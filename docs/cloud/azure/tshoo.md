---
title: "Azure Troubleshooting Notes"
description: "tshoot notes"
---

#### Creating Virtual Hub

- Virtual hub is not enabled to allow Router ASN modification. However, it's not grayed out so you can change with without having the option enabled. See error below when changing it.

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
  
  - VPN (Site to site) Gateway: ```This gateway is being updated. It may take upto 30 minutes for the update```. 
