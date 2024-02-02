## TAP-VLAN Inspection Service

${\large{\textbf{\textsf{\color{red}Definition}}}}$

A TAP inspection service is a passive device, typically with no external IP addresses. SSL Orchestrator passes receive-only copies of the payload packets to a TAP service. 

___

${\large{\textbf{\textsf{\color{red}Create\ TAP\ Inspection\ Service}}}}$

For the **TAP-VLAN** variant, the only requirements for defining are name, description, type (tap-vlan), and corresponding VLAN (L1-network). Optionally, a **destinationMacAddress** can be asserted within the **network** block. The resulting ID will be at ```.id```.

**Basic**
```bash
POST {{CM}}/api/v1/spaces/default/security/inspection-services
```
```json
{
  "name": "my-sslo-tap",
  "description": "My SSLO Tap Inspection Service",
  "type": "tap-vlan",
  "network": {
    "vlan": "sslo-insp-tap"
  }
}
```
**Curl**
```bash
INSP=$(cat <<EOF
{
  "name": "my-sslo-tap",
  "description": "My SSLO Tap Inspection Service",
  "type": "tap-vlan",
  "network": {
    "vlan": "sslo-insp-tap"
  }
}
EOF
)
insp_id=$(curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/security/inspection-services" -d "${INSP}" |jq -r '.id')
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Reference}}}}$

[table]



