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





${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference:\ TrafficRuleSets\ Rule\ Actions}}}}$




