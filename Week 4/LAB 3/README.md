# Working with placement Groups using AWS CLI

Tasks:

1. Create a placement group
2. Tag a placement group
3. Launch instances in a placement group
4. Describe instances in a placement group
5. Change the placement group for an instance
6. Delete a placement group

Guide:
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html#using-placement-groups

## Notes

1. Create a placement group

> input

```bash
aws ec2 create-placement-group --group-name my-cluster --strategy cluster --tag-specifications 'ResourceType=placement-group,Tags={Key=purpose,Value=production}'
```

> Output

```bash
{
    "PlacementGroup": {
        "GroupName": "my-cluster",
        "State": "available",
        "Strategy": "cluster",
        "GroupId": "pg-022a240f88abe4532",
        "Tags": [
            {
                "Key": "purpose",
                "Value": "production"
            }
        ],
        "GroupArn": "arn:aws:ec2:us-east-2:949303776906:placement-group/my-cluster"
    }
}
```

2. Tag a placement group

> input

```bash
aws ec2 describe-tags \
    --filters Name=resource-type,Values=placement-group
```

> Output

```bash
{
    "Tags": [
        {
            "Key": "purpose",
            "ResourceId": "pg-01c1b61d1d73bece6",
            "ResourceType": "placement-group",
            "Value": "production"
        },
        {
            "Key": "purpose",
            "ResourceId": "pg-022a240f88abe4532",
            "ResourceType": "placement-group",
            "Value": "production"
        },
        {
            "Key": "purpose",
            "ResourceId": "pg-0d25db08bf8ce38ea",
            "ResourceType": "placement-group",
            "Value": "production"
        }
    ]
}
```

- Use the describe-placement-groups command to view the configuration of the specified placement group, which includes any tags that were specified for the placement group.

> input

```bash
aws ec2 describe-tags \
    --filters Name=resource-id,Values=pg-01c1b61d1d73bece6
```

> Output

```bash
{
    "Tags": [
        {
            "Key": "purpose",
            "ResourceId": "pg-01c1b61d1d73bece6",
            "ResourceType": "placement-group",
            "Value": "production"
        }
    ]
}
```

3. Launch instances in a placement group

> input

```bash
aws ec2 run-instances --placement "GroupName = my-cluster"
```

> Output

```bash
{
    "PlacementGroups": [
        {
            "GroupName": "my-cluster",
            "State": "available",
            "Strategy": "cluster",
            "GroupId": "pg-022a240f88abe4532",
            "Tags": [
                {
                    "Key": "purpose",
                    "Value": "production"
                }
            ],
            "GroupArn": "arn:aws:ec2:us-east-2:949303776906:placement-group/my-cluster"
        }
    ]
}
```

T2 micro cannot be used for cluster instances.
