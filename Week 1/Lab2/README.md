# Task 2: Create Amazon S3 bucket with bucket policies and life cycle management

1. Launch AWS Console
2. Create a S3 bucket with enforced ownership
3. Create a single lifecycle policy
4. Create a single bucket policy
5. Delete all policies
6. Delete S3 bucket

## Notes:

To Create an S3 bucket with enforced ownership, ACLs (Access Control List), would have to be disabled.

1. Create S3 Bucket with enforced ownership

- Using the S3 Console, disable ACL while setting up the bucket.
- Using the AWS CLI:

> Code input

```
aws s3api create-bucket --bucket schull-test --region us-east-2 --create-bucket-configuration LocationConstraint=us-east-2 --object-ownership BucketOwnerEnforced
```

> Code output

```
{
    "Location": "http://schull-test.s3.amazonaws.com/"
}
```

2. Create a single lifecycle policy

A single lifecycle policy to move current versions of objects between storage classes was created.
The basis of the rule was to transition objects from standard storage class to Standard-IA storage class after 30 days.

3. Create a single bucket policy
   To create or edit a bucket policy, navigate to the edit bucket policy or select policy generator.

> Bucket policy generated

```
{
    "Version": "2012-10-17",
    "Id": "Policy1649947131245",
    "Statement": [
        {
            "Sid": "Stmt1649946540563",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:GetObjectVersion"
            ],
            "Resource": "arn:aws:s3:::schull-test/*"
        }
    ]
}
```

4. Policies and buckets crested were deleted through the AWS console.

For guide, kindly visit

https://docs.aws.amazon.com/cli/latest/reference/s3api/create-bucket.html

https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html

https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3-commands.html
