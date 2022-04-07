# Litmus Provisioning Tool
This Readme refers to the installation and usage of the CompliancX Provisioning tool.

# About Tool
This tool is for provisioning compliant resources in AWS Cloud.resources can be provisioned in two ways, either with the provision or multi-provision commands. In provision we can create a single resource at one time, for example, we can provision an S3 bucket in one request while in multi-provision we can create multiple resources at one time.
Some resources are interdependent to each other for example EC2 instance needs a subnet id where this instance will be provisioned so the EC2 resource is dependent on VPC. in this scenario we can use cross resource referencing to provide values from one resource to another. please find examples of cross resource referencing in the below sections. When the tool starts to provision any resource it first checks for all required parameters to make sure that they are provided. After that tool runs a compliance check on the values provided and if the compliance check passes then it proceeds to provision the resource.

# Installation and Login
For installation of Provisioning tool first, install python3 on your machine but recommended version in python 3.8 version.
Clone repo to your machine and go to ComplianceCli directory and install tool with pip command i-e pip install .
now configure login with your credentials, client, region, and account details in which you want to provision resources. below are some commands to get help menu and login to the compliance cli.

- help manu
```
ltscale --help
```

- login through local Credentials
```
ltscale login -l
```
- login through IDP Credentials
```
ltscale login
```

# Resource Provisioning

After login into the compliance tool, we are all set to provision resources in the account you just configured in the login process. Now there are two ways to provision resources either by passing values to variables of Resource one by one on run time or the other way is to pass JSON file of the resource having module of key-value pairs to define properties of resource in a single file and then pass that file path to compliance command.

# Reference Name
The reference name is a unique name Per Account Per region. The reference name is assigned to the resource request so that after provisioning the request if we need to change something in that resource we can change it and can apply it through the reference name of that resource to make sure that changes are applied to the same resource. Reference name should be unique for every region in account hence we cant use one reference name twice in a single region. 


# Provisioning Single Resource request

- getting resource list
```
ltscale provision -h
```

- You will get list of Resources which can be provisioned, now pick your resour ce and work on it with below command. for example i will pick KMS. this command will take me through every Parameter key of KMS and will ask for value of it.
```
ltscale provision KMS
```
- To get JSON formate template for a single resource we can generate it with following command. i will take KMS from last example to demonstrate.
```
ltscale provision KMS -v
```
- After saing that JSON template of resource and providing values to parameters we can now provision that Resource with following command.
```
ltscale provision KMS --data file://path-to-file
```

# Provisioning multi Resource request

We can provide JSON files of any resource for provisioning.
 
In a single resource, we will have single resource configurations in JSON file while in multi-provision we will have multiple resource configurations in a single file so that we can create multiple resources in a single request. For both of the provisioning types, every request will have a Unique Resource name per region.

to get that JSON file of Resource run below command.

```
ltscale multi-provision -v
```
this will prompt a list of Resources. inter a number of resources, you want to create, and on inter tool will give JSON file of that Resource. copy that to file on your machine. provide values to keys and save the file.provide this file path in the below command to provision resource.

```
ltscale multi-provision --data file://Path-to-file
```
# Resource Referencing
If resources are dependent on each other then we can use cross resource referencing to feed values of one resource into other. Here is the main use of REFERENCE name of request because through this REFERENCE name will refer values in between resource requests if.

Cross-referencing will be of two type 
- Referring to a resource from the current request.
- Referring a previously provisioned resource

#  Referring to a resource from the current request

The format for referencing a resources from same request:

```
${RESOURCE_NAME[RESOURCE_INDEX].RESOURCE_VARIABLE}
E.g 
${VPC[0].VPC_ID}
${VPC[0].PUBLIC_SUBNET_IDS}

${RESOURCE_NAME[RESOURCE_INDEX].RESOURCE_VARIABLE[RESOURCE_VARIABLE_INDEX]]}
E.g
${VPC[0].PUBLIC_SUBNET_IDS[0]}
${VPC[0].PRIVATE_SUBNETS_WITHOUT_NG_IDS[0]}

```
here VPC[0] means the first VPC in the current Request so if we have two VPCs,s in the current request and we want to refer 2nd VPC then we will pick it from indexing i-e VPC[1] , in the below example we are reffering VPC id in Security Group and both are in same Request.

```
{
   "REFERENCE_NAME": "sameRequestReference",
   "REGION": "us-east-1",
   "VPC": [
       {
           "NAME": "dev",
           "VPC_CIDR": "10.0.0.0/16",
           "AZS_COUNT": 3,
           "PUBLIC_SUBNETS": [],
           "PRIVATE_SUBNETS": [],
           "PRIVATE_SUBNETS_WITHOUT_NG": [
               "10.0.1.0/24"
           ],
           "TAGS": {},
           "ENABLE_DNS_SUPPORT": false,
           "ENABLE_DNS_HOSTNAMES": false,
           "INSTANCE_TENANCY": "default",
           "MAP_PUBLIC_IP_ON_LAUNCH": true,
       }
   ],
   "SecurityGroup": [
       {
           "SG_NAME": "dev",
           "SG_DESCRIPTION": "dev",
           "VPC_ID": "${VPC[0].VPC_ID}",
           "SG_INGRESS": [
               {
                   "sg_ingress_from_port": 22,
                   "sg_ingress_to_port": 22,
                   "sg_ingress_protocol": "tcp",
                   "sg_ingress_description": "dev",
                   "sg_ingress_cidr_blocks": [
                       "192.168.10.10/32"
                   ],
                   "sg_ingress_security_groups": [],
                   "sg_ingress_self": false
               }
           ],
           "TAGS": {},
       }
   ]

}

```

# Referring a previously provisioned resource
if we have separate requests and we want to refer Resource values of Request A to Request B then we will use the REFERENCE name with key "import".
this import key will look for the Reference name and then to its resources and will pic the desired value from the request.

The format for referencing a previous resources is:

```
REFERENCE_NAME =  Request reference name added by the user during provisioning of resource
RESOURCE_NAME = Cloud resource name which can be created through complianceX . Available resources are enlisted below
RESOURCE_INDEX = Index at which the resource request was initiated
RESOURCE_VARIABLE = Variable which can be referenced for associating a particular information about the resource. Available Variables for each resource are enlisted below.
RESOURCE_VARIABLE_INDEX = If the RESOURCE_VARIABLE data type is list index can be used to refer to a single value from list. 

import ${REFERENCE_NAME.RESOURCE_NAME[RESOURCE_INDEX].RESOURCE_VARIABLE}
e.g
import ${referenceABC.VPC[0].VPC_ID}
import ${referenceABC.VPC[0].PUBLIC_SUBNET_IDS}

import ${REFERENCE_NAME.RESOURCE_NAME[RESOURCE_INDEX].RESOURCE_VARIABLE[RESOURCE_VARIABLE_INDEX]]}
e.g 
import ${referenceABC.VPC[0].PUBLIC_SUBNET_IDS[0]}
import ${referenceABC.VPC[0].PRIVATE_SUBNETS_WITHOUT_NG_IDS[0]}

```
In the below example we are creating VPC in Request A with the reference name “Multi_Reference”
And then we are importing its VPC ID and SUBNET ID to the Request B Resources

- Request A ..!

```
{
   "REFERENCE_NAME": "Multi_Reference",
   "REGION": "us-east-1",
   "VPC": [
       {
           "NAME": "dev",
           "VPC_CIDR": "10.0.0.0/16",
           "AZS_COUNT": 3,
           "PUBLIC_SUBNETS": ["10.0.1.0/24"],
           "PRIVATE_SUBNETS": [],
           "PRIVATE_SUBNETS_WITHOUT_NG": [],
           "TAGS": {},
           "ENABLE_DNS_SUPPORT": false,
           "ENABLE_DNS_HOSTNAMES": false,
           "INSTANCE_TENANCY": "default",
           "MAP_PUBLIC_IP_ON_LAUNCH": true
       }
   ]
}

```

- Request B ..!

```
{
   "REFERENCE_NAME": "currentRequest",
   "REGION": "us-east-1",
   "SecurityGroup": [
       {
           "SG_NAME": "dev",
           "SG_DESCRIPTION": "dev",
           "VPC_ID": "import ${Multi_Reference.VPC[0].VPC_ID}",
           "SG_INGRESS": [
               {
                   "sg_ingress_from_port": 22,
                   "sg_ingress_to_port": 22,
                   "sg_ingress_protocol": "tcp",
                   "sg_ingress_description": "dev",
                   "sg_ingress_cidr_blocks": [
                       "192.168.10.10/32"
                   ],
                   "sg_ingress_security_groups": [],
                   "sg_ingress_self": false
               }
           ],
           "TAGS": {}
       },
       "EC2": [
              {
                "NAME": "test",                                      
                "AMI": "str",                                       
                "INSTANCE_TYPE": "str",                             
                "SUBNET_ID": ["import ${Multi_Reference.VPC[0].PUBLIC_SUBNET_IDS[0]}"],                                       
                "ROOT_BLOCK_DEVICE": {                                
                  "volume_size": "int",                                   
                  "volume_type": "str",                                    
                  "kms_key_id": "str"                                     
              }
          }
       ]
   ]
}

```

# Defining Tags in JSON file
Tags are key-value pairs and can be defined in our JSON file as following

```
"TAGS": {
       "Key": "Value"
      }
```
