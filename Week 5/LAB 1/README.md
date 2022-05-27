# Working with Elastic Load balancing

Tasks: Using AWS CLI,

1. Create an Application Load Balancer
2. Create a Network Load Balancer
3. Create a Gateway Load Balancer
4. Create a Classic Load Balancer
5. Perform clean up operations

NB: You need to launch at least two instances to do this.

Guide
https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/load-balancer-getting-started.html

## Notes

After launching test instances and creating a target group for them, next phase was to attempt to create an application load balancer with the CLI.

1. Create an Application Load Balancer

- To create a load balancer, use the create-load-balancer command to create a load balancer. You must specify two subnets that are not from the same Availability Zone.

> input

```bash
aws elbv2 create-load-balancer --name target-balancer  \
> --subnets subnet-0ee41c702f91849b9 subnet-0ce41ddcc188475ba --security-groups sg-07c1b7c674666c3c9
```

> Output

```bash
{
    "LoadBalancers": [
        {
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/app/target-balancer/d414143b9d915631",
            "DNSName": "target-balancer-744419818.us-east-2.elb.amazonaws.com",
            "CanonicalHostedZoneId": "Z3AADJGX6KTTL2",
            "CreatedTime": "2022-05-06T13:24:44+00:00",
            "LoadBalancerName": "target-balancer",
            "Scheme": "internet-facing",
            "VpcId": "vpc-0ebf076edb2fbcc5f",
            "State": {
                "Code": "provisioning"
            },
            "Type": "application",
            "AvailabilityZones": [
                {
                    "ZoneName": "us-east-2a",
                    "SubnetId": "subnet-0ce41ddcc188475ba",
                    "LoadBalancerAddresses": []
                },
                {
                    "ZoneName": "us-east-2b",
                    "SubnetId": "subnet-0ee41c702f91849b9",
                    "LoadBalancerAddresses": []
                }
            ],
            "SecurityGroups": [
                "sg-07c1b7c674666c3c9"
            ],
            "IpAddressType": "ipv4"
        }
    ]
}
```

- Use the `register-targets` command to register your instances with your target group:

  > Instances already registered throught the management console while creating target group.

- Use the `create-listener` command to create a listener for your load balancer with a default rule that forwards requests to your target group:

> input

```bash
aws elbv2 create-listener --load-balancer-arn arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/app/target-balancer/d414143b9d915631 \
> --protocol HTTP --port 80  \
> --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0f54d25b87a5fccb
```

> Output

```bash
{
    "Listeners": [
        {
            "ListenerArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:listener/app/target-balancer/d414143b9d915631/6683e1774b4dce47",
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/app/target-balancer/d414143b9d915631",
            "Port": 80,
            "Protocol": "HTTP",
            "DefaultActions": [
                {
                    "Type": "forward",
                    "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0f54d25b87a5fccb",
                    "ForwardConfig": {
                        "TargetGroups": [
                            {
                                "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0f54d25b87a5fccb",
                                "Weight": 1
                            }
                        ],
                        "TargetGroupStickinessConfig": {
                            "Enabled": false
                        }
                    }
                }
            ]
        }
```

- Add path-based routing
- Use the create-rule command to add a rule to your listener that forwards requests to the target group if the URL contains the specified pattern:

> input

```bash
aws elbv2 create-rule --listener-arn arn:aws:elasticloadbalancing:us-east-2:949303776906:listener/app/target-balancer/d414143b9d915631/6683e1774b4dce47 --priority 10 \
--conditions Field=path-pattern,Values='/img/*' \
--actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0f54d25b87a5fccb
```

> Output

```bash
      {
            "RuleArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:listener-rule/app/target-balancer/d414143b9d915631/6683e1774b4dce47/56766dfe43951e6d",
            "Priority": "10",
            "Conditions": [
                {
                    "Field": "path-pattern",
                    "Values": [
                        "/img/*"
                    ],
                    "PathPatternConfig": {
                        "Values": [
                            "/img/*"
                        ]
                    }
                }
            ],
            "Actions": [
                {
                    "Type": "forward",
                    "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0f54d25b87a5fccb",
                    "ForwardConfig": {
                        "TargetGroups": [
                            {
                                "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0f54d25b87a5fccb",
                                "Weight": 1
                            }
                        ],
                        "TargetGroupStickinessConfig": {
                            "Enabled": false
                        }
                    }
                }
:
```

- Delete your load balancer
  > input

```bash
aws elbv2 delete-load-balancer --load-balancer-arn arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/app/target-balancer/d414143b9d915631
aws elbv2 delete-target-group --target-group-arn arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0f54d25b87a5fccb
```

2. Create a Network Load Balancer

- Create Load Balancer

> input

```bash
aws elbv2 create-load-balancer --name schull-balancer --type network --subnets subnet-0ee41c702f91849b9 subnet-0ce41ddcc188475ba
```

> Output

```bash
{
    "LoadBalancers": [
        {
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/net/schull-balancer/6974552856c13104",
            "DNSName": "schull-balancer-6974552856c13104.elb.us-east-2.amazonaws.com",
            "CanonicalHostedZoneId": "ZLMOA37VPKANP",
            "CreatedTime": "2022-05-06T21:45:58.683000+00:00",
            "LoadBalancerName": "schull-balancer",
            "Scheme": "internet-facing",
            "VpcId": "vpc-0ebf076edb2fbcc5f",
            "State": {
                "Code": "provisioning"
            },
            "Type": "network",
            "AvailabilityZones": [
                {
                    "ZoneName": "us-east-2a",
                    "SubnetId": "subnet-0ce41ddcc188475ba",
                    "LoadBalancerAddresses": []
                },
                {
                    "ZoneName": "us-east-2b",
                    "SubnetId": "subnet-0ee41c702f91849b9",
                    "LoadBalancerAddresses": []
                }
            ],
            "IpAddressType": "ipv4"
        }
    ]
}
```

- Create Target group.
  > input

```bash
aws elbv2 create-target-group --name instance-target --protocol TCP --port 80 --vpc-id vpc-0ebf076edb2fbcc5f
```

> Output

```bash
{
    "TargetGroups": [
        {
            "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0ce3a8f8e0fbb683",
            "TargetGroupName": "instance-target",
            "Protocol": "TCP",
            "Port": 80,
            "VpcId": "vpc-0ebf076edb2fbcc5f",
            "HealthCheckProtocol": "TCP",
            "HealthCheckPort": "traffic-port",
            "HealthCheckEnabled": true,
            "HealthCheckIntervalSeconds": 30,
            "HealthCheckTimeoutSeconds": 10,
            "HealthyThresholdCount": 3,
            "UnhealthyThresholdCount": 3,
            "TargetType": "instance",
            "IpAddressType": "ipv4"
        }
    ]
}
```

- Register Targets

> input

```bash
aws elbv2 register-targets --target-group-arn arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0ce3a8f8e0fbb683 --targets Id=i-08af7c91b7b048dc4 Id=i-0a58f6fb453b7e421
```

- Use the create-listener command to create a listener for your load balancer with a default rule that forwards requests to your target group:

> input

```bash
aws elbv2 create-listener --load-balancer-arn arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/net/schull-balancer/6974552856c13104 --protocol TCP --port 80  \
--default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0ce3a8f8e0fbb683
```

> Output

```bash
{
    "Listeners": [
        {
            "ListenerArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:listener/net/schull-balancer/6974552856c13104/3468738ddfa021dd",
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/net/schull-balancer/6974552856c13104",
            "Port": 80,
            "Protocol": "TCP",
            "DefaultActions": [
                {
                    "Type": "forward",
                    "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0ce3a8f8e0fbb683",
                    "ForwardConfig": {
                        "TargetGroups": [
                            {
                                "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0ce3a8f8e0fbb683"
                            }
                        ]
                    }
                }
            ]
        }
    ]
}
```

- Delete your load balancer

```bash
aws elbv2 delete-load-balancer --load-balancer-arn arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/net/schull-balancer/6974552856c13104
aws elbv2 delete-target-group --target-group-arn arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/instance-target/0ce3a8f8e0fbb68
```

3. Create a Gateway Load Balancer

> input

```bash
aws elbv2 create-load-balancer --name gateway-balancer --type gateway --subnets subnet-0ce41ddcc188475ba subnet-0ee41c702f91849b9
```

> Output

```bash
{
    "LoadBalancers": [
        {
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/gwy/gateway-balancer/3e92af7d297d893a",
            "CreatedTime": "2022-05-15T15:16:17.826000+00:00",
            "LoadBalancerName": "gateway-balancer",
            "VpcId": "vpc-0ebf076edb2fbcc5f",
            "State": {
                "Code": "provisioning"
            },
            "Type": "gateway",
            "AvailabilityZones": [
                {
                    "ZoneName": "us-east-2b",
                    "SubnetId": "subnet-0ee41c702f91849b9"
                },
                {
                    "ZoneName": "us-east-2a",
                    "SubnetId": "subnet-0ce41ddcc188475ba"
                }
            ]
        }

```

- Create Target Group

> input

```bash
aws elbv2 create-target-group --name gateway-targets --protocol GENEVE --port 6081 --vpc-id vpc-0ebf076edb2fbcc5f
```

> Output

```bash
{
            "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/gateway-targets/0048bf29618d6066fc"
        }
    ]

```

Use the `register-targets` command to register your instances with your target group.

> input

```bash
aws elbv2 register-targets --target-group-arn arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/gateway-targets/0048bf29618d6066fc --targets Id=i-0eaf895353cc35802 Id=i-04bcf7cc181a21420
```

- Use the `create-listener` command to create a listener for your load balancer with a default rule that forwards requests to your target group.

> input

```bash
aws elbv2 create-listener --load-balancer-arn arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/gwy/gateway-balancer/3e92af7d297d893a --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/gateway-targets/0048bf29618d6066fc
```

> Output

```bash
{
    "Listeners": [
        {
            "ListenerArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:listener/gwy/gateway-balancer/3e92af7d297d893a/0b9b230da5e31b49",
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/gwy/gateway-balancer/3e92af7d297d893a",
            "DefaultActions": [
                {
                    "Type": "forward",
                    "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/gateway-targets/0048bf29618d6066fc",
                    "ForwardConfig": {
                        "TargetGroups": [
                            {
                                "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-2:949303776906:targetgroup/gateway-targets/0048bf29618d6066fc"
                            }
                        ]
                    }
                }
            ]
        }
    ]
}
```

- Create a Gateway Load Balancer endpoint

> input

```bash
aws ec2 create-vpc-endpoint-service-configuration --gateway-load-balancer-arns arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/gwy/gateway-balancer/3e92af7d297d893a --no-acceptance-required
```

> Output

```bash
{
    "ServiceConfiguration": {
        "ServiceType": [
            {
                "ServiceType": "GatewayLoadBalancer"
            }
        ],
        "ServiceId": "vpce-svc-01431857fc01a5569",
        "ServiceName": "com.amazonaws.vpce.us-east-2.vpce-svc-01431857fc01a5569",
        "ServiceState": "Available",
        "AvailabilityZones": [
            "us-east-2a",
            "us-east-2b"
        ],
        "AcceptanceRequired": false,
        "ManagesVpcEndpoints": false,
        "GatewayLoadBalancerArns": [
            "arn:aws:elasticloadbalancing:us-east-2:949303776906:loadbalancer/gwy/gateway-balancer/3e92af7d297d893a"
        ]
    }
}
```

- Use the modify-vpc-endpoint-service-permissions command to allow service consumers to create an endpoint to your service.

> input

```bash
aws ec2 modify-vpc-endpoint-service-permissions --service-id vpce-svc-01431857fc01a5569 --add-allowed-principals arn:aws:iam::949303776906:role/aws-elasticbeanstalk-ec2-role arn:aws:iam::949303776906:user/tega
```

- Use the `create-vpc-endpoint` command to create the Gateway Load Balancer endpoint for your service.

> input

Use consumers vpc id and sunnets

```bash
aws ec2 create-vpc-endpoint --vpc-endpoint-type GatewayLoadBalancer --service-name com.amazonaws.vpce.us-east-2.vpce-svc-01431857fc01a5569 --vpc-id vpc-0ebf076edb2fbcc5f --subnet-ids subnet-0ce41ddcc188475ba
```

> Output

```bash
{
    "VpcEndpoint": {
        "VpcEndpointId": "vpce-0ff717f5570ada51e",
        "VpcEndpointType": "GatewayLoadBalancer",
        "VpcId": "vpc-0ebf076edb2fbcc5f",
        "ServiceName": "com.amazonaws.vpce.us-east-2.vpce-svc-01431857fc01a5569",
        "State": "pending",
        "SubnetIds": [
            "subnet-0ce41ddcc188475ba"
        ],
        "RequesterManaged": false,
        "NetworkInterfaceIds": [
            "eni-047ed191b33aac71f"
        ],
        "CreationTimestamp": "2022-05-15T21:58:25.966000+00:00",
        "OwnerId": "949303776906"

```

- Configure routing

> input

```bash
aws ec2 create-route --route-table-id rtb-02a39741c3c59025c --destination-cidr-block 10.0.1.0/24 --vpc-endpoint-id vpce-0ff717f5570ada51e
```

- Use the `create-route` command to add an entry to the route table for the subnet with the application servers that routes all traffic from the application servers to the Gateway Load Balancer endpoint.

> input

```bash
aws ec2 create-route --route-table-id rtb-0174f660e834ae8a7 --destination-cidr-block 0.0.0.0/0 --vpc-endpoint-id vpce-0ff717f5570ada51e
```

- Use the create-route command to add an entry to the route table for the subnet with the Gateway Load Balancer endpoint that routes all traffic that originated from the application servers to the internet gateway.

> input

```bash
aws ec2 create-route --route-table-id rtb-0174f660e834ae8a7 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-01234567890abcdefe
```

- Repeat for each application subnet route table in each zone.

4. Create a Classic Load Balancer

- Set up instnces using port 80, HTTP
- Navigate t load balancers and create Classic Load Balancer.
- Define load balancer
- Add 2 subnets in different availability zones to increase Avilability of the load balancer.
- Assign security groups to your load balancer in a VPC
- Configure health checks for your EC2 instances
- Register EC2 instances with your load balancer
- Create and verify your load balancer
- Detele balancer.
