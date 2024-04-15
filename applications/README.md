## SSL Orchestrator Applications

${\large{\textbf{\textsf{\color{red}Definition}}}}$

An **Application** is the entry point for network traffic into SSL Orchestrator, and is expressed as a number of use cases based on type and direction of traffic flowing through the BIG-IP:

* Inbound Application - Generic inbound application
* Inbound Gateway - Inbound routed "next hop"
* Outbound Gateway - Outbound routed "next hop"
* Outbound Explicit - Outbound explicit proxy
* Inbound/Outbound Layer 2 - A "bump-in-the-wire" pathway

An **Inbound** flow is generally characterized as a reverse proxy, external users sending requests into a locally-managed server environment. In a reverse proxy, TLS offload and certificate management are handled by locally-managed certificates. An **Outbound** flow is generally characterized as a forward proxy, internal users sending requests to external (ex. Internet) resources. In a forward proxy, TLS decryption and re-encryption is handled through an *SSL Forward Proxy* process that forges a copy of the remote server certificate to the internal client.


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Common\ Application\ Patterns}}}}$

SSL Orchestrator applications all share a common set of API patterns that can be described here:

* Getting the application information
* Creating an application
* Deleting an application

For the sake of this completeness, there are multiple ways to deploy an application through CM: FAST template, and AS3 declaration. This documentation strictly addresses the AS3 methods.

___

*Note:*

* *Values below in {{double curly brackets}} indicate typical environment variables defined in Postman or the Visual Studio Code Thunder Client.*
* *The {{CM}} variable would be a statically defined Postman/Thunder environment variable for the Central Manager host/IP.*
* *The ${CM} variable in the Bash commands is the export of the Central Manager host/IP (ex. ```export CM=10.1.1.6```)*
___


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Get\ Applications}}}}$

To fetch an SSL Orchestrator application use the following GET. The ID will be at ```_embedded.appsvcs[X].id``` where X is the index.

**Basic**
```bash
GET {{CM}}/api/v1/spaces/default/appsvcs/documents
```
**Filtered** by application *name* string (ex. my_app), where resulting ID is at ```_embedded.appsvcs[0].id```
```bash
GET {{CM}}/api/v1/spaces/default/appsvcs/documents/?filter=name+eq+%27my_app%27&select=name,id
```
**Curl**
```bash
chain_id=$(curl -sk -H "Authorization: Bearer ${token}" "https://${CM}/api/v1/spaces/default/appsvcs/documents/?filter=name+eq+%27my_app%27&select=name,id" |jq -r '._embedded.appsvcs[0].id')
```

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

${\large{\textbf{\textsf{\color{red}Delete\ an\ Application}}}}$

To delete an SSL Orchestrator application, it must first be removed from any associated BIG-IP Next instances. The REST call to delete requires the policy ID.

**Basic: Get Application**
```bash
GET {{CM}}/api/v1/spaces/default/appsvcs/documents
```
You'll need the application ID at **_embedded.appsvcs[0].id**, and the deployment ID at **_embedded.appsvcs[0].deployments[0].id**.

**Basic: Delete Application Deployment**
```bash
DELETE {{CM}}/api/v1/spaces/default/appsvcs/documents/{{application_id}}/deployments/{{deployment_id}}
```
You can now delete the application in CM.

**Basic: Delete Application**
```bash
DELETE {{CM}}/api/v1/spaces/default/appsvcs/documents/{{application_id}}
```

**Curl: Get Application**
```bash
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" -X GET "https://${CM}/api/v1/spaces/default/appsvcs/documents"
```
**Curl: Delete Application Deployment**
```bash
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" -X DELETE "https://${CM}/api/v1/spaces/default/appsvcs/documents/{{application_id}}/deployments/{{deployment_id}}"
```
**Curl: Delete Application**
```bash
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" -X DELETE "https://${CM}/api/v1/spaces/default/appsvcs/documents/{{application_id}}"
```








