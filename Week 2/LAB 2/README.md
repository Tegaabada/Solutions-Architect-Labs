Task: Create a nondefault VPC using AWS CLI

1. Open your CLI and run a configuration setting
2. Lauch a VPC
3. Create two subnets (a public and a private subnet)
4. Make your subnet public by creating and attaching an internet gateway
5. Create a security group and add a SSH access from anywhere
6. Lauch an instance into your subnet
7. Clean Up

### Notes

1. Launch a VPC
   In this lab, we would launch a VPC using the AWS CLI.

> Create a VPC:

```bash
aws ec2 create-vpc
--cidr-block 10.0.0.0/16
--query Vpc.VpcId
--output text
```

> output

```bash
vpc-05512707749f3efa0
```

2. Create 2 Subnets

> Create subnet 1:

```bash
 aws ec2 create-subnet
 --vpc-id vpc-05512707749f3efa0
 --cidr-block 10.0.0.0/24
```

Subnet 1 ID: `"SubnetId": "subnet-03a89f2146264eb49"`

> Create subnet 2:

```bash
 aws ec2 create-subnet
 --vpc-id vpc-05512707749f3efa0
 --cidr-block 10.0.1.0/24
```

Subnet 2 ID: "SubnetId": `"subnet-060fa3527b7929104"`

3. Make one subnet public by creating and attaching an internet gateway

- Create Internet Gateway:

> input

```bash
aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
```

> Output

```bash
igw-0ce9fb9a7d0fb4b75
```

- Attach IGW to VPC:

```bash
aws ec2 attach-internet-gateway --vpc-id vpc-05512707749f3efa0 --internet-gateway-id igw-0ce9fb9a7d0fb4b75
```

- Create a custom route table for your VPC

> input

```bash
aws ec2 create-route-table
--vpc-id vpc-05512707749f3efa0
--query RouteTable.RouteTableId
--output text
```

> output

```bash
rtb-0320fa00099b5c9e0
```

- Create a route in the route table that points all traffic `(0.0.0.0/0)` to the internet gateway:

> input

```bash
aws ec2 create-route
--route-table-id rtb-0320fa00099b5c9e0
--destination-cidr-block 0.0.0.0/0
--gateway-id igw-0ce9fb9a7d0fb4b75
```

> Output

```bash
{
    "Return": true
}
```

- Associate route table with a subnet:
  To Associate route table with a subnet, query the subnet ID by filtering the subnets associated with the custom VPC

> input

```bash

aws ec2 describe-subnets
--filters "Name=vpc-id,Values=vpc-05512707749f3efa0" --query "Subnets[*].{ID:SubnetId,CIDR:CidrBlock}"
```

> Output

```bash
[
    {
        "ID": "subnet-060fa3527b7929104",
        "CIDR": "10.0.0.0/24"
    },
    {
        "ID": "subnet-03a89f2146264eb49",
        "CIDR": "10.0.1.0/24"
    }
]
```

- Choose which subnet to associate with the custom route table

> input

```bash
aws ec2 associate-route-table  --subnet-id subnet-060fa3527b7929104 --route-table-id rtb-0320fa00099b5c9e0
```

> Output

```bash
{
    "AssociationId": "rtbassoc-09d09dd7f04d0b7d9",
    "AssociationState": {
        "State": "associated"
    }
}
```

- Modify the public IP addressing behavior of the subnet so that an instance launched into the subnet automatically receives a public IP address:

> input

```bash
aws ec2 modify-subnet-attribute
--subnet-id subnet-060fa3527b7929104
--map-public-ip-on-launch
```

4. Create a Security Group and add a SSH access from anywhere

- Create a security group in your VPC

> input

```bash
aws ec2 create-security-group
--group-name schullaccess
--description "Security group for Schull SSH access"
--vpc-id vpc-05512707749f3efa0
```

> output

```bash
{
    "GroupId": "sg-0d9458d0faec3e152"
}
```

- Add a rule that allows SSH access

```bash
aws ec2 authorize-security-group-ingress
--group-id sg-0d9458d0faec3e152
--protocol tcp --port 22 --cidr 0.0.0.0/0
```

> output

```bash
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-00d6f479d906d9ee3",
            "GroupId": "sg-0d9458d0faec3e152",
            "GroupOwnerId": "949303776906",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

- Use the following command to set the permissions of your private key file so that only you can read it.

```bash
chmod 400 schullpair.pem
```

5. Launch a Linux instance into your public subnet, using the security group and key pair you've created.

> input

```bash
aws ec2 run-instances
--image-id ami-0d718c3d715cec4a7
--count 1
--instance-type t2.micro
--key-name schullpair
--security-group-id sg-0d9458d0faec3e152
--subnet-id subnet-060fa3527b7929104
```

> Partial output

```bash
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0d718c3d715cec4a7",
            "InstanceId": "i-042335d2dd3065f8a",
            "InstanceType": "t2.micro",
            "KeyName": "schullpair",
            "LaunchTime": "2022-04-19T17:50:27+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-2b",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-10-0-0-158.us-east-2.compute.internal",
            "PrivateIpAddress": "10.0.0.158",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },

```

- Query state of the instance:
  > input

```bash
aws ec2 describe-instances
--instance-id i-042335d2dd3065f8a
--query "Reservations[*].Instances[*].{State:State.Name,Address:PublicIpAddress}"
```

> output

```bash
    [
        {
            "State": "running",
            "Address": "18.224.169.155"
        }
    ]
]
```

- SSH into the instance:
  > input

```bash
ssh -i "schullpair.pem" ec2-user@18.224.169.155
```

> Output

```bash
The authenticity of host '18.224.169.155 (18.224.169.155)' can't be established.
ECDSA key fingerprint is SHA256:beuy+0fdlUrZij14KoUAjoHTDX5TgYKoemChjrJygbg.
ECDSA key fingerprint is MD5:05:aa:2c:90:8f:18:d0:a0:d3:19:e3:d6:14:0d:a0:c9.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '18.224.169.155' (ECDSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
19 package(s) needed for security, out of 38 available
Run "sudo yum update" to apply all updates.
```

6. Clean up

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-commands-example.html

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-commands-example-ipv6.html

https://docs.aws.amazon.com/vpc/index.html
