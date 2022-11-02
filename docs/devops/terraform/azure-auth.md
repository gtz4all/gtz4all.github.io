---
title: "Terraform Azure Authentication"
---

#### Terraform Authentication

there are multiple ways to provide terraform authentication. the simplest one is using  a Microsoft account using ```az login```.
However this is good for running terraform locally. To run terraform in a CI/CD Pipeline, you need to use a service principal.

#### Requirements
- [Install Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
- [Install Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt)


#### Azure Service Principal

Reference: [Terraform AzureRM Authentication](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/service_principal_client_secret#creating-a-service-principal-in-the-azure-portal)

An Azure service principal is a user identity used by user-created apps, services, and automation tools to access specific Azure resources. 
It is used to grant the minimum permissions level needed to perform specific tasks. It can contain either a password or certificate used for authentication.


1. Go to [Azure App Register](https://portal.azure.com/#view/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) > New Registration

    - **Name**: terraform

    - **Supported Account Types**: Accounts in this organizational directory only (Default Directory only - Single tenant)

    - **Redirect URI**: web

    - Leave Redirect URI ```web``` Value Blank

      ![image](https://user-images.githubusercontent.com/40032360/199315117-6e3ad267-f8ad-43d5-89a2-d302b2a96677.png)

    - Proceed with **[ Register ]**


2. The newly created App Registration ```terraform``` should be visible on-screen

      ![image](https://user-images.githubusercontent.com/40032360/199318610-bf58a5b7-4777-4502-82a2-6963aaf80c60.png)

       At the top of this page, note the "Application (client) ID" and the "Directory (tenant) ID".


3. Generate a Client Secret

      ![image](https://user-images.githubusercontent.com/40032360/199319697-accf2dfc-e12e-4b03-b4ce-e950d103069e.png)

       Add **Client Secrete**

      ![image](https://user-images.githubusercontent.com/40032360/199319856-72f5893c-8a10-4afa-85c7-45e3b3dccf14.png)
      ![image](https://user-images.githubusercontent.com/40032360/199320032-f8b796c2-00cd-493e-988d-299c602a246c.png)
      
      !!! note 

          the **Value** Id. This is the required value for the environment variable: ```ARM_CLIENT_SECRET```.  this value is only displayed once.

      ![image](https://user-images.githubusercontent.com/40032360/199321282-7b096ee6-822e-42c1-8514-ad02f6357fe0.png)


4. Granting the Application access to manage resources in a Subscription

     - [navigate to the Subscriptions blade within the Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)
     
     - select the Subscription you wish to use

     - click Access Control (IAM),  [+] Add > Add role assignment.

     - ![image](https://user-images.githubusercontent.com/40032360/199324438-94134dbb-290f-42d3-a8c4-0291f7851139.png)

     - Specify a Role which grants the appropriate permissions needed for the Service Principal(for example, Network Contributor will grant Read/Write on all network resources in the Subscription).

         - **Role**: 

            A role definition is a collection of permissions. You can use the built-in roles or you can create your own custom roles.
            
            [```Network Contributor```]

         - **Members**: 

            **Selected role**: Network Contributor

            **Assign access to**: User, group, or service principal

            **Members**: [+]Select member (search for the newly created service principal)

         - [Review + Assign]
           
     - ![image](https://user-images.githubusercontent.com/40032360/199544783-221613fc-d493-48fc-824d-e3c1827ad87b.png)
    
    
5. Export Terraform Variables

    ```bash
    export ARM_CLIENT_ID="00000000-0000-0000-0000-000000000000"
    export ARM_CLIENT_SECRET="00000000-0000-0000-0000-000000000000"
    export ARM_SUBSCRIPTION_ID="00000000-0000-0000-0000-000000000000"
    export ARM_TENANT_ID="00000000-0000-0000-0000-000000000000"
    ```

6. Validate credentials using **AZ CLI**

    ```bash
    az login --service-principal -u $ARM_CLIENT_ID -p $ARM_CLIENT_SECRET --tenant $ARM_TENANT_ID
    ```

    ```bash
    [
      {
        "cloudName": "AzureCloud",
        "id": "97bd4d2e-0b8b-4cab-88d6-445e32c5f2b8",
        "isDefault": true,
        "name": "sandbox-services",
        "state": "Enabled",
        "tenantId": "90b97141-b1c4-453e-8f76-10db96d5d950",
        "user": {
          "name": "9bc261bc-84bf-4c41-af58-c4da7acabd86",
          "type": "servicePrincipal"
        }
      }
    ]
    ```

#### Terrafom Example

1. Terraform main.tf content

    ```hlc
    ## terraform provider
    terraform {
      required_version = ">= 0.12.29, < 2.0"
      required_providers {
        azurerm = {
          source  = "hashicorp/azurerm"
          version = "~> 3.7"
        }
      }
    }

    ## Azure Provider configuration
    provider "azurerm" {
       features {}
       skip_provider_registration = true
    }

    ## Variables
    locals {
      resource_group_name = "sandbox-rg1"
      location = "East US"
      name = "lab1"
      address_space = "10.100.0.0/16"
    }

    ## Virtual Network
    resource "azurerm_virtual_network" "sandbox" {
      name                = "vnet${local.name}"
      location            = local.location
      resource_group_name = local.resource_group_name
      address_space       = [local.address_space]
      subnet {
        name           = "vnet${local.name}-snet1"
        address_prefix = cidrsubnets(local.address_space, 8,8)[0]
      }
      subnet {
        name           = "vnet${local.name}-snet2"
        address_prefix = cidrsubnets(local.address_space, 8,8)[1]
      }

      tags = {
        environment = "sandbox"
      }
    }
    ```

2. terraform init
3. terraform plan
4. terraform apply
5. terraform destroy

