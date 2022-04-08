# VPC
> Amazon Virtual Private Cloud (Amazon VPC) enables you to launch AWS resources into a virtual network that you've defined. This virtual network closely resembles a traditional network that you'd operate in your own data center, with the benefits of using the scalable infrastructure of AWS.

To create this entity in your **Litmus Scale** template, use the following data:
```
{
    "NAME": "str",
    "VPC_CIDR": "str",
    "AZS_COUNT": "int",
    "PUBLIC_SUBNETS": "list",
    "PRIVATE_SUBNETS": "list",
    "PRIVATE_SUBNETS_WITHOUT_NG": "list",
    "TAGS": "str",
    "ENABLE_DNS_SUPPORT": "bool",
    "ENABLE_DNS_HOSTNAMES": "bool",
    "INSTANCE_TENANCY": "str",
    "MAP_PUBLIC_IP_ON_LAUNCH": "bool"
    "TAGS": "TAG"
}
```
## Properties
---
### Name
The name of the vpc which is going to be provisioned
 - **Required** : Yes
 - **Type** : String

### VPC_CIDR
The primary IPv4 CIDR block for the VPC.
- **Required** : Yes
- **Type** : String

### AZS_COUNT
Number of availability zones in which you want to span your subnets
- **Required** : No
- **Type** : Integer
- **Default** : 3

### PUBLIC_SUBNETS
List of IPv4 CIDR block for each subnet that's associated with a route table that has a route to an internet gateway.
- **Required** : No
- **Type**: List of Strings
- **Default** : []

### PRIVATE_SUBNETS
List of IPv4 CIDR block for each subnet that's associated with a route table that has no route to an internet gateway and a nat gateway associated.
- **Required** : No
- **Type**: List of Strings
- **Default** : []

### PRIVATE_SUBNETS_WITHOUT_NG
List of IPv4 CIDR block for each subnet that's associated with a route table that has no route to an internet gateway and without a nat gateway association.
- **Required** : Conditional (Required if No Public Subnets are created otherwise optional)
- **Type**: List of Strings
- **Default** : []

### ENABLE_DNS_SUPPORT
A boolean flag to enable/disable DNS support in the VPC
- **Required** : No
- **Type** : Boolean
- **Default** :  True

### ENABLE_DNS_HOSTNAMES
A boolean flag to enable/disable DNS hostnames in the VPC
- **Required** : No
- **Type** : Boolean
- **Default** :  True

### INSTANCE_TENANCY
The allowed tenancy of instances launched into the VPC.
- **Required**: No
- **Type** : String
- **Default**: Default
- **Allowed Values** : 
    - "default": An instance launched into the VPC runs on shared hardware by default, unless you explicitly specify a different tenancy during instance launch.

    - "dedicated": An instance launched into the VPC is a Dedicated Instance by default, unless you explicitly specify a tenancy of host during instance launch. You cannot specify a tenancy of default during instance launch.

### MAP_PUBLIC_IP_ON_LAUNCH
A boolean flag to enable/disable assigning of public ip address to an instance on launch
- **Required** : No
- **Type** : Boolean
- **Default** :  True

### TAGS
The tags for the VPC.
- **Required**: No
- **Type**: JSON
---
##  Resource Referencing Variables
Resource referencing variables avaialbe for VPC which can be used for cross resource referencing
> Note : Replace index and reference with your desired value 

### VPC_ID
VPC ID of the created VPC.
- **Type** : String
- **Usage** :
    - Same Request
        - ${VPC[index].VPC_ID}
    - Previous Request
        - import ${\<reference name \>.VPC[index].VPC_ID}

### VPC_CIDR
Primary VPC CIDR of the created VPC.
- **Type** : String
- **Usage** :
    - Same Request
        - ${VPC[index].VPC_CIDR}
    - Previous Request
        - import ${\<reference name \>.VPC[index].VPC_CIDR}

### PUBLIC_SUBNET_IDS
IDs of the public subnets created.
- **Type** : LIST
- **Usage** :
    - Same Request
        - Referring all public subnets ids
            - ${VPC[index].PUBLIC_SUBNET_IDS}
        - Referring one public subnet id
            - ${VPC[index].PUBLIC_SUBNET_IDS[index]}
        
    - Previous Request
        - import ${\<reference name \>.VPC[index].PUBLIC_SUBNET_IDS}

