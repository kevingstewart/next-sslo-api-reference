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
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" -X DELETE "https://${CM}/api/v1/spaces/default/security/service-chains/${chain_id}"
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
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" -X PUT "https://${CM}/api/v1/spaces/default/security/service-chains/${chain_id}" -d "${CHAIN}"
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Ansible\ Reference}}}}$

Execute with:
```
bigip_next_cm_mgmt_ip="10.1.1.6"
bigip_next_password="my_password"
ansible-playbook -i notahost, sslo-service-chain.yaml --extra-vars "bigip_next_cm_mgmt_ip=$bigip_next_cm_mgmt_ip bigip_next_password=$bigip_next_password"
```

```yaml
---
- hosts: all
  connection: local

  tasks:
    - name: Check if BIG-IP Next Central Manager instance is available (HTTPS responding 405 on /api/login)
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/login
        method: GET
        status_code: 405
        validate_certs: false
      until: json_response.status == 405
      retries: 50
      delay: 30
      register: json_response

    - name: Authenticate to BIG-IP Next CM API
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/login
        method: POST
        headers:
          Content-Type: application/json
        body: |
          {
              "username": "admin",
              "password": "{{ bigip_next_password }}"
          }
        body_format: json
        timeout: 60
        status_code: 200
        validate_certs: false
      register: bigip_next_cm_token
      retries: 30
      delay: 30

    - name: Set the BIG-IP Next CM token
      set_fact:
        bigip_next_cm_token: "{{ bigip_next_cm_token.json.access_token }}"

    - debug:
        var: bigip_next_cm_token
    
    - name: Get Inspection Services (filter by name: "my-sslo-tap")
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/v1/spaces/default/security/inspection-services?filter=name+eq+%27my-sslo-tap%27&select=name,id
        method: GET
        headers:
          Authorization: "Bearer {{ bigip_next_cm_token }}"
          Content-Type: application/json
        timeout: 60
        status_code: 200
        validate_certs: false
      register: json_response
      retries: 30
      delay: 30

    - name: Set BIG-IP Instance ID
      set_fact:
        insp_ids: "{{ json_response.json._embedded.inspection_services | map(attribute='id') }}"

    - name: Create Service Chain (add filtered inspection services)
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/v1/spaces/default/security/service-chains
        method: POST
        headers:
          Authorization: "Bearer {{ bigip_next_cm_token }}"
          Content-Type: application/json
        body: |
          {
            "name": "my-sslo-service-chain",
            "inspection_services": {{ insp_ids }}
          }
        body_format: json
        timeout: 60
        status_code: 200
        validate_certs: false
      register: json_response
      retries: 30
      delay: 30

    - debug:
        var: json_response
```
