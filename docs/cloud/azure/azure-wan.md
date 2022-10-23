---
title: "Azure Virtual WAN Components"
description: "vWAN"
---

![Gtz4All Azure Connectivity](../../assets/azureconnectivity.drawio){: style="height:150px;width:150px"}

#### Virtual Hub Deployment

- IP Addressing

  https://docs.microsoft.com/en-us/azure/virtual-wan/virtual-wan-faq#what-is-the-recommended-hub-address-space-during-hub-creation
  The recommended Virtual WAN hub address space is /23. Virtual WAN hub assigns subnets to various gateways (ExpressRoute, Site-to-site VPN, Point-to-site VPN, Azure Firewall, Virtual hub Router). For scenarios where NVAs are deployed inside a virtual hub, a /28 is typically carved out for the NVA instances. However, if the user were to provision multiple NVAs, a /27 subnet may be assigned. Therefore, keeping a future architecture in mind, while Virtual WAN hubs are deployed with a minimum size of /24, the recommended hub address space at creation time for user to input is /23.



- BGP Configuration

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

###### ASNs (Autonomous System Numbers)

Reference: https://github.com/MicrosoftDocs/azure-docs/blob/main/includes/vpn-gateway-faq-bgp-include.md

You can use your own public ASNs or private ASNs for both your on-premises networks and Azure virtual networks. You can't use the ranges reserved by Azure or IANA.

The following ASNs are reserved by Azure or IANA:
* ASNs reserved by Azure:

  * Public ASNs: 8074, 8075, 12076
  * Private ASNs: 65515, 65517, 65518, 65519, 65520
* ASNs [reserved by IANA](http://www.iana.org/assignments/iana-as-numbers-special-registry/iana-as-numbers-special-registry.xhtml):

  * 23456, 64496-64511, 65535-65551 and 429496729

###### Gateway Limitation

Reference: https://github.com/MicrosoftDocs/azure-docs/blob/main/articles/expressroute/expressroute-howto-coexist-resource-manager.md

If you want to use transit routing between ExpressRoute and VPN, **the ASN of Azure VPN Gateway must be set to 65515** and Azure Route Server should be used. Azure VPN Gateway supports the BGP routing protocol. For ExpressRoute and Azure VPN to work together, you must keep the Autonomous System Number of your Azure VPN gateway at its default value, 65515. If you previously selected an ASN other than 65515 and you change the setting to 65515, you must reset the VPN gateway for the setting to take effect.
