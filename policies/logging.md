## Inbound Policy Logging Rulesets

${\large{\textbf{\textsf{\color{red}Definition}}}}$

A logging ruleset in an SSL Orchestrator policy takes any number of traffic conditions and generates a logging action on the matching traffic. Currently the logging ruleset has a single **COLLECT_DATA** action.

___

${\large{\textbf{\textsf{\color{red}Create\ Inbound\ Policy\ Logging\ Rulesets}}}}$

The below example provides the bare minimum policy declaration with only an All Traffic rule in the traffic ruleset, and a simple logging ruleset. For the **inbound policy traffic and logging rulesets**, the _minimum_ requirements for defining are illustrated in the following examples. The resulting ID will be at ```.id```.

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
  ],
  "loggingRuleSets": [
    {
      "ruleType": "logging",
      "rules": [
        {
          "name": "all-logging",
          "conditions": [
            {
              "conditionType": "IP_PROTOCOL",
              "operator": "equals",
              "values": [
                6
              ],
              "local": true
            }
          ],
          "actions": [
            {
              "actionType": "COLLECT_DATA"
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
  ],
  "loggingRuleSets": [
    {
      "ruleType": "logging",
      "rules": [
        {
          "name": "all-logging",
          "conditions": [
            {
              "conditionType": "IP_PROTOCOL",
              "operator": "equals",
              "values": [
                6
              ],
              "local": true
            }
          ],
          "actions": [
            {
              "actionType": "COLLECT_DATA"
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
|          | loggingRuleSets           |   none   | a dictionary of one element containing rules and ruleType  |
|          | loggingRuleSets: ruleType |   none   | must be "logging"                                          |
|          | loggingRuleSets: rules    |   none   | dictionary of all logging rules                            |


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference:\ LoggingRuleSets\ Rules}}}}$

| Required | Attribute  | Defaults | Notes                            |
|:--------:|------------|:--------:|----------------------------------|
|     *    | name       |   none   | string: rule-name                |
|     *    | conditions |   none   | dictionary of logging conditions |
|     *    | actions    |   none   | dictionary of actions            |


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference:\ LoggingRuleSets\ Rule\ Conditions}}}}$

| conditionType                                     | operator                                                                                                                                                                                | values                      | datagroup         | local                                        | Example                                                                                                                                                                                                               | Notes                                                     |
|---------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|-------------------|----------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------|
| IS_IPV4                                           |                                                                                                                                                                                         |                             |                   |                                              | {<br /> &emsp;"conditionType":"IS_IPV4",<br /> }                                                                                                                                                                      | IP Version is IPv4                                        |
| IS_IPV6                                           |                                                                                                                                                                                         |                             |                   |                                              | {<br /> &emsp;"conditionType":"IS_IPV6",<br /> }                                                                                                                                                                      | IP Version is IPv6                                        |
| IP_PROTOCOL                                       | "equals"<br /> "not-equals"                                                                                                                                                             | 6 for TCP<br /> 11 for UDP  |                   |                                              | {<br /> &emsp;"conditionType":"IP_PROTOCOL",<br /> &emsp;"operator":"equals",<br /> &emsp;"values":[6]<br /> }                                                                                                        | IP protocol version currently supports TCP(6) and UDP(11) |
| L4_PORT                                           | "equals"<br /> "not-equals"<br /> "less"<br /> "less-or-greater"<br /> "greater"<br /> "greater-or-equal"                                                                               | 0-65535                     |                   | true (server-side)<br /> false (client-side) | {<br /> &emsp;"conditionType":"L4_PORT",<br /> &emsp;"operator":"equals",<br /> &emsp;"values":[443],<br /> &emsp;"local":true<br /> }                                                                                | Client or server-side port match (integers)               |
| L4_PORT<br /> (data group match)                  | "equals"<br /> "not-equals"                                                                                                                                                             |                             | DG UUID reference | true (server-side)<br /> false (client-side) | {<br /> &emsp;"conditionType":"L4_PORT",<br /> &emsp;"operator":"equals",<br /> &emsp;"datagroup":DG-UUID,<br /> &emsp;"local":true<br /> }                                                                           | Client or server-side port match (data group)             |
| IP_ADDRESS                                        | "matches"<br /> "not-matches"                                                                                                                                                           | IP4/6 IP or<br /> IP subnet |                   | true (server-side)<br /> false (client-side) | {<br /> &emsp;"conditionType":"IP_ADDRESS",<br /> &emsp;"operator":"equals",<br /> &emsp;"values":[<br /> &emsp;&emsp;"10.10.0.10",<br /> &emsp;&emsp;"10.20.0.0/16"<br /> &emsp;],<br /> &emsp;"local":false<br /> } | Client or server-side IP/IP-subnet match (IPv4/IPv6)      |
| IP_ADDRESS<br /> (data group match)               | "matches"<br /> "not-matches"                                                                                                                                                           |                             | DG UUID reference | true (server-side)<br /> false (client-side) | {<br /> &emsp;"conditionType":"IP_ADDRESS",<br /> &emsp;"operator":"equals",<br /> &emsp;"datagroup":DG-UUID<br /> &emsp;"local":false<br /> }                                                                        | Client or server-side IP/IP-subnet match (datagroup)      |


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference:\ LoggingRuleSets\ Rule\ Actions}}}}$


|                 actionType                 | Example                                                                                           | Notes                                   |
|:------------------------------------------:|---------------------------------------------------------------------------------------------------|-----------------------------------------|
|                COLLECT_DATA                | {<br /> &emsp;"actionType":"COLLECT_DATA"<br />}                                                  | This action is implied in CM UI policy. |





