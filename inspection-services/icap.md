## ICAP Inspection Service

${\large{\textbf{\textsf{\color{red}Definition}}}}$

The Internet Content Adaption Protocol (ICAP) is defined by RFC3507 and constitutes an encapsulation protocol. Packets are encapsulated by an ICAP client and passed to an ICAP server. What the ICAP server does with the encapsulated data depends on the underlying service, and typically ranges from malware and antivirus detection, to data loss prevention (DLP). An ICAP device is generally deployed as a service that listens on a specific IP and port, and accepts ICAP-enclosed requests as defined by RFC3057.

___

${\large{\textbf{\textsf{\color{red}Create\ ICAP\ Inspection\ Service}}}}$

For the **ICAP** inspection service, the _minimum_ requirements for defining are illustrated in the following examples. The resulting ID will be at ```.id```.

**Basic**
```bash
POST {{CM}}/api/v1/spaces/default/security/inspection-services
```
```json
{
  "name": "my-sslo-icap",
  "description": "My SSLO ICAP Inspection Service",
  "type": "icap",
  "requestModificationURI": "avscan",
  "responseModificationURI": "avscan",
  "oneConnect": { "sourceMask": "0.0.0.0" },
  "serviceDownAction": "ignore",
  "network": {
    "vlan": "sslo-insp-icap",
    "endpoints": [
      {
        "address": "198.19.97.10:1344"
      },
      {
        "address": "198.19.97.11:1344"
      }
    ]
  }
}
```
**Curl**
```bash
INSP=$(cat <<EOF
{
  "name": "my-sslo-icap",
  "description": "My SSLO ICAP Inspection Service",
  "type": "icap",
  "requestModificationURI": "avscan",
  "responseModificationURI": "avscan",
  "oneConnect": { "sourceMask": "0.0.0.0" },
  "serviceDownAction": "ignore",
  "network": {
    "vlan": "sslo-insp-icap",
    "endpoints": [
      {
        "address": "198.19.97.10:1344"
      },
      {
        "address": "198.19.97.11:1344"
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

| Required | Attribute               | Defaults | Notes                                                                                                                                                                                                                                                                                                                     |
|:--------:|-------------------------|:--------:|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| *        | name                    |          |                                                                                                                                                                                                                                                                                                                           |
| *        | description             |          |                                                                                                                                                                                                                                                                                                                           |
| *        | type                    |          | string: must be "**icap**"                                                                                                                                                                                                                                                                                                |
| *        | requestModificationURI  |          |                                                                                                                                                                                                                                                                                                                           |
| *        | responseModificationURI |          |                                                                                                                                                                                                                                                                                                                           |
| *        | serviceDownAction       | none     | "ignore", "drop", or "reset"                                                                                                                                                                                                                                                                                              |
| *        | oneConnect              | none     | {   "sourceMask": "0.0.0.0" }                                                                                                                                                                                                                                                                                             |
| *        | monitor                 | none     | "tcp": {<br /> &emsp;"interval": 10,<br /> &emsp;"timeout": 10<br /> }<br /> <br /> "http" {<br /> &emsp;"interval": 10,<br /> &emsp;"timeout": 10,<br /> &emsp;"sendString": "",<br /> &emsp;"receiveString": "",<br /> &emsp;"receiveDisableString": "",<br /> &emsp;"username": "",<br /> &emsp;"password": ""<br /> } |
| *        | network: vlan           |          | string: vlan-name                                                                                                                                                                                                                                                                                                         |
| *        | network: endpoints      |          | array: { "address":"ip-address:port" }                                                                                                                                                                                                                                                                                    |
|          | previewLength           | 0        |                                                                                                                                                                                                                                                                                                                           |
|          | headerFrom              |          | string                                                                                                                                                                                                                                                                                                                    |
|          | host                    |          | string                                                                                                                                                                                                                                                                                                                    |
|          | referer                 |          | string                                                                                                                                                                                                                                                                                                                    |
|          | userAgent               |          | string                                                                                                                                                                                                                                                                                                                    |
|          | allowHTTP1.0            | false    | boolean                                                                                                                                                                                                                                                                                                                   |


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Ansible\ Reference}}}}$

Execute with:
```
bigip_next_cm_mgmt_ip="10.1.1.6"
bigip_next_password="my_password"
ansible-playbook -i notahost, sslo-insp-icap.yaml --extra-vars "bigip_next_cm_mgmt_ip=$bigip_next_cm_mgmt_ip bigip_next_password=$bigip_next_password"
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
    
    - name: Create SSLO ICAP Inspection Service
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/v1/spaces/default/security/inspection-services
        method: POST
        headers:
          Authorization: "Bearer {{ bigip_next_cm_token }}"
          Content-Type: application/json
        body: |
          {
            "name": "my-sslo-icap",
            "description": "My SSLO ICAP Inspection Service",
            "type": "icap",
            "requestModificationURI": "avscan",
            "responseModificationURI": "avscan",
            "oneConnect": { "sourceMask": "0.0.0.0" },
            "serviceDownAction": "ignore",
            "network": {
              "vlan": "sslo-insp-icap",
              "endpoints": [
                {
                  "address": "198.19.97.10:1344"
                },
                {
                  "address": "198.19.97.11:1344"
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

    - name: Deploy SSLO ICAP Inspection Service to BIG-IP Instance
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


