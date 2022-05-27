Task : Create a dual Stack VPC and subnets using AWS CLI

Step:

1. Launch a non-default VPC
2. Create two subnets
3. Create an internet gateway to one of the subnets with route table
4. Configure an egress-only private subnet
5. Modify the IPv6 addressing behavior of the subnets
6. Lauch an instance into your public subnet
7. Launch an instance into your private subnet
8. Perform clean up operations.

### Notes

1. Launch a VPC
   In this lab, we would launch a VPC using the AWS CLI.

> Create a VPC:

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --amazon-provided-ipv6-cidr-block
```

> output

```bash
vpc-0b3ade90ef03311fb
```

- Describe your VPC to get the IPv6 CIDR block that's associated with the VPC.

```bash
aws ec2 describe-vpcs --vpc-id vpc-0b3ade90ef03311fb
```

> output

```bash
{
    "Vpcs": [
        {
            "CidrBlock": "10.0.0.0/16",
            "DhcpOptionsId": "dopt-00c763ea61f14474a",
            "State": "available",
            "VpcId": "vpc-0b3ade90ef03311fb",
            "OwnerId": "949303776906",
            "InstanceTenancy": "default",
            "Ipv6CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-0d82c2db4df90d9c7",
                    "Ipv6CidrBlock": "2600:1f16:993:5e00::/56",
                    "Ipv6CidrBlockState": {
                        "State": "associated"
                    },
                    "NetworkBorderGroup": "us-east-2",
                    "Ipv6Pool": "Amazon"
                }
            ],
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-08e0fa7c4c2f5b382",
                    "CidrBlock": "10.0.0.0/16",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "IsDefault": false
        }
    ]
}
```

2. Create 2 Subnets

- Create a subnet with a `10.0.0.0/24` IPv4 CIDR block and a `2600:1f16:993:5e00::/64` IPv6 CIDR block (from the ranges that were returned in the previous step).

> Create subnet 1:

```bash
aws ec2 create-subnet --vpc-id vpc-0b3ade90ef03311fb --cidr-block 10.0.0.0/24 --ipv6-cidr-block 2600:1f16:993:5e00::/64
```

Subnet-1 (public) ID: `"SubnetId": "subnet-0677f445a3aa0be68"`

> Create subnet 2:

```bash
aws ec2 create-subnet --vpc-id vpc-0b3ade90ef03311fb --cidr-block 10.0.1.0/24 --ipv6-cidr-block 2600:1f16:993:5e00::/64
```

Subnet-2 (private) ID: "SubnetId": `"subnet-0fc6396c3d00696e9"`

3. Create an internet gateway to one of the subnets with route table

- Create Internet Gateway:

> input

```bash
aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
```

> Output

```bash
igw-07e92a48ba0997373
```

- Attach IGW to VPC:

```bash
aws ec2 attach-internet-gateway --vpc-id vpc-0b3ade90ef03311fb --internet-gateway-id igw-07e92a48ba0997373
```

- Create a custom route table for your VPC

> input

```bash
aws ec2 create-route-table
--vpc-id vpc-0b3ade90ef03311fb
--query RouteTable.RouteTableId
--output text
```

> output

```bash
rtb-0278cf1cd206f1dca
```

- Create a route in the route table that points all IPv6 traffic `(::/0)` to the internet gateway

> input

```bash
aws ec2 create-route --route-table-id rtb-0278cf1cd206f1dca --destination-ipv6-cidr-block ::/0 --gateway-id igw-07e92a48ba0997373
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

aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-0b3ade90ef03311fb" --query "Subnets[*].{ID:SubnetId,IPv4CIDR:CidrBlock,IPv6CIDR:Ipv6CidrBlockAssociationSet[*].Ipv6CidrBlock}"
```

> Output

```bash
[
    {
        "ID": "subnet-0677f445a3aa0be68",
        "IPv4CIDR": "10.0.0.0/24",
        "IPv6CIDR": [
            "2600:1f16:993:5e00::/64"
        ]
    },
    {
        "ID": "subnet-0fc6396c3d00696e9",
        "IPv4CIDR": "10.0.1.0/24",
        "IPv6CIDR": [
            "2600:1f16:993:5e01::/64"
        ]
    }
]
```

- Choose which subnet to associate with the custom route table

> input

```bash
aws ec2 associate-route-table  --subnet-id subnet-0677f445a3aa0be68 --route-table-id rtb-0278cf1cd206f1dca
```

> Output

```bash
{
    "AssociationId": "rtbassoc-0741f3adef6b32372",
    "AssociationState": {
        "State": "associated"
    }
}
[c
```

4. Configure an egress-only private subnet

You can configure the second subnet in your VPC to be an IPv6 egress-only private subnet. Instances that are launched in this subnet are able to access the internet over IPv6 (for example, to get software updates) through an egress-only internet gateway, but hosts on the internet cannot reach your instances.

- Create an egress-only internet gateway for your VPC. In the output that's returned, take note of the gateway ID.

> input

```bash
aws ec2 create-egress-only-internet-gateway --vpc-id vpc-0b3ade90ef03311fb
```

> output

```bash
{
    "EgressOnlyInternetGateway": {
        "Attachments": [
            {
                "State": "attached",
                "VpcId": "vpc-0b3ade90ef03311fb"
            }
        ],
        "EgressOnlyInternetGatewayId": "eigw-0fd6c818f45e45b0d"
    }
}
```

- Create a custom route table for your VPC

> input

```bash
aws ec2 create-route-table --vpc-id vpc-0b3ade90ef03311fb
```

> output

```bash
rtb-0f0a9912c3379c6a3
```

- Create a route in the route table that points all IPv6 traffic `(::/0)` to the egress-only Internet gateway.

> input

```bash
aws ec2 create-route --route-table-id rtb-0f0a9912c3379c6a3 --destination-ipv6-cidr-block ::/0 --egress-only-internet-gateway-id eigw-0fd6c818f45e45b0d
```

> Output

```bash
{
    "Return": true
}
```

- Associate the route table with the second subnet in your VPC

> input

```bash
aws ec2 associate-route-table --subnet-id subnet-0fc6396c3d00696e9 --route-table-id rtb-0f0a9912c3379c6a3
```

> Output

```bash
{
    "AssociationId": "rtbassoc-042e69a8ee10b23b8",
    "AssociationState": {
        "State": "associated"
    }
}
```

5. Modify the IPv6 addressing behavior of the subnets

> Subnet 1

```bash
aws ec2 modify-subnet-attribute --subnet-id subnet-0677f445a3aa0be68 --assign-ipv6-address-on-creation
```

> Subnet 2

```bash
aws ec2 modify-subnet-attribute --subnet-id subnet-0fc6396c3d00696e9 --assign-ipv6-address-on-creation
```

6. Lauch an instance into your public subnet

- Use the following command to set the permissions of your private key file so that only you can read it.

```bash
chmod 400 schullpair.pem
```

- Create a security group in your VPC

> input

```bash
aws ec2 create-security-group --group-name schullaccess --description "Security group for Schull SSH access" --vpc-id vpc-0b3ade90ef03311fb
```

> output

```bash
{
    "GroupId": "sg-01d1a1c024fac001d"
}
```

- Add a rule that allows SSH access

```bash
aws ec2 authorize-security-group-ingress --group-id sg-01d1a1c024fac001d --ip-permissions '[{"IpProtocol": "tcp", "FromPort": 22, "ToPort": 22, "Ipv6Ranges": [{"CidrIpv6": "::/0"}]}]'
```

> output

```bash
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-09126c5901c8e8558",
            "GroupId": "sg-01d1a1c024fac001d",
            "GroupOwnerId": "949303776906",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv6": "::/0"
        }
    ]
}
```

- Launch a Linux instance into your public subnet, using the security group and key pair you've created.

> input

```bash
aws ec2 run-instances
--image-id ami-0d718c3d715cec4a7
--count 1
--instance-type t2.micro
--key-name schullpair
--security-group-id sg-01d1a1c024fac001d
--subnet-id subnet-0677f445a3aa0be68
```

> Partial output

```bash
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0d718c3d715cec4a7",
            "InstanceId": "i-02b52c3133c7bd1fa",
            "InstanceType": "t2.micro",
            "KeyName": "schullpair",
            "LaunchTime": "2022-04-25T17:31:26+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-2c",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-10-0-0-211.us-east-2.compute.internal",
            "PrivateIpAddress": "10.0.0.211",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-0677f445a3aa0be68",
            "VpcId": "vpc-0b3ade90ef03311fb",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "7c529316-39f4-47a4-9bec-68ca8bc0cf2f",
            "EbsOptimized": false,
            "EnaSupport": true,

```

- Query state of the instance:
  > input

```bash
aws ec2 describe-instances --instance-id i-02b52c3133c7bd1fa --query "Reservations[*].Instances[*].{State:State.Name,Address:PrivateIpAddress,Ipv6Addresses:Ipv6Address}"
```

> output

```bash
    [
        {
            "State": "running",
            "Address": "10.0.0.211",
            "Ipv6Addresses": "2600:1f16:993:5e00:9567:d447:9ac:84a9"
        }
    ]
]
```

- SSH into the instance:
  > input

```bash
ssh -i "schullpair.pem" ec2-user@2600:1f16:993:5e00:9567:d447:9ac:84a9
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

7. Launch an instance into your private subnet

- Create a security group in your VPC using the `create-security-group` command.

> input

```bash
aws ec2 create-security-group
--group-name schullRestrictedaccess
--description "Security group for Schull SSH access from bastion"
--vpc-id vpc-0b3ade90ef03311fb
```

> output

```bash
{
    "GroupId": "sg-083698cfe2ee7b31f"
}
```

- Add a rule that allows inbound SSH access from the IPv6 address of the instance in your public subnet, and a rule that allows all ICMPv6 traffic using the authorize-security-group-ingress command

> Rule 1

```bash
aws ec2 authorize-security-group-ingress --group-id sg-083698cfe2ee7b31f --ip-permissions '[{"IpProtocol": "tcp", "FromPort": 22, "ToPort": 22, "Ipv6Ranges": [{"CidrIpv6": "2001:db8:1234:1a00::123/128"}]}]'
```

> output

```bash
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0ff016f560e315fe8",
            "GroupId": "sg-083698cfe2ee7b31f",
            "GroupOwnerId": "949303776906",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv6": "2001:db8:1234:1a00::123/128"
        }
    ]
}
```

> Rule 2

```bash
aws ec2 authorize-security-group-ingress --group-id sg-083698cfe2ee7b31f --ip-permissions '[{"IpProtocol": "58", "FromPort": -1, "ToPort": -1, "Ipv6Ranges": [{"CidrIpv6": "::/0"}]}]'
```

> output

```bash
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0b58ba47724bb887e",
            "GroupId": "sg-083698cfe2ee7b31f",
            "GroupOwnerId": "949303776906",
            "IsEgress": false,
            "IpProtocol": "icmpv6",
            "FromPort": -1,
            "ToPort": -1,
            "CidrIpv6": "::/0"
        }
    ]
}
```

- Launch an instance into your private subnet, using the security group you've created and the same key pair you used to launch the instance in the public subnet.

> input

```bash
aws ec2 run-instances --image-id ami-0ab0af576059bb208 --count 1 --instance-type t2.micro --key-name schullpair --security-group-ids sg-083698cfe2ee7b31f --subnet-id subnet-0fc6396c3d00696e9
```

> output

```bash
{
    "GroupId": "sg-083698cfe2ee7b31f"
}
```

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-commands-example.html

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-commands-example-ipv6.html

https://docs.aws.amazon.com/vpc/index.html
