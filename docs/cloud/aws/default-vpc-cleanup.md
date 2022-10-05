---
title: "Default Vpc Cleanup"
date: 2021-05-23T17:58:49-04:00
draft: false
Description: "Lambda - Default VPC Cleanup"
---

## Delete DefaultVPC Lambda
    This lambda is used to delete all DefaultVPCs and their resources in every region
    * Log all work
    * push to BD for historical reference
### IAM Roles
In order to provide this solution we need to create IAM roles in both core and child account.

###### Core Network  Lambda Role
* Access DynamoDB Table 'DefaultVPC_Cleanup' used for inventory purposes.
* Allows lambda to assume role 'core-network-delete-defaultvpc-lambda-role' on other accounts
* Logging
##### core-network-delete-defaultvpc-lambda-policy
```python
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DefaultVPCCleanupDB",
            "Effect": "Allow",
            "Action": [
                "dynamodb:DeleteItem",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem"
            ],
            "Resource": "arn:aws:dynamodb:us-east-1:860014166701:table/DefaultVPC_Cleanup"
        },
        {
            "Sid": "SwitchRole",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": [
               "arn:aws:iam::*:role/core-network-delete-defaultvpc-lambda-role",
               "arn:aws:iam::*:role/core-network-restricted-access-lambda-role"
              ]
        },
        {
            "Sid": "Logging",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```
##### Trust Relationship
```python
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
##### Lambda
```python
import sys
import boto3
import botocore
import json
import requests
from datetime import datetime
from botocore.exceptions import ClientError

# a way to print messages - VERBOSE = 0 will not print messages
VERBOSE = 1
DeleteDefaultVpcDB = 'DefaultVPC_Cleanup'

print('Loading function ' + datetime.now().time().isoformat())
    
def lambda_handler(event,context):
  print(event)
  # API Gateway Response
  response = {
      "statusCode": 200,
      "headers": {
          "Access-Control-Allow-Origin": "*",
          "Content-Type": "application/json;",
      },
      "isBase64Encoded": False
  }
  if "FormAwsAcct" in event['queryStringParameters']:
    #get Org Accounts
    getOrgAccounts = requests.get('https://7bzbqo4224.execute-api.us-east-1.amazonaws.com/Production/listaccounts')
    OrgAccountsDB = getOrgAccounts.json()
    #print(OrgAccountsDB)
    
    for account in OrgAccountsDB:
      print('Validating Account: ' + event['queryStringParameters']['FormAwsAcct'])
      #print(account['Id'])
      #print(event['queryStringParameters']['FormAwsAcct'])
      if account['Id'] == event['queryStringParameters']['FormAwsAcct']:
        account_id = event['queryStringParameters']['FormAwsAcct']
        role_id = event['queryStringParameters']['FormSwitchRole']
        region_id = 'us-east-1' 
        type_id = 'ec2'
        # get account regions
        ec2, client = switch_role(account_id, role_id, region_id, type_id)
        regions = get_regions(client)
        account_list = {}
        account_list['AccountCleanup'] = {}
        account_list['AccountId'] = account_id
        account_list['DateTime'] = datetime.now().strftime("%d-%m-%Y %H:%M:%S")
        for region in regions:
          #if region == 'us-east-1':
          print(region)
          account_list[region] = {}
          try:
            ec2, client = switch_role(account_id, role_id, region, type_id)
            vpcs = get_default_vpcs(client)
            print(vpcs)
          except ClientError as e:
            #print(e.response['Error']['Message'] + 'this is my msg test' + "\n")
            ## for any errors add a dynamodb tab with the error msg per region
            account_list['AccountCleanup'][region] = ('GetDefaultVpc: ' + (e.response['Error']['Message']))
            continue
          #else:
          if (vpcs):
            #print(vpcs)
            for vpc in vpcs:
              #print(vpc)
              print("REGION:" + region + " - " + "DefaultVPC Id: " + vpc)
              account_list[region]['DefaultVpcId'] = (vpc)
              igw = del_igw(ec2, vpc)
              account_list[region]['DefaultVpcIgw'] = (igw)
              subnets = del_sub(ec2, vpc)
              account_list[region]['DefaultVpcSubnets'] = (subnets)
              rtb = del_rtb(ec2, vpc)
              account_list[region]['DefaultVpcRouteTable'] = (rtb)
              rtb_acls = del_acl(ec2, vpc)
              account_list[region]['DefaultVpcRouteTableAcls'] = (rtb_acls)
              secgroups = del_sgp(ec2, vpc)
              account_list[region]['DefaultVpcSecurityGroups'] = (secgroups)
              cleanup = del_vpc(ec2, vpc)
              account_list[region]['DefaultVpcCleanup'] = (cleanup)
              
              if ('Deleted Successfully' in account_list[region]['DefaultVpcCleanup']):
                account_list['AccountCleanup'][region] = 'Complete'
              else:
                account_list['AccountCleanup'][region] = 'Incomplete'
          else:
            print("REGION:" + region + " - " + "DefaultVPC Id: " + 'None' + "\n")
            account_list[region] = {'DefaultVpcId': 'None' }
            account_list['AccountCleanup'][region] = 'Complete'
            
          # update DynamoDB Tables            
          AccountList_dump = json.dumps(account_list)
          AccountList = json.loads(AccountList_dump)
          table = boto3.resource('dynamodb').Table(DeleteDefaultVpcDB)
          table.put_item(Item = AccountList)
          
          
          response['body'] = (AccountList_dump)
        print(response)  
        return(response)
        break
      else:
        response['body'] = ('<p style="text-align: center;">You have provided an invalid BCH Managed AWS Account. Please resubmit your request - <a href="http://websvc4:8090/display/AWS/Default+VPC+Removal+Webform" target="_blank">BCH AWS Default VPC Removal Webform.</a></p>')
    print(response)
    return (response)

              
      #update DynamoDB Tables            
      #AccountList_dump = json.dumps(account_list)
      #AccountList = json.loads(AccountList_dump)
      #table = boto3.resource('dynamodb').Table(DeleteDefaultVpcDB)
      #table.put_item(Item = AccountList)
      
      #response['body'] = (AccountList_dump)
      #return(response)

def switch_role(acct, role, region, type):
  '''
  acct = aws_account#
  role = assume_role_name
  region = assume_role_region
  type = service type, ie ec2,s3
  '''
  ## some services have 'resource'
  resource_type = ["cloudformation","cloudwatch","dynamodb","ec2","glacier","iam","opsworks","s3","sns","sqs"]
  role_arn = "arn:aws:iam::%s:role/%s" % (acct, role)
  sts_connection = boto3.client('sts')
  try:
    assume_role = sts_connection.assume_role(
      RoleArn = role_arn,
      RoleSessionName = "switch_role_session"
    )
  except ClientError as e:
    print(e.response['Error']['Message'])
    sys.exit(1)
    
  try:
    access_key = assume_role['Credentials']['AccessKeyId']
    secret_key = assume_role['Credentials']['SecretAccessKey']
    session_token = assume_role['Credentials']['SessionToken']
    
    # Creates services using Assumed role credentials
    if type in resource_type:
      resource_creds = boto3.resource(
        type,
        aws_access_key_id=access_key,
        aws_secret_access_key=secret_key,
        aws_session_token=session_token,
        region_name = region
      )
      client_creds = boto3.client(
        type,
        aws_access_key_id=access_key,
        aws_secret_access_key=secret_key,
        aws_session_token=session_token,
        region_name = region
      )
    else:
      resource_creds = 'none'
      client_creds = boto3.client(
        type,
        aws_access_key_id=access_key,
        aws_secret_access_key=secret_key,
        aws_session_token=session_token,
        region_name = region
      )
    
  except ClientError as e:
    return(e.response['Error']['Message'])
    
  return(resource_creds, client_creds)
  
def get_regions(client):
  """ Build a region list """
  reg_list = []

  regions = client.describe_regions()
  #print(regions)
  data_str = json.dumps(regions)
  resp = json.loads(data_str)
  region_str = json.dumps(resp['Regions'])
  region = json.loads(region_str)
  for reg in region:
    reg_list.append(reg['RegionName'])
  #print (reg_list)
  return reg_list

    
def get_default_vpcs(client):
  vpc_list = []
  vpcs = client.describe_vpcs(
    Filters=[
      {
          'Name' : 'isDefault',
          'Values' : [
            'true',
          ],
      },
    ]
  )
  vpcs_str = json.dumps(vpcs)
  resp = json.loads(vpcs_str)
  data = json.dumps(resp['Vpcs'])
  vpcs = json.loads(data)
  
  for vpc in vpcs:
    vpc_list.append(vpc['VpcId'])  
  
  return vpc_list

def del_igw(ec2, vpcid):
  """ Detach and delete the internet-gateway """
  vpc_resource = ec2.Vpc(vpcid)
  igws = vpc_resource.internet_gateways.all()
  if igws:
    for igw in igws:
      try:
        print("Detaching and Removing igw-id: ", igw.id) if (VERBOSE == 1) else ""
        igw.detach_from_vpc(
          VpcId=vpcid
        )
        igw.delete(
          #DryRun=True
        )
        return(igw.id + ': detached and deleted')
      except ClientError as e:
        return(e.response['Error']['Message'])
  #else:
  #  return(vpcid + ": no IGW attached")

def del_sub(ec2, vpcid):
  """ Delete the subnets """
  vpc_resource = ec2.Vpc(vpcid)
  subnets = vpc_resource.subnets.all()
  vpc_subnets = [ec2.Subnet(subnet.id) for subnet in subnets ]  
  if vpc_subnets:
    subnets =[]
    try:
      for sub in vpc_subnets: 
        print("Removing sub-id: ", sub.id) if (VERBOSE == 1) else ""
        subnets.append(sub.id)
        sub.delete(
           #DryRun=True
        )
    except ClientError as e:
      return(e.response['Error']['Message'])

    return(subnets)

def del_rtb(ec2, vpcid):
  """ Delete the route-tables """
  vpc_resource = ec2.Vpc(vpcid)
  rtbs = vpc_resource.route_tables.all()
  if rtbs:
    rtables =[]
    try:
      for rtb in rtbs:
        if (rtb.associations_attribute):
          if (rtb.associations_attribute[0]['Main'] == True):
            print(rtb.id + " is the main route table, continue...") if (VERBOSE == 1) else ""
            rtables.append('main route table: ' + rtb.id)
          continue
        else:
          print("Removing rtb-id: ", rtb.id) if (VERBOSE == 1) else ""
          rtables.append(rtb.id)
          table = ec2.RouteTable(rtb.id)
          table.delete(
            #DryRun=True
          )
    except ClientError as e:
      return(e.response['Error']['Message'])

    return(rtables)

def del_acl(ec2, vpcid):
  """ Delete the network-access-lists """
  vpc_resource = ec2.Vpc(vpcid)      
  acls = vpc_resource.network_acls.all()
  if acls:
    rtable_acls = []
    try:
      for acl in acls: 
        if acl.is_default:
          print(acl.id + " is the default NACL, continue...") if (VERBOSE == 1) else ""
          rtable_acls.append('default_acl: ' + acl.id)
          continue
        else:
          print("Removing acl-id: ", acl.id) if (VERBOSE == 1) else ""
          rtable_acls.append(acl.id)
          acl.delete(
            #DryRun=True
          )
    except ClientError as e:
      return(e.response['Error']['Message'])
    
    return(rtable_acls)
      
      
def del_sgp(ec2, vpcid):
  """ Delete any security-groups """
  vpc_resource = ec2.Vpc(vpcid)
  sgps = vpc_resource.security_groups.all()
  if sgps:
    sgps_list = []
    try:
      for sg in sgps: 
        if sg.group_name == 'default':
          print(sg.id + " is the default security group, continue...") if (VERBOSE == 1) else ""
          sgps_list.append('default_security_group: ' + sg.id)
          continue
        else:
          print("Removing sg-id: ", sg.id) if (VERBOSE == 1) else ""
          sgps_list.append(sg.id)
          sg.delete(
            # DryRun=True
          )
    except ClientError as e:
      return(e.response['Error']['Message'])

    return(sgps_list)

def del_vpc(ec2, vpcid):
  """ Delete the VPC """
  vpc_resource = ec2.Vpc(vpcid)
  try:
    print("Removing vpc-id: ", vpc_resource.id)
    vpc_resource.delete(
      # DryRun=True
    )
    return(vpc_resource.id + ' Deleted Successfully.')
  except ClientError as e:
      return (e.response['Error']['Message'])
      print("Please remove dependencies and delete VPC manually.")
```

##### Confluence Webform

## Default VPC Removal Webform

#### HTML/JavaScript
```HTML

<div>
<!--- Submitted Animation-->
<div class="aui" id="spinner" style="display: none" >
       <aui-spinner size="large"></aui-spinner>
	    <div class="aui" id="ClonfluenceUserName"></div>
        <div class="aui" id="ClonfluenceUserId"></div>
		<p>Your request is being processed...Please wait</p>
</div>
<!--- Submitted Animation-->

<!--- Webform-->
<form class="aui" id="RequestForm" style="margin: 40px;">
    <h3 style="text-align: center;">AWS Account Default VPC Deletion Form</h3>
    <p style="text-align: center;">Please fill in this form to delete Default VPCs from an AWS Account. in the <a href="http://websvc4:8090/display/AWS/BCH+Cloud+Standard+Offerings" target="_blank">BCH AWS Organization.</a></p>

    <label for="fname">AWS Account Number</label>
    <input type="text" id="aws_acct" name="FormAwsAcct" placeholder="Valid AWS Account Number" required>

    <label for="lname">AWS Switch Role</label>
    <input type="text" id="switch_role" name="FormSwitchRole" placeholder="AWS Assume Role - core-network-restricted-access-lambda-role" required>

	 <label for="email">Email Address</label>
    <input type="text" id="email" name="FormEmail" placeholder="Requester BCH Email Address - This is not currently being used but it is a required field" required>

    <button class="aui-button aui-button-primary" type="button" id="button-spin-2" onclick="runAPIGetRequest()">Submit</button>
    <p><small>When you click the "Submit" button, an API Call will be made with the information you've provided. If you need any assistance, please reach out to the BCH Cloud Team.</small></p>
</form>
<!--- Webform-->

<!--- Response Placeholder-->
	<div class="aui" id="submittedMsg"></div>
<!--- Response Placeholder-->

</div>
<script type="text/javascript"> 
//Get Information from confluence
var UserName = AJS.params.userDisplayName;
var UserId = AJS.params.remoteUser.toUpperCase();
//alert(AJS.Meta.get("user-display-name"));
//alert(AJS.Meta.get("remote-user"));

function checkRequired() {
	var i, GetInputs, allowSubmit;
	allowSubmit = true;
	GetInputs = document.getElementsByTagName('input');
	for (i=0; i<GetInputs.length; i+=1) {
		if(GetInputs[i].hasAttribute('required')){
			if ((GetInputs[i].value == null)||(GetInputs[i].value == "")) {
				GetInputs[i].style.backgroundColor = "#e6f2ff";
				allowSubmit = false;
				console.log("checkRequired():" + "Name:" + GetInputs[i].name + " Missing: " + GetInputs[i].placeholder)
			} else {GetInputs[i].style.backgroundColor = "#ffffff";
		    	console.log("checkRequired():" + "Name:" + GetInputs[i].name + " Type: " + GetInputs[i].type.toLowerCase() + " Value: " + GetInputs[i].value)
		        }
			}
	}
	return allowSubmit;
}

function buildParms() {
	var GetInputs;
	var i;
	var parms, tok,
	parms = "";
	GetInputs = document.getElementsByTagName('input');
	for (i=0; i<GetInputs.length; i+=1) {
		if (parms === "") {tok = "?";} else {tok = "&";}
			parms = parms + tok + GetInputs[i].name + "=" + encodeURIComponent(GetInputs[i].value);
		console.log(GetInputs[i].name + "=" + encodeURIComponent(GetInputs[i].value));
		}
	console.log(parms);
	return parms;
}
function runAPIGetRequest() {
	if (checkRequired()) {
    // Public API DNS Names
	//myAPI = "https://609k7k2nn4.execute-api.us-east-1.amazonaws.com/core/getjson"
    //myAPI = "https://609k7k2nn4.execute-api.us-east-1.amazonaws.com/core/defaultvpcremoval"
    // Private API DNS Names
	//myAPI = "https://609k7k2nn4-vpce-0897455697c88d3cd.execute-api.us-east-1.amazonaws.com/core/getjson"
    myAPI = "https://609k7k2nn4-vpce-0897455697c88d3cd.execute-api.us-east-1.amazonaws.com/core/defaultvpcremoval"
	formParms = "";
    formParms = buildParms();
    // adding Confluence Information
    formParms = (formParms + "&" + "UserName" + "=" + UserName +  "&" + "UserId" + "=" + UserId)
	document.getElementById("RequestForm").style.display = "none" ; // hide buttons

	document.getElementById("spinner").style.display = "block" ; // submitted animation
    document.getElementById("ClonfluenceUserName").innerHTML = UserName
    document.getElementById("ClonfluenceUserId").innerHTML = UserId

	var urlWithParms = myAPI + formParms;
	xmlhttp=new XMLHttpRequest();
	xmlhttp.open("GET", urlWithParms, true);
	xmlhttp.send();
	xmlhttp.onreadystatechange = function() {
		if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
		    document.getElementById("submittedMsg").innerHTML = xmlhttp.responseText;
	        document.getElementById("spinner").style.display = "none" ; // submitted animation
        } else if (xmlhttp.status != 200) { 
		    document.getElementById("submittedMsg").innerHTML = '<h6 style="color: #e63900; font-weight: bold;" > There was an issue with your request. Please reach out to BCH Cloud Team</h6>';
	        document.getElementById("spinner").style.display = "none" ; // submitted animation
		}
	};
	}
}


</script>
```
#### CSS 
```CSS
<style>
.sectionMacroRow form{
border-radius: 5px;
width: 800px;
padding: 20px;
-moz-box-shadow:5px 5px 5px 5px rgba(0,0,0,0.3);
-webkit-box-shadow:5px 5px 5px 5px rgba(0,0,0,0.3);
box-shadow:5px 5px 5px 5px rgba(0,0,0,0.3);
}

.sectionMacroRow form input[type=text], select {
  width: 100%;
  padding: 12px 20px;
  margin: 8px 0;
  display: block;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.sectionMacroRow form input[type=submit] {
  width: 100%;
  background-color: #42526e;
  color: white;
  padding: 14px 20px;
  margin: 8px 0;
  display: block;
  text-align: center;
  border: 3px solid #deebff;
  border-radius: 4px;
  cursor: pointer;
}

.sectionMacroRow form input[type=submit]:hover {
  background-color: #0049b0;
}

</style>
```