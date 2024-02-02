## ICAP Inspection Service

${\large{\textbf{\textsf{\color{red}Definition}}}}$

The Internet Content Adaption Protocol (ICAP) is defined by RFC3507 and constitutes an encapsulation protocol. Packets are encapsulated by an ICAP client and passed to an ICAP server. What the ICAP server does with the encapsulated data depends on the underlying service, and typically ranges from malware and antivirus detection, to data loss prevention (DLP). An ICAP device is generally deployed as a service that listens on a specific IP and port, and accepts ICAP-enclosed requests as defined by RFC3057.

___

${\large{\textbf{\textsf{\color{red}Create\ ICAP\ Inspection\ Service}}}}$

For the **ICAP** inspection service, the _minimum_ requirements for defining are illustrated in the following examples. The resulting ID will be at ```.id```.

**Basic**
```bash
POST {{CM}}/api/v1/spaces/default/security/inspection-services
```
```json
{
  "name": "my-sslo-icap",
  "description": "My SSLO ICAP Inspection Service",
  "type": "icap",
  "requestModificationURI": "avscan",
  "responseModificationURI": "avscan",
  "previewLength": 0,
  "serviceDownAction": "ignore",
  "monitor": {
    "tcp": {
      "interval": 10,
      "timeout": 10
    }
  },
  "network": {
    "vlan": "sslo-insp-icap",
    "endpoints": [
      {
        "address": "198.19.97.10:1344"
      },
      {
        "address": "198.19.97.11:1344"
      }
    ]
  }
}
```
**Curl**
```bash
INSP=$(cat <<EOF
{
  "name": "my-sslo-icap",
  "description": "My SSLO ICAP Inspection Service",
  "type": "icap",
  "requestModificationURI": "avscan",
  "responseModificationURI": "avscan",
  "previewLength": 0,
  "serviceDownAction": "ignore",
  "monitor": {
    "tcp": {
      "interval": 10,
      "timeout": 10
    }
  },
  "network": {
    "vlan": "sslo-insp-icap",
    "endpoints": [
      {
        "address": "198.19.97.10:1344"
      },
      {
        "address": "198.19.97.11:1344"
      }
    ]
  }
}
EOF
)
insp_id=$(curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/security/inspection-services" -d "${INSP}" |jq -r '.id')
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Reference}}}}$

[table]
