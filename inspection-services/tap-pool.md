## TAP-Pool Inspection Service

${\large{\textbf{\textsf{\color{red}Definition}}}}$

A TAP inspection service is a passive device, typically with no external IP addresses. SSL Orchestrator passes receive-only copies of the payload packets to a TAP service. 

___

${\large{\textbf{\textsf{\color{red}Create\ TAP\ Inspection\ Service}}}}$

For the **TAP-Pool** variant, the requirements for defining are name, description, type (tap-clone-pool), a corresponding VLAN (L1-network), and an array of endpoint IP addresses. The resulting ID will be at ```.id```.

**Basic**
```bash
POST {{CM}}/api/v1/spaces/default/security/inspection-services
```
```json
{
  "name": "my-sslo-tap",
  "description": "My SSLO Tap Inspection Service",
  "type": "tap-clone-pool",
  "network": {
    "vlan": "sslo-insp-tap",
    "endpoints": [
      {
        "address": "198.19.97.10"
      },
      {
        "address": "198.19.97.11"
      }
    ]
  }
}
```
**Curl**
```bash
INSP=$(cat <<EOF
{
  "name": "my-sslo-tap",
  "description": "My SSLO Tap Inspection Service",
  "type": "tap-clone-pool",
  "network": {
    "vlan": "sslo-insp-tap",
    "endpoints": [
      {
        "address": "198.19.97.10"
      },
      {
        "address": "198.19.97.11"
      }
    ]
  }
}
EOF
)
insp_id=$(curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/security/inspection-services" -d "${INSP}" |jq -r '.id')
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference}}}}$

| Required | Attribute | Defaults | Notes |
|:-:|---|---|---|
| * | name |  |  |
| * | description |  |  |
| * | type |  | string: must be "**tap-clone-pool**" |
| * | network: vlan |  |  |
| * | network: endpoints |  | array: { "address":"ip-address" } |


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Ansible\ Reference}}}}$

Execute with:
```
bigip_next_cm_mgmt_ip="10.1.1.6"
bigip_next_password="my_password"
ansible-playbook -i notahost, sslo-insp-tap.yaml --extra-vars "bigip_next_cm_mgmt_ip=$bigip_next_cm_mgmt_ip bigip_next_password=$bigip_next_password"
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
    
    - name: Create SSLO TAP Inspection Service
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/v1/spaces/default/security/inspection-services
        method: POST
        headers:
          Authorization: "Bearer {{ bigip_next_cm_token }}"
          Content-Type: application/json
        body: |
          {
            "name": "my-sslo-tap",
            "description": "My SSLO Tap Inspection Service",
            "type": "tap-clone-pool",
            "network": {
              "vlan": "sslo-insp-tap",
              "endpoints": [
                {
                  "address": "198.19.97.10"
                },
                {
                  "address": "198.19.97.11"
                }
              ]
            }
          }
        body_format: json
        timeout: 60
        status_code: 200
        validate_certs: false
      register: json_response

    - name: Set Inspection Service ID
      set_fact:
        insp_id: "{{ json_response.json.id}}"

    - name: Get BIG-IP Next ID
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/v1/spaces/default/instances?filter=hostname+eq+%27bigip-next.f5labs.com%27&select=hostname,id
        method: GET
        headers:
          Authorization: "Bearer {{ bigip_next_cm_token }}"
          Content-Type: application/json
        timeout: 60
        status_code: 200
        validate_certs: false
      register: json_response

    - name: Set BIG-IP Instance ID
      set_fact:
        bigip_id: "{{ json_response.json._embedded.devices | map(attribute='id') }}"

    - name: Deploy SSLO TAP Inspection Service to BIG-IP Instance
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/v1/spaces/default/security/inspection-services/{{ insp_id }}/deployment
        method: POST
        headers:
          Authorization: "Bearer {{ bigip_next_cm_token }}"
          Content-Type: application/json
        body: |
          {
            "deploy-instances": {{ bigip_id }},
            "undeploy-instances": []
          }
        body_format: json
        timeout: 60
        status_code: 200
        validate_certs: false
      register: json_response

    - debug:
        var: json_response
```
