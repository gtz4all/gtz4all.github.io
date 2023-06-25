---
title: AWS Landing Zones
description: AWS Cloudformation
---
## Build Landing Zone

Custom Settings After Landing Zone Deployment

- Change Pipeline Source
  - Current: S3 bucket
    - requires KMS policy adjustments to allow zip download
    - Add root ARN
  - CodePipeline - Github source requires ‘SourceApp’ matching current S3 settings
    - Remove S3 as source

- Service Catalog
  - Administrator Portfolio requires Policy Changes to allow Root ARN to see Products and be able to launch account vending machine( not Administrator Products)
  - Created ‘Network’ After adding ARN to Product Policy

#### Account Inventory
Account name|Email|Account ID|Assume Role
------------|-----|----------|-----------
gtzroot|kervingtz@gmail.com|571219239203.00|User: gtzroot
shared-services|kervingtz+aws-sharedsvcs@gmail.com|176041802277.00|AWSCloudFormationStackSetExecutionRole
security|kervingtz+aws-security@gmail.com|588179906098.00|AWSCloudFormationStackSetExecutionRole
network|kervingtz+aws-network@gmail.com|507533005210.00|
log-archive|kervingtz+aws-logs@gmail.com|346432656342.00|AWSCloudFormationStackSetExecutionRole



#### Roles
During initial LZ Deployment, log-archive, security, and shared-services accounts get created with role - AWSCloudFormationStackSetExecutionRole - User in Root Account can assume role into these 3 accounts.

- Role: AWSCloudFormationStackSetExecutionRole

  `Permission Policy`
  ```json
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": "*",
              "Resource": "*"
          }
      ]
  }
  ```

  `Trust Relationship`
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "AWS": "arn:aws:iam::571219239203:root"
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }
  ```
  
#### Service Catalog - Account Vending Machine
Service Catalog → Portfolios → AWS Landing Zone - Baseline → Add Groups, roles and users = Add Root User ARN - arn:aws:iam::571219239203:user/gtzroot

Name|Type|ARN
----|----|---
gtzroot|IAM|arn:aws:iam::571219239203:user/gtzroot
StateMachineLambdaRole|IAM|arn:aws:iam::571219239203:role/StateMachineLambdaRole

#### AWS-Landing-Zone-Account-Vending-Machine

- Product List - (SO0045) - AWS Landing Zone - Account Vending Machine Template → Launch Product → Network
  - Note: No Default VPCs are created.

- Using the Vending Machine - Role AWSCloudFormationStackSetExecutionRole only allows other roles from root account to assume this role. No changes required.
  - Role: AWSCloudFormationStackSetExecutionRole
  `Permission Policy`
  ```json
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": "*",
              "Resource": "*"
          }
      ]
  }
  ```
  `Trust Relationship`
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "AWS": [
            "arn:aws:iam::571219239203:role/LandingZoneLambdaRole",
            "arn:aws:iam::571219239203:role/StateMachineLambdaRole",
            "arn:aws:iam::571219239203:role/StateMachineTriggerLambdaRole",
            "arn:aws:iam::571219239203:role/AWSCloudFormationStackSetAdministrationRole",
            "arn:aws:iam::571219239203:role/LandingZoneHandshakeSMLambdaRole"
          ]
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }
  ```

#### AWS Single-Sign-On
- Created user in AWS SSO

  Sign-in URL|Username
  -----------|--------
  https://gtz.awsapps.com/start|gtzssoadmin

#### References
- https://medium.com/@superluminar/tested-for-you-multi-account-setups-with-aws-landing-zone-b934154cfc78
- https://medium.com/@superluminar/extending-aws-landing-zone-a-real-world-example-63b8d46115dc
- https://virtualbonzo.com/2019/11/08/aws-multi-account-architecture-part-2-aws-landing-zone/
