# Working with EC2 instance using the AWS CLI / Programmable access

Tasks:

1. Using your default vpc, find the public subnet
2. Create a security group
3. Launch an instance with a web server with termination protection enabled
4. Monitor Your EC2 instance; view the types of metrics that are collected for an EC2 instance
5. Modify the security group that your web server is using to allow HTTP access
6. Resize your Amazon EC2 instance to scale

Guide
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html

## Notes

1. Using default VPC, create a security group

> input

```bash
aws ec2 create-security-group
--group-name schullaccess
--description "Security group for Schull SSH access"
--vpc-id vpc-0ebf076edb2fbcc5f
```

> output

```bash
{
    "GroupId": "sg-0395cd9e64df97adc"
}
```

- Add a rule that allows SSH access

```bash
aws ec2 authorize-security-group-ingress
--group-id sg-0395cd9e64df97adc
--protocol tcp --port 22 --cidr 0.0.0.0/0
```

> output

```bash
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0685c75d427a7d487",
            "GroupId": "sg-0395cd9e64df97adc",
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

2. Launch an instance with a web server with termination protection enabled

An instance with a SQL server was launched

> input

```bash
aws ec2 run-instances --image-id ami-0d718c3d715cec4a7 --count 1 --instance-type t2.micro --key-name schullpair --security-group-id sg-0395cd9e64df97adc --subnet-id subnet-05acf97595c528380
```

> Partial output

```bash
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0d718c3d715cec4a7",
            "InstanceId": "i-09964aef96f27a136",
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

- SSH into the instance and install Web Server
  > input

```bash
ssh -i "schullpair.pem" ec2-user@ec2-3-21-35-95.us-east-2.compute.amazonaws.com
```

> Output:

```bash
 ec2-18-188-211-193.us-east-2.compute.amazonaws.com

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2
```

- Install Apache server

> input:

```bash
sudo yum update -y
sudo yum install -y httpd
```

- Start server

> input

```
sudo systemctl start httpd

```

- Enable Termination protection of the instance

> input:

```bash
aws ec2 modify-instance-attribute \
  --instance-id i-09964aef96f27a136 \
  --block-device-mappings "[{\"DeviceName\": \"/dev/sda1\",\"Ebs\":{\"DeleteOnTermination\":false}}]"
```
