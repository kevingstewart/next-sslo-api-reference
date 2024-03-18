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












