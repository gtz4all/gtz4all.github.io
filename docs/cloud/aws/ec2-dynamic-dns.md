---
title: "EC2 Cross-Account Dynamic DNS Design"
date: 2021-08-23T18:19:36-04:00
draft: false
description: "EC2 Cross-Account Dynamic DNS Design"
---

EC2 Dynamic DNS provides a simple integration betwee private and hybrid cloud.

Dynamic DNS solutions for EC2's and ALB's 
- Tag Key: DNSName
- Tag Value: must not include special characters

Combine into a functional lambda Tagging solution
- 3 into One
- VPC must be shared
-- maybe look for shared tag on remote accounts
--
Function to tell the difference between EC2's Tags and ALBs
split function and pull fuctions as needed 



* Import event data as json 
```py
def load_event(file_name):
    """This function load file as variable."""
    try:
      with open(file_name) as fileobj:
          event = json.loads(fileobj.read()) 
          
    except ClientError as e:
        print(e.response['Error']['Message'])
        
    return event
```

###### CloudWatch EC2 State Event
```json
{
   "version":"0",
   "id":"44265205-9501-dd5f-39b5-6af68926b286",
   "detail-type":"EC2 Instance State-change Notification",
   "source":"aws.ec2",
   "account":"089714066692",
   "time":"2021-08-29T17:03:52Z",
   "region":"us-east-1",
   "resources":[
      "arn:aws:ec2:us-east-1:089714066692:instance/i-0389a92bd5b886c39"
   ],
   "detail":{
      "instance-id":"i-0389a92bd5b886c39",
      "state":"running"
   }
}

```
#### Steps

1. Create VPC
   - https://gitlab.com/gtz4all/aws-three-tier-vpc-cf/-/blob/main/gtz4all-three-tier-vpc-nested-conditions.yml

2. Create Hosted Zone based on project name and domain
   * sbox.gtz4all.com
   Associate Private Zone with VPC

3. Create Lambda Role
   * sbox-dev-ddns-lambda-role/policy

```json
{
    "Statement": [
        {
            "Action": [
                "logs:CreateLogStream",
                "ec2:Describe*",
                "ec2:CreateTags",
                "ec2:DeleteTags",
                "route53:*",
                "dynamodb:*",
                "logs:CreateLogGroup",
                "logs:PutLogEvents"
            ],
            "Condition": {
                "ForAllValues:StringEquals": {
                    "aws:TagKeys": [
                        "DDNS"
                    ]
                },
                "StringEquals": {
                    "aws:RequestedRegion": [
                        "us-east-1",
                        "us-east-2"
                    ]
                }
            },
            "Effect": "Allow",
            "Resource": "*",
            "Sid": "sboxDDNSNetwork"
        },
        {
            "Action": "sts:AssumeRole",
            "Condition": {
                "ForAllValues:StringEquals": {
                    "aws:TagKeys": [
                        "DDNS"
                    ]
                },
                "StringEquals": {
                    "aws:RequestedRegion": [
                        "us-east-1",
                        "us-east-2"
                    ]
                }
            },
            "Effect": "Allow",
            "Resource": "*",
            "Sid": "sboxDDNSClient"
        }
    ],
    "Version": "2012-10-17"
}
```

3. Create lambda and associate created role
    * Adjust timeout to 5min

4. Create EventBridge

   * sbox-dev-ec2-state
   

