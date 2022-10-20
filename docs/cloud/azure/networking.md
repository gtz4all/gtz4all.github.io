---
title: "Azure Networking"
description: "Cloud Computing Documentation"
---

#### Virtual networks
###### Features
- private and secure infrastructure

    VM residing inside a virtual network are able to privately communicate with each other access other azure resources

- Network Segmentation

    Virtual Networks are isolated from each other
      
- Per Subscription / Region

    VNETs are region and subscription specific. 

###### Components
- Subnets

    Subnets devide a virtual network into smaller segments. These segments can have unique security policies from each other. Some 
    services require dedicated subnets such as network gateways, Bastion, and SQL Managed Instances
    
- Network Interfaces

    Network interfaces are use to connect virtuals machines to virtual networks. a virtual machine can have network interfaces in different subnets
    
- Network Security Groups (NSGs)

    Security groups filter inbound/outbound source/destination IP address/Port traffic based on a priority list of rules. they can be associated to 
    either ***subnets*** or ***network interfaces***.

    NSGs are region specific ( just like VMs )
    
- Application Security Groups (ASGs)

    ASGs are used to group network interfaces based on purpose/usage. an ASG is similar to a ***tag*** on the network interface. a Network Security Group Rule can
    it destination set to an ASG.
    
    ```mermaid
    graph TD
    subgraph D[Virtual Private Network 10.0.0.0/26]
      subgraph E[Subnet 10.0.1.0/24]
        1[Virtual Machine 1]
        2[Application Security Group: ASG1] --> 1 
      end
      subgraph F[Subnet 10.0.2.0/24]
        3[Virtual Machine 3]
        4[Application Security Group: ASG1] --> 3 
      end
      5[Network Security Group: NSG1<br>Direction: Inbound<br>Source: Internet<br>Destination: ASG1<br>Service: HTTPS<br>Action: Allow]
    end

    style 4 fill:#F0FFFF
    style 2 fill:#F0FFFF
    style D fill:#FFFFFF
    style 5 fill:#F0FFFF
    style E fill:#F0F8FF
    style F fill:#F0F8FF
    ```
#### Azure Firewalls
###### Features
- Azure Firewall as a Service is used to provide monitor and protection based on rules between virtual network egress/ingress traffic
- provides defense in depth
- Azure firewall is deployed between VNET 1 and VNET2 and the Internet. incomming traffic from the internet will be inspected by the Azure firewall before allowing or denying access to VNET1/VNET2

    ```mermaid
    graph TD
    subgraph D[Azure]
      subgraph A[Virtual Network1]
      subgraph AA[Subnet]
        AAA[VM]
      end
      end

      subgraph B[Virtual Network1]
      subgraph BB[Subnet]
        BBB[VM]
      end
      end

      subgraph C[Virtual Network1]
      subgraph CC[Subnet]
        CCC[Azure Firewall]
      end
      end

    end
      F[Internet]

    style D fill:#FFFFFF

    style A fill:#F0FFFF
    style B fill:#F0FFFF
    style C fill:#F0FFFF

    style AA fill:#FFFFFF
    style BB fill:#FFFFFF
    style CC fill:#FFFFFF

    style AAA fill:#F0FFFF
    style BBB fill:#F0FFFF
    style CCC fill:#F0FFFF
    ```
    
#### Endpoint Services

- A service endpoints provide private connectivity between Virtual Networks and a range of Azure services, including Azure Storage, SQL Databases, and Key Vault.
- communication travels across Azure backbone

    ```mermaid
    graph TD
    subgraph D[Azure]
      subgraph A[fa:fa-bucket Azure Storage]
      end
      subgraph C[fa:fa-network-wired Virtual Network]
      subgraph CC[Subnet]
        CCC[Azure Firewall]
      end
      end

    end
      F[Internet]
    style D fill:#FFFFFF
    style A fill:#F0FFFF
    style C fill:#F0FFFF
    style CC fill:#FFFFFF
    style CCC fill:#F0FFFF
    ```
    
#### Private Links
- provide private connectivity to Azure PaaS services.
- accessible on connected virtual networks and globally peered networks.
- DNS Integration supported

###### Private Endpoint
- a managed network interface used to provide connectivity to a private link

=== "Service Endpoints"
    - Provides connectivity from a subnet to an entire Azure service
    - cannot be used by on-premises networks
    - uses routes to direct traffic

=== "Private Endpoints"
    - Provides connectivity to single instance of an entire Azure service
    - supports connected routes ( transitive routing )
    - integrates with DNS
    - data transfer fees(inbound/outbound)

#### Azure Bastion

- provides a managed secure remote management portal
- supports RDP and SSH
- supports Network Security Groups (NSGs)

#### Virtual Network Peering
Virtual Network Peerings provide connectivity between virtual networks within regions, across regions and subscriptions

- Peering connections must be created in both directions
- Non-transitive
- CIDR must not overlap
- Default Routes - The default route list has grown and is Azure managed. Azure is using the "more specific" approach - if there is no more specific drop all traffic.

Address Prefix | Next Hop | Description
---------------|----------|------------
Virtual network address range | Virtual Network | routes traffic within the virtual network
0.0.0.0/0 | Internet | Default route to the internet
10.0.0.0/8 <br> 172.16.0.0/12 <br> 192.168.0.0/16 | None | Drops private Ip address ranges that are not part of the virtual network
100.64.0.0/10 | None | Drops shared address space traffic

- Custom Routes are user-defined or learned via BGP

#### Route Tables
- can be associated to subnets in different VPCs. ( what would be a use-case for this?)

#### Gateways

- Virtual Network Gateway
    - provides either VPN or ExpressRoute connectivity to Azure. 
    - requires a special gateway subnet with a minimun of a /27 CIDR block  
    - Similar to an AWS VGW - its VPC specific

- VPN Gateway
    - provides external connectivity - on-premises networks, cloud providers, or remote devices 
    - supports site to site, multiple sites, and point to site connections
    - route-based VPN and policy-based connections

- ExpressRoute
    - direct, private connection between on-premises network and Azure
    - connectivity to Azure VNets using private peering or Azure services using microsoft services
    - connection types:
        - IPVPN - integrates with MPLS (VPLS)
        - Point-to-Point Ethernet (Ethernet Private Links - EPLs)
        - CloudExchange to connect from co-locations

<table><tr>
<th>VPN Gateway</th>
<th>Express Route</th>
</tr>
<tr><td>

- connects on-premises networks and other clouds<br>
- connects only to virtual networks<br>
- connects via the internet<br>

</td><td>

- connects on-premises networks<br>
- connects to virtual networks and microsoft services<br>
- connects via dedicated, private connections<br>

</td></tr></table>

![image](https://user-images.githubusercontent.com/40032360/196560564-d0d24907-e959-4c3e-9bfb-62e9a814572d.png)

#### Virtual WAN

Virtual WANs are used to manage communication between multiple virtual networks, on-premises networks, and remote sites. It also improves performance by using azure backbone.

- While the concept of Virtual WAN is global, the actual Virtual WAN resource is Resource Manager-based and deployed regionally. If the virtual WAN region itself were to have an issue, all hubs in that virtual WAN will continue to function as is, but the user won't be able to create new hubs until the virtual WAN region is available.

- Virtual WAN is assigned a region. However, Virtual WAN resources such as Hubs and Gateways can be in any region.

- managed hub-and-spoke provides connecitivity between virtual networks, on-premises networks and remote workers
- one hub per region
- regional hubs are dynamically connected to each other
- similar to an AWS Transit Gateway

![Gtz4All WAN Hubs](../../assets/azure.drawio)


#### Load balancing

<table><tr>
<th>Load Balancer</th>
<th>Application Gateway</th>
</tr>
<tr><td>

- Layer 4 / TCP/IP Applications<br>
- load balancing based on source/destination<br>
- low latency<br>

</td><td>

- Layer 7 - HTTP/HTTPs based application<br>
- load balancing based on source/destination/hostname/path<br>
- Web Application Firewall is supported<br>

</td></tr></table>


