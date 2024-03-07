## Inbound Policy Traffic Rulesets

${\large{\textbf{\textsf{\color{red}Definition}}}}$

A traffic ruleset in an SSL Orchestrator policy takes any number of traffic conditions and performs the following action on matching traffic:

- Allowing or resetting the traffic
- Intercepting (decrypting) or bypassing decryption
- Assigning the flow to a service chain of inspection services

___

${\large{\textbf{\textsf{\color{red}Create\ Inbound\ Policy\ Traffic\ Rulesets}}}}$

*Note that while Central Manager will create an "All Traffic" traffic rule by default, it is not created by default when using API to manage traffic rules in a policy. It is highly recommended to include an All Traffic rule in your traffic ruleset declarations **at the end of the ruleset** so that anything not matching other (higher) conditions will have some default actions.*

The below example provides the bare minimum policy declaration with only an All Traffic rule and no other rules in a traffic ruleset. The below All Traffic rule has no conditions, but will allow (implied), SSL intercept, and pass to a service chain. Where it indicates [service-chain-id], replace this with a valid service-chain ID. For the **inbound policy traffic ruleset**, the _minimum_ requirements for defining are illustrated in the following examples. The resulting ID will be at ```.id```.

**Basic**
```bash
POST {{CM}}/api/v1/spaces/default/security/policies
```
```json
{
  "policyName": "my-sslo-policy",
  "policyType": "default",
  "trafficRuleSets": [
    {
      "ruleType": "traffic",
      "rules": [
        {
          "name": "All Traffic",
          "conditions": [],
          "actions": [
            {
              "actionType": "SSL_PROXY_INTERCEPT"
            },
            {
              "actionType": "SERVICE_CHAIN",
              "serviceChain": "[service-chain-id]"
            }
          ]
        }
      ]
    }
  ]
}
```
**Curl**
```bash
POLICY=$(cat <<EOF
{
  "policyName": "my-sslo-policy",
  "policyType": "default",
  "trafficRuleSets": [
    {
      "ruleType": "traffic",
      "rules": [
        {
          "name": "All Traffic",
          "conditions": [],
          "actions": [
            {
              "actionType": "SSL_PROXY_INTERCEPT"
            },
            {
              "actionType": "SERVICE_CHAIN",
              "serviceChain": "[service-chain-id]"
            }
          ]
        }
      ]
    }
  ]
}
EOF
)
policy_id=$(curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/security/policies" -d "${POLICY}" |jq -r '.id')
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference:\ Policy\ Block}}}}$

| Required | Attribute                 | Defaults | Notes                                                      |
|:--------:|---------------------------|:--------:|------------------------------------------------------------|
|     *    | policyName                |   none   | string: policy-name                                        |
|     *    | policyType                |   none   | "default" for inbound app policy, or "inbound-gateway"     |
|     *    | trafficRuleSets           |   none   | a dictionary of one element containing rules and ruleType  |
|     *    | trafficRuleSets: ruleType |   none   | must be "traffic"                                          |
|     *    | trafficRuleSets: rules    |   none   | dictionary of all traffic rules                            |


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference:\ TrafficRuleSets\ Rules}}}}$

| Required | Attribute  | Defaults | Notes                            |
|:--------:|------------|:--------:|----------------------------------|
|     *    | name       |   none   | string: rule-name                |
|     *    | conditions |   none   | dictionary of traffic conditions |
|     *    | actions    |   none   | dictionary of actions            |


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference:\ TrafficRuleSets\ Rule\ Conditions}}}}$

|                   conditionType                   |                                                                       operator&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;                                                                      | values&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; |     datagroup     | local&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; | Example                                                                                                                                                                                                                                       | Notes&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; |
|:-------------------------------------------------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:------------------------------------------:|:-----------------:|:-----------------------------------------------:|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------:|
|                      IS_IPV4                      |                                                                                                                                                                                         |                                            |                   | true                                            | {<br /> &emsp;"conditionType":"IS_IPV4",<br /> &emsp;"local":true<br /> }                                                                                                                                                                     | IP Version is IPv4                                                                                                                  |
|                      IS_IPV6                      |                                                                                                                                                                                         |                                            |                   | true                                            | {<br /> &emsp;"conditionType":"IS_IPV6",<br /> &emsp;"local":true<br /> }                                                                                                                                                                     | IP Version is IPv6                                                                                                                  |
|                    IP_PROTOCOL                    | "equals"<br /> "not-equals"                                                                                                                                                             | 6 for TCP<br /> 11 for UDP                 |                   | true                                            | {<br /> &emsp;"conditionType":"IP_PROTOCOL",<br /> &emsp;"operator":"equals",<br /> &emsp;"values":[6],<br /> &emsp;"local":true<br /> }                                                                                                      | IP protocol version currently supports TCP(6) and UDP(11)                                                                           |
|                      L4_PORT                      | "equals"<br /> "not-equals"<br /> "less"<br /> "less-or-greater"<br /> "greater"<br /> "greater-or-equal"                                                                               | 0-65535                                    |                   | true (client-side)<br /> false (server-side)    | {<br /> &emsp;"conditionType":"L4_PORT",<br /> &emsp;"operator":"equals",<br /> &emsp;"values":[443],<br /> &emsp;"local":false<br /> }                                                                                                       | Client or server-side port match (integers)                                                                                         |
|          L4_PORT<br /> (data group match)         | "equals"<br /> "not-equals"                                                                                                                                                             |                                            | DG UUID reference | true (client-side)<br /> false (server-side)    | {<br /> &emsp;"conditionType":"L4_PORT",<br /> &emsp;"operator":"equals",<br /> &emsp;"datagroup":DG-UUID,<br /> &emsp;"local":false<br /> }                                                                                                  | Client or server-side port match (data group)                                                                                       |
|                     IP_ADDRESS                    | "matches"<br /> "not-matches"                                                                                                                                                           | IP4/6 IP or IP subnet                      |                   | true (client-side)<br /> false (server-side)    | {<br /> &emsp;"conditionType":"IP_ADDRESS",<br /> &emsp;"operator":"equals",<br /> &emsp;"values":[<br /> &emsp;&emsp;"10.10.0.10",<br /> &emsp;&emsp;"10.20.0.0/16",<br /> &emsp;],<br /> &emsp;"local":true<br /> }                         | Client or server-side IP/IP-subnet match (IPv4/IPv6)                                                                                |
|        IP_ADDRESS<br /> (data group match)        | "matches"<br /> "not-matches"                                                                                                                                                           |                                            | DG UUID reference | true (client-side)<br /> false (server-side)    | {<br /> &emsp;"conditionType":"IP_ADDRESS",<br /> &emsp;"operator":"equals",<br /> &emsp;"datagroup":DG-UUID<br /> &emsp;"local":true<br /> }                                                                                                 | Client or server-side IP/IP-subnet match (datagroup)                                                                                |
|              SSL_EXTENSION_SERVERNAME             | "equals"<br /> "not-equals"<br /> "starts-with"<br /> "not-starts-with"<br /> "ends-with"<br /> "not-ends-with"<br /> "contains"<br /> "not-contains"<br /> "exists"<br /> "not-exists" | server-name strings                        |                   | true                                            | {<br /> &emsp;"conditionType":"SSL_EXTENSION_SERVERNAME",<br /> &emsp;"operator":"equals",<br /> &emsp;"values":[<br /> &emsp;&emsp;"test1.f5labs.com",<br /> &emsp;&emsp;"test1.f5labs.com",<br /> &emsp;],<br /> &emsp;"local":true<br /> } | SNI Server Name match (server-name strings)                                                                                         |
| SSL_EXTENSION_SERVERNAME<br /> (data group match) | "equals"<br /> "starts-with"<br /> "ends-with"<br /> "contains"                                                                                                                         |                                            | DG UUID reference | true                                            | {<br /> &emsp;"conditionType":"SSL_EXTENSION_SERVERNAME",<br /> &emsp;"operator":"equals",<br /> &emsp;"datagroup":DG-UUID<br /> &emsp;"local":true<br /> }                                                                                   | SNI Server Name match (data group)                                                                                                  |



${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference:\ TrafficRuleSets\ Rule\ Actions}}}}$




