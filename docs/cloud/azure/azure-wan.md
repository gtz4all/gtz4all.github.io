---
title: "Azure Virtual WAN Components"
description: "vWAN"
---

#### Virtual Hub Deployment

- Error

```json
{
  "code": "DeploymentFailed",
  "message": "At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/DeployOperations for usage details.",
  "details": [
    {
      "code": "CustomAsnNotEnabledForVirtualHub",
      "message": "Virtual hub is not enabled to allow Router ASN modification. Please set ASN to 65515 or contact Support for enabling virtual hub to allow modification."
    }
  ]
}
```

- Review

```yml
Basics
  Region: South Central US
  Name: vhub2
  Hub private address space: 10.0.0.0/24
  Virtual hub capacity: 2 Routing Infrastructure Units, 3 Gbps Router, Supports 2000 VMs
  Hub routing preference: ExpressRoute
  Router ASN: 65515 ##<--- This ASN cannot be changed

Site to site
  Site to site (VPN gateway): Enabled
  AS Number: 64525 ##<--- This ASN can be changed during build. It cannot be modified later.
  Gateway scale units: 1 scale unit - 500 Mbps x 2
```
