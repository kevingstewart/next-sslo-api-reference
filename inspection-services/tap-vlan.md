## TAP-VLAN Inspection Service

${\large{\textbf{\textsf{\color{red}Definition}}}}$

A TAP inspection service is a passive device, typically with no external IP addresses. SSL Orchestrator passes receive-only copies of the payload packets to a TAP service. For the **TAP-VLAN** variant, the only requirements for defining are name, description, type (tap-vlan), and corresponding VLAN (L1-network). Optionally, a MAC address can be asserted.

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Get\ BIG-IP\ Instance}}}}$

Deploying an inspection service requires two steps: defining the inspection service, and deploying to a BIG-IP Next instance. The following Create API Prototype illustrates both steps. The second step (deployment) requires the ID the of the BIG-IP Next instance, which is obtained from the following GET request. The ID will be at ```_embedded.devices[X].id``` where X is the index.

```bash
GET {{CM}}/api/v1/spaces/default/instances?select=hostname,id
```
or, to grab a specific BIG-IP Next instance, apply a filter, in which case your target ID value will be at ```_embedded.devices[0].id```

```bash
GET {{CM}}/api/v1/spaces/default/instances?filter=hostname%20eq%20%27bigip-next.f5labs.com%27&select=hostname,id
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Get\ Inspection\ Services}}}}$

To fetch the list of inspection services:

```bash
GET {{CM}}/api/v1/spaces/default/security/inspection-services
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Create\ TAP\ Inspection\ Service}}}}$

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

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}API\ Reference}}}}$
