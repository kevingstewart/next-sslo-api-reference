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

${\large{\textbf{\textsf{\color{red}Deploy\ Inspection\ Service}}}}$

Deploying to a BIG-IP instance requires the inspection service ID in the REST URL, and the BIG-IP Next instance ID in the JSON payload.

**Basic**
```bash
POST {{CM}}/api/v1/spaces/default/security/inspection-services/{{insp_id}}/deployment
```
```json
{
  "deploy-instances": [
    "{{bigip_id}}"
  ],
  "undeploy-instances": []
}
```
**Curl**
```bash
DEPLOY=$(cat <<EOF
{
  "deploy-instances": [
    "${bigip_id}"
  ],
  "undeploy-instances": []
}
EOF
)
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/security/inspection-services/${insp_id}/deployment" -d "${DEPLOY}"
```
