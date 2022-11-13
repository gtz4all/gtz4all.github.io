---
title: "Azure CLI Useful Commands"
---

- Get Subscription Details
  ```
  az account list
  ```
  ```hcl
  ## Output
  [
    {
      "cloudName": "AzureCloud",
      "id": "4cedc5dd-e3ad-468d-bf66-32e31bdb9148",
      "isDefault": true,
      "name": "P1-Real Hands-On Labs",
      "state": "Enabled",
      "tenantId": "3617ef9b-98b4-40d9-ba43-e1ed6709cf0d",
      "user": {
        "name": "cloud_user_p_27fd299f@azurelabs.linuxacademy.com",
        "type": "user"
      }
    }
  ]
  ```
  
- Get Subscription Id
  ```
  az account list --query '[].id' -o tsv
  ```
  
- Get resource group Name
  ```
  az group list --query '[].name' -o tsv
  ```

- Get resource group location
  ```
  az group list --query '[].location' -o tsv
  ```
