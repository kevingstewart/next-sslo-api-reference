# F5 BIG-IP Next SSL Orchestrator API Examples

This project is an *unofficial* compendium of API examples for creating and managing BIG-IP Next SSL Orchestrator objects. Please always refer to the official F5 Next API pages for official API reference. SSL Orchestrator objects are under the *Security* section. 

https://clouddocs.f5.com/products/bigip-next/mgmt-api/latest/ApiReferences/bigip_public_api_ref/r_openapi-next.html.

The purpose of this project is to demonstrate the ease and usefulness of **automated** SSL Orchestrator configuration on BIG-IP Next. The SSL Orchestrator object space in BIG-IP Next can essentially be broken down into the following categories. Separate sections and pages are provided for each type of object.

* **Inspection Services**: represent the security "inspection" products that will receive decrypted, service chained traffic from the SSL Orchestrator. These can generally be categorized as:
  * TAP inspection services
  * ICAP inspection services
  * Inline Layer 3 inspection services
  * HTTP Transparent Proxy inspection services
  * HTTP Explicit Proxy inspection services

* **Service Chains**: a logical grouping of inspection services in a defined order through which SSL Orchestrator processes traffic.

* **Security Policies**: a set of rules that match traffic patterns and perform actions, including allowing or blocking traffic, decrypting or bypassing decryption, and sending flows to service chains.

* **Applications**: not specifically an SSL Orchestrator object, the security policy is applied to different types of application flows.
  
----

This project contains three automation "modes" for SSL Orchestrator objects to suit different environmental preferences:

* Native BIG-IP Next Central Manager "CM" API for the SSL Orchestrator objects, plus Application Services 3 "AS3" API for the application stacks.
* Curl command line utilizing the CM and AS3 APIs
* Ansible playbooks utilizing the CM and AS3 APIs

----

Before creating any objects through the CM, you first need to authenticate and fetch a token that will be used in subsequent API calls. The basic API for this looks like the following:

**Basic**
```bash
POST {{CM}}/api/login
```
```json
{
  "username": "{{CMUSER}}",
  "password": "{{CMPASS}}"
}
```
where {{CM}} is the IP or hostname of the Central Manager instance, and {{CMUSER}} and {{CMPASS}} are valid user:password credentials on the CM. This will return an "access_token" value in the successful JSON response that you'll use in other API calls as the **Authorization: Bearer** token. To do the same with command line Curl, the below will populate the *TOKEN* variable with your Bearer token.

**Curl**
```bash
export CM=172.16.1.148
export CMUSER=admin
export CMPASS='mypassword'

LOGON=$(cat <<EOF
{
  "username": "${CMUSER}",
  "password": "${CMPASS}"
}
EOF
)
TOKEN=$(curl -sk "https://${CM}/api/login" -H 'Content-Type: application/json' -d "${LOGON}" | jq -r '.access_token')
echo $TOKEN
```
And finally, to do the same with raw Ansible:

**Ansible**
```yaml
---
- hosts: all
  connection: local

  tasks:
    - name: Check if BIG-IP Next Central Manager instance is available (HTTPS responding 405 on /api/login)
      uri:
        url: https://{{ BIGIP_NEXT_CM }}/api/login
        method: GET
        status_code: 405
        validate_certs: false
      until: json_response.status == 405
      retries: 50
      delay: 30
      register: json_response


    - name: Authenticate to BIG-IP Next CM API
      uri:
        url: https://{{ BIGIP_NEXT_CM }}/api/login
        method: POST
        headers:
          Content-Type: application/json
        body: |
          {
              "username": "{{ BIGIP_NEXT_CMUSER }}",
              "password": "{{ BIGIP_NEXT_CMPASS }}"
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
        bearer_token: "{{ bigip_next_cm_token.json.access_token }}"

    - debug:
        var: bearer_token
```
```bash
export BIGIP_NEXT_CM=172.16.1.148
export BIGIP_NEXT_CMUSER=admin
export BIGIP_NEXT_CMPASS='mypassword'
ansible-playbook -i notahost, next-cm-auth.yaml --extra-vars "BIGIP_NEXT_CM=${BIGIP_NEXT_CM} BIGIP_NEXT_CMUSER=${BIGIP_NEXT_CMUSER} BIGIP_NEXT_CMPASS=${BIGIP_NEXT_CMPASS}"
```



[//]: # (Markdown Editor: https://www.tablesgenerator.com/markdown_tables)
