# Working with EBS

Tasks:

1. Create an Amazon EBS volume
2. Attach and mount your volume to an EC2 instance
3. Create a snapshot of your volume
4. Create a new volume from your snapshot
5. Attach and mount the new volume to your EC2 instance

Guide:

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes.html

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html

## Notes

1. Create an Amazon EBS volume

> input

```bash
aws ec2 create-volume \
    --volume-type io1 \
    --size 80 \
    --availability-zone us-east-1a
```

> output

```bash
{
    "AvailabilityZone": "us-east-2a",
    "CreateTime": "2022-04-23T22:06:48+00:00",
    "Encrypted": false,
    "Size": 80,
    "SnapshotId": "snap-0ffed374b724399b0",
    "State": "creating",
    "VolumeId": "vol-0796df260f4d6c315",
    "Iops": 240,
    "Tags": [],
    "VolumeType": "io1",
    "MultiAttachEnabled": false
}
```

2. Attach and mount your volume to an EC2 instance

After spinning an instance, input this cmd.

> input

```bash
aws ec2 attach-volume --volume-id vol-0e1567383d6874e27 --instance-id i-0c35bc0992b3001e8 --device /dev/sdf
```

> output

```bash
{
    "AttachTime": "2022-04-23T21:43:30.080000+00:00",
    "Device": "/dev/sdf",
    "InstanceId": "i-0c35bc0992b3001e8",
    "State": "attaching",
    "VolumeId": "vol-0e1567383d6874e27"
}
```

3. Create a snapshot of your volume

> input

```bash
aws ec2 create-snapshot --volume-id vol-0e1567383d6874e27 --description "This is my root volume ebs snapshot"
```

> output

```bash
{
    "Description": "This is my root volume ebs snapshot",
    "Encrypted": false,
    "OwnerId": "949303776906",
    "Progress": "",
    "SnapshotId": "snap-0ffed374b724399b0",
    "StartTime": "2022-04-23T21:48:52.550000+00:00",
    "State": "pending",
    "VolumeId": "vol-0e1567383d6874e27",
    "VolumeSize": 80,
    "Tags": []
}
```

4. Create a new volume from your snapshot

> input

```bash
aws ec2 create-volume \
    --volume-type gp2 \
    --iops 240 \
    --snapshot-id snap-0ffed374b724399b0 \
    --availability-zone us-east-1a
```

> output

```bash
{
    "AvailabilityZone": "us-east-2a",
    "CreateTime": "2022-04-23T22:06:48+00:00",
    "Encrypted": false,
    "Size": 80,
    "SnapshotId": "snap-0ffed374b724399b0",
    "State": "creating",
    "VolumeId": "vol-0796df260f4d6c315",
    "Iops": 240,
    "Tags": [],
    "VolumeType": "io1",
    "MultiAttachEnabled": false
}
```

5. Attach and mount the new volume to your EC2 instance

> input

```bash
aws ec2 attach-volume --volume-id vol-0796df260f4d6c315 --instance-id i-05d44f4014cc6724f --device /dev/sdf
```

> output

```bash
{
    "AttachTime": "2022-04-23T22:12:57.904000+00:00",
    "Device": "/dev/sdf",
    "InstanceId": "i-05d44f4014cc6724f",
    "State": "attaching",
    "VolumeId": "vol-0796df260f4d6c315"
}
```
