## SSL Orchestrator Policies

${\large{\textbf{\textsf{\color{red}Definition}}}}$

A **Policy** is a set of rules that match traffic patterns. When a rule condition is matched, it assigns a set of actions to the matching flow. A single SSL Orchestrator policy applied to an application can contain multiple "rulesets", including a traffic ruleset, logging ruleset, and potentially other types. All of the different rulesets will have similar traffic condition options, but take different actions. For example, the traffic ruleset can take the following actions:

* Allowing or resetting the traffic
* Intercepting (decrypting) or bypassing decryption
* Assigning the flow to a service chain of inspection services

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Common\ Policy\ Patterns}}}}$

SSL Orchestrator policies all share a common set of API patterns that can be described here:

* Getting the policy information
* Creating a policy
* Deleting a policy
* Updating a policy

As previously notes, a single policy may include multiple rulesets, minimally a traffic rule set, but could also contain a logging ruleset and other types. The logging ruleset is described in a separate document.

___

*Note:*

* *Values below in {{double curly brackets}} indicate typical environment variables defined in Postman or the Visual Studio Code Thunder Client.*
* *The {{CM}} variable would be a statically defined Postman/Thunder environment variable for the Central Manager host/IP.*
* *The ${CM} variable in the Bash commands is the export of the Central Manager host/IP (ex. ```export CM=10.1.1.6```)*
___


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Get\ Policies}}}}$

To fetch an SSL Orchestrator policy use the following GET. The ID will be at ```_embedded.policies[X].id``` where X is the index.

**Basic**
```bash
GET {{CM}}/api/v1/spaces/default/security/policies
```
**Filtered** by *name* string (ex. my-sslo-policy), where resulting ID is at ```_embedded.policies[0].id```
```bash
GET {{CM}}/api/v1/spaces/default/security/policies?filter=name+eq+%27my-sslo-policy%27&select=name,id
```
**Curl**
```bash
chain_id=$(curl -sk -H "Authorization: Bearer ${token}" "https://${CM}/api/v1/spaces/default/security/policies?filter=name+eq+%27my-sslo-policy%27&select=name,id" |jq -r '._embedded.policies[0].id')
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Create\ a\ Policy}}}}$

SSL Orchestrator policies have a robust and flexible workflow that requires one or more "rules". Each rule will then contain "conditions" (the traffic patterns to match on), and "actions" (the actions to take on a matching traffic condition).

*Note that while Central Manager will create an "All Traffic" traffic rule by default, it is not created by default when using API to manage traffic rules in a policy. It is highly recommended to include an All Traffic rule in your traffic ruleset declarations **at the end of the ruleset** so that anything not matching other (higher) conditions will have some default actions.*

The below example provides the bare minimum policy declaration with only an All Traffic rule and no other rules in a traffic ruleset. The below All Traffic rule has no conditions, but will allow (implied), SSL intercept, and pass to a service chain. Where it indicates [service-chain-id], replace this with a valid service-chain ID.

**Basic**
```bash
POST /api/v1/spaces/default/security/policies
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
policy_id=$(curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/security/policies" -d "${POLICY}")
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Delete\ a\ Policy}}}}$

To delete an SSL Orchestrator policy, it must first be removed from any associated applications. The REST call to delete requires the policy ID.

**Basic**
```bash
DELETE {{CM}}/api/v1/spaces/default/security/policies/{{policy_id}}
```
**Curl**
```bash
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" -X DELETE "https://${CM}/api/v1/spaces/default/security/policies/${policy_id}"
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Update\ a\ Policy}}}}$

Updating an existing SSL Orchestrator policy requires the ID of the target policy.

**Basic**
```bash
PUT {{CM}}/api/v1/spaces/default/security/policies/{{policy_id}}
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
              "actionType": "SSL_PROXY_BYPASS"
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
              "actionType": "SSL_PROXY_BYPASS"
            }
          ]
        }
      ]
    }
  ]
}
EOF
)
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" -X PUT "https://${CM}/api/v1/spaces/default/security/policies/${policy_id}" -d "${POLICY}"
```






