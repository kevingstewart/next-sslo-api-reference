## Inline HTTP Transparent Inspection Service

${\large{\textbf{\textsf{\color{red}Definition}}}}$

The inline HTTP Transparent inspection service is defined by the following characteristics:

- It assigns IP addresses to network interfaces and participates in traffic routing.
- It has physically or logically separate inbound (to-device) and outbound (from-device) interfaces.
- It performs HTTP proxy functions and will typically change some if not all of the L3/L4 flow characteristics (ex. client/server IPs and ports).

Many modern security products fit into this category, including products by Cisco, Symantec, McAfee and Forcepoint. Inline HTTP devices route traffic, very much like layer 3 devices, and can modify or break (i.e. reset, drop) that traffic in real time. The SSL Orchestrator routes traffic to the service from one VLAN, and the service will typically gateway route the traffic back to the F5 BIG-IP on another VLAN. Inline HTTP devices can also be defined as explicit or transparent forward proxy.

___

${\large{\textbf{\textsf{\color{red}Create\ Inline\ HTTP\ Transparent\ Inspection\ Service}}}}$

For the **Inline HTTP Transparent** inspection service, the _minimum_ requirements for defining are illustrated in the following examples. The resulting ID will be at ```.id```.

**Basic**
```bash
POST {{CM}}/api/v1/spaces/default/security/inspection-services
```
```json
{
  "name": "my-sslo-tproxy",
  "description": "My SSLO HTTP Transparent Service",
  "type": "http-transparent",
  "serviceDownAction": "ignore",
  "to": {
    "network": {
      "vlan": "sslo-insp-tproxy-in",
      "endpoints": [
        {
          "address": "198.19.98.30:0"
        }
      ],
      "snat": {
        "snatType": "NONE"
      }
    },
    "monitor": {
      "icmp": {
        "interval": 5,
        "timeout": 16
      }
    }
  },
  "from": {
    "network": {
      "vlan": "sslo-insp-tproxy-out"
    }
  }
}
```
**Curl**
```bash
INSP=$(cat <<EOF
{
  "name": "my-sslo-tproxy",
  "description": "My SSLO HTTP Transparent Service",
  "type": "http-transparent",
  "serviceDownAction": "ignore",
  "to": {
    "network": {
      "vlan": "sslo-insp-tproxy-in",
      "endpoints": [
        {
          "address": "198.19.98.30:0"
        }
      ],
      "snat": {
        "snatType": "NONE"
      }
    },
    "monitor": {
      "icmp": {
        "interval": 5,
        "timeout": 16
      }
    }
  },
  "from": {
    "network": {
      "vlan": "sslo-insp-tproxy-out"
    }
  }
}
EOF
)
insp_id=$(curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/security/inspection-services" -d "${INSP}" |jq -r '.id')
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference}}}}$

| Required | Attribute                         | Defaults | Notes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|:--------:|-----------------------------------|:--------:|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| *        | name                              | none     |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| *        | description                       | none     |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| *        | type                              | none     | string: must be "**http-transparent**"                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| *        | serviceDownAction                 | none     | "ignore", "drop", or "reset"                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| *        | to: network: vlan                 | none     | string: vlan-name                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| *        | to: network: endpoints: address   | none     | string: ip-address:0                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| *        | to: network: snat: snatType       | none     | "NONE", "AUTOMAP", or "POOL"                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|          | to: network: snat: snatType: POOL | none     | "addresses": [<BR /> &emsp;"10.0.0.200",<BR /> &emsp;"10.0.0.201"<BR /> ]                                                                                                                                                                                                                                                                                                                                                                                                                                |
| *        | monitor                           |          | "icmp": {<br /> &emsp;"interval": 5,<br /> &emsp;"timeout": 16<br /> }<br /> <br /> "http" {<br /> &emsp;"interval": 5,<br /> &emsp;"timeout": 16,<br /> &emsp;"sendString": "",<br /> &emsp;"receiveString": "",<br /> &emsp;"receiveDisableString": "",<br /> &emsp;"username": "",<br /> &emsp;"password": ""<br /> }<br /> "tcp": {<br /> &emsp;"interval": 5,<br /> &emsp;"timeout": 16<br /> &emsp;"sendString": ""<br /> &emsp;"receiveString": ""<br /> &emsp;"receiveDisableString": ""<br /> } |
| *        | from: network: vlan               |          | string: vlan-name                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Ansible\ Reference}}}}$

Execute with:
```
bigip_next_cm_mgmt_ip="10.1.1.6"
bigip_next_password="my_password"
ansible-playbook -i notahost, sslo-insp-thttp.yaml --extra-vars "bigip_next_cm_mgmt_ip=$bigip_next_cm_mgmt_ip bigip_next_password=$bigip_next_password"
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
    
    - name: Create SSLO HTTP Transparent Inspection Service
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/v1/spaces/default/security/inspection-services
        method: POST
        headers:
          Authorization: "Bearer {{ bigip_next_cm_token }}"
          Content-Type: application/json
        body: |
          {
            "name": "my-sslo-tproxy",
            "description": "My SSLO HTTP Transparent Service",
            "type": "http-transparent",
            "serviceDownAction": "ignore",
            "to": {
              "network": {
                "vlan": "sslo-insp-tproxy-in",
                "endpoints": [
                  {
                    "address": "198.19.98.30:0"
                  }
                ],
                "snat": {
                  "snatType": "NONE"
                }
              },
              "monitor": {
                "icmp": {
                  "interval": 5,
                  "timeout": 16
                }
              }
            },
            "from": {
              "network": {
                "vlan": "sslo-insp-tproxy-out"
              }
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

    - name: Deploy SSLO HTTP Transparent Inspection Service to BIG-IP Instance
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
