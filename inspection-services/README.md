## SSL Orchestrator Inspection Services

${\large{\textbf{\textsf{\color{red}Definition}}}}$

An **Inspection Service** represents a single security product (ex. FireEye NX, Palo Alto NGFW, McAfee DLP, etc.). However, a service and a “device” refer to different things in SSL Orchestrator. A device in this case is a single appliance, while a service represents the load balanced and monitored set of like devices. For example, a single FireEye service might define a set of multiple FireEye NX appliances (devices). SSL Orchestrator allows administrators to create many services, each containing a number of like devices. Services fall into five categories based on how they consume network traffic. Almost all modern security products today fit into at least one category, but in some cases a security product can be configured to operate in different modes. SSL Orchestrator supports the following service types:

* **Inline Layer 3** - an inline L3 device is an IP-based device that can route. It is “inline” in the sense that network traffic enters one (physical or logical) interface and routes out another. In this case, the SSL Orchestrator routes to it, and it routes back.
* **TAP** - a TAP device is a passive receive-only service. Intrusion Detection Systems (IDS) are commonly deployed as TAP, and simply receive an out-of-band copy of the network packets.
* **ICAP** - an ICAP device is one that adheres to RFC 3507 specifications. ICAP itself is a payload encapsulation protocol and is often used to encapsulate payloads to various types of services, including DLP and malware detection products. In this case SSL Orchestrator is the ICAP client, encapsulating HTTP request and response payloads to an ICAP service.
* **Inline HTTP** - an inline HTTP device is a subset of inline L3 devices, except that it is a proxy-type device. Proxy-type devices can be transparent or explicit, but the most important difference between HTTP and L3 devices is that HTTP devices, as a function of proxying, change the TCP flows. In particular, a proxy device will always minimally change the source port but may also change source and destination addresses.
* **Inline Layer 2** - an inline L2 device does not have IP addresses and does not participate in routing. It is effectively no more than a physical presence on the wire.

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Common\ Inspection\ Service\ Patterns}}}}$

SSL Orchestrator inspection services all share a common set of API patterns that can be described here:

* Getting the list of BIG-IP instances (needed for instance IDs)
* Getting the list of inspection services (needed for inspection service IDs)
* Creating an inspection service (with specific documentation for each in different pages)
* Deploying a created inspection service to a BIG-IP instance
* Un-deploying an inspection service from a BIG-IP instance
* Deleting an inspection service
* Updating an inspection service

___

${\large{\textbf{\textsf{\color{red}Get\ BIG-IP\ Next\ Instances}}}}$

Deploying an inspection service requires two steps: defining the inspection service, and deploying to a BIG-IP Next instance. Creating an inspection service is detailed in the individual sections, but all involve the same deploy to BIG-IP instance pattern. To deploy to an instance, you first need the ID of the BIG-IP Next instance. The ID returned in the following request will be at ```_embedded.devices[X].id``` where X is the index.

**Basic**
```bash
GET {{CM}}/api/v1/spaces/default/instances?select=hostname,id
```
**Filtered** by *hostname* string, where resulting ID is at ```_embedded.devices[0].id```
```bash
GET {{CM}}/api/v1/spaces/default/instances?filter=hostname+eq+%27bigip-next.f5labs.com%27&select=hostname,id
```
**Curl**
```bash
bigip_id=$(curl -sk -H "Authorization: Bearer ${token}" "https://${CM}/api/v1/spaces/default/instances?filter=hostname+eq+%27bigip-next.f5labs.com%27&select=hostname,id" |jq -r '._embedded.devices[0].id')
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Get\ Inspection\ Services}}}}$

To fetch the list of inspection services use the following GET. The ID will be at ```_embedded.inspection_services[X].id``` where X is the index.

**Basic**
```bash
GET {{CM}}/api/v1/spaces/default/security/inspection-services
```
**Filtered** by *name* string, where resulting ID is at ```_embedded.inspection_services[0].id```
```bash
GET {{CM}}/api/v1/spaces/default/security/inspection-services?filter=name+eq+%27my-sslo-tap%27&select=name,id
```
**Curl**
```bash
insp_id=$(curl -sk -H "Authorization: Bearer ${token}" "https://${CM}/api/v1/spaces/default/security/inspection-services?filter=name+eq+%27my-sslo-tap%27&select=name,id" |jq -r '._embedded.inspection_services[0].id')
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Create\ an\ Inspection\ Service}}}}$

All inspection services will have a similar creation workflow, except for the specific JSON payload. The resulting ID value in the JSON response will be at ```.id```.

**Basic**
```bash
POST {{CM}}/api/v1/spaces/default/security/inspection-services
```
```json
{
  <-- inspection service specific properties -->
}
```
**Curl**
```bash
INSP=$(cat <<EOF
{
  <-- inspection service specific properties -->
}
EOF
)
insp_id=$(curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/security/inspection-services" -d "${INSP}")
```


${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Deploy\ an\ Inspection\ Service}}}}$

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

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Un-deploy\ an\ Inspection\ Service}}}}$

Un-deploying to a BIG-IP instance requires the inspection service ID in the REST URL, and the BIG-IP Next instance ID in the JSON payload. If the inspection service is part of deployed service chains, it must first also be removed from those service chains before un-deploying and deleting.

**Basic**
```bash
POST {{CM}}/api/v1/spaces/default/security/inspection-services/{{insp_id}}/deployment
```
```json
{
  "deploy-instances": [],
  "undeploy-instances": [
    "{{bigip_id}}"
  ]
}
```
**Curl**
```bash
UNDEPLOY=$(cat <<EOF
{
  "deploy-instances": [],
  "undeploy-instances": [
    "${bigip_id}"
  ]
}
EOF
)
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/security/inspection-services/${insp_id}/deployment" -d "${UNDEPLOY}"
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Delete\ an\ Inspection\ Service}}}}$

To delete an inspection service, it must fist be un-deployed from any BIG-IP Next instances. The REST call to delete requires the inspection service ID.

**Basic**
```bash
DELETE {{CM}}/api/v1/spaces/default/security/inspection-services/{{insp_id}}
```

${\normalsize{\textsf{\color{white}===}}}$

${\large{\textbf{\textsf{\color{red}Update\ an\ Inspection\ Service}}}}$

Updating an existing inspection service requires the ID of the target inspection service.

**Basic**
```bash
PUT {{CM}}/api/v1/spaces/default/security/inspection-services/{{insp_id}}
```
```json
{
  "name": "my-sslo-tap",
  "description": "My SSLO Tap Inspection Service",
  "type": "tap-vlan",
  "network": {
    "vlan": "sslo-insp-tap",
    "destinationmacAddress": "f5:f5:f5:f5:f5:f5"
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
    "vlan": "sslo-insp-tap",
    "destinationmacAddress": "f5:f5:f5:f5:f5:f5"
  }
}
EOF
)
curl -sk -H "Authorization: Bearer ${token}" -H "Content-Type: application/json" "https://${CM}/api/v1/spaces/default/security/inspection-services/${insp_id}" -d "${INSP}"
```
