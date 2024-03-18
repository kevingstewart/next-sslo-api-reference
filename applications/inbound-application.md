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












