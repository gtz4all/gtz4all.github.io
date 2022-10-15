---
title: "Azure Virtual Networks"
description: "Cloud Computing Documentation"
---

#### Features
- private and secure infrastructure

    VM residing inside a virtual network are able to privately communicate with each other access other azure resources

- Network Segmentation

    Virtual Networks are isolated from each outer
      
- Per Subscription / Region

    VNETs are region and subscription (Account/Project) specific

#### Components
- Subnets

    Subnets devides a virtual network into smaller segments. This segments can have unique security policies from each other. Some 
    services require dedicated subnets such as network gateways, Bastion, and SQL Managed Instances
    
- Network Interfaces

    Network interfaces are use to connect virtuals machines to virtual networks. a virtual machine can have network interfaces in different subnets
    
- Network Security Groups (NSGs)

    Security groups filter inbound/outbound source/destination IP address/Port traffic based on a priority list of rules. they can be associated to 
    either ***subnets*** or ***network interfaces***.
