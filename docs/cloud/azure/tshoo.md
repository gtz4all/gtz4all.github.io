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


#### vhub routing

###### Before vhub Firewall
- VPN Gateway
  ![image](https://user-images.githubusercontent.com/40032360/202820743-9587965e-3c15-4ed4-841d-40ca65b1e62a.png)
  
  ![image](https://user-images.githubusercontent.com/40032360/202820882-cdb74164-5105-4396-9b5d-c5606e7f7033.png)

- Default Route Table - Effective Routes
  Route table 
  ![image](https://user-images.githubusercontent.com/40032360/202821542-5f17afeb-396b-463d-9470-6144ae7cb96e.png)

  
