## Inbound Policy Traffic Rulesets

${\large{\textbf{\textsf{\color{red}Definition}}}}$

A traffic ruleset in an SSL Orchestrator policy takes any number of traffic conditions and performs the following action on matching traffic:

- Allowing or resetting the traffic
- Intercepting (decrypting) or bypassing decryption
- Assigning the flow to a service chain of inspection services

SSL Orchestrator policies possess the following structure and logic:

- A policy contains one more rulesets, minimally containing a traffic ruleset, but potentially also a logging ruleset and others. Each ruleset can have similar traffic matching conditions, but will perform different actions.
- Within a ruleset is one or more rules. The rules are processed top to bottom in a "first-match" strategy, meaning traffic is compared to each rule in the ruleset, starting from the top, until a match is made, and then stops.
- Within a rule is one or more **conditions** (the properties of the traffic to match on), and one or more corresponding **actions** (the actions to take when the conditions are matched). The one exception is an "All Traffic" rule which will have no conditions. This would normally be placed at the bottom of a ruleset to catch any traffic that does not match any other rules.
- Each rule can have one or more conditions, again except for the All Traffic rule. When there is more than one condition, a logical **AND** is applied. For example, if the conditions are "L4_PORT equals 443" and "IP_ADDRESS equals 10.0.0.0/8", then both of these must be true for the rule to match.
- Each rule can have one or more actions. Like conditions, these are all additive.
- Internally, traffic matching follows both the order of the rules and the "present" OSI layer (walking up the stack). For example, if an L4_PORT rule is placed at the top of the ruleset, then that traffic condition will be evaluated at OSI layer 4. However, if an SSL_EXTENSION_SERVERNAME rule is placed above the L4_PORT rule, then the L4_PORT will be evaluated at the SSL/TLS OSI layer (6), because this is the OSI layer needed to acquire the SNI value. It is not imperative to place the rules in a particular order, but you can gain some efficiencies by placing lower layer rules and conditions higher up in the ruleset.

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

| conditionType                                     | operator                                                                                                                                                                                | values                      | datagroup         | local                                        | Example                                                                                                                                                                                                               | Notes                                                     |
|---------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|-------------------|----------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------|
| IS_IPV4                                           |                                                                                                                                                                                         |                             |                   |                                              | {<br /> &emsp;"conditionType":"IS_IPV4",<br /> }                                                                                                                                                                      | IP Version is IPv4                                        |
| IS_IPV6                                           |                                                                                                                                                                                         |                             |                   |                                              | {<br /> &emsp;"conditionType":"IS_IPV6",<br /> }                                                                                                                                                                      | IP Version is IPv6                                        |
| IP_PROTOCOL                                       | "equals"<br /> "not-equals"                                                                                                                                                             | 6 for TCP<br /> 11 for UDP  |                   |                                              | {<br /> &emsp;"conditionType":"IP_PROTOCOL",<br /> &emsp;"operator":"equals",<br /> &emsp;"values":[6]<br /> }                                                                                                        | IP protocol version currently supports TCP(6) and UDP(11) |
| L4_PORT                                           | "equals"<br /> "not-equals"<br /> "less"<br /> "less-or-greater"<br /> "greater"<br /> "greater-or-equal"                                                                               | 0-65535                     |                   | true (server-side)<br /> false (client-side) | {<br /> &emsp;"conditionType":"L4_PORT",<br /> &emsp;"operator":"equals",<br /> &emsp;"values":[443],<br /> &emsp;"local":true<br /> }                                                                                | Client or server-side port match (integers)               |
| L4_PORT<br /> (data group match)                  | "equals"<br /> "not-equals"                                                                                                                                                             |                             | DG UUID reference | true (server-side)<br /> false (client-side) | {<br /> &emsp;"conditionType":"L4_PORT",<br /> &emsp;"operator":"equals",<br /> &emsp;"datagroup":DG-UUID,<br /> &emsp;"local":true<br /> }                                                                           | Client or server-side port match (data group)             |
| IP_ADDRESS                                        | "matches"<br /> "not-matches"                                                                                                                                                           | IP4/6 IP or<br /> IP subnet |                   | true (server-side)<br /> false (client-side) | {<br /> &emsp;"conditionType":"IP_ADDRESS",<br /> &emsp;"operator":"equals",<br /> &emsp;"values":[<br /> &emsp;&emsp;"10.10.0.10",<br /> &emsp;&emsp;"10.20.0.0/16"<br /> &emsp;],<br /> &emsp;"local":false<br /> } | Client or server-side IP/IP-subnet match (IPv4/IPv6)      |
| IP_ADDRESS<br /> (data group match)               | "matches"<br /> "not-matches"                                                                                                                                                           |                             | DG UUID reference | true (server-side)<br /> false (client-side) | {<br /> &emsp;"conditionType":"IP_ADDRESS",<br /> &emsp;"operator":"equals",<br /> &emsp;"datagroup":DG-UUID<br /> &emsp;"local":false<br /> }                                                                        | Client or server-side IP/IP-subnet match (datagroup)      |
| SSL_EXTENSION_SERVERNAME                          | "equals"<br /> "not-equals"<br /> "starts-with"<br /> "not-starts-with"<br /> "ends-with"<br /> "not-ends-with"<br /> "contains"<br /> "not-contains"<br /> "exists"<br /> "not-exists" | server-name strings         |                   |                                              | {<br /> &emsp;"conditionType":"SSL_EXTENSION_SERVERNAME",<br /> &emsp;"operator":"equals",<br /> &emsp;"values":[<br /> &emsp;&emsp;"test1.f5labs.com",<br /> &emsp;&emsp;"test2.f5labs.com"<br /> &emsp;]<br /> }    | SNI Server Name match (server-name strings)               |
| SSL_EXTENSION_SERVERNAME<br /> (data group match) | "equals"<br /> "starts-with"<br /> "ends-with"<br /> "contains"                                                                                                                         |                             | DG UUID reference |                                              | {<br /> &emsp;"conditionType":"SSL_EXTENSION_SERVERNAME",<br /> &emsp;"operator":"equals",<br /> &emsp;"datagroup":DG-UUID<br /> }                                                                                    | SNI Server Name match (data group)                        |



${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference:\ TrafficRuleSets\ Rule\ Actions}}}}$


|                 actionType                 | Example                                                                                           | Notes                                                                                                                          |
|:------------------------------------------:|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------|
|                    RESET                   | {<br /> &emsp;"actionType":"RESET"<br /> }                                                        | This actionType corresponds to the Flow Action. The default action is to **allow** traffic.                                    |
| SSL_PROXY_INTERCEPT<br /> SSL_PROXY_BYPASS | {<br /> &emsp;"actionType":"SSL_PROXY_INTERCEPT"<br /> }                                          | This actionType corresponds to the SSL Action and determines TLS decryption or bypass.                                         |
|                SERVICE_CHAIN               | {<br /> &emsp;"actionType":"SERVICE_CHAIN",<br /> &emsp;"serviceChain":Service-Chain-UUID<br /> } | This actionType corresponds to the Service Chain attribute and must additionally supply the ID value of a valid service chain. |
