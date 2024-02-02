## SSL Orchestrator Service Chains

${\large{\textbf{\textsf{\color{red}Definition}}}}$

A **Service Chain** is a logical grouping of services in a defined order through which SSL Orchestrator processes traffic. Security policies match traffic flow conditions, which then assign flows to service chains. The service chain then orchestrates the traffic through the defined set of services in the defined order. An SSL Orchestrator service chain is uniquely different than the “daisy chain” approach of other SSL visibility products. In the SSL Orchestrator service chain model, all services are connected (physically or logically) to SSL Orchestrator. The service chain model improves over the daisy chain model in the following ways:

* If some services are not designed to inspect some types of traffic, a traffic policy can simply send traffic to services that are interested, bypassing those that are not. For example, the policy could be configured to only send outbound HTTP traffic to a HTTP proxy device.
* If a specific throughput requirement is needed for the solution, but not all traffic needs to go to all services, then not all services have to scale at the same rate.
* As these services are independently orchestrated, it becomes easy to scale and perform maintenance (i.e. add/remove devices from individual service pools), without incurring any downtime.


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Common\ Service\ Chain\ Patterns}}}}$

SSL Orchestrator service chains all share a common set of API patterns that can be described here:

* Getting the list of service chains (needed for service chain IDs)
* Creating a service chain
* Deleting a service chain
* Updating a service chain

___

*Note:*

* *Values below in {{double curly brackets}} indicate typical environment variables defined in Postman or the Visual Studio Code Thunder Client.*
* *The {{CM}} variable would be a statically defined Postman/Thunder environment variable for the Central Manager host/IP.*
* *The ${CM} variable in the Bash commands is the export of the Central Manager host/IP (ex. ```export CM=10.1.1.6```)*
___


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Get\ Service\ Chains}}}}$

To fetch the list of service chains use the following GET. The ID will be at ```_embedded.service_chains[X].id``` where X is the index.

**Basic**
```bash
GET {{CM}}/api/v1/spaces/default/security/service-chains
```
**Filtered** by *name* string, where resulting ID is at ```_embedded.service_chains[0].id```
```bash
GET {{CM}}/api/v1/spaces/default/security/service-chains?filter=name+eq+%27my-api-service-chain%27&select=name,id
```
**Curl**
```bash
chain_id=$(curl -sk -H "Authorization: Bearer ${token}" "https://${CM}/api/v1/spaces/default/security/service-chains?filter=name+eq+%27my-api-service-chain%27&select=name,id" |jq -r '._embedded.service_chains[0].id')
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Create\ a\ Service\ Chain}}}}$

Service chains have a simple workflow that simply requires an array of the inspection service ID values, in the order prescribed. The resulting ID value in the JSON response will be at ```.id```. You will use the service chain ID in policy declarations.

**Basic**
```bash
POST /api/v1/spaces/default/security/service-chains
```
```json
{
  "name": "my-api-service-chain",
  "inspection_services": [
    "inspection-service-id",
    "inspection-service-id"
  ]
}
```
**Curl**
```bash
CHAIN=$(cat <<EOF
{
  "name": "my-api-service-chain",
  "inspection_services": [
    "inspection-service-id",
    "inspection-service-id"
  ]
}
EOF
)
chain_id=$(curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/security/service-chains" -d "${CHAIN}")
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Delete\ a\ Service\ Chain}}}}$

To delete a service chain, it must first be removed from any associated policies. The REST call to delete requires the service chain ID.

**Basic**
```bash
DELETE {{CM}}/api/v1/spaces/default/security/service-chains/{{chain_id}}
```
**Curl**
```bash
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" -X DELETE "https://${CM}/api/v1/spaces/default/security/service-chains/{{chain_id}}"
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Update\ a\ Service\ Chain}}}}$

Updating an existing service chain requires the ID of the target service chain.

**Basic**
```bash
PUT {{CM}}/api/v1/spaces/default/security/service-chains/{{chain_id}}
```
```json
{
  "name": "my-api-service-chain",
  "inspection_services": [
    "inspection-service-id",
    "inspection-service-id",
    "inspection-service-id"
  ]
}
```
**Curl**
```bash
CHAIN=$(cat <<EOF
{
  "name": "my-api-service-chain",
  "inspection_services": [
    "inspection-service-id",
    "inspection-service-id",
    "inspection-service-id"
  ]
}
EOF
)
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" -X PUT "https://${CM}/api/v1/spaces/default/security/service-chains/{{chain_id}}" -d "${CHAIN}"
```
