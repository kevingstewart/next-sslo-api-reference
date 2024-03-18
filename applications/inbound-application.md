## SSL Orchestrator Inbound Application

${\large{\textbf{\textsf{\color{red}Definition}}}}$

An **Inbound Application** is the entry point mode where the client targets an address and port hosted on the BIG-IP. The primary attributes of the inbound application are an IP address and port-specific listener, pool assignment, and address and port translation (NAT and PAT). The SSL Orchestrator inbound application deployment essentially binds a policy to a typical "default" application template.


___

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Create\ an\ Application}}}}$

The basic implementation of SSL Orchestrator in Next is to simply attach a policy to an application AS3 declaration.

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
  "id": "adc-canonical",
  "schemaVersion": "3.43.0",
  "my_tenant": {
    "class": "Tenant",
    "my_app": {
      "class": "Application",
      "my_server_tls": {
        "class": "TLS_Server",
        "certificates": [
          {
            "certificate": "webcert"
          }
        ],
        "ciphers": "DEFAULT",
        "tls1_1Enabled": true,
        "tls1_2Enabled": true,
        "tls1_3Enabled": false
      },
      "my_client_tls": {
        "class": "TLS_Client",
        "ciphers": "DEFAULT",
        "tls1_1Enabled": true,
        "tls1_2Enabled": true,
        "tls1_3Enabled": false
      },
      "my_pool": {
        "class": "Pool",
        "loadBalancingMode": "round-robin",
        "members": [
          {
            "serverAddresses": [
              "192.168.100.11",
              "192.168.100.12",
              "192.168.100.13"
            ],
            "servicePort": 443
          }
        ],
        "monitors": [
          "https"
        ]
      },
      "my_pool_service": {
        "class": "Service_Pool",
        "pool": "my_pool"
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
          "cm": "my-api-policy"
        },
        "clientTLS": "my_client_tls",
        "serverTLS": "my_server_tls",
        "pool": "my_pool",
        "snat": "auto",
        "virtualAddresses": [
          "10.1.10.22"
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
The response to successful creation will contain a JSON payload including an ID value. That ID value is then used in the following request to deploy the application to a BIG-IP Next instance, where **{{Next-Instance-IP-Address}}** is the IP address of the target instance.

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
  "id": "adc-canonical",
  "schemaVersion": "3.43.0",
  "my_tenant": {
    "class": "Tenant",
    "my_app": {
      "class": "Application",
      "my_server_tls": {
        "class": "TLS_Server",
        "certificates": [
          {
            "certificate": "webcert"
          }
        ],
        "ciphers": "DEFAULT",
        "tls1_1Enabled": true,
        "tls1_2Enabled": true,
        "tls1_3Enabled": false
      },
      "my_client_tls": {
        "class": "TLS_Client",
        "ciphers": "DEFAULT",
        "tls1_1Enabled": true,
        "tls1_2Enabled": true,
        "tls1_3Enabled": false
      },
      "my_pool": {
        "class": "Pool",
        "loadBalancingMode": "round-robin",
        "members": [
          {
            "serverAddresses": [
              "192.168.100.11",
              "192.168.100.12",
              "192.168.100.13"
            ],
            "servicePort": 443
          }
        ],
        "monitors": [
          "https"
        ]
      },
      "my_pool_service": {
        "class": "Service_Pool",
        "pool": "my_pool"
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
          "cm": "my-api-policy"
        },
        "clientTLS": "my_client_tls",
        "serverTLS": "my_server_tls",
        "pool": "my_pool",
        "snat": "auto",
        "virtualAddresses": [
          "10.1.10.22"
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
ansible-playbook -i notahost, sslo-inbound-application.yaml --extra-vars "bigip_next_cm_mgmt_ip=$bigip_next_cm_mgmt_ip bigip_next_password=$bigip_next_password"
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
    
    
    - name: Create SSLO Inbound Application
      uri:
        url: https://{{ bigip_next_cm_mgmt_ip }}/api/v1/spaces/default/appsvcs/documents
        method: POST
        headers:
          Authorization: "Bearer {{ bigip_next_cm_token }}"
          Content-Type: application/json
        body: |
          {
            "class": "ADC",
            "id": "adc-canonical",
            "schemaVersion": "3.43.0",
            "my_tenant": {
              "class": "Tenant",
              "my_app": {
                "class": "Application",
                "my_server_tls": {
                  "class": "TLS_Server",
                  "certificates": [
                    {
                      "certificate": "webcert"
                    }
                  ],
                  "ciphers": "DEFAULT",
                  "tls1_1Enabled": true,
                  "tls1_2Enabled": true,
                  "tls1_3Enabled": false
                },
                "my_client_tls": {
                  "class": "TLS_Client",
                  "ciphers": "DEFAULT",
                  "tls1_1Enabled": true,
                  "tls1_2Enabled": true,
                  "tls1_3Enabled": false
                },
                "my_pool": {
                  "class": "Pool",
                  "loadBalancingMode": "round-robin",
                  "members": [
                    {
                      "serverAddresses": [
                        "192.168.100.11",
                        "192.168.100.12",
                        "192.168.100.13"
                      ],
                      "servicePort": 443
                    }
                  ],
                  "monitors": [
                    "https"
                  ]
                },
                "my_pool_service": {
                  "class": "Service_Pool",
                  "pool": "my_pool"
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
                    "cm": "my-api-policy"
                  },
                  "clientTLS": "my_client_tls",
                  "serverTLS": "my_server_tls",
                  "pool": "my_pool",
                  "snat": "auto",
                  "virtualAddresses": [
                    "10.1.10.22"
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


    - name: Deploy Inbound Application to BIG-IP Instance
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










