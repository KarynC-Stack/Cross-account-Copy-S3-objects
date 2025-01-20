# Copy S3 bucket objects from one account to another account

1. Configure AWS CLI
2. Create an S3 bucket (Source Bucket)
3. Create another S3 bucket (Target/Destination Bucket)
4. Attach the relevant policies
5. Copy Objects

**Install [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)**

**Configure AWS CLI**
```bash
aws configure
```

**Create one S3 bucket(source) in one account**
```
aws s3 mb s3://<<sourcebucket>>
```

**Attach the following bucket policy to the bucket in the console**
```json
{
    "Version": "2012-10-17",
    "Id": "Policy1611277539797",
    "Statement": [
        {
            "Sid": "S3Crossaccountaccess",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::************:user/London"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::mydestinationbucketproject/*",
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-acl": "bucket-owner-full-control"
                }
            }
        },
        {
            "Sid": "Allowlistbucket",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::************:user/London"
            },
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::mydestinationbucketproject"
        }
    ]
}

```
**Create a new user for the destination account**
```bash
aws iam create-user --user-name <username>
```
**Attach an in-line policy for the user in the console. Copy the below code below-Replace sourcebucket and destinationbucket**
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"s3:ListBucket",
				"s3:GetObject"
			],
			"Resource": [
				"arn:aws:s3:::arn:aws:s3:::<sourcebucket>",
				"arn:aws:s3:::arn:aws:s3:::<sourcebucket>/*"
			]
		},
		{
			"Effect": "Allow",
			"Action": [
				"s3:ListBucket",
				"s3:PutObject",
				"s3:PutObjectAcl"
			],
			"Resource": [
				"arn:aws:s3:::arn:aws:s3:::arn:aws:s3:::mydestinationbucket",
				"arn:aws:s3:::arn:aws:s3:::arn:aws:s3:::mydestinationbucket/*"
			]
		}
	]
}
```

**Create one S3 bucket(destination) in the new account**
```bash
aws s3 mb s3://<destinationbucket>
```

**Create access keys for the user**
```bash
aws iam create-access-key --user-name <username>
```

**Input both access and secret keys for the above username**

**Copy S3 objects from the source bucket into the destination bucket. Replace source bucket and destination bucket name**
```bash
aws s3 sync s3://<sourcebucket> s3://<mydestinationbucket> --acl bucket-owner-full-control
```

**Confirm objects have been successfully copied**
```bash
aws s3 ls <mydestinationbucket>
```
