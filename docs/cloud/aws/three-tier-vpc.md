
An AWS Three-Tier VPC Design provides a baseline to deploy software applications that consist of frontend, backend and database.This design provides Availability Zone High Availability and Fault Tolerant. In addition this design also provides the required topoligy for a highly secured environment.

![three-tier-vpc-design.jpeg](images/three-tier-vpc-design.jpeg)

#### Public Tier
The Public Tier has both inbound/outbound VPC External access. It is used to provide external services and an entry point into the environment

***Recommended Resources***:
* Public Application Load Balancer
    Users will only be allowed to access application via public Application Load Balancer
* NAT Gateways
    Provides outbound access for private tier. This is required for software updates or patches
* Bastion Hosts
    A bastion host provides secure management access into all three tiers. These servers should be secured with MFA. 

#### Private Tier
The Private Tier only has outbound VPC External Access which is mainly used for software patches and updates. This is where the application servers and internal load balancer reside.

***Recommended Resources***:
* Internal Application Load Balancer
    This type of ALBs and NLBs are used to provide a Highly Available Service to the Public Tier ALBs
* Application Servers
    EC2s, Lambda or EKS, ECS

#### Local Tier
This tier does not have inbound/outbould VPC External Access. It is meant to host database services. Services inside this tier should only be access by the application services. 

***Recommended Resources***:
* Database Instances

## Cloudformation
Cloudformation is a tool to quickly and consistently deploy infrastructure by building the designed resources and managing them as stacks. it is a declarative programming language with some basic features specific to this use case.

#### Componentes
* Metadata: 

  ```AWS::CloudFormation::Interface``` is used to modify how the AWS Cloudformation groups, labels and orders user input parameters.
  https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/metadata-section-structure.html 
  ```yaml
  Metadata: 
  AWS::CloudFormation::Interface: 
    ParameterGroups:
      - Label: 
          default: "Availability Zones"
        Parameters:
          - NumberOfAZs
  ```
* Parametes:

  Parameters are used to request user input custom values which are later used by the ```Resources``` section to build infrastructure.
  https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html
  ```yaml
  Parameters:
    NumberOfAZs:
      Type: Number
      AllowedValues: 
      - 2
      - 4
      Default: 2
  ```

* Conditions:

  Conditions are used to set parameters based on other parameter's values provided by the user. These conditions are later used to set conditial resources. ex: If ```Condition: Value``` is true, create resource.
  https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html
  ```yaml
  ## Conditions are either True or False and are used to choose whether a rosource should be created or not. 
  Conditions: 
    ## Build4AZs is True if NumberofAZs is set to '4'
    Build4AZs:    !Equals [ !Ref NumberOfAZs, 4 ] 
    ## Build4AZs is True if NumberofAZs is set to '4'
    ProdVPC:    !Equals [ !Ref Environment, prod ] 
    ## BuildProdEnvironemt is True if both Build4AZs and ProdVPC are True
    BuildProdEnvironemt:   !And [ !Condition Build4AZs, !Condition ProdVPC ]

  Resources:
    PublicSubnet3:
      Type: AWS::EC2::Subnet
      ## PublicSubnet3 will be created only if Build4AZs is True
      Condition: Build4AZs
  ```

#### Built-in Intrinsic Functions

An intrinsic function used to perform a mathematical, character, or logical operation against a data item whose value is derived automatically during execution.

###### Fn::Cidr ( !Cidr )

  This Subnetting calculator function returns an CIDR address block list based on the ```count`` provided. 

  ``` !Cidr [ ipBlock, count, cidrBits ] ```

  * ***Parameters***
    - ```ipBlock```: CIDR address block to be split into smaller CIDR blocks.
    - ```count```: The number of smaller CIDRs blocks to generate. Valid range 1 - 256.
    - ```cidrBits```: The number of subnet bits for the CIDR. 


  * ***Example***
    - Requirementes: create 12(count) /24s(cidrBits) out of a /16 (ipBlock) and pick the 6th(!Select) CIDR block.
    
<table>
<tr>
<th>!Select [ 5, !Cidr [ "10.10.0.0/16", 12, 5 ]</th>
<th>Generated Table</th>
</tr>
<tr>
<td>

  - !Select - select the 6th block - starts from 0 - 7 if there are 8 blocks.

    ```!Select [ 5,``` 
  - !Cidr [CidrBlock = (/16),  12 = break /16 into 12 block, what size for each block? look at host-bits...

    ```!Cidr [ "10.10.0.0/16", 12,```
  - SubnetBits/cidrBits = 8 - which means host-bits of 8, (/24s)
  
    ```8]]```

***cidrBit Table***

Suffix|IPs|CIDR|Borrowed Bits|Binary
-|-|-|-|-
.255|1  |/32|0|11111111 
.254|2  |/31|1|11111110
.252|4  |/30|2|11111100
.248|8  |/29|3|11111000
.240|16 |/28|4|11110000
.224|32 |/27|5|11100000
.192|64 |/26|6|11000000
.128|128|/25|7|10000000
.0|256|/24|8|00000000

</td>
<td>    

Count|CidrBlocks|Select
-|-|-
0|10.10.0.0/24| 
1|10.10.1.0/24| 
2|10.10.2.0/24| 
3|10.10.3.0/24| 
4|10.10.4.0/24| 
5|10.10.5.0/24|!Select
6|10.10.6.0/24| 
7|10.10.7.0/24| 
8|10.10.8.0/24| 
9|10.10.9.0/24| 
10|10.10.10.0/24|
11|10.10.11.0/24|
12|10.10.12.0/24| 

</td>
</tr>
</table>

  * ***Conditional Subnetting***
    ```yaml
    !Select [ !If [BuildProd, 0, 0], !Cidr [ !Ref CidrBlock, !If [BuildProd, 16, 8], !If [BuildProd, 5, 6]]]
    ```
<table>
<tr>
<th>Conditial !Cidr</th>
<th>Condition !if</th>
</tr>
<tr>
<td>

  - if BuildPord is True - Create 16 /27s (Host-Bits - 5 ) blocks
  - if BuildPord is False - Create 8 /26s (Host-Bits - 6 ) blocks 

    ```yaml
    !Select [ !If [BuildProd, 0, 0], 
    !Cidr [ !Ref CidrBlock, !If [BuildProd, 16, 8], 
    !If [BuildProd, 5, 6]]]
    ```

</td>
<td>

  !If Condition|Boolean|Value
  -|-|-
  BluidProd|True|firstValue
  BluidProd|False|secondValue

  ***!Cidr*** 

  ipBlock|count|cidrBits
  -|-|-
  !Ref CidrBlock|!If [BuildProd, 16, 8]|!If [BuildProd, 5, 6]
    
</td>
</tr>
</table>

###### Fn::GetAZs (!GetAZs)

The intrinsic function Fn::GetAZs returns a Availability Zone Name List based for a region. 

```
AZ Name to AZ ID mapping is different per account.
```

* ```!GetAZs ""``` in ```us-east-1``` returns a list of all Availability Zones: ```[ "us-east-1a", "us-east-1b", "us-east-1c", "us-east-1d", "us-east-1e", "us-east-1f" ]``` 

  0|1|2|3|4|5
  -|-|-|-|-|-
  "us-east-1a"|"us-east-1b"|"us-east-1c"|"us-east-1d"|"us-east-1e"|"us-east-1f"

* ```!Select``` can pick an AZ based on the order list. 

  ```!Select [3, !GetAZs '']``` returns  ```"us-east-1d"```.


#### Three Tier VPC Cloudformation Template
This template builds the Three Tier VPC based on the above topology using the built-in cloudformation functions recently mentioned. 

[Gtz4all Three Tier VPC Template](gtz4all-three-tier-vpc.yml)

<div style="width:1200px;height:600px;overflow:auto;">

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: GTZ Three Tier VPC Template
Metadata: 
  AWS::CloudFormation::Interface: 
    ParameterGroups:
      - Label: 
          default: "Stack Name - {{ProjectName}}-{{Environment}}-three-tier-vpc-cf (i.e jira-dev-three-tier-vpc-cf)"
      - Label: 
          default: "|"
      - Label: 
          default: "VPC Configuration"
        Parameters:
          - ProjectName
          - Environment          
          - CidrBlock
      - Label: 
          default: "Subnet 8 = /24(/20 CidrBlock Required), 7 = /25 (/21 CidrBlock Required), 6 = /26 (/22 CidrBlock Required)"
        Parameters:
          - SubnetBits
          
      - Label: 
          default: "Availability Zones"
        Parameters:
          - NumberOfAZs

Parameters:
  ProjectName:
    Description: Project Name ( ex. jira /  sfpt / sec )
    Type: String
    
  CidrBlock:
    AllowedPattern: ^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\/(1[6-9]|2[0-8]))$
    ConstraintDescription: CIDR block parameter must be in the form x.x.x.x/16-28
    Description: VPC valid IP CIDR Blocks /20's, /21's and /22's( ex. 10.168.0.0/20)
    Default: 10.168.0.0/20
    Type: String
    
  SubnetBits:
    Type: Number
    AllowedValues: 
    - 8
    - 7
    - 6
    Default: 8
    Description:  Subnet Bits 8 = /24(/20 CidrBlock Required), 7 = /25 (/21 CidrBlock Required), 6 = /26 (/22 CidrBlock Required)
    
  NumberOfAZs:
    Type: Number
    AllowedValues: 
    - 2
    - 4
    Default: 2
    Description:  Availability Zones - 2 = 2 per Tier, 4 = 4 per Tier
    
  Environment:
    Description: Choose VPC Type 
    Type: String
    AllowedValues: 
    - dev
    - test
    - prod
    
Conditions: 
  Build4AZs:    !Equals [ !Ref NumberOfAZs, 4 ] 

###############################################
### Three Tier VPC                         ####
###############################################  
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: {Ref: CidrBlock}
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName,!Ref Environment, "three-tier", "vpc" ]]

  DhcpOptions: 
      Type: AWS::EC2::DHCPOptions
      Properties: 
          DomainName: !Join ['-', [!Ref ProjectName, "three-tier.org" ]]
          DomainNameServers: 
            - AmazonProvidedDNS
          Tags: 
            - Key: Name
              Value: !Join ['-', [!Ref ProjectName,!Ref Environment, "three-tier", "dopt" ]]

  VPCDHCPOptionsAssociation:
    Type: AWS::EC2::VPCDHCPOptionsAssociation
    Properties:
      VpcId: {Ref: VPC}
      DhcpOptionsId: {Ref: DhcpOptions}
    
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    DependsOn: VPC
    Properties:
      Tags:
      - Key: Name
        Value: !Join ['-', [!Ref ProjectName,!Ref Environment, "three-tier", "igw" ]]
  VPCGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

###############################################
### Public/ALB Tier (inbound/outbound)     ####
############################################### 
  PublicRouteTableA:
    Type: AWS::EC2::RouteTable
    DependsOn: VPCGatewayAttachment
    Properties:
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "rt-1a" ]]
      VpcId: !Ref VPC
  PublicRouteTableB:
    Type: AWS::EC2::RouteTable
    DependsOn: VPCGatewayAttachment
    Properties:
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "rt-1b" ]]
      VpcId: !Ref VPC       
###############################################
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [0, !GetAZs '']
      CidrBlock: !Select [ 0, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "subnet-1a" ]]
      VpcId: !Ref VPC
  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [1, !GetAZs '']     
      CidrBlock: !Select [ 1, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "subnet-1b" ]]
      VpcId: !Ref VPC
  PublicSubnet3:
    Type: AWS::EC2::Subnet
    Condition: Build4AZs
    Properties:
      AvailabilityZone: !Select [2, !GetAZs '']
      CidrBlock: !Select [ 2, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "subnet-1c" ]]
      VpcId: !Ref VPC
  PublicSubnet4:
    Type: AWS::EC2::Subnet
    Condition: Build4AZs
    Properties:
      AvailabilityZone: !Select [3, !GetAZs '']     
      CidrBlock: !Select [ 3, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "subnet-1d" ]]
      VpcId: !Ref VPC
###############################################
  PublicSubnetAssoc1:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTableA
      SubnetId: !Ref PublicSubnet1
  PublicSubnetAssoc2:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTableB
      SubnetId: !Ref PublicSubnet2
  PublicSubnetAssoc3:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Condition: Build4AZs
    Properties:
      RouteTableId: !Ref PublicRouteTableA
      SubnetId: !Ref PublicSubnet3
  PublicSubnetAssoc4:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Condition: Build4AZs
    Properties:
      RouteTableId: !Ref PublicRouteTableB
      SubnetId: !Ref PublicSubnet4

###############################################
### Private/Application Tier (inbound)     ####
############################################### 
  AppRouteTableA:
    Type: AWS::EC2::RouteTable
    Properties:
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "private", "rt-1a" ]]
      VpcId:
        Ref: VPC
  AppRouteTableB:
    Type: AWS::EC2::RouteTable
    Properties:
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "private", "rt-1b" ]]
      VpcId:
        Ref: VPC        
###############################################
  AppSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [0, !GetAZs '']
      CidrBlock: !Select [ 4, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "private", "subnet-1a" ]]
      VpcId: !Ref VPC
  AppSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [1, !GetAZs '']     
      CidrBlock: !Select [ 5, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "private", "subnet-1b" ]]
      VpcId: !Ref VPC
  AppSubnet3:
    Type: AWS::EC2::Subnet
    Condition: Build4AZs
    Properties:
      AvailabilityZone: !Select [2, !GetAZs '']
      CidrBlock: !Select [ 6, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "private", "subnet-1c" ]]
      VpcId: !Ref VPC
  AppSubnet4:
    Type: AWS::EC2::Subnet
    Condition: Build4AZs
    Properties:
      AvailabilityZone: !Select [3, !GetAZs '']     
      CidrBlock: !Select [ 7, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "private", "subnet-1d" ]]
      VpcId: !Ref VPC
###############################################
  AppSubnetAssoc1:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref AppRouteTableA
      SubnetId: !Ref AppSubnet1
  AppSubnetAssoc2:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref AppRouteTableB
      SubnetId: !Ref AppSubnet2
  AppSubnetAssoc3:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Condition: Build4AZs
    Properties:
      RouteTableId: !Ref AppRouteTableA
      SubnetId: !Ref AppSubnet3
  AppSubnetAssoc4:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Condition: Build4AZs
    Properties:
      RouteTableId: !Ref AppRouteTableB
      SubnetId: !Ref AppSubnet4
###############################################
  NatGatewayAEIP:
    Type: 'AWS::EC2::EIP'
    DependsOn: VPCGatewayAttachment  
    Properties:
      Domain: vpc
  NatGatewayA:
    Type: 'AWS::EC2::NatGateway'
    DependsOn: NatGatewayAEIP  
    Properties:
      AllocationId: 
          Fn::GetAtt:
          - NatGatewayAEIP
          - AllocationId
      SubnetId: !Ref AppSubnet1
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "natgw", "1a" ]]
###############################################
  NatGatewayBEIP:
    Type: 'AWS::EC2::EIP'
    DependsOn: VPCGatewayAttachment  
    Properties:
      Domain: vpc
  NatGatewayB:
    Type: 'AWS::EC2::NatGateway'
    DependsOn: NatGatewayBEIP 
    Properties:
      AllocationId:
          Fn::GetAtt:
          - NatGatewayBEIP
          - AllocationId
      SubnetId: !Ref AppSubnet2
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "natgw", "1b" ]]
###############################################
  AppRouteTableARoute: 
    Type: 'AWS::EC2::Route'
    DependsOn: VPCGatewayAttachment
    Properties:
      RouteTableId: !Ref AppRouteTableA
      DestinationCidrBlock: '0.0.0.0/0'
      NatGatewayId: !Ref NatGatewayA
  AppRouteTableBRoute: 
    Type: 'AWS::EC2::Route'
    DependsOn: VPCGatewayAttachment
    Properties:
      RouteTableId: !Ref AppRouteTableB
      DestinationCidrBlock: '0.0.0.0/0'
      NatGatewayId: !Ref NatGatewayB
      
###############################################
### Local/Database Tier (local-access)     ####
###############################################
  BDRouteTableA:
    Type: AWS::EC2::RouteTable
    Properties:
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "local", "rt-1a" ]]
      VpcId:
        Ref: VPC
  BDRouteTableB:
    Type: AWS::EC2::RouteTable
    Properties:
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "local", "rt-1b" ]]
      VpcId:
        Ref: VPC          
###############################################
  BDSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [0, !GetAZs '']
      CidrBlock: !Select [ 8, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "local", "subnet-1a" ]]
      VpcId: !Ref VPC
  BDSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [1, !GetAZs '']     
      CidrBlock: !Select [ 9, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "local", "subnet-1b" ]]
      VpcId: !Ref VPC
  BDSubnet3:
    Type: AWS::EC2::Subnet
    Condition: Build4AZs
    Properties:
      AvailabilityZone: !Select [2, !GetAZs '']
      CidrBlock: !Select [ 10, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "local", "subnet-1c" ]]
      VpcId: !Ref VPC
  BDSubnet4:
    Type: AWS::EC2::Subnet
    Condition: Build4AZs
    Properties:
      AvailabilityZone: !Select [3, !GetAZs '']     
      CidrBlock: !Select [ 11, !Cidr [ !Ref CidrBlock, 12,!Ref SubnetBits]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "local", "subnet-1d" ]]
      VpcId: !Ref VPC
###############################################
  BDSubnetAssoc1:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref BDRouteTableA
      SubnetId: !Ref BDSubnet1
  BDSubnetAssoc2:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref BDRouteTableB
      SubnetId: !Ref BDSubnet2
  BDSubnetAssoc3:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Condition: Build4AZs
    Properties:
      RouteTableId: !Ref BDRouteTableA
      SubnetId: !Ref BDSubnet3
  BDSubnetAssoc4:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Condition: Build4AZs
    Properties:
      RouteTableId: !Ref BDRouteTableB
      SubnetId: !Ref BDSubnet4
###############################################
  PublicRouteTableRouteA: 
    Type: 'AWS::EC2::Route'
    Properties:
      RouteTableId: !Ref PublicRouteTableA
      DestinationCidrBlock: '0.0.0.0/0'
      GatewayId: !Ref InternetGateway
  PublicRouteTableRouteB: 
    Type: 'AWS::EC2::Route'
    Properties:
      RouteTableId: !Ref PublicRouteTableB
      DestinationCidrBlock: '0.0.0.0/0'
      GatewayId: !Ref InternetGateway
````

</div>

#### Cloudformation Nested Functions
The above template is prone to error depending on the user inputs. Howerver there are complicated ways to reduce some of them.

* ***Problem***: 
  Assume user provides a CidrBlock: ```10.168.64.0/22``` and CidrBits:```8```. In order to generate enough Cidr Blocks for 4 AZs, the templace is requesting ```12``` smaller blocks. Since The ```/22``` CidrBlock only contains 4 ```/24's```, the cloudformation stack build will fail.

* ***Solution***: 
  One possible solution would be to pick from a list of ```AllowedValues``` in the ```Parameters`` section and use ```Conditions``` to set ```CidrBits``` based on user choice using nested conditions in ```resources```.

```yaml
Conditions: 
  CidrBit8:    !Equals [ !Ref Cidr, 20 ]
  CidrBit7:    !Equals [ !Ref Cidr, 21 ]
  CidrBit6:    !Equals [ !Ref Cidr, 22 ]

Resources: 
  !If [CidrBit8, 8, !If [CidrBit7, 7, 6]]
```

* ```!If [CidrBit8, 8, !If [CidrBit7, 7, 6]]``` 
```
    if CidrBit8 == True:
      cidrBit == 8
    elif CidrBit7 == True:
      cidrBit == 7
    else:
      cidrBit = 6
```

###### Cloudformation Snippet

<div style="width:1200px;height:600px;overflow:auto;">

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: GTZ Three Tier VPC Template
Metadata: 
  AWS::CloudFormation::Interface: 
    ParameterGroups:
      - Label: 
          default: "Stack Name - {{ProjectName}}-{{Environment}}-three-tier-vpc-cf (i.e jira-dev-three-tier-vpc-cf)"
      - Label: 
          default: "|"
      - Label: 
          default: "VPC Configuration"
        Parameters:
          - ProjectName
          - Environment          
          - CidrBlock
      - Label: 
          default: "Subnet 8 = /24(/20 CidrBlock Required), 7 = /25 (/21 CidrBlock Required), 6 = /26 (/22 CidrBlock Required)"
        Parameters:
          - CidrMask
          
      - Label: 
          default: "Availability Zones"
        Parameters:
          - NumberOfAZs

Parameters:
  ProjectName:
    Description: Project Name ( ex. jira /  sfpt / sec )
    Type: String
    
  CidrBlock1:
    ConstraintDescription: CIDR block parameter must be in the form x.x.x.x
    Description: VPC valid IP Block ( ex. 10.168.0.0)
    Default: 10.168.0.0
    Type: String
    
    
  Cidr:
    Type: Number
    AllowedValues: 
    - 20
    - 22
    - 21
    Default: 20
    Description:  VPC valid IP CIDR Blocks /20's ( /24's smaller blocks )  , /21's ( /25's smaller blocks )and /22's ( /26's smaller blocks )
    
  NumberOfAZs:
    Type: Number
    AllowedValues: 
    - 2
    - 4
    Default: 2
    Description:  Availability Zones - 2 = 2 per Tier, 4 = 4 per Tier
    
  Environment:
    Description: Choose VPC Type 
    Type: String
    AllowedValues: 
    - dev
    - test
    - prod
    
Conditions: 
  Build4AZs:    !Equals [ !Ref NumberOfAZs, 4 ]
  CidrBit8:    !Equals [ !Ref Cidr, 20 ]
  CidrBit7:    !Equals [ !Ref Cidr, 21 ]
  CidrBit6:    !Equals [ !Ref Cidr, 22 ]

###############################################
### Three Tier VPC                         ####
###############################################  
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Join ['', [!Ref CidrBlock1, "/", !Ref Cidr ]]
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName,!Ref Environment, "three-tier", "vpc" ]]

  DhcpOptions: 
      Type: AWS::EC2::DHCPOptions
      Properties: 
          DomainName: !Join ['-', [!Ref ProjectName, "three-tier.org" ]]
          DomainNameServers: 
            - AmazonProvidedDNS
          Tags: 
            - Key: Name
              Value: !Join ['-', [!Ref ProjectName,!Ref Environment, "three-tier", "dopt" ]]

  VPCDHCPOptionsAssociation:
    Type: AWS::EC2::VPCDHCPOptionsAssociation
    Properties:
      VpcId: {Ref: VPC}
      DhcpOptionsId: {Ref: DhcpOptions}
    
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    DependsOn: VPC
    Properties:
      Tags:
      - Key: Name
        Value: !Join ['-', [!Ref ProjectName,!Ref Environment, "three-tier", "igw" ]]
  VPCGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

###############################################
### Public/ALB Tier (inbound/outbound)     ####
############################################### 
  PublicRouteTableA:
    Type: AWS::EC2::RouteTable
    DependsOn: VPCGatewayAttachment
    Properties:
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "rt-1a" ]]
      VpcId: !Ref VPC
  PublicRouteTableB:
    Type: AWS::EC2::RouteTable
    DependsOn: VPCGatewayAttachment
    Properties:
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "rt-1b" ]]
      VpcId: !Ref VPC       
###############################################
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [0, !GetAZs '']
      CidrBlock: !Select [ 0, !Cidr [ !Join ['', [!Ref CidrBlock1, "/", !Ref Cidr ]], 12, !If [CidrBit8, 8, !If [CidrBit7, 7, 6]]]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "subnet-1a" ]]
      VpcId: !Ref VPC
  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [1, !GetAZs '']     
      CidrBlock: !Select [ 1, !Cidr [ !Join ['', [!Ref CidrBlock1, "/", !Ref Cidr ]], 12, !If [CidrBit8, 8, !If [CidrBit7, 7, 6]]]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "subnet-1b" ]]
      VpcId: !Ref VPC
  PublicSubnet3:
    Type: AWS::EC2::Subnet
    Condition: Build4AZs
    Properties:
      AvailabilityZone: !Select [2, !GetAZs '']
      CidrBlock: !Select [ 2, !Cidr [ !Join ['', [!Ref CidrBlock1, "/", !Ref Cidr ]], 12, !If [CidrBit8, 8, !If [CidrBit7, 7, 6]]]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "subnet-1c" ]]
      VpcId: !Ref VPC
  PublicSubnet4:
    Type: AWS::EC2::Subnet
    Condition: Build4AZs
    Properties:
      AvailabilityZone: !Select [3, !GetAZs '']     
      CidrBlock: !Select [ 3, !Cidr [ !Join ['', [!Ref CidrBlock1, "/", !Ref Cidr ]], 12, !If [CidrBit8, 8, !If [CidrBit7, 7, 6]]]]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Join ['-', [!Ref ProjectName, "public", "subnet-1d" ]]
      VpcId: !Ref VPC
###############################################
  PublicSubnetAssoc1:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTableA
      SubnetId: !Ref PublicSubnet1
  PublicSubnetAssoc2:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTableB
      SubnetId: !Ref PublicSubnet2
  PublicSubnetAssoc3:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Condition: Build4AZs
    Properties:
      RouteTableId: !Ref PublicRouteTableA
      SubnetId: !Ref PublicSubnet3
  PublicSubnetAssoc4:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Condition: Build4AZs
    Properties:
      RouteTableId: !Ref PublicRouteTableB
      SubnetId: !Ref PublicSubnet4
```
</div>
