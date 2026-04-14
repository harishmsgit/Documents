# AWS CLI Operations Reference — Solutions Architect (Production-Ready)

> **Audience:** AWS Solutions Architect  
> **CLI Version:** AWS CLI v2  
> **Scope:** Full lifecycle management of core AWS services

---

## Prerequisites — AWS CLI Setup

Install AWS CLI v2 (Linux / WSL):
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

Configure credentials:
```bash
aws configure
# AWS Access Key ID:
# AWS Secret Access Key:
# Default region: ap-south-1
# Default output format: json
```

Configure a named profile:
```bash
aws configure --profile production
aws configure --profile staging
```

Use a specific profile for a command:
```bash
aws s3 ls --profile production
export AWS_PROFILE=production
```

Verify identity:
```bash
aws sts get-caller-identity
```

List configured profiles:
```bash
aws configure list-profiles
aws configure list --profile production
```

---

## 1. IAM — Identity and Access Management

### Users
```bash
# List all IAM users
aws iam list-users

# Create a user
aws iam create-user --user-name devops-user

# Delete a user
aws iam delete-user --user-name devops-user

# Create access key for a user
aws iam create-access-key --user-name devops-user

# List access keys for a user
aws iam list-access-keys --user-name devops-user

# Deactivate an access key
aws iam update-access-key --user-name devops-user --access-key-id <key-id> --status Inactive

# Delete an access key
aws iam delete-access-key --user-name devops-user --access-key-id <key-id>
```

### Groups
```bash
# List groups
aws iam list-groups

# Create a group
aws iam create-group --group-name DevOpsTeam

# Add user to group
aws iam add-user-to-group --user-name devops-user --group-name DevOpsTeam

# Remove user from group
aws iam remove-user-from-group --user-name devops-user --group-name DevOpsTeam
```

### Policies
```bash
# List all managed policies
aws iam list-policies --scope AWS

# Attach a managed policy to a user
aws iam attach-user-policy \
  --user-name devops-user \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# Attach managed policy to a group
aws iam attach-group-policy \
  --group-name DevOpsTeam \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess

# Detach policy from user
aws iam detach-user-policy \
  --user-name devops-user \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# Create a custom inline policy
aws iam put-user-policy \
  --user-name devops-user \
  --policy-name S3ReadOnly \
  --policy-document file://s3-policy.json
```

### Roles
```bash
# List roles
aws iam list-roles

# Create a role (e.g., for EC2)
aws iam create-role \
  --role-name EC2-S3-Role \
  --assume-role-policy-document file://trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
  --role-name EC2-S3-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Get role details
aws iam get-role --role-name EC2-S3-Role

# Create instance profile
aws iam create-instance-profile --instance-profile-name EC2-S3-Profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-Profile \
  --role-name EC2-S3-Role

# Delete a role
aws iam delete-role --role-name EC2-S3-Role
```

### Password Policy
```bash
# Set account password policy
aws iam update-account-password-policy \
  --minimum-password-length 12 \
  --require-symbols \
  --require-numbers \
  --require-uppercase-characters \
  --require-lowercase-characters \
  --max-password-age 90 \
  --password-reuse-prevention 5

# Get current password policy
aws iam get-account-password-policy
```

---

## 2. EC2 — Elastic Compute Cloud

### Key Pairs
```bash
# Create a key pair
aws ec2 create-key-pair --key-name my-key --query 'KeyMaterial' --output text > my-key.pem
chmod 400 my-key.pem

# List key pairs
aws ec2 describe-key-pairs

# Delete a key pair
aws ec2 delete-key-pair --key-name my-key
```

### Instances
```bash
# Launch an EC2 instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.medium \
  --key-name my-key \
  --security-group-ids sg-xxxxxxxx \
  --subnet-id subnet-xxxxxxxx \
  --iam-instance-profile Name=EC2-S3-Profile \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=prod-server},{Key=Env,Value=production}]' \
  --count 1

# List running instances
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' \
  --output table

# Get instance by name tag
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=prod-server" \
  --query 'Reservations[*].Instances[*].[InstanceId,PublicIpAddress]' \
  --output table

# Start / Stop / Reboot / Terminate
aws ec2 start-instances --instance-ids i-xxxxxxxxxxxxxxxxx
aws ec2 stop-instances --instance-ids i-xxxxxxxxxxxxxxxxx
aws ec2 reboot-instances --instance-ids i-xxxxxxxxxxxxxxxxx
aws ec2 terminate-instances --instance-ids i-xxxxxxxxxxxxxxxxx

# Get instance system log
aws ec2 get-console-output --instance-id i-xxxxxxxxxxxxxxxxx

# Describe instance types
aws ec2 describe-instance-types --instance-types t3.micro t3.medium t3.large
```

### AMIs
```bash
# List your own AMIs
aws ec2 describe-images --owners self --output table

# Create an AMI from an instance
aws ec2 create-image \
  --instance-id i-xxxxxxxxxxxxxxxxx \
  --name "prod-server-ami-$(date +%Y%m%d)" \
  --no-reboot

# Copy AMI to another region
aws ec2 copy-image \
  --source-image-id ami-xxxxxxxxxxxxxxxxx \
  --source-region us-east-1 \
  --region ap-south-1 \
  --name "prod-server-ami-copy"

# Deregister an AMI
aws ec2 deregister-image --image-id ami-xxxxxxxxxxxxxxxxx
```

### Security Groups
```bash
# List security groups
aws ec2 describe-security-groups \
  --query 'SecurityGroups[*].[GroupId,GroupName,Description]' --output table

# Create a security group
aws ec2 create-security-group \
  --group-name prod-sg \
  --description "Production Security Group" \
  --vpc-id vpc-xxxxxxxxx

# Add inbound rule (SSH from specific IP)
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp --port 22 --cidr 10.0.0.0/16

# Add inbound rule (HTTP from anywhere)
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp --port 80 --cidr 0.0.0.0/0

# Add inbound rule (HTTPS)
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# Remove inbound rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp --port 22 --cidr 10.0.0.0/16

# Delete a security group
aws ec2 delete-security-group --group-id sg-xxxxxxxxx
```

### Elastic IPs
```bash
# Allocate an Elastic IP
aws ec2 allocate-address --domain vpc

# Associate EIP with an instance
aws ec2 associate-address \
  --instance-id i-xxxxxxxxxxxxxxxxx \
  --allocation-id eipalloc-xxxxxxxxxxxxxxxxx

# Disassociate EIP
aws ec2 disassociate-address --association-id eipassoc-xxxxxxxxxxxxxxxxx

# Release EIP
aws ec2 release-address --allocation-id eipalloc-xxxxxxxxxxxxxxxxx

# List all EIPs
aws ec2 describe-addresses
```

### EBS Volumes
```bash
# List volumes
aws ec2 describe-volumes --output table

# Create a volume
aws ec2 create-volume \
  --volume-type gp3 \
  --size 100 \
  --availability-zone ap-south-1a \
  --encrypted

# Attach a volume to an instance
aws ec2 attach-volume \
  --volume-id vol-xxxxxxxxxxxxxxxxx \
  --instance-id i-xxxxxxxxxxxxxxxxx \
  --device /dev/xvdf

# Detach a volume
aws ec2 detach-volume --volume-id vol-xxxxxxxxxxxxxxxxx

# Create a snapshot
aws ec2 create-snapshot \
  --volume-id vol-xxxxxxxxxxxxxxxxx \
  --description "prod-vol-backup-$(date +%Y%m%d)"

# List snapshots
aws ec2 describe-snapshots --owner-ids self --output table

# Delete a snapshot
aws ec2 delete-snapshot --snapshot-id snap-xxxxxxxxxxxxxxxxx

# Delete a volume
aws ec2 delete-volume --volume-id vol-xxxxxxxxxxxxxxxxx
```

---

## 3. VPC — Virtual Private Cloud

```bash
# List all VPCs
aws ec2 describe-vpcs --output table

# Create a VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=prod-vpc}]'

# Enable DNS hostnames
aws ec2 modify-vpc-attribute --vpc-id vpc-xxxxxxxxx --enable-dns-hostnames

# Create subnets
aws ec2 create-subnet \
  --vpc-id vpc-xxxxxxxxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone ap-south-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=prod-public-1a}]'

aws ec2 create-subnet \
  --vpc-id vpc-xxxxxxxxx \
  --cidr-block 10.0.2.0/24 \
  --availability-zone ap-south-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=prod-private-1b}]'

# Enable auto-assign public IP on subnet
aws ec2 modify-subnet-attribute \
  --subnet-id subnet-xxxxxxxxx \
  --map-public-ip-on-launch

# Create Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=prod-igw}]'

# Attach IGW to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-xxxxxxxxx \
  --vpc-id vpc-xxxxxxxxx

# Create Route Table
aws ec2 create-route-table --vpc-id vpc-xxxxxxxxx \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=prod-public-rt}]'

# Add route to Internet via IGW
aws ec2 create-route \
  --route-table-id rtb-xxxxxxxxx \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-xxxxxxxxx

# Associate route table with subnet
aws ec2 associate-route-table \
  --route-table-id rtb-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx

# Create NAT Gateway (for private subnets)
aws ec2 create-nat-gateway \
  --subnet-id subnet-xxxxxxxxx \
  --allocation-id eipalloc-xxxxxxxxx \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=prod-nat}]'

# Create VPC Peering
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-xxxxxxxxx \
  --peer-vpc-id vpc-yyyyyyyyy

# Accept VPC Peering
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-xxxxxxxxx

# Create VPC Endpoint for S3
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxxxxxx \
  --service-name com.amazonaws.ap-south-1.s3 \
  --route-table-ids rtb-xxxxxxxxx

# Delete a VPC
aws ec2 delete-vpc --vpc-id vpc-xxxxxxxxx
```

---

## 4. S3 — Simple Storage Service

### Buckets
```bash
# List all buckets
aws s3 ls
aws s3api list-buckets --query 'Buckets[*].[Name,CreationDate]' --output table

# Create a bucket
aws s3 mb s3://my-prod-bucket-2026 --region ap-south-1

# Create bucket with versioning enabled
aws s3api create-bucket \
  --bucket my-prod-bucket-2026 \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1

aws s3api put-bucket-versioning \
  --bucket my-prod-bucket-2026 \
  --versioning-configuration Status=Enabled

# Enable server-side encryption (AES-256)
aws s3api put-bucket-encryption \
  --bucket my-prod-bucket-2026 \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'

# Block all public access (production default)
aws s3api put-public-access-block \
  --bucket my-prod-bucket-2026 \
  --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# Enable lifecycle rules (expire objects after 90 days)
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-prod-bucket-2026 \
  --lifecycle-configuration file://lifecycle.json

# Delete a bucket (must be empty)
aws s3 rb s3://my-prod-bucket-2026

# Force delete a bucket with all contents
aws s3 rb s3://my-prod-bucket-2026 --force
```

### Objects
```bash
# List objects in a bucket
aws s3 ls s3://my-prod-bucket-2026/
aws s3 ls s3://my-prod-bucket-2026/ --recursive --human-readable --summarize

# Upload a file
aws s3 cp ./app.jar s3://my-prod-bucket-2026/releases/app.jar

# Upload a directory
aws s3 cp ./dist/ s3://my-prod-bucket-2026/static/ --recursive

# Download a file
aws s3 cp s3://my-prod-bucket-2026/releases/app.jar ./app.jar

# Sync a directory (only uploads changed files)
aws s3 sync ./dist/ s3://my-prod-bucket-2026/static/ --delete

# Move an object
aws s3 mv s3://my-prod-bucket-2026/old-path/file.txt s3://my-prod-bucket-2026/new-path/file.txt

# Delete an object
aws s3 rm s3://my-prod-bucket-2026/releases/app.jar

# Delete all objects in a prefix
aws s3 rm s3://my-prod-bucket-2026/releases/ --recursive

# Generate a presigned URL (valid 1 hour)
aws s3 presign s3://my-prod-bucket-2026/releases/app.jar --expires-in 3600

# Get object metadata
aws s3api head-object --bucket my-prod-bucket-2026 --key releases/app.jar
```

---

## 5. RDS — Relational Database Service

```bash
# List all DB instances
aws rds describe-db-instances \
  --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceClass,Engine,DBInstanceStatus,Endpoint.Address]' \
  --output table

# Create RDS MySQL instance (production)
aws rds create-db-instance \
  --db-instance-identifier prod-mysql \
  --db-instance-class db.t3.medium \
  --engine mysql \
  --engine-version 8.0 \
  --master-username admin \
  --master-user-password <secure-password> \
  --allocated-storage 100 \
  --storage-type gp3 \
  --storage-encrypted \
  --vpc-security-group-ids sg-xxxxxxxxx \
  --db-subnet-group-name prod-db-subnet-group \
  --multi-az \
  --backup-retention-period 7 \
  --preferred-backup-window "02:00-03:00" \
  --preferred-maintenance-window "sun:05:00-sun:06:00" \
  --deletion-protection \
  --tags Key=Env,Value=production

# Create DB Subnet Group
aws rds create-db-subnet-group \
  --db-subnet-group-name prod-db-subnet-group \
  --db-subnet-group-description "Production DB Subnet Group" \
  --subnet-ids subnet-xxxxxxxxx subnet-yyyyyyyyy

# Create a DB snapshot
aws rds create-db-snapshot \
  --db-instance-identifier prod-mysql \
  --db-snapshot-identifier prod-mysql-snapshot-$(date +%Y%m%d)

# List snapshots
aws rds describe-db-snapshots \
  --db-instance-identifier prod-mysql \
  --query 'DBSnapshots[*].[DBSnapshotIdentifier,Status,SnapshotCreateTime]' --output table

# Restore DB from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier prod-mysql-restored \
  --db-snapshot-identifier prod-mysql-snapshot-20260415 \
  --db-instance-class db.t3.medium

# Modify DB instance
aws rds modify-db-instance \
  --db-instance-identifier prod-mysql \
  --db-instance-class db.r5.large \
  --apply-immediately

# Reboot DB instance
aws rds reboot-db-instance --db-instance-identifier prod-mysql

# Delete DB instance (with final snapshot)
aws rds delete-db-instance \
  --db-instance-identifier prod-mysql \
  --final-db-snapshot-identifier prod-mysql-final-snapshot

# Delete without final snapshot (non-production only)
aws rds delete-db-instance \
  --db-instance-identifier dev-mysql \
  --skip-final-snapshot
```

---

## 6. ECS — Elastic Container Service

```bash
# List clusters
aws ecs list-clusters
aws ecs describe-clusters --clusters prod-cluster

# Create a cluster
aws ecs create-cluster \
  --cluster-name prod-cluster \
  --capacity-providers FARGATE FARGATE_SPOT \
  --tags key=Env,value=production

# Register a task definition
aws ecs register-task-definition --cli-input-json file://task-def.json

# List task definitions
aws ecs list-task-definitions --family-prefix my-app

# Describe a task definition
aws ecs describe-task-definition --task-definition my-app:1

# List services
aws ecs list-services --cluster prod-cluster

# Create a service
aws ecs create-service \
  --cluster prod-cluster \
  --service-name my-app-service \
  --task-definition my-app:1 \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration \
    "awsvpcConfiguration={subnets=[subnet-xxxxxxxxx],securityGroups=[sg-xxxxxxxxx],assignPublicIp=ENABLED}"

# Update service (deploy new task definition)
aws ecs update-service \
  --cluster prod-cluster \
  --service my-app-service \
  --task-definition my-app:2 \
  --force-new-deployment

# Scale service
aws ecs update-service \
  --cluster prod-cluster \
  --service my-app-service \
  --desired-count 5

# List running tasks
aws ecs list-tasks --cluster prod-cluster --service-name my-app-service

# Describe a task
aws ecs describe-tasks \
  --cluster prod-cluster \
  --tasks <task-arn>

# Stop a task
aws ecs stop-task --cluster prod-cluster --task <task-arn>

# Delete a service (scale to 0 first)
aws ecs update-service --cluster prod-cluster --service my-app-service --desired-count 0
aws ecs delete-service --cluster prod-cluster --service my-app-service

# Delete cluster
aws ecs delete-cluster --cluster prod-cluster
```

---

## 7. EKS — Elastic Kubernetes Service

```bash
# List clusters
aws eks list-clusters

# Create EKS cluster
aws eks create-cluster \
  --name prod-eks \
  --role-arn arn:aws:iam::<account-id>:role/EKSClusterRole \
  --resources-vpc-config subnetIds=subnet-xxxxxxxxx,subnet-yyyyyyyyy,securityGroupIds=sg-xxxxxxxxx \
  --kubernetes-version 1.29

# Describe cluster
aws eks describe-cluster --name prod-eks

# Update kubeconfig for kubectl access
aws eks update-kubeconfig --name prod-eks --region ap-south-1

# Create a node group
aws eks create-nodegroup \
  --cluster-name prod-eks \
  --nodegroup-name prod-nodes \
  --scaling-config minSize=2,maxSize=10,desiredSize=3 \
  --instance-types t3.medium \
  --ami-type AL2_x86_64 \
  --node-role arn:aws:iam::<account-id>:role/EKSNodeRole \
  --subnets subnet-xxxxxxxxx subnet-yyyyyyyyy

# List node groups
aws eks list-nodegroups --cluster-name prod-eks

# Update node group (scale)
aws eks update-nodegroup-config \
  --cluster-name prod-eks \
  --nodegroup-name prod-nodes \
  --scaling-config minSize=2,maxSize=10,desiredSize=5

# Update EKS cluster version
aws eks update-cluster-version \
  --name prod-eks \
  --kubernetes-version 1.30

# Delete node group
aws eks delete-nodegroup --cluster-name prod-eks --nodegroup-name prod-nodes

# Delete cluster
aws eks delete-cluster --name prod-eks
```

---

## 8. Lambda — Serverless Functions

```bash
# List functions
aws lambda list-functions \
  --query 'Functions[*].[FunctionName,Runtime,LastModified]' --output table

# Create a function (zip-based)
aws lambda create-function \
  --function-name my-function \
  --runtime python3.12 \
  --role arn:aws:iam::<account-id>:role/LambdaExecutionRole \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --timeout 30 \
  --memory-size 256 \
  --environment Variables='{ENV=production,DB_HOST=prod-db.example.com}'

# Deploy / update function code
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip

# Update environment variables
aws lambda update-function-configuration \
  --function-name my-function \
  --environment Variables='{ENV=production,NEW_VAR=value}'

# Invoke a function
aws lambda invoke \
  --function-name my-function \
  --payload '{"key": "value"}' \
  --cli-binary-format raw-in-base64-out \
  output.json
cat output.json

# Get function details
aws lambda get-function --function-name my-function

# List function versions
aws lambda list-versions-by-function --function-name my-function

# Publish a new version
aws lambda publish-version --function-name my-function

# Create an alias
aws lambda create-alias \
  --function-name my-function \
  --name production \
  --function-version 5

# Add S3 trigger
aws lambda add-permission \
  --function-name my-function \
  --statement-id s3-trigger \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::my-prod-bucket-2026

# Delete a function
aws lambda delete-function --function-name my-function
```

---

## 9. Elastic Load Balancer (ALB / NLB)

```bash
# List all load balancers
aws elbv2 describe-load-balancers \
  --query 'LoadBalancers[*].[LoadBalancerName,Type,State.Code,DNSName]' --output table

# Create an Application Load Balancer
aws elbv2 create-load-balancer \
  --name prod-alb \
  --type application \
  --subnets subnet-xxxxxxxxx subnet-yyyyyyyyy \
  --security-groups sg-xxxxxxxxx \
  --tags Key=Env,Value=production

# Create a target group
aws elbv2 create-target-group \
  --name prod-tg \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-xxxxxxxxx \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3

# Register targets (EC2 instances)
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:...:targetgroup/prod-tg/xxx \
  --targets Id=i-xxxxxxxxxxxxxxxxx Port=80

# Create a listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:...:loadbalancer/app/prod-alb/xxx \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:...:certificate/xxx \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...:targetgroup/prod-tg/xxx

# Describe target health
aws elbv2 describe-target-health \
  --target-group-arn arn:aws:elasticloadbalancing:...:targetgroup/prod-tg/xxx

# Delete a load balancer
aws elbv2 delete-load-balancer \
  --load-balancer-arn arn:aws:elasticloadbalancing:...:loadbalancer/app/prod-alb/xxx
```

---

## 10. Auto Scaling

```bash
# List Auto Scaling Groups
aws autoscaling describe-auto-scaling-groups \
  --query 'AutoScalingGroups[*].[AutoScalingGroupName,DesiredCapacity,MinSize,MaxSize]' --output table

# Create a launch template
aws ec2 create-launch-template \
  --launch-template-name prod-lt \
  --version-description "v1" \
  --launch-template-data '{
    "ImageId": "ami-0c55b159cbfafe1f0",
    "InstanceType": "t3.medium",
    "KeyName": "my-key",
    "SecurityGroupIds": ["sg-xxxxxxxxx"]
  }'

# Create an Auto Scaling Group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name prod-asg \
  --launch-template LaunchTemplateName=prod-lt,Version='$Latest' \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 3 \
  --vpc-zone-identifier "subnet-xxxxxxxxx,subnet-yyyyyyyyy" \
  --target-group-arns arn:aws:elasticloadbalancing:...:targetgroup/prod-tg/xxx \
  --health-check-type ELB \
  --health-check-grace-period 300 \
  --tags Key=Name,Value=prod-asg-instance,PropagateAtLaunch=true

# Set desired capacity
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name prod-asg \
  --desired-capacity 5

# Create a scaling policy (target tracking CPU 70%)
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name prod-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 70.0
  }'

# Put instance in standby (for maintenance)
aws autoscaling enter-standby \
  --instance-ids i-xxxxxxxxxxxxxxxxx \
  --auto-scaling-group-name prod-asg \
  --should-decrement-desired-capacity

# Return from standby
aws autoscaling exit-standby \
  --instance-ids i-xxxxxxxxxxxxxxxxx \
  --auto-scaling-group-name prod-asg

# Delete ASG
aws autoscaling delete-auto-scaling-group \
  --auto-scaling-group-name prod-asg \
  --force-delete
```

---

## 11. Route 53 — DNS

```bash
# List hosted zones
aws route53 list-hosted-zones \
  --query 'HostedZones[*].[Name,Id,Config.PrivateZone]' --output table

# Create a hosted zone
aws route53 create-hosted-zone \
  --name example.com \
  --caller-reference $(date +%s)

# List DNS records in a zone
aws route53 list-resource-record-sets \
  --hosted-zone-id Z1234567890ABC

# Create / update a DNS record (A record)
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "app.example.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "1.2.3.4"}]
      }
    }]
  }'

# Create an Alias record pointing to ALB
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "app.example.com",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z35SXDOTRQ7X7K",
          "DNSName": "prod-alb-1234567890.ap-south-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }]
  }'

# Create a health check
aws route53 create-health-check \
  --caller-reference $(date +%s) \
  --health-check-config '{
    "IPAddress": "1.2.3.4",
    "Port": 443,
    "Type": "HTTPS",
    "ResourcePath": "/health",
    "FullyQualifiedDomainName": "app.example.com",
    "RequestInterval": 30,
    "FailureThreshold": 3
  }'

# Delete a hosted zone
aws route53 delete-hosted-zone --id Z1234567890ABC
```

---

## 12. CloudFront — CDN

```bash
# List distributions
aws cloudfront list-distributions \
  --query 'DistributionList.Items[*].[Id,DomainName,Status]' --output table

# Create a distribution (S3 origin)
aws cloudfront create-distribution \
  --distribution-config file://cf-distribution.json

# Get distribution details
aws cloudfront get-distribution --id <distribution-id>

# Invalidate cache (clear CDN cache)
aws cloudfront create-invalidation \
  --distribution-id <distribution-id> \
  --paths "/*"

aws cloudfront create-invalidation \
  --distribution-id <distribution-id> \
  --paths "/static/*" "/index.html"

# Disable a distribution
aws cloudfront update-distribution \
  --id <distribution-id> \
  --if-match <etag> \
  --distribution-config file://cf-disabled.json

# Delete a distribution (must be disabled first)
aws cloudfront delete-distribution \
  --id <distribution-id> \
  --if-match <etag>
```

---

## 13. SQS — Simple Queue Service

```bash
# List queues
aws sqs list-queues

# Create a standard queue
aws sqs create-queue \
  --queue-name prod-queue \
  --attributes '{
    "MessageRetentionPeriod": "86400",
    "VisibilityTimeout": "30",
    "ReceiveMessageWaitTimeSeconds": "20"
  }'

# Create FIFO queue
aws sqs create-queue \
  --queue-name prod-queue.fifo \
  --attributes '{
    "FifoQueue": "true",
    "ContentBasedDeduplication": "true"
  }'

# Send a message
aws sqs send-message \
  --queue-url https://sqs.ap-south-1.amazonaws.com/<account-id>/prod-queue \
  --message-body "Hello from prod"

# Receive messages
aws sqs receive-message \
  --queue-url https://sqs.ap-south-1.amazonaws.com/<account-id>/prod-queue \
  --max-number-of-messages 10 \
  --wait-time-seconds 20

# Delete a message
aws sqs delete-message \
  --queue-url https://sqs.ap-south-1.amazonaws.com/<account-id>/prod-queue \
  --receipt-handle <receipt-handle>

# Get queue attributes
aws sqs get-queue-attributes \
  --queue-url https://sqs.ap-south-1.amazonaws.com/<account-id>/prod-queue \
  --attribute-names All

# Purge a queue
aws sqs purge-queue \
  --queue-url https://sqs.ap-south-1.amazonaws.com/<account-id>/prod-queue

# Delete a queue
aws sqs delete-queue \
  --queue-url https://sqs.ap-south-1.amazonaws.com/<account-id>/prod-queue
```

---

## 14. SNS — Simple Notification Service

```bash
# List topics
aws sns list-topics

# Create a topic
aws sns create-topic --name prod-alerts

# Create FIFO topic
aws sns create-topic --name prod-alerts.fifo \
  --attributes FifoTopic=true,ContentBasedDeduplication=true

# Subscribe an email endpoint
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-south-1:<account-id>:prod-alerts \
  --protocol email \
  --notification-endpoint devops@example.com

# Subscribe an SQS queue
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-south-1:<account-id>:prod-alerts \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:ap-south-1:<account-id>:prod-queue

# Publish a message
aws sns publish \
  --topic-arn arn:aws:sns:ap-south-1:<account-id>:prod-alerts \
  --subject "ALERT: High CPU" \
  --message "CPU utilization exceeded 90% on prod-server"

# List subscriptions
aws sns list-subscriptions-by-topic \
  --topic-arn arn:aws:sns:ap-south-1:<account-id>:prod-alerts

# Delete a topic
aws sns delete-topic --topic-arn arn:aws:sns:ap-south-1:<account-id>:prod-alerts
```

---

## 15. CloudWatch — Monitoring & Logs

```bash
# List all alarms
aws cloudwatch describe-alarms \
  --query 'MetricAlarms[*].[AlarmName,StateValue,MetricName]' --output table

# Create a CPU alarm
aws cloudwatch put-metric-alarm \
  --alarm-name prod-high-cpu \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-xxxxxxxxxxxxxxxxx \
  --alarm-actions arn:aws:sns:ap-south-1:<account-id>:prod-alerts \
  --ok-actions arn:aws:sns:ap-south-1:<account-id>:prod-alerts

# Get metric statistics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-xxxxxxxxxxxxxxxxx \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 300 \
  --statistics Average

# List log groups
aws logs describe-log-groups \
  --query 'logGroups[*].[logGroupName,retentionInDays]' --output table

# Create a log group
aws logs create-log-group --log-group-name /prod/myapp

# Set retention policy (30 days)
aws logs put-retention-policy \
  --log-group-name /prod/myapp \
  --retention-in-days 30

# Tail logs live
aws logs tail /prod/myapp --follow

# Filter log events
aws logs filter-log-events \
  --log-group-name /prod/myapp \
  --filter-pattern "ERROR" \
  --start-time $(date -d '1 hour ago' +%s)000

# Delete a log group
aws logs delete-log-group --log-group-name /prod/myapp

# Delete an alarm
aws cloudwatch delete-alarms --alarm-names prod-high-cpu
```

---

## 16. CloudFormation — Infrastructure as Code

```bash
# List stacks
aws cloudformation list-stacks \
  --query 'StackSummaries[?StackStatus!=`DELETE_COMPLETE`].[StackName,StackStatus]' --output table

# Validate a template
aws cloudformation validate-template --template-body file://stack.yaml

# Create a stack
aws cloudformation create-stack \
  --stack-name prod-stack \
  --template-body file://stack.yaml \
  --parameters ParameterKey=Env,ParameterValue=production \
  --capabilities CAPABILITY_NAMED_IAM \
  --tags Key=Project,Value=myapp

# Wait for stack creation to complete
aws cloudformation wait stack-create-complete --stack-name prod-stack

# Describe stack events
aws cloudformation describe-stack-events \
  --stack-name prod-stack \
  --query 'StackEvents[*].[Timestamp,ResourceStatus,ResourceType,LogicalResourceId]' --output table

# Update a stack
aws cloudformation update-stack \
  --stack-name prod-stack \
  --template-body file://stack.yaml \
  --parameters ParameterKey=Env,ParameterValue=production \
  --capabilities CAPABILITY_NAMED_IAM

# Create a change set (preview changes)
aws cloudformation create-change-set \
  --stack-name prod-stack \
  --change-set-name my-changeset \
  --template-body file://stack.yaml \
  --capabilities CAPABILITY_NAMED_IAM

# Describe change set
aws cloudformation describe-change-set \
  --stack-name prod-stack \
  --change-set-name my-changeset

# Execute change set
aws cloudformation execute-change-set \
  --stack-name prod-stack \
  --change-set-name my-changeset

# Delete a stack
aws cloudformation delete-stack --stack-name prod-stack

# List stack outputs
aws cloudformation describe-stacks \
  --stack-name prod-stack \
  --query 'Stacks[0].Outputs' --output table
```

---

## 17. Secrets Manager & Parameter Store

### Secrets Manager
```bash
# List secrets
aws secretsmanager list-secrets \
  --query 'SecretList[*].[Name,LastChangedDate]' --output table

# Create a secret
aws secretsmanager create-secret \
  --name prod/db/password \
  --secret-string '{"username":"admin","password":"SecureP@ss123"}'

# Retrieve a secret value
aws secretsmanager get-secret-value --secret-id prod/db/password

# Update a secret
aws secretsmanager update-secret \
  --secret-id prod/db/password \
  --secret-string '{"username":"admin","password":"NewP@ss456"}'

# Rotate a secret
aws secretsmanager rotate-secret \
  --secret-id prod/db/password \
  --rotation-lambda-arn arn:aws:lambda:...:function:rotate-db-password

# Delete a secret (with 7-day recovery window)
aws secretsmanager delete-secret \
  --secret-id prod/db/password \
  --recovery-window-in-days 7

# Force delete immediately (no recovery)
aws secretsmanager delete-secret \
  --secret-id prod/db/password \
  --force-delete-without-recovery
```

### SSM Parameter Store
```bash
# Put a parameter
aws ssm put-parameter \
  --name /prod/db/host \
  --value "prod-db.cluster.ap-south-1.rds.amazonaws.com" \
  --type SecureString \
  --overwrite

# Get a parameter
aws ssm get-parameter --name /prod/db/host --with-decryption

# Get multiple parameters by path
aws ssm get-parameters-by-path \
  --path /prod/ \
  --recursive \
  --with-decryption

# Delete a parameter
aws ssm delete-parameter --name /prod/db/host
```

---

## 18. KMS — Key Management Service

```bash
# List all KMS keys
aws kms list-keys

# Describe a key
aws kms describe-key --key-id <key-id>

# Create a symmetric encryption key
aws kms create-key \
  --description "Production encryption key" \
  --key-usage ENCRYPT_DECRYPT \
  --tags TagKey=Env,TagValue=production

# Create an alias
aws kms create-alias \
  --alias-name alias/prod-key \
  --target-key-id <key-id>

# Encrypt data
aws kms encrypt \
  --key-id alias/prod-key \
  --plaintext "my-secret-value" \
  --output text \
  --query CiphertextBlob | base64 --decode > encrypted.bin

# Decrypt data
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.bin \
  --output text \
  --query Plaintext | base64 --decode

# Schedule key deletion (7-30 days)
aws kms schedule-key-deletion \
  --key-id <key-id> \
  --pending-window-in-days 7

# Disable a key
aws kms disable-key --key-id <key-id>

# Enable a key
aws kms enable-key --key-id <key-id>
```

---

## 19. ACM — AWS Certificate Manager

```bash
# List certificates
aws acm list-certificates \
  --query 'CertificateSummaryList[*].[DomainName,CertificateArn,Status]' --output table

# Request a public certificate (DNS validation)
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names "*.example.com" \
  --validation-method DNS \
  --tags Key=Env,Value=production

# Describe certificate (get DNS validation records)
aws acm describe-certificate \
  --certificate-arn arn:aws:acm:...:certificate/xxx

# Import an existing certificate
aws acm import-certificate \
  --certificate fileb://cert.pem \
  --private-key fileb://private.key \
  --certificate-chain fileb://chain.pem

# Delete a certificate
aws acm delete-certificate \
  --certificate-arn arn:aws:acm:...:certificate/xxx
```

---

## 20. Systems Manager (SSM) — Instance Management

```bash
# List managed instances
aws ssm describe-instance-information \
  --query 'InstanceInformationList[*].[InstanceId,PingStatus,PlatformName]' --output table

# Start a session (SSH-less shell access)
aws ssm start-session --target i-xxxxxxxxxxxxxxxxx

# Run a remote command on instances
aws ssm send-command \
  --instance-ids i-xxxxxxxxxxxxxxxxx \
  --document-name "AWS-RunShellScript" \
  --parameters commands='["systemctl status myapp"]' \
  --output text

# Get command output
aws ssm get-command-invocation \
  --command-id <command-id> \
  --instance-id i-xxxxxxxxxxxxxxxxx

# Run a command on tagged instances
aws ssm send-command \
  --targets Key=tag:Env,Values=production \
  --document-name "AWS-RunShellScript" \
  --parameters commands='["df -h"]'
```

---

## 21. Cost & Billing

```bash
# Get cost and usage (last 30 days)
aws ce get-cost-and-usage \
  --time-period Start=$(date -d '30 days ago' +%Y-%m-%d),End=$(date +%Y-%m-%d) \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --group-by Type=DIMENSION,Key=SERVICE

# Get cost by tag
aws ce get-cost-and-usage \
  --time-period Start=$(date -d '30 days ago' +%Y-%m-%d),End=$(date +%Y-%m-%d) \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --filter '{"Tags":{"Key":"Env","Values":["production"]}}'

# Create a cost budget alert
aws budgets create-budget \
  --account-id <account-id> \
  --budget file://budget.json \
  --notifications-with-subscribers file://notifications.json
```

---

## 22. Quick Reference Summary

| Service | Key Command |
|---|---|
| Identity | `aws sts get-caller-identity` |
| IAM User | `aws iam create-user --user-name <name>` |
| EC2 Launch | `aws ec2 run-instances --image-id <ami> --instance-type <type>` |
| EC2 List | `aws ec2 describe-instances --filters Name=instance-state-name,Values=running` |
| S3 Upload | `aws s3 cp <file> s3://<bucket>/` |
| S3 Sync | `aws s3 sync ./dist/ s3://<bucket>/static/ --delete` |
| RDS Create | `aws rds create-db-instance --db-instance-identifier <id> ...` |
| ECS Deploy | `aws ecs update-service --cluster <c> --service <s> --force-new-deployment` |
| EKS Access | `aws eks update-kubeconfig --name <cluster>` |
| Lambda Deploy | `aws lambda update-function-code --function-name <fn> --zip-file fileb://fn.zip` |
| CloudWatch Logs | `aws logs tail <log-group> --follow` |
| SSM Session | `aws ssm start-session --target <instance-id>` |
| Secret Get | `aws secretsmanager get-secret-value --secret-id <name>` |
| CFN Deploy | `aws cloudformation deploy --stack-name <s> --template-file <f>` |
| Cache Invalidate | `aws cloudfront create-invalidation --distribution-id <id> --paths "/*"` |
| Cost Report | `aws ce get-cost-and-usage --time-period Start=<date>,End=<date> --granularity MONTHLY --metrics BlendedCost` |
