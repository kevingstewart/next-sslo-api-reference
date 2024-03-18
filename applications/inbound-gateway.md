## SSL Orchestrator Inbound Gateway

${\large{\textbf{\textsf{\color{red}Definition}}}}$

An **Inbound Gateway** is the entry point mode where the client targets an address and port hosted *behind* the BIG-IP, where the BIG-IP is a routed next-hop in that path. The primary attributes of the inbound gateway are a wildcard IP (and optional wildcard port) listener, no pool assignment, and address and port translation (NAT and PAT) disabled. The SSL Orchestrator inbound gateway deployment essentially binds a policy to a specialized application template.


___

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Create\ an\ Inbound\ Gateway}}}}$

The basic implementation of SSL Orchestrator in Next is to attach a policy to an application AS3 declaration. The Inbound Gateway requires a few additional options compared to Inbound Application.

```json
"policySslOrchestrator": {
   "cm": "my-sslo-gw-policy"
}
```

This is essentially all that is required for an inbound application workflow, using the "default" application template. The other use cases defined above require specialized application templates in the CM UI, and also some modifications to the AS3 declaration as addressed in the separate pages in this section. This document cannot be exhaustive on all the AS3 attributes available in an application declaration, but all application deployments require two steps: Create the application in CM, deploy that application to a BIG-IP Next instance. The below example provides the bare minimum inbound application declaration. 

**Basic: Create Application**
```bash
POST /api/v1/spaces/default/appsvcs/documents
```
```json
{
  "class": "ADC",
  "id": "http-canonical-cm-example-template",
  "label": "Central Manager Example Template",
  "schemaVersion": "3.43.0",
  "my_tenant": {
    "class": "Tenant",
    "my_app": {
      "class": "Application",
      "my_server_tls": {
        "class": "TLS_Server",
        "certificates": [
          {
           "certificate": "webcert",
           "sniDefault": true
          }
        ],
        "ciphers": "DEFAULT",
        "tls1_1Enabled": false,
        "tls1_2Enabled": true,
        "tls1_3Enabled": true
      },
      "my_client_tls": {
        "class": "TLS_Client",
        "ciphers": "DEFAULT",
        "tls1_1Enabled": false,
        "tls1_2Enabled": true,
        "tls1_3Enabled": true
      },
      "my_service": {
        "class": "Service_HTTPS",
        "allowNetworks": [
          {
            "bigip": "Default L3-Network"
          }
        ],
        "persistenceMethods": [],
        "policySslOrchestrator": {
          "cm": "my-sslo-gw-policy"
        },
        "clientTLS": "my_client_tls",
        "serverTLS": "my_server_tls",
        "snat": "auto",
        "translateServerAddress": false,
        "virtualAddresses": [
          "0.0.0.0"
        ],
        "virtualPort": 443
      },
      "webcert": {
        "class": "Certificate",
        "certificate": {
          "cm": "wildcard.f5labs.com.crt"
        },
        "privateKey": {
          "cm": "wildcard.f5labs.com.pem"
        }
      }
    }
  }
}
```
Notice that there is no pool defined in this declaration, and that "translateServerAddress" is false. The virtualAddress is a wildcard listener (0.0.0.0), and in this case the virtualPort is 443 to specifically intercept HTTPS traffic. The response to successful creation will contain a JSON payload including an ID value. That ID value is then used in the following request to deploy the application to a BIG-IP Next instance, where **{{Next-Instance-IP-Address}}** is the IP address of the target instance.

**Basic: Deploy Application**
```bash
POST /api/v1/spaces/default/appsvcs/documents/{{application_id}}/deployments
```
```json
{
  "target": "{{Next-Instance-IP-Address}}"
}
```

**Curl: Create Application**
```bash
APP=$(cat <<EOF
{
  "class": "ADC",
  "id": "http-canonical-cm-example-template",
  "label": "Central Manager Example Template",
  "schemaVersion": "3.43.0",
  "my_tenant": {
    "class": "Tenant",
    "my_app": {
      "class": "Application",
      "my_server_tls": {
        "class": "TLS_Server",
        "certificates": [
          {
           "certificate": "webcert",
           "sniDefault": true
          }
        ],
        "ciphers": "DEFAULT",
        "tls1_1Enabled": false,
        "tls1_2Enabled": true,
        "tls1_3Enabled": true
      },
      "my_client_tls": {
        "class": "TLS_Client",
        "ciphers": "DEFAULT",
        "tls1_1Enabled": false,
        "tls1_2Enabled": true,
        "tls1_3Enabled": true
      },
      "my_service": {
        "class": "Service_HTTPS",
        "allowNetworks": [
          {
            "bigip": "Default L3-Network"
          }
        ],
        "persistenceMethods": [],
        "policySslOrchestrator": {
          "cm": "my-sslo-gw-policy"
        },
        "clientTLS": "my_client_tls",
        "serverTLS": "my_server_tls",
        "snat": "auto",
        "translateServerAddress": false,
        "virtualAddresses": [
          "0.0.0.0"
        ],
        "virtualPort": 443
      },
      "webcert": {
        "class": "Certificate",
        "certificate": {
          "cm": "wildcard.f5labs.com.crt"
        },
        "privateKey": {
          "cm": "wildcard.f5labs.com.pem"
        }
      }
    }
  }
}
EOF
)
app_id=$(curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/appsvcs/documents" -d "${APP}")
```

**Curl: Deploy Application**
```bash
DEPLOY=$(cat <<EOF
{
  "target": "${Next-Instance-IP-Address}"
}
EOF
)
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/appsvcs/documents/${app_id}/deployments" -d "${DEPLOY}"
```


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Ansible\ Reference}}}}$

Execute with:
```
bigip_next_cm_mgmt_ip="10.1.1.6"
bigip_next_password="my_password"
ansible-playbook -i notahost, sslo-inbound-gateway.yaml --extra-vars "bigip_next_cm_mgmt_ip=$bigip_next_cm_mgmt_ip bigip_next_password=$bigip_next_password"
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
    
    
    - name: Create SSLO Inbound Gateway
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/v1/spaces/default/appsvcs/documents
        method: POST
        headers:
          Authorization: "Bearer {{ bigip_next_cm_token }}"
          Content-Type: application/json
        body: |
          {
             "class": "ADC",
             "id": "http-canonical-cm-example-template",
             "label": "Central Manager Example Template",
             "schemaVersion": "3.43.0",
             "my_tenant": {
               "class": "Tenant",
               "my_app": {
                 "class": "Application",
                 "my_server_tls": {
                   "class": "TLS_Server",
                   "certificates": [
                     {
                       "certificate": "webcert",
                       "sniDefault": true
                     }
                   ],
                   "ciphers": "DEFAULT",
                   "tls1_1Enabled": false,
                   "tls1_2Enabled": true,
                   "tls1_3Enabled": true
                 },
                 "my_client_tls": {
                   "class": "TLS_Client",
                   "ciphers": "DEFAULT",
                   "tls1_1Enabled": false,
                   "tls1_2Enabled": true,
                   "tls1_3Enabled": true
                 },
                 "my_service": {
                   "class": "Service_HTTPS",
                   "allowNetworks": [
                     {
                       "bigip": "Default L3-Network"
                     }
                   ],
                   "persistenceMethods": [],
                   "policySslOrchestrator": {
                     "cm": "my-sslo-gw-policy"
                   },
                   "clientTLS": "my_client_tls",
                   "serverTLS": "my_server_tls",
                   "snat": "auto",
                   "translateServerAddress": false,
                   "virtualAddresses": [
                     "0.0.0.0"
                   ],
                   "virtualPort": 443
                 },
                 "webcert": {
                   "class": "Certificate",
                   "certificate": {
                     "cm": "wildcard.f5labs.com.crt"
                   },
                   "privateKey": {
                     "cm": "wildcard.f5labs.com.pem"
                   }
                 }
               }
             }
           }
        body_format: json
        timeout: 60
        status_code: 200
        validate_certs: false
      register: json_response


    - name: Set application ID
      set_fact:
        app_id: "{{ json_response.json.id}}"


    - name: Deploy Inbound Gateway to BIG-IP Instance
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/v1/spaces/default/appsvcs/documents/{{ app_id }}/deployments
        method: POST
        headers:
          Authorization: "Bearer {{ bigip_next_cm_token }}"
          Content-Type: application/json
        body: |
          {
            "target": "10.1.1.7"
          }
        body_format: json
        timeout: 60
        status_code: 202
        validate_certs: false
      register: json_response


    - debug:
        var: json_response
```










