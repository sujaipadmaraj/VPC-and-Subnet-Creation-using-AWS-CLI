# VPC and Subnet Creation using AWS CLI

Let me demonstrate how to create a VPC, subnets, route tables, security groups and instances without leaving the terminal. I know you must be wondering why can't we use the most popular Terraform insted of AWS Command Line Interface (AWS CLI) here. I'm trying this as just a learning exercise to familiarize myself and others on the detailed configuration steps required to create a custom VPC

## Getting Started

Here in this project, we will creating a custom IPv4 VPC, having both public and private subnets, a NAT gateway, required Route tables, keypairs and three instances. One instance would be in private subnet and the other two would be on public.

### Proccess

Following would be a quick summery of the steps that we are going do in this project.

   - Creating a custom VPC
   - Creating 3 subnets
   - Creating an Internet Gateway
   - Creating a NAT Gateway
   - Updating route tables
   - Creating requires security groups
   - Creating keypair
   - Launching instances
   


### Requirements

We would need a local machine with AWS CLI configured. You can find the steps required to install AWS CLI  [here](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

   ```
 #aws configure
AWS Access Key ID [None]: XXXXXXXXXXXXXXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Default region name [None]: ap-south-1
Default output format [None]: json
   ````

### Create a VPC

Let's start off by creating a VPC.The command below creates a VPC with a 172.16.0.0/16 CIDR block and returns the ID of the new VPC.

     #aws ec2 create-vpc --cidr-block 172.16.0.0/16 --query Vpc.VpcId --output text
     vpc-0cea13535c6cc6163

The command listed below can be used to add a proper nametag to the VPC. Adding tags is always recommended as it enables you to categorize your AWS resources.

    #aws ec2 create-tags --resources vpc-0cea13535c6cc6163 --tags Key=Name,Value=MyVPC


### Creating Subnets and its name tags

Create the first subnet in your VPC using the CIDR block 172.16.0.0/18.

```
#aws ec2 create-subnet --vpc-id vpc-0cea13535c6cc6163 --cidr-block 172.16.0.0/18 --availability-zone ap-south-1a
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "aps1-az1",
        "AvailableIpAddressCount": 16379,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:ap-south-1:563053030797:subnet/subnet-091c1837f7353fe0e",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0cea13535c6cc6163",
        "State": "available",
        "AvailabilityZone": "ap-south-1a",
        "SubnetId": "subnet-091c1837f7353fe0e",
        "OwnerId": "563053030797",
        "CidrBlock": "172.16.0.0/18",
        "AssignIpv6AddressOnCreation": false
    }
}

```
Create tags for subnet 1

```
aws ec2 create-tags --resources subnet-091c1837f7353fe0e --tags Key=Name,Value=Subnet1
```

Create the second subnet in your VPC using the CIDR block 172.16.64.0/18.

```
aws ec2 create-subnet --vpc-id vpc-0cea13535c6cc6163 --cidr-block 172.16.64.0/18 --availability-zone ap-south-1b
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "aps1-az3",
        "AvailableIpAddressCount": 16379,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:ap-south-1:563053030797:subnet/subnet-05a5654029f8b9d47",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0cea13535c6cc6163",
        "State": "available",
        "AvailabilityZone": "ap-south-1b",
        "SubnetId": "subnet-05a5654029f8b9d47",
        "OwnerId": "563053030797",
        "CidrBlock": "172.16.64.0/18",
        "AssignIpv6AddressOnCreation": false
    }
}
```
Create a tag for the second subnet

```
aws ec2 create-tags --resources subnet-05a5654029f8b9d47 --tags Key=Name,Value=Subnet2
```
Create the third subnet in your VPC using the CIDR block 172.16.128.0/18. This serves as our private subnet.

```
aws ec2 create-subnet --vpc-id vpc-0cea13535c6cc6163 --cidr-block 172.16.128.0/18 --availability-zone ap-south-1b
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "aps1-az3",
        "AvailableIpAddressCount": 16379,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:ap-south-1:563053030797:subnet/subnet-0a2bc33f670f19968",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0cea13535c6cc6163",
        "State": "available",
        "AvailabilityZone": "ap-south-1b",
        "SubnetId": "subnet-0a2bc33f670f19968",
        "OwnerId": "563053030797",
        "CidrBlock": "172.16.128.0/18",
        "AssignIpv6AddressOnCreation": false
    }
}
```
Create a tag for the private subnet

```
aws ec2 create-tags --resources subnet-0a2bc33f670f19968 --tags Key=Name,Value=Subnet3
```
Enable public IP for the subnet1 and subnet2.

```
#aws ec2 modify-subnet-attribute --subnet-id subnet-091c1837f7353fe0e  --map-public-ip-on-launch
# aws ec2 modify-subnet-attribute --subnet-id subnet-05a5654029f8b9d47  --map-public-ip-on-launch
```
Using the command below, we can view all of the details of the VPC and subnets.

```
aws ec2 describe-subnets  --filters "Name=vpc-id,Values=vpc-0cea13535c6cc6163"
{
    "Subnets": [
        {
            "MapPublicIpOnLaunch": true,
            "AvailabilityZoneId": "aps1-az1",
            "Tags": [
                {
                    "Value": "Subnet1",
                    "Key": "Name"
                }
            ],
            "AvailableIpAddressCount": 16379,
            "DefaultForAz": false,
            "SubnetArn": "arn:aws:ec2:ap-south-1:563053030797:subnet/subnet-091c1837f7353fe0e",
            "Ipv6CidrBlockAssociationSet": [],
            "VpcId": "vpc-0cea13535c6cc6163",
            "MapCustomerOwnedIpOnLaunch": false,
            "AvailabilityZone": "ap-south-1a",
            "SubnetId": "subnet-091c1837f7353fe0e",
            "OwnerId": "563053030797",
            "CidrBlock": "172.16.0.0/18",
            "State": "available",
            "AssignIpv6AddressOnCreation": false
        },
        {
            "MapPublicIpOnLaunch": true,
            "AvailabilityZoneId": "aps1-az3",
            "Tags": [
                {
                    "Value": "Subnet2",
                    "Key": "Name"
                }
            ],
            "AvailableIpAddressCount": 16379,
            "DefaultForAz": false,
            "SubnetArn": "arn:aws:ec2:ap-south-1:563053030797:subnet/subnet-05a5654029f8b9d47",
            "Ipv6CidrBlockAssociationSet": [],
            "VpcId": "vpc-0cea13535c6cc6163",
            "MapCustomerOwnedIpOnLaunch": false,
            "AvailabilityZone": "ap-south-1b",
            "SubnetId": "subnet-05a5654029f8b9d47",
            "OwnerId": "563053030797",
            "CidrBlock": "172.16.64.0/18",
            "State": "available",
            "AssignIpv6AddressOnCreation": false
        },
        {
            "MapPublicIpOnLaunch": false,
            "AvailabilityZoneId": "aps1-az3",
            "Tags": [
                {
                    "Value": "Subnet3",
                    "Key": "Name"
                }
            ],
            "AvailableIpAddressCount": 16379,
            "DefaultForAz": false,
            "SubnetArn": "arn:aws:ec2:ap-south-1:563053030797:subnet/subnet-0a2bc33f670f19968",
            "Ipv6CidrBlockAssociationSet": [],
            "VpcId": "vpc-0cea13535c6cc6163",
            "MapCustomerOwnedIpOnLaunch": false,
            "AvailabilityZone": "ap-south-1b",
            "SubnetId": "subnet-0a2bc33f670f19968",
            "OwnerId": "563053030797",
            "CidrBlock": "172.16.128.0/18",
            "State": "available",
            "AssignIpv6AddressOnCreation": false
        }
    ]
}
```
### Create an Internet Gateway

The following command generates an Internet Gateway and returns the new Internet Gateway ID.

```
aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
igw-03350a80f6e2f9220
```

Now we need to attach the internet gateway to our VPC to allow the instances(and NAT) to communicate with the outside world.

```
aws ec2 attach-internet-gateway --vpc-id vpc-0cea13535c6cc6163 --internet-gateway-id igw-03350a80f6e2f9220
```

### Create a NAT Gateway.

Using the allocate-address command, assign an elastic IP and then create a NAT Gateway.

```
aws ec2 allocate-address
{
    "Domain": "vpc",
    "PublicIpv4Pool": "amazon",
    "PublicIp": "65.1.155.61",
    "AllocationId": "eipalloc-0ae4324012b353bfd",
    "NetworkBorderGroup": "ap-south-1"
}
```

Command to create NAT gateway.

```
aws ec2 create-nat-gateway --subnet-id subnet-05a5654029f8b9d47 --allocation-id eipalloc-0ae4324012b353bfd
{
    "NatGateway": {
        "NatGatewayAddresses": [
            {
                "AllocationId": "eipalloc-0ae4324012b353bfd"
            }
        ],
        "VpcId": "vpc-0cea13535c6cc6163",
        "State": "pending",
        "NatGatewayId": "nat-019e4fbdb0adce594",
        "SubnetId": "subnet-05a5654029f8b9d47",
        "CreateTime": "2022-12-16T18:00:31.000Z"
    },
    "ClientToken": "3ebb1a95-8c0d-4f34-9725-f546dcff1b5a"
}
```
### Update route tables

By default, a route table will be created. The command below can be used to access the VPC's default route table information.

```
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-0cea13535c6cc6163"
{
    "RouteTables": [
        {
            "Associations": [
                {
                    "AssociationState": {
                        "State": "associated"
                    },
                    "RouteTableAssociationId": "rtbassoc-05e1a398c6e3676ab",
                    "Main": true,
                    "RouteTableId": "rtb-0bd1340de9d13d9d5"
                }
            ],
            "RouteTableId": "rtb-0bd1340de9d13d9d5",
            "VpcId": "vpc-0cea13535c6cc6163",
            "PropagatingVgws": [],
            "Tags": [],
            "Routes": [
                {
                    "GatewayId": "local",
                    "DestinationCidrBlock": "172.16.0.0/16",
                    "State": "active",
                    "Origin": "CreateRouteTable"
                }
            ],
            "OwnerId": "563053030797"
        }
    ]
}
```
Create a custom route table for private subnet using the below command and associate the private subnet with it.

```
aws ec2 create-route-table --vpc-id vpc-0cea13535c6cc6163 --query RouteTable.RouteTableId --output text
rtb-0e921bd2bfd37b164
```

```
aws ec2 associate-route-table  --subnet-id subnet-0a2bc33f670f19968 --route-table-id rtb-0e921bd2bfd37b164
{
    "AssociationState": {
        "State": "associated"
    },
    "AssociationId": "rtbassoc-0edec3cd23da697f2"
}
```

Add route table entry for Internet Gateway in the default route table to allow the instances to communicate with the internet.

```
aws ec2 create-route --route-table-id rtb-0bd1340de9d13d9d5 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-03350a80f6e2f9220
{
    "Return": true
}
```

To allow instances on the private subnet to communicate with the NAT gateway, add a route table entry to the custom route table.

```
aws ec2 create-route --route-table-id rtb-0e921bd2bfd37b164 --destination-cidr-block 0.0.0.0/0 --nat-gateway-id nat-019e4fbdb0adce594
{
    "Return": true
}
```

Using the following describe-route-tables command, we can describe the route table and verify that the route has been created and is active.

```
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-0cea13535c6cc6163"
{
    "RouteTables": [
        {
            "Associations": [
                {
                    "SubnetId": "subnet-0a2bc33f670f19968",
                    "AssociationState": {
                        "State": "associated"
                    },
                    "RouteTableAssociationId": "rtbassoc-0edec3cd23da697f2",
                    "Main": false,
                    "RouteTableId": "rtb-0e921bd2bfd37b164"
                }
            ],
            "RouteTableId": "rtb-0e921bd2bfd37b164",
            "VpcId": "vpc-0cea13535c6cc6163",
            "PropagatingVgws": [],
            "Tags": [],
            "Routes": [
                {
                    "GatewayId": "local",
                    "DestinationCidrBlock": "172.16.0.0/16",
                    "State": "active",
                    "Origin": "CreateRouteTable"
                },
                {
                    "Origin": "CreateRoute",
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "NatGatewayId": "nat-019e4fbdb0adce594",
                    "State": "active"
                }
            ],
            "OwnerId": "563053030797"
        },
        {
            "Associations": [
                {
                    "AssociationState": {
                        "State": "associated"
                    },
                    "RouteTableAssociationId": "rtbassoc-05e1a398c6e3676ab",
                    "Main": true,
                    "RouteTableId": "rtb-0bd1340de9d13d9d5"
                }
            ],
            "RouteTableId": "rtb-0bd1340de9d13d9d5",
            "VpcId": "vpc-0cea13535c6cc6163",
            "PropagatingVgws": [],
            "Tags": [],
            "Routes": [
                {
                    "GatewayId": "local",
                    "DestinationCidrBlock": "172.16.0.0/16",
                    "State": "active",
                    "Origin": "CreateRouteTable"
                },
                {
                    "GatewayId": "igw-03350a80f6e2f9220",
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "State": "active",
                    "Origin": "CreateRoute"
                }
            ],
            "OwnerId": "563053030797"
        }
    ]
}
```

### Create security groups

Let's create a Security group for bastion server. The bastion host acts as a jump server allowing secure connection to instances provisioned without a public IP address. To reduce exposure of servers within the VPC you will create and use a bastion host.

```
aws ec2 create-security-group --group-name bastion --description "Allow SSH access" --vpc-id vpc-0cea13535c6cc6163
{
    "GroupId": "sg-07f6c3b7fa1b13047"
}
```

Now create a security group for the frontend web server

```
aws ec2 create-security-group --group-name webserver --description "Allow SSH access from bastion and  http access from all" --vpc-id vpc-0cea13535c6cc6163
{
    "GroupId": "sg-07afb1e28ad90aafa"
}
```

Now create a security group for the backend database server also

```
aws ec2 create-security-group --group-name dbserver --description "Allow SSH access from bastion" --vpc-id vpc-0cea13535c6cc6163
{
    "GroupId": "sg-00f88e969b2ddcd23"
}
```
Now we need to add rules to the security groups.

Adding a rule to allow SSH access to the bastion server.

```
aws ec2 authorize-security-group-ingress --group-id sg-07f6c3b7fa1b13047 --protocol tcp --port 22 --cidr 0.0.0.0/0
```

Adding a rule to allow SSH access to the frontend server from the bastion server

```
aws ec2 authorize-security-group-ingress --group-id sg-07afb1e28ad90aafa --protocol tcp --port 22 --source-group sg-07f6c3b7fa1b13047
```

Add a rule to allow HTTP access to the frontend server from the outside world.

```
aws ec2 authorize-security-group-ingress --group-id sg-07afb1e28ad90aafa --protocol tcp --port 80 --cidr 0.0.0.0/0
```
Adding a rule to allow SSH access to the backend server from the bastion server.

```
aws ec2 authorize-security-group-ingress --group-id sg-00f88e969b2ddcd23 --protocol tcp --port 22 --source-group sg-07f6c3b7fa1b13047
```
Adding a rule to enable the frontend server to access mysql on the backend server.

```
aws ec2 authorize-security-group-ingress --group-id sg-00f88e969b2ddcd23 --protocol tcp --port 3306 --source-group sg-07afb1e28ad90aafa
```

### Create a key pair

The following commands will create a key pair, save it to a file named aws.pem and update the permission of the file.

```
 aws ec2 create-key-pair --key-name aws.pem --query 'KeyMaterial' --output text > aws.pem
[ec2-user@ip-172-31-11-130 ~]$ chmod 400 aws.pem
```

Enable public DNS hostname for the VPC and verified it using describe-vpc-attribute command.

```
aws ec2 modify-vpc-attribute --vpc-id vpc-0cea13535c6cc6163 --enable-dns-hostnames "{\"Value\":true}"
aws ec2 describe-vpc-attribute --vpc-id vpc-0cea13535c6cc6163 --attribute enableDnsHostnames
{
    "VpcId": "vpc-0cea13535c6cc6163",
    "EnableDnsHostnames": {
        "Value": true
    }
}
```

### Launch instances

Launch the bastion server with the necessary security group, keypair, subnet, and AMI using the following command.

```
aws ec2 run-instances --image-id ami-074dc0a6f6c764218 --count 1 --instance-type t2.micro --key-name aws.pem --security-group-ids sg-07f6c3b7fa1b13047 --subnet-id subnet-05a5654029f8b9d47
{
    "Instances": [
        {
            "Monitoring": {
                "State": "disabled"
            },
            "PublicDnsName": "",
            "StateReason": {
                "Message": "pending",
                "Code": "pending"
            },
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "EbsOptimized": false,
            "LaunchTime": "2022-12-16T19:07:28.000Z",
            "PrivateIpAddress": "172.16.100.69",
            "ProductCodes": [],
            "VpcId": "vpc-0cea13535c6cc6163",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "StateTransitionReason": "",
            "InstanceId": "i-0c0308a136c62849a",
            "EnaSupport": true,
            "ImageId": "ami-074dc0a6f6c764218",
            "PrivateDnsName": "ip-172-16-100-69.ap-south-1.compute.internal",
            "KeyName": "aws.pem",
            "SecurityGroups": [
                {
                    "GroupName": "bastion",
                    "GroupId": "sg-07f6c3b7fa1b13047"
                }
            ],
            "ClientToken": "d9c85c8a-77e8-4d34-85a7-f88e4eeaef7d",
            "SubnetId": "subnet-05a5654029f8b9d47",
            "InstanceType": "t2.micro",
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "NetworkInterfaces": [
                {
                    "Status": "in-use",
                    "MacAddress": "0a:04:d4:6f:d0:fc",
                    "SourceDestCheck": true,
                    "VpcId": "vpc-0cea13535c6cc6163",
                    "Description": "",
                    "NetworkInterfaceId": "eni-071c29b77cece272c",
                    "PrivateIpAddresses": [
                        {
                            "PrivateDnsName": "ip-172-16-100-69.ap-south-1.compute.internal",
                            "Primary": true,
                            "PrivateIpAddress": "172.16.100.69"
                        }
                    ],
                    "PrivateDnsName": "ip-172-16-100-69.ap-south-1.compute.internal",
                    "InterfaceType": "interface",
                    "Attachment": {
                        "Status": "attaching",
                        "DeviceIndex": 0,
                        "DeleteOnTermination": true,
                        "AttachmentId": "eni-attach-087cba35852840465",
                        "AttachTime": "2022-12-16T19:07:28.000Z"
                    },
                    "Groups": [
                        {
                            "GroupName": "bastion",
                            "GroupId": "sg-07f6c3b7fa1b13047"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "OwnerId": "563053030797",
                    "SubnetId": "subnet-05a5654029f8b9d47",
                    "PrivateIpAddress": "172.16.100.69"
                }
            ],
            "SourceDestCheck": true,
            "Placement": {
                "Tenancy": "default",
                "GroupName": "",
                "AvailabilityZone": "ap-south-1b"
            },
            "Hypervisor": "xen",
            "BlockDeviceMappings": [],
            "Architecture": "x86_64",
            "RootDeviceType": "ebs",
            "RootDeviceName": "/dev/xvda",
            "VirtualizationType": "hvm",
            "MetadataOptions": {
                "State": "pending",
                "HttpEndpoint": "enabled",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1
            },
            "AmiLaunchIndex": 0
        }
    ],
    "ReservationId": "r-09de2755ca42c9179",
    "Groups": [],
    "OwnerId": "563053030797"
}
```
Let's create a nametag for bastion server

```
aws ec2 create-tags --resources i-0c0308a136c62849a --tags Key=Name,Value=bastion
```

Similarly, launch the frontend webserver with the required security group, keypair, subnet, and AMI

```
aws ec2 run-instances --image-id  ami-074dc0a6f6c764218 --count 1 --instance-type t2.micro --key-name aws.pem --security-group-ids sg-07afb1e28ad90aafa --subnet-id subnet-091c1837f7353fe0e
{
    "Instances": [
        {
            "Monitoring": {
                "State": "disabled"
            },
            "PublicDnsName": "",
            "StateReason": {
                "Message": "pending",
                "Code": "pending"
            },
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "EbsOptimized": false,
            "LaunchTime": "2022-12-16T19:13:14.000Z",
            "PrivateIpAddress": "172.16.11.160",
            "ProductCodes": [],
            "VpcId": "vpc-0cea13535c6cc6163",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "StateTransitionReason": "",
            "InstanceId": "i-0b9ea7e1699dee5cc",
            "EnaSupport": true,
            "ImageId": "ami-074dc0a6f6c764218",
            "PrivateDnsName": "ip-172-16-11-160.ap-south-1.compute.internal",
            "KeyName": "aws.pem",
            "SecurityGroups": [
                {
                    "GroupName": "webserver",
                    "GroupId": "sg-07afb1e28ad90aafa"
                }
            ],
            "ClientToken": "e1c8d55f-3158-4fb0-9af0-b0144d0d3633",
            "SubnetId": "subnet-091c1837f7353fe0e",
            "InstanceType": "t2.micro",
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "NetworkInterfaces": [
                {
                    "Status": "in-use",
                    "MacAddress": "02:d2:d6:f3:63:b4",
                    "SourceDestCheck": true,
                    "VpcId": "vpc-0cea13535c6cc6163",
                    "Description": "",
                    "NetworkInterfaceId": "eni-071f9ce137ea57ba6",
                    "PrivateIpAddresses": [
                        {
                            "PrivateDnsName": "ip-172-16-11-160.ap-south-1.compute.internal",
                            "Primary": true,
                            "PrivateIpAddress": "172.16.11.160"
                        }
                    ],
                    "PrivateDnsName": "ip-172-16-11-160.ap-south-1.compute.internal",
                    "InterfaceType": "interface",
                    "Attachment": {
                        "Status": "attaching",
                        "DeviceIndex": 0,
                        "DeleteOnTermination": true,
                        "AttachmentId": "eni-attach-07dd93632c0b37ee7",
                        "AttachTime": "2022-12-16T19:13:14.000Z"
                    },
                    "Groups": [
                        {
                            "GroupName": "webserver",
                            "GroupId": "sg-07afb1e28ad90aafa"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "OwnerId": "563053030797",
                    "SubnetId": "subnet-091c1837f7353fe0e",
                    "PrivateIpAddress": "172.16.11.160"
                }
            ],
            "SourceDestCheck": true,
            "Placement": {
                "Tenancy": "default",
                "GroupName": "",
                "AvailabilityZone": "ap-south-1a"
            },
            "Hypervisor": "xen",
            "BlockDeviceMappings": [],
            "Architecture": "x86_64",
            "RootDeviceType": "ebs",
            "RootDeviceName": "/dev/xvda",
            "VirtualizationType": "hvm",
            "MetadataOptions": {
                "State": "pending",
                "HttpEndpoint": "enabled",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1
            },
            "AmiLaunchIndex": 0
        }
    ],
    "ReservationId": "r-0caecc50c1baec475",
    "Groups": [],
    "OwnerId": "563053030797"
}
```

Now launch the backend database server with the required security group, keypair, subnet, and AMI.

```
aws ec2 run-instances --image-id  ami-074dc0a6f6c764218 --count 1 --instance-type t2.micro --key-name aws.pem --security-group-ids sg-00f88e969b2ddcd23 --subnet-id subnet-0a2bc33f670f19968
{
    "Instances": [
        {
            "Monitoring": {
                "State": "disabled"
            },
            "PublicDnsName": "",
            "StateReason": {
                "Message": "pending",
                "Code": "pending"
            },
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "EbsOptimized": false,
            "LaunchTime": "2022-12-16T19:16:40.000Z",
            "PrivateIpAddress": "172.16.131.89",
            "ProductCodes": [],
            "VpcId": "vpc-0cea13535c6cc6163",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "StateTransitionReason": "",
            "InstanceId": "i-0b8e7fbd51e1864bf",
            "EnaSupport": true,
            "ImageId": "ami-074dc0a6f6c764218",
            "PrivateDnsName": "ip-172-16-131-89.ap-south-1.compute.internal",
            "KeyName": "aws.pem",
            "SecurityGroups": [
                {
                    "GroupName": "dbserver",
                    "GroupId": "sg-00f88e969b2ddcd23"
                }
            ],
            "ClientToken": "f92c52ae-483e-4b68-a628-9c8c32963d61",
            "SubnetId": "subnet-0a2bc33f670f19968",
            "InstanceType": "t2.micro",
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "NetworkInterfaces": [
                {
                    "Status": "in-use",
                    "MacAddress": "0a:3b:a1:98:95:4a",
                    "SourceDestCheck": true,
                    "VpcId": "vpc-0cea13535c6cc6163",
                    "Description": "",
                    "NetworkInterfaceId": "eni-02a337be765c1d3b7",
                    "PrivateIpAddresses": [
                        {
                            "PrivateDnsName": "ip-172-16-131-89.ap-south-1.compute.internal",
                            "Primary": true,
                            "PrivateIpAddress": "172.16.131.89"
                        }
                    ],
                    "PrivateDnsName": "ip-172-16-131-89.ap-south-1.compute.internal",
                    "InterfaceType": "interface",
                    "Attachment": {
                        "Status": "attaching",
                        "DeviceIndex": 0,
                        "DeleteOnTermination": true,
                        "AttachmentId": "eni-attach-0ef62a33a95b262be",
                        "AttachTime": "2022-12-16T19:16:40.000Z"
                    },
                    "Groups": [
                        {
                            "GroupName": "dbserver",
                            "GroupId": "sg-00f88e969b2ddcd23"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "OwnerId": "563053030797",
                    "SubnetId": "subnet-0a2bc33f670f19968",
                    "PrivateIpAddress": "172.16.131.89"
                }
            ],
            "SourceDestCheck": true,
            "Placement": {
                "Tenancy": "default",
                "GroupName": "",
                "AvailabilityZone": "ap-south-1b"
            },
            "Hypervisor": "xen",
            "BlockDeviceMappings": [],
            "Architecture": "x86_64",
            "RootDeviceType": "ebs",
            "RootDeviceName": "/dev/xvda",
            "VirtualizationType": "hvm",
            "MetadataOptions": {
                "State": "pending",
                "HttpEndpoint": "enabled",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1
            },
            "AmiLaunchIndex": 0
        }
    ],
    "ReservationId": "r-0fa0de9dbf3466a8e",
    "Groups": [],
    "OwnerId": "563053030797"
}
```

SSH to bastion server use describe command with instance ID to get the public IP

Describe bastion server with following command

```
aws ec2 describe-instances --instance-ids i-0c0308a136c62849a
{
    "Reservations": [
        {
            "Instances": [
                {
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "PublicDnsName": "ec2-15-207-54-194.ap-south-1.compute.amazonaws.com",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "EbsOptimized": false,
                    "LaunchTime": "2022-12-16T19:07:28.000Z",
                    "PublicIpAddress": "15.207.54.194",
                    "PrivateIpAddress": "172.16.100.69",
                    "ProductCodes": [],
                    "VpcId": "vpc-0cea13535c6cc6163",
                    "CpuOptions": {
                        "CoreCount": 1,
                        "ThreadsPerCore": 1
                    },
                    "StateTransitionReason": "",
                    "InstanceId": "i-0c0308a136c62849a",
                    "EnaSupport": true,
                    "ImageId": "ami-074dc0a6f6c764218",
                    "PrivateDnsName": "ip-172-16-100-69.ap-south-1.compute.internal",
                    "KeyName": "aws.pem",
                    "SecurityGroups": [
                        {
                            "GroupName": "bastion",
                            "GroupId": "sg-07f6c3b7fa1b13047"
                        }
                    ],
                    "ClientToken": "d9c85c8a-77e8-4d34-85a7-f88e4eeaef7d",
                    "SubnetId": "subnet-05a5654029f8b9d47",
                    "InstanceType": "t2.micro",
                    "CapacityReservationSpecification": {
                        "CapacityReservationPreference": "open"
                    },
                    "NetworkInterfaces": [
                        {
                            "Status": "in-use",
                            "MacAddress": "0a:04:d4:6f:d0:fc",
                            "SourceDestCheck": true,
                            "VpcId": "vpc-0cea13535c6cc6163",
                            "Description": "",
                            "NetworkInterfaceId": "eni-071c29b77cece272c",
                            "PrivateIpAddresses": [
                                {
                                    "PrivateDnsName": "ip-172-16-100-69.ap-south-1.compute.internal",
                                    "PrivateIpAddress": "172.16.100.69",
                                    "Primary": true,
                                    "Association": {
                                        "PublicIp": "15.207.54.194",
                                        "PublicDnsName": "ec2-15-207-54-194.ap-south-1.compute.amazonaws.com",
                                        "IpOwnerId": "amazon"
                                    }
                                }
                            ],
                            "PrivateDnsName": "ip-172-16-100-69.ap-south-1.compute.internal",
                            "InterfaceType": "interface",
                            "Attachment": {
                                "Status": "attached",
                                "DeviceIndex": 0,
                                "DeleteOnTermination": true,
                                "AttachmentId": "eni-attach-087cba35852840465",
                                "AttachTime": "2022-12-16T19:07:28.000Z"
                            },
                            "Groups": [
                                {
                                    "GroupName": "bastion",
                                    "GroupId": "sg-07f6c3b7fa1b13047"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "OwnerId": "563053030797",
                            "PrivateIpAddress": "172.16.100.69",
                            "SubnetId": "subnet-05a5654029f8b9d47",
                            "Association": {
                                "PublicIp": "15.207.54.194",
                                "PublicDnsName": "ec2-15-207-54-194.ap-south-1.compute.amazonaws.com",
                                "IpOwnerId": "amazon"
                            }
                        }
                    ],
                    "SourceDestCheck": true,
                    "Placement": {
                        "Tenancy": "default",
                        "GroupName": "",
                        "AvailabilityZone": "ap-south-1b"
                    },
                    "Hypervisor": "xen",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "Status": "attached",
                                "DeleteOnTermination": true,
                                "VolumeId": "vol-0f16ea459aa3c3421",
                                "AttachTime": "2022-12-16T19:07:29.000Z"
                            }
                        }
                    ],
                    "Architecture": "x86_64",
                    "RootDeviceType": "ebs",
                    "RootDeviceName": "/dev/xvda",
                    "VirtualizationType": "hvm",
                    "Tags": [
                        {
                            "Value": "bastion",
                            "Key": "Name"
                        }
                    ],
                    "HibernationOptions": {
                        "Configured": false
                    },
                    "MetadataOptions": {
                        "State": "applied",
                        "HttpEndpoint": "enabled",
                        "HttpTokens": "optional",
                        "HttpPutResponseHopLimit": 1
                    },
                    "AmiLaunchIndex": 0
                }
            ],
            "ReservationId": "r-09de2755ca42c9179",
            "Groups": [],
            "OwnerId": "563053030797"
        }
    ]
}
```

Now let try to SSH to bastion server.

```
ssh -i aws.pem ec2-user@15.207.54.194
The authenticity of host '15.207.54.194 (15.207.54.194)' can't be established.
ECDSA key fingerprint is SHA256:1fOwuVCWrn88CVstTnj63NbUQFyI8tBf93oIEjxWGMs.
ECDSA key fingerprint is MD5:d0:b5:b2:12:41:93:78:fc:86:37:04:c2:58:37:ca:97.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '15.207.54.194' (ECDSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
19 package(s) needed for security, out of 38 available
Run "sudo yum update" to apply all updates.
[ec2-user@ip-172-16-100-69 ~]$ sudo hostnamectl set-hostname bastion.ap-south-1.compute.internal
[ec2-user@ip-172-16-100-69 ~]$ hostname
bastion.ap-south-1.compute.internal
```

Now, let's try to SSH from bastion server to frontend server using private IP from describe command after copying the keypair to bastion server.

```
sudo ssh -i aws.pem ec2-user@172.16.11.160

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
19 package(s) needed for security, out of 38 available
Run "sudo yum update" to apply all updates.
```

Now, lets try to SSH to backend server from bastion server

```
sudo ssh -i aws.pem ec2-user@172.16.131.89
The authenticity of host '172.16.131.89 (172.16.131.89)' can't be established.
ECDSA key fingerprint is SHA256:KuBbOmoqU1Q1Nb3vBczUluxM5q+vLXgIvu+iHcLYk3o.
ECDSA key fingerprint is MD5:f1:34:5e:59:e2:f2:50:a3:0b:09:43:b1:17:5c:a5:68.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.131.89' (ECDSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
19 package(s) needed for security, out of 38 available
Run "sudo yum update" to apply all updates.
[ec2-user@ip-172-16-131-89 ~]$ sudo hostnamectl set-hostname backend.ap-south-1.compute.internal
```


Since we are able to access the servers from the bastion server today’s experiment is a success
