## Inline L3 Inspection Service

${\large{\textbf{\textsf{\color{red}Definition}}}}$

The inline layer 3 inspection service is defined by the following characteristics:

- It assigns IP addresses to network interfaces and participates in traffic routing.
- It has physically or logically separate inbound (to-device) and outbound (from-device) interfaces.

Many modern security products fit into this category, including products by Cisco, Palo Alto, Fortinet and Checkpoint. Inline layer 3 devices route traffic, and can modify or break (i.e. reset, drop) that traffic in real time. The SSL Orchestrator routes traffic to the service from one VLAN, and the service will typically gateway route the traffic back to the F5 BIG-IP on another VLAN.

___

${\large{\textbf{\textsf{\color{red}Create\ Inline\ L3\ Inspection\ Service}}}}$

For the **Inline L3** inspection service, the _minimum_ requirements for defining are illustrated in the following examples. The resulting ID will be at ```.id```.

**Basic**
```bash
POST {{CM}}/api/v1/spaces/default/security/inspection-services
```
```json
{
  "name": "my-sslo-ngfw",
  "description": "My SSLO L3 Inspection Service",
  "type": "l3",
  "serviceDownAction": "ignore",
  "to": {
    "network": {
      "vlan": "sslo-insp-l3-in",
      "endpoints": [
        {
          "address": "198.19.64.30:0"
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
      "vlan": "sslo-insp-l3-out"
    }
  }
}
```
**Curl**
```bash
INSP=$(cat <<EOF
{
  "name": "my-sslo-ngfw",
  "description": "My SSLO L3 Inspection Service",
  "type": "l3",
  "serviceDownAction": "ignore",
  "to": {
    "network": {
      "vlan": "sslo-insp-l3-in",
      "endpoints": [
        {
          "address": "198.19.64.30:0"
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
      "vlan": "sslo-insp-l3-out"
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
| *        | type                              | none     | string: must be "**l3**"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| *        | serviceDownAction                 | none     | "ignore", "drop", or "reset"                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| *        | to: network: vlan                 | none     | string: vlan-name                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| *        | to: network: endpoints: address   | none     | string: ip-address:0                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| *        | to: network: snat: snatType       | none     | "NONE", "AUTOMAP", or "POOL"                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|          | to: network: snat: snatType: POOL | none     | "addresses": [<BR /> &emsp;"10.0.0.200",<BR /> &emsp;"10.0.0.201"<BR /> ]                                                                                                                                                                                                                                                                                                                                                                                                                                |
| *        | monitor                           |          | "icmp": {<br /> &emsp;"interval": 5,<br /> &emsp;"timeout": 16<br /> }<br /> <br /> "http" {<br /> &emsp;"interval": 5,<br /> &emsp;"timeout": 16,<br /> &emsp;"sendString": "",<br /> &emsp;"receiveString": "",<br /> &emsp;"receiveDisableString": "",<br /> &emsp;"username": "",<br /> &emsp;"password": ""<br /> }<br /> "tcp": {<br /> &emsp;"interval": 5,<br /> &emsp;"timeout": 16<br /> &emsp;"sendString": ""<br /> &emsp;"receiveString": ""<br /> &emsp;"receiveDisableString": ""<br /> } |
| *        | from: network: vlan               |          | string: vlan-name                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
