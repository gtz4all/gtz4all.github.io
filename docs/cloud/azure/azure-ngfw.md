---
title: "Azure Firewall Notes"
---

#### Azure Firewall forced tunneling

- When you configure a new Azure Firewall, you can route all Internet-bound traffic to a designated next hop instead of going directly to the Internet.
 
- Forced tunneling lets you ```redirect``` all internet bound traffic from Azure Firewall to your ```on-premises firewall``` or to chain it to a nearby network virtual appliance (NVA)for additional inspection. You enable a firewall for forced tunneling when you create a new firewall. As of today, it is not possible to migrate an existing firewall deployment to a forced tunneling mode.

- To support forced tunneling, service management traffic is separated from customer traffic. ```An additional``` dedicated subnet named ```AzureFirewallManagementSubnet``` is required with its own associated public IP address. The only route allowed on this subnet is a default route to the internet, and Border Gateway Protocol (BGP) route propagation must be disabled.

- Within this configuration, the ```AzureFirewallSubnet``` can now include routes to any on-premises firewall or NVA to process traffic before it's passed to the internet. You can also publish these routes via BGP to AzureFirewallSubnet if BGP route propagation is enabled on this subnet.
