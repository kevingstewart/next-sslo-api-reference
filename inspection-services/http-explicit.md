## Inline HTTP Explicit Inspection Service

${\large{\textbf{\textsf{\color{red}Definition}}}}$

The inline HTTP Explicit inspection service is defined by the following characteristics:

- It assigns IP addresses to network interfaces and participates in traffic routing.
- It has physically or logically separate inbound (to-device) and outbound (from-device) interfaces.
- It performs HTTP proxy functions and will typically change some if not all of the L3/L4 flow characteristics (ex. client/server IPs and ports).
- It listens on a specific IP and port and expects traffic to be directed this listener.

Many modern security products fit into this category, including products by Cisco, Symantec, McAfee and Forcepoint. Inline HTTP devices route traffic, very much like layer 3 devices, and can modify or break (i.e. reset, drop) that traffic in real time. The SSL Orchestrator routes traffic to the service from one VLAN, and the service will typically gateway route the traffic back to the F5 BIG-IP on another VLAN. Inline HTTP devices can also be defined as explicit or transparent forward proxy.

___

${\large{\textbf{\textsf{\color{red}Create\ Inline\ HTTP\ Explicit\ Inspection\ Service}}}}$

For the **Inline HTTP Explicit** inspection service, the _minimum_ requirements for defining are illustrated in the following examples. The resulting ID will be at ```.id```.

**Basic**
```bash
POST {{CM}}/api/v1/spaces/default/security/inspection-services
```
```json
{
  "name": "my-sslo-eproxy",
  "description": "My SSLO HTTP Explicit Service",
  "type": "http-explicit",
  "serviceDownAction": "ignore",
  "to": {
    "network": {
      "vlan": "sslo-insp-eproxy-in",
      "endpoints": [
        {
          "address": "198.19.96.30:3128"
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
      "vlan": "sslo-insp-eproxy-out"
    }
  }
}
```
**Curl**
```bash
INSP=$(cat <<EOF
{
  "name": "my-sslo-eproxy",
  "description": "My SSLO HTTP Explicit Service",
  "type": "http-explicit",
  "serviceDownAction": "ignore",
  "to": {
    "network": {
      "vlan": "sslo-insp-eproxy-in",
      "endpoints": [
        {
          "address": "198.19.96.30:3128"
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
      "vlan": "sslo-insp-eproxy-out"
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
| *        | type                              | none     | string: must be "**http-explicit**"                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| *        | serviceDownAction                 | none     | "ignore", "drop", or "reset"                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| *        | to: network: vlan                 | none     | string: vlan-name                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| *        | to: network: endpoints: address   | none     | string: ip-address:port                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| *        | to: network: snat: snatType       | none     | "NONE", "AUTOMAP", or "POOL"                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|          | to: network: snat: snatType: POOL | none     | "addresses": [<BR /> &emsp;"10.0.0.200",<BR /> &emsp;"10.0.0.201"<BR /> ]                                                                                                                                                                                                                                                                                                                                                                                                                                |
| *        | monitor                           |          | "icmp": {<br /> &emsp;"interval": 5,<br /> &emsp;"timeout": 16<br /> }<br /> <br /> "http" {<br /> &emsp;"interval": 5,<br /> &emsp;"timeout": 16,<br /> &emsp;"sendString": "",<br /> &emsp;"receiveString": "",<br /> &emsp;"receiveDisableString": "",<br /> &emsp;"username": "",<br /> &emsp;"password": ""<br /> }<br /> "tcp": {<br /> &emsp;"interval": 5,<br /> &emsp;"timeout": 16<br /> &emsp;"sendString": ""<br /> &emsp;"receiveString": ""<br /> &emsp;"receiveDisableString": ""<br /> } |
| *        | from: network: vlan               |          | string: vlan-name                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
