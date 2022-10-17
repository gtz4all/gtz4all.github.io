---
title: "Azure Management Groups"
description: "Cloud Computing Documentation"
---

#### Management groups
- used to efficiently manage access, policies, and compliance in a multi-subscription environment. 
- simplify management and security of subscriptions, resource groups, and resources.
- all associated subscriptions inherit governance conditions.
    A policy applied to management group that restrict resource creation to only authorized regions would be applied to all nested management groups,
    subscriptions and resources

![image](https://user-images.githubusercontent.com/40032360/196037187-4fccd9ac-1e13-4a01-9f21-50fa311ff7cd.png)

#### Management Group and Subscription Hierarchy
- Management groups can be nested to provide governance based on usage, role, or environment. This simplifies security policy management.
- A subscription cam be moved from one management group to another inheriting any security policy associated with the new management group.

![image](https://user-images.githubusercontent.com/40032360/196039582-c9026de9-9220-4155-aee1-1a9235869910.png)


#### Management Groups Facts
- 10,000 Management Groups per directory
- 6 levels of depth per management group not including the Root or Subscription level
- Management Groups or Subscription can only belong to a single parents.
- A management group can have multiple children - sub management groups and multiple subscriptions

#### Root Management Group Facts
- A directory has a single top-level root management group call ***Tentant root group***
- Root management group cannot be moved or deleted

#### Landing Zones
Reference: [Management Groups - Landing Zone](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/design-area/resource-org-management-groups)

![image](https://user-images.githubusercontent.com/40032360/196042025-5d57b22e-6ea2-4474-9ba0-8ce83fcf33de.png)

Management group|Description
----------------|-----------
Intermediate Root Management Group|This management group is located directly under the tenant root group. Created with a prefix provided by the organization, which purposely avoids the usage of the root group so that organizations can move existing Azure subscriptions into the hierarchy. It also enables future scenarios. This management group is a parent to all the other management groups created by the Azure landing zone accelerator.
Platform|This management group contains all the platform child management groups, like management, connectivity, and identity.
Management|This management group contains a dedicated subscription for management, monitoring, and security. This subscription will host an Azure Log Analytics workspace, including associated solutions, and an Azure Automation account.
Connectivity|This management group contains a dedicated subscription for connectivity. This subscription will host the Azure networking resources required for the platform, like Azure Virtual WAN, Azure Firewall, and Azure DNS private zones.
Identity|This management group contains a dedicated subscription for identity. This subscription is a placeholder for Windows Server Active Directory Domain Services (AD DS) virtual machines (VMs) or Azure Active Directory Domain Services. The subscription also enables AuthN or AuthZ for workloads within the landing zones. Specific Azure policies are assigned to harden and manage the resources in the identity subscription.
Landing Zones|The parent management group for all the landing zone child management groups. It will have workload agnostic Azure policies assigned to ensure workloads are secure and compliant.
Online|The dedicated management group for online landing zones. This group is for workloads that might require direct internet inbound/outbound connectivity or for workloads that might not require a virtual network.
Corp|The dedicated management group for corporate landing zones. This group is for workloads that require connectivity or hybrid connectivity with the corporate network via the hub in the connectivity subscription.
Sandboxes|The dedicated management group for subscriptions that will only be used for testing and exploration by an organization. These subscriptions will be securely disconnected from the corporate and online landing zones. Sandboxes also have a less restrictive set of policies assigned to enable testing, exploration, and configuration of Azure services.
Decommissioned|The dedicated management group for landing zones that are being canceled. Canceled landing zones will be moved to this management group before deletion by Azure after 30-60 days.

!!! Note
    The above is just a proposal. Some organizations may consider making changes to this hierarchy in order to make relevant to their requirements and environment.
    
 
