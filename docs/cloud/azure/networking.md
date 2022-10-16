---
title: "Azure Virtual Networks"
description: "Cloud Computing Documentation"
---

#### Virtual networks
###### Features
- private and secure infrastructure

    VM residing inside a virtual network are able to privately communicate with each other access other azure resources

- Network Segmentation

    Virtual Networks are isolated from each outer
      
- Per Subscription / Region

    VNETs are region and subscription (Account/Project) specific. 

###### Components
- Subnets

    Subnets devides a virtual network into smaller segments. This segments can have unique security policies from each other. Some 
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
    
    
