# AWS Certified Solutions Architect — Associate (SAA-C03)
## Exam Preparation Guide — Questions & Answers

> **Exam:** AWS Certified Solutions Architect – Associate (SAA-C03)  
> **Format:** 65 questions | 130 minutes | Score 720/1000 to pass  
> **Domains:**
> - Domain 1: Design Secure Architectures (30%)
> - Domain 2: Design Resilient Architectures (26%)
> - Domain 3: Design High-Performing Architectures (24%)
> - Domain 4: Design Cost-Optimized Architectures (20%)

---

## Domain 1 — Design Secure Architectures

---

**Q1.** A company needs to store sensitive customer data in S3. The security team requires that all data be encrypted at rest using keys that the company fully controls. Which solution meets this requirement?

- A. S3 Server-Side Encryption with Amazon S3-managed keys (SSE-S3)
- B. S3 Server-Side Encryption with AWS KMS keys (SSE-KMS) using a customer-managed key
- C. S3 Server-Side Encryption with customer-provided keys (SSE-C)
- D. Client-side encryption using an AWS-managed key

**Answer: B**  
**Explanation:** SSE-KMS with a customer-managed key (CMK) gives the company full control over key policies, rotation, and auditing via CloudTrail. SSE-S3 uses AWS-managed keys (less control). SSE-C requires the customer to manage and send the key with every request.

---

**Q2.** A Solutions Architect needs to grant an EC2 instance access to an S3 bucket. What is the most secure and operationally efficient method?

- A. Embed AWS credentials in the application code
- B. Store credentials in an environment variable on the EC2 instance
- C. Attach an IAM role with the required S3 permissions to the EC2 instance
- D. Create an IAM user and store the access keys in a config file on the instance

**Answer: C**  
**Explanation:** IAM roles for EC2 instances provide temporary, automatically rotated credentials via instance metadata. Hardcoding or storing credentials on instances is a security antipattern and violates the AWS security best practices.

---

**Q3.** A company wants to ensure that only encrypted connections are made to their S3 bucket. Which S3 bucket policy condition enforces this?

- A. `"aws:SecureTransport": "true"`
- B. `"s3:x-amz-server-side-encryption": "AES256"`
- C. `"aws:RequestedRegion": "us-east-1"`
- D. `"s3:prefix": "/secure"`

**Answer: A**  
**Explanation:** The `aws:SecureTransport` condition key evaluates to true when the request is made over HTTPS (TLS). Setting it to `"false"` with a `Deny` effect blocks all HTTP requests.

---

**Q4.** An application running on EC2 needs to access AWS Secrets Manager to retrieve database credentials. The EC2 instance is in a private subnet with no internet access. What is the most secure way to enable this access?

- A. Add a NAT Gateway to allow internet traffic
- B. Create a VPC endpoint for Secrets Manager
- C. Store the secrets in SSM Parameter Store instead
- D. Open port 443 outbound in the security group

**Answer: B**  
**Explanation:** A VPC Interface Endpoint for Secrets Manager allows the EC2 instance to communicate with Secrets Manager privately without internet access. This keeps traffic within the AWS network and improves security.

---

**Q5.** A company has multiple AWS accounts and needs to enforce a rule that no account can disable AWS CloudTrail. Which AWS service should be used?

- A. AWS Config Rules
- B. AWS Service Control Policies (SCPs) via AWS Organizations
- C. IAM Permission Boundaries
- D. AWS CloudFormation StackSets

**Answer: B**  
**Explanation:** SCPs in AWS Organizations allow the management account to enforce guardrails across all member accounts. An SCP can deny the `cloudtrail:StopLogging` and `cloudtrail:DeleteTrail` actions globally.

---

**Q6.** A web application is receiving malicious HTTP requests. The security team needs to block specific IP addresses and protect against SQL injection and XSS attacks. Which AWS service should be used?

- A. AWS Shield Standard
- B. AWS Network Firewall
- C. AWS WAF (Web Application Firewall)
- D. Security Groups

**Answer: C**  
**Explanation:** AWS WAF provides managed rule groups for OWASP Top 10 threats including SQL injection and XSS. It can be applied to CloudFront, ALB, API Gateway, and AppSync. Security Groups operate at Layer 4 (TCP/IP), not Layer 7 (HTTP).

---

**Q7.** A company requires that all API calls to AWS services in their account are logged for auditing. Which service achieves this at the account level?

- A. Amazon CloudWatch
- B. AWS X-Ray
- C. AWS CloudTrail
- D. AWS Config

**Answer: C**  
**Explanation:** AWS CloudTrail records every API call made in an AWS account including the identity of the caller, the time, source IP, and parameters. It provides governance, compliance, and operational auditing.

---

**Q8.** A Solutions Architect needs to limit the maximum permissions that an IAM user can have, regardless of what policies are attached to them. What should be used?

- A. IAM Resource-based policies
- B. AWS Organizations SCPs
- C. IAM Permission Boundaries
- D. IAM Condition keys

**Answer: C**  
**Explanation:** IAM Permission Boundaries set the maximum permissions an IAM entity (user or role) can have. Even if more permissive policies are attached, effective permissions are the intersection of attached policies and the boundary.

---

**Q9.** An application requires users to authenticate before accessing resources. The users already have identities in an on-premises Active Directory. Which AWS service enables federation without creating IAM users?

- A. AWS IAM Identity Center (SSO) with AD Connector
- B. Amazon Cognito User Pools
- C. AWS Directory Service for Microsoft AD
- D. Both A and C

**Answer: D**  
**Explanation:** AWS IAM Identity Center with AD Connector federates on-premises AD identities to AWS. AWS Managed Microsoft AD can also trust an on-premises AD. Both allow users to authenticate with existing credentials without creating IAM users.

---

**Q10.** A company wants to detect if any S3 bucket becomes publicly accessible in their account. Which service provides the fastest automated detection?

- A. AWS Trusted Advisor
- B. AWS Config with managed rule `s3-bucket-public-read-prohibited`
- C. Amazon Macie
- D. AWS Security Hub

**Answer: B**  
**Explanation:** AWS Config continuously evaluates resource configurations against rules. The managed rule `s3-bucket-public-read-prohibited` flags non-compliant S3 buckets in near real-time. Macie focuses on sensitive data discovery, not configuration compliance.

---

## Domain 2 — Design Resilient Architectures

---

**Q11.** A company runs a critical e-commerce application on EC2 instances behind an ALB. The RTO is 1 hour and the RPO is 15 minutes. Which DR strategy meets these requirements at the lowest cost?

- A. Multi-site active/active
- B. Warm standby
- C. Pilot light
- D. Backup and restore

**Answer: C**  
**Explanation:** Pilot light maintains a minimal version of the environment running in AWS (core DB replicated, EC2 AMIs ready). Recovery time is typically under 1 hour since you just scale up the pilot. Backup and restore would exceed the RTO. Warm standby costs more. Multi-site active/active has near-zero RTO but is the most expensive.

---

**Q12.** An application must survive the failure of an entire AWS Availability Zone with no data loss and automatic failover in under 60 seconds. What architecture achieves this?

- A. Multi-AZ RDS with Auto Scaling EC2 across two AZs behind an ALB
- B. Single RDS instance with automated backups
- C. RDS Read Replica in a second AZ
- D. EC2 with daily EBS snapshots

**Answer: A**  
**Explanation:** Multi-AZ RDS automatically fails over to a synchronous standby replica in another AZ (typically within 60 seconds, no data loss). EC2 Auto Scaling with ALB across multiple AZs replaces failed instances automatically.

---

**Q13.** Where should session state be stored in a stateless, horizontally scalable web application running on EC2?

- A. In the EC2 instance local memory
- B. In an EBS volume attached to each instance
- C. In Amazon ElastiCache (Redis) or DynamoDB
- D. In an S3 bucket

**Answer: C**  
**Explanation:** Storing sessions in ElastiCache (Redis) or DynamoDB externalizes state so any EC2 instance can serve any request. Local memory or EBS breaks horizontal scaling because the user must always reach the same instance.

---

**Q14.** A company needs a message queue that guarantees each message is processed exactly once and in the order it was sent. Which SQS queue type should be used?

- A. Standard Queue
- B. FIFO Queue with content-based deduplication
- C. Dead Letter Queue
- D. Delay Queue

**Answer: B**  
**Explanation:** SQS FIFO queues guarantee exactly-once processing and strict message ordering. Standard queues offer at-least-once delivery and best-effort ordering. Content-based deduplication prevents duplicate messages within a 5-minute window.

---

**Q15.** An application uses DynamoDB. During peak traffic, the ProvisionedThroughputExceededException error occurs. What is the best solution to handle sudden traffic spikes automatically?

- A. Increase provisioned read/write capacity manually
- B. Enable DynamoDB Auto Scaling
- C. Switch to DynamoDB On-Demand capacity mode
- D. Add a DynamoDB DAX cluster

**Answer: C**  
**Explanation:** On-Demand capacity mode instantly accommodates any traffic spike without capacity planning. Auto Scaling can lag during sudden spikes. DAX improves read latency (caching) but doesn't solve write throttling. For completely unpredictable traffic, On-Demand is the best fit.

---

**Q16.** A company needs to distribute traffic across EC2 instances in multiple AWS Regions for global low-latency access. Which service should be used?

- A. Application Load Balancer
- B. AWS Global Accelerator
- C. Amazon Route 53 with latency-based routing
- D. Both B and C are valid

**Answer: D**  
**Explanation:** Route 53 latency-based routing directs users to the lowest-latency region at DNS resolution. AWS Global Accelerator uses static Anycast IPs and routes traffic over the AWS backbone to the nearest endpoint, providing faster failover (seconds vs. DNS TTL). Both are valid multi-region routing solutions.

---

**Q17.** A Lambda function processes messages from an SQS queue. After multiple failed processing attempts, unprocessed messages must be preserved for investigation. What should be configured?

- A. Lambda retry policy set to 0
- B. SQS visibility timeout increased to 12 hours
- C. A Dead Letter Queue (DLQ) on the SQS queue
- D. SNS topic subscription for failed messages

**Answer: C**  
**Explanation:** An SQS Dead Letter Queue automatically receives messages that fail processing after a configured number of `maxReceiveCount` attempts. This preserves failed messages for debugging without blocking the main queue.

---

**Q18.** An S3 bucket stores critical business data. The company wants to prevent accidental deletion of any version of an object. What combination of features should be enabled?

- A. S3 Versioning only
- B. S3 Versioning + MFA Delete
- C. S3 Replication to another bucket
- D. S3 Object Lock in Governance mode

**Answer: B**  
**Explanation:** S3 Versioning retains all versions of objects. MFA Delete requires multi-factor authentication to permanently delete a version or suspend versioning, preventing accidental or unauthorized deletions. S3 Object Lock (Compliance mode) is used for regulatory retention requirements.

---

**Q19.** A company runs a stateful application on a single EC2 instance. They want to make it fault-tolerant with automatic recovery if the instance fails. What is the simplest solution?

- A. Create an Auto Scaling Group with min=1, max=1
- B. Create a CloudWatch alarm to recover the EC2 instance on failure
- C. Use an Elastic Load Balancer health check
- D. Create an AMI and manually launch a replacement

**Answer: B**  
**Explanation:** A CloudWatch alarm with the `recover` action automatically recovers an EC2 instance on the same host or a new one (retaining instance ID, EIP, and EBS data) when a system-level failure is detected. An ASG with min=max=1 would also replace the instance but on a new instance ID.

---

**Q20.** A company needs an RTO near zero and RPO near zero for a mission-critical application. Cost is not a concern. Which DR strategy should be used?

- A. Backup and restore
- B. Pilot light
- C. Warm standby
- D. Multi-site active/active

**Answer: D**  
**Explanation:** Multi-site active/active runs identical infrastructure in multiple regions simultaneously. Traffic is served from all regions, and failover is instantaneous (near-zero RTO/RPO). It is the most expensive strategy but provides the highest availability.

---

## Domain 3 — Design High-Performing Architectures

---

**Q21.** A company has a read-heavy application backed by a MySQL RDS database. Query response times are too slow. What is the best solution to improve read performance with minimal code changes?

- A. Upgrade to a larger RDS instance class
- B. Create RDS Read Replicas and route read traffic to them
- C. Enable Multi-AZ on the RDS instance
- D. Use DynamoDB instead of RDS

**Answer: B**  
**Explanation:** RDS Read Replicas offload read queries from the primary instance, improving performance for read-heavy workloads. Multi-AZ improves availability (not read performance). Upgrading the instance is a short-term fix that doesn't scale as well.

---

**Q22.** An application requires microsecond-latency data access for a real-time leaderboard. Which AWS service is most appropriate?

- A. Amazon RDS
- B. Amazon DynamoDB
- C. Amazon ElastiCache for Redis
- D. Amazon S3

**Answer: C**  
**Explanation:** ElastiCache for Redis provides microsecond read/write latency for in-memory data structures like sorted sets (perfect for leaderboards). DynamoDB offers single-digit millisecond latency, which may not meet microsecond requirements.

---

**Q23.** A company serves a static website globally. Users in Asia report slow load times. The content is hosted in an S3 bucket in us-east-1. What is the best solution?

- A. Move the S3 bucket to ap-southeast-1
- B. Enable S3 Transfer Acceleration
- C. Deploy the website behind Amazon CloudFront
- D. Use Route 53 latency-based routing to a second S3 bucket

**Answer: C**  
**Explanation:** CloudFront caches static content at 400+ edge locations worldwide. Asian users receive content from the nearest edge location instead of fetching from us-east-1, dramatically reducing latency. S3 Transfer Acceleration improves uploads, not static content delivery.

---

**Q24.** A data processing pipeline reads from S3, processes records, and writes results. Throughput needs to scale automatically based on queue depth. Which architecture is best?

- A. EC2 instances with manual scaling
- B. Lambda triggered by SQS with Kinesis for streaming
- C. AWS Batch for batch processing jobs
- D. EC2 Auto Scaling Group with SQS-based scaling policy

**Answer: D**  
**Explanation:** EC2 Auto Scaling with a custom CloudWatch metric tracking SQS queue depth automatically scales instances to match workload. AWS Batch is also a strong candidate for large-scale batch processing. Lambda + SQS works for event-driven but has execution time limits.

---

**Q25.** An application stores and queries large amounts of time-series IoT data. The query pattern is always by time range and device ID. Which database is most appropriate?

- A. Amazon RDS MySQL
- B. Amazon DynamoDB with a time-based sort key
- C. Amazon Timestream
- D. Amazon Redshift

**Answer: C**  
**Explanation:** Amazon Timestream is purpose-built for time-series data with built-in time-based query functions, automatic data tiering (in-memory → magnetic), and massive scale. DynamoDB works but requires manual time-series design. Redshift is for analytics data warehousing.

---

**Q26.** A company's data analytics team runs complex SQL queries on petabytes of structured data. Queries currently take hours. Which service provides the best performance for this use case?

- A. Amazon RDS
- B. Amazon Aurora
- C. Amazon Redshift
- D. Amazon Athena

**Answer: C**  
**Explanation:** Amazon Redshift is a fully managed petabyte-scale data warehouse with columnar storage and massively parallel processing (MPP), optimized for analytical queries on large datasets. Athena is serverless and uses S3, suitable for ad-hoc queries but not sustained complex analytics at this scale.

---

**Q27.** A microservices application needs asynchronous communication between services. There are multiple consumer services that each need to receive every message. Which pattern should be used?

- A. SQS Standard Queue with multiple consumers
- B. SNS topic with SQS queue subscriptions for each consumer (Fan-out)
- C. SQS FIFO Queue
- D. Amazon Kinesis Data Streams

**Answer: B**  
**Explanation:** The SNS fan-out pattern publishes a message to an SNS topic, which delivers copies to multiple SQS queues (one per consumer service). Each consumer independently processes its own copy of the message. A single SQS queue can only deliver each message to one consumer.

---

**Q28.** A company needs to process and analyze real-time streaming data from thousands of IoT sensors and load it into S3 for later analytics. Which service combination is most appropriate?

- A. SQS → Lambda → S3
- B. Kinesis Data Streams → Kinesis Data Firehose → S3
- C. SNS → SQS → Lambda → S3
- D. Kinesis Data Streams → EC2 → RDS

**Answer: B**  
**Explanation:** Kinesis Data Streams ingests high-throughput real-time data. Kinesis Data Firehose buffers, batches, compresses, and reliably delivers the stream to S3 (or Redshift/Elasticsearch) with zero administration. This is the standard AWS real-time streaming → S3 pipeline.

---

**Q29.** An API Gateway REST API is receiving heavy traffic. The backend Lambda functions are being throttled. What is the fastest way to reduce the load without changing code?

- A. Increase Lambda concurrency limit
- B. Enable API Gateway caching
- C. Switch to HTTP API
- D. Add CloudFront in front of API Gateway

**Answer: B**  
**Explanation:** API Gateway caching stores responses at the API Gateway level for a configurable TTL, serving repeated identical requests without invoking Lambda. This immediately reduces backend calls and throttling at the cost of cache storage.

---

**Q30.** An application needs to serve hundreds of thousands of concurrent WebSocket connections for a real-time chat application. Which AWS service natively supports WebSockets?

- A. Application Load Balancer
- B. API Gateway WebSocket API
- C. Amazon CloudFront
- D. Network Load Balancer

**Answer: B**  
**Explanation:** API Gateway WebSocket APIs support maintaining persistent, bidirectional connections. ALB also supports WebSockets but requires managing backend EC2/ECS instances. API Gateway WebSocket API with Lambda is fully serverless and scales automatically.

---

## Domain 4 — Design Cost-Optimized Architectures

---

**Q31.** A company runs a batch processing workload on EC2 that can be interrupted. The job takes 4 hours and runs nightly. What is the most cost-effective EC2 purchasing option?

- A. On-Demand Instances
- B. Reserved Instances (1-year)
- C. Spot Instances with checkpointing
- D. Dedicated Hosts

**Answer: C**  
**Explanation:** Spot Instances offer up to 90% discount over On-Demand. Since the workload can be interrupted and checkpointed (saves progress), Spot is ideal for batch jobs. Reserved Instances save cost for steady-state workloads, not nightly batch jobs.

---

**Q32.** A company has EC2 instances that are always running 24/7 for 3 years. What is the most cost-effective purchasing option?

- A. On-Demand Instances
- B. Spot Instances
- C. 3-year Reserved Instances (All Upfront)
- D. Savings Plans

**Answer: C**  
**Explanation:** 3-year Reserved Instances (All Upfront) provide the maximum discount (up to 75%) for steady-state, always-on workloads. Savings Plans offer similar savings with more flexibility. On-Demand has no commitment but is most expensive. Spot instances are not reliable for 24/7 applications.

---

**Q33.** A company stores 100 TB of infrequently accessed compliance data in S3 Standard. Data must be retrievable within 12 hours. How can they reduce storage costs?

- A. Move data to S3 One Zone-IA
- B. Move data to S3 Glacier Instant Retrieval
- C. Move data to S3 Glacier Flexible Retrieval
- D. Enable S3 Intelligent-Tiering

**Answer: C**  
**Explanation:** S3 Glacier Flexible Retrieval (formerly Glacier) costs $0.004/GB/month vs $0.023/GB for S3 Standard. Retrieval in Expedited (1-5 min), Standard (3-5 hours), or Bulk (5-12 hours) modes satisfies the 12-hour requirement. S3 Glacier Instant Retrieval is for millisecond access at higher cost.

---

**Q34.** An application uses NAT Gateway to allow private EC2 instances to access the internet. The team notices very high NAT Gateway data processing costs. What can reduce this cost?

- A. Replace NAT Gateway with a NAT Instance
- B. Create VPC Endpoints for AWS services (S3, DynamoDB) accessed by the instances
- C. Move EC2 instances to public subnets
- D. Use AWS Direct Connect

**Answer: B**  
**Explanation:** VPC Gateway Endpoints for S3 and DynamoDB route traffic directly to these services without going through the NAT Gateway, eliminating NAT Gateway data processing charges for that traffic. This is a significant cost saving with no ongoing management overhead.

---

**Q35.** A company runs a development environment with EC2 instances active only during business hours (8AM–6PM, Mon–Fri). What is the most cost-effective solution?

- A. Use Reserved Instances
- B. Use Savings Plans
- C. Use Instance Scheduler to stop instances outside business hours
- D. Use On-Demand Instances

**Answer: C**  
**Explanation:** AWS Instance Scheduler (or EventBridge + Lambda) automatically stops instances outside business hours. Instances only incur charges when running. This eliminates ~70% of runtime costs. Reserved Instances commit to payment regardless of usage, wasting money when instances are stopped.

---

**Q36.** A company has unstructured log data in S3 that is queried ad-hoc 2-3 times per month. Which query service minimizes cost?

- A. Amazon Redshift
- B. Amazon Athena
- C. Amazon EMR
- D. Amazon RDS

**Answer: B**  
**Explanation:** Amazon Athena is serverless and charges only per query ($5/TB scanned). For infrequent ad-hoc queries on S3 data, there is no infrastructure to maintain. Using Parquet/ORC format + partitioning reduces scan size and cost. Redshift has ongoing cluster costs unsuitable for occasional queries.

---

**Q37.** A company uses data transfer heavily between EC2 instances in different Availability Zones. The bill shows high data transfer charges. What can reduce this?

- A. Move all instances into the same AZ
- B. Use VPC Peering between AZs
- C. Use Placement Groups (Partition) to keep instances in same AZ
- D. Use AWS Direct Connect

**Answer: A / C**  
**Explanation:** Data transfer between EC2 instances in the same AZ using private IP addresses is free. Cross-AZ data transfer incurs charges ($0.01/GB). Consolidating workloads that communicate heavily into the same AZ (accepting reduced redundancy) eliminates this cost. Partition Placement Groups organize instances within or across AZs.

---

**Q38.** A company wants to identify underutilized EC2 instances and reduce costs without manual analysis. Which AWS service provides these recommendations automatically?

- A. AWS Cost Explorer
- B. AWS Trusted Advisor
- C. AWS Compute Optimizer
- D. AWS Config

**Answer: C**  
**Explanation:** AWS Compute Optimizer uses machine learning to analyze CloudWatch metrics and recommends optimal EC2 instance types based on actual usage. Trusted Advisor also checks for low utilization but Compute Optimizer provides more detailed right-sizing recommendations.

---

**Q39.** A company transfers 10 TB of data per day from their on-premises data center to S3. The internet connection is slow and unreliable. What is the most cost-effective solution for a one-time large migration?

- A. AWS Direct Connect
- B. AWS Snowball Edge
- C. AWS DataSync over existing internet connection
- D. S3 Transfer Acceleration

**Answer: B**  
**Explanation:** AWS Snowball Edge is a physical device that stores up to 80TB and is shipped to the data center, loaded, and shipped back to AWS for ingestion. For large one-time migrations with slow/unreliable internet, physical transfer (Snowball) is faster and cheaper than online methods.

---

**Q40.** A company's DynamoDB table has predictable low usage on weekdays and spikes on weekends. What pricing model minimizes cost while handling both patterns?

- A. Provisioned capacity with fixed settings
- B. On-Demand capacity mode
- C. Provisioned capacity with Auto Scaling
- D. Add a DAX cluster

**Answer: C**  
**Explanation:** Provisioned capacity with Auto Scaling is the most cost-effective for predictable patterns. It scales capacity up for weekend spikes and scales down during weekday lows, paying only for provisioned capacity. On-Demand mode costs more per request than provisioned at high scales.

---

## Mixed Scenario & Architecture Questions

---

**Q41.** A three-tier web application experiences variable traffic. The web tier (EC2), application tier (EC2), and database tier (RDS MySQL Multi-AZ) serve customers globally. Which set of changes improves both performance and cost-efficiency?

- A. Add CloudFront in front of ALB, add ElastiCache for session state, use EC2 Auto Scaling, add RDS Read Replicas
- B. Move everything to Lambda and DynamoDB
- C. Use Reserved Instances for web and app tiers, use Spot for database
- D. Replicate RDS to all global regions

**Answer: A**  
**Explanation:** CloudFront reduces origin load. ElastiCache offloads session/cache from RDS. Auto Scaling matches capacity to demand. RDS Read Replicas handle read-heavy workloads. This is the classic, well-architected three-tier improvement pattern.

---

**Q42.** A company needs to run a machine learning training job that requires 500 GB of shared storage accessible by 100 EC2 instances simultaneously. Which storage solution is most appropriate?

- A. EBS volumes on each EC2 instance
- B. Amazon S3
- C. Amazon EFS (Elastic File System)
- D. Instance store

**Answer: C**  
**Explanation:** Amazon EFS provides a fully managed Network File System (NFS) accessible by thousands of EC2 instances simultaneously across AZs. EBS is block storage attached to a single instance (Multi-Attach has limitations). S3 is object storage requiring API calls, not a file system mount.

---

**Q43.** An application needs to trigger processing whenever a new image is uploaded to S3. The processing must complete within 15 minutes. Which architecture is most appropriate?

- A. EC2 polling S3 with a cron job
- B. S3 Event Notification → SQS → Lambda
- C. S3 Event Notification → Lambda directly
- D. CloudWatch Events → Lambda

**Answer: C**  
**Explanation:** S3 Event Notifications can invoke Lambda directly when objects are created. Lambda handles the processing within its 15-minute timeout. Adding SQS in between (option B) is recommended when you need buffering for high-volume events or retry logic, but for straightforward cases, direct invocation is simpler.

---

**Q44.** A company uses an on-premises data center and needs a dedicated, private, high-bandwidth connection to AWS with consistent latency for a production workload. Which connectivity option should they use?

- A. AWS Site-to-Site VPN
- B. AWS Client VPN
- C. AWS Direct Connect
- D. AWS Transit Gateway with VPN

**Answer: C**  
**Explanation:** AWS Direct Connect provides a dedicated physical network connection from the data center to AWS with consistent low latency and high bandwidth. Site-to-Site VPN goes over the public internet and has variable latency — it's a cost-effective backup but not suitable for latency-sensitive production workloads.

---

**Q45.** A disaster recovery strategy requires minimal cost while maintaining a copy of the application's database in AWS, with an RTO of 4 hours and RPO of 1 hour. Which DR pattern and AWS configuration is most appropriate?

- A. Multi-site active/active with RDS Multi-AZ
- B. Warm standby with EC2 ASG and RDS Read Replica
- C. Pilot light with replicated RDS and pre-built AMIs
- D. Backup and restore with automated RDS snapshots every hour

**Answer: C**  
**Explanation:** Pilot light keeps a minimal version running — an RDS replica receives continuous replication (RPO ≤ minutes), and EC2 AMIs are pre-built. During recovery, the RDS replica is promoted and EC2 instances are launched (within 4 hours). This meets the RTO/RPO at minimal cost vs. warm standby.

---

**Q46.** An application writes 1 MB objects to S3 thousands of times per second. What S3 feature improves write performance automatically?

- A. S3 Transfer Acceleration
- B. S3 Multi-part Upload
- C. S3 automatic prefix-based partitioning (no action needed after 2018 update)
- D. S3 Intelligent-Tiering

**Answer: C**  
**Explanation:** Since 2018, S3 automatically scales to at least 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests per second per prefix. No special key naming is required. S3 handles partitioning transparently. Multipart upload is for individual large objects (>100 MB).

---

**Q47.** A company's application uses API Gateway + Lambda + DynamoDB. During a spike, DynamoDB writes are throttled. What is the recommended solution?

- A. Enable DynamoDB Auto Scaling
- B. Switch to RDS
- C. Use DynamoDB Streams
- D. Increase Lambda concurrency

**Answer: A**  
**Explanation:** DynamoDB Auto Scaling adjusts provisioned read/write capacity in response to traffic changes using CloudWatch alarms. Alternatively, switching to On-Demand mode eliminates throttling entirely for unpredictable workloads. Lambda concurrency does not affect DynamoDB capacity.

---

**Q48.** Which AWS service enables you to run containers without managing servers or clusters?

- A. Amazon ECS with EC2 launch type
- B. Amazon EKS with managed node groups
- C. AWS Fargate
- D. AWS Elastic Beanstalk

**Answer: C**  
**Explanation:** AWS Fargate is a serverless compute engine for containers. You define CPU/memory requirements and AWS manages the underlying infrastructure. There are no EC2 instances to patch, scale, or manage. It works with both ECS and EKS.

---

**Q49.** A company needs to run an ETL job every day at midnight to process data from RDS and store results in S3. The job completes in 30 minutes. What is the most cost-effective solution?

- A. EC2 instance running 24/7 with a cron job
- B. AWS Glue ETL job triggered by EventBridge Scheduler
- C. ECS task triggered by EventBridge Scheduler
- D. Lambda function (note: 15-min max timeout)

**Answer: B**  
**Explanation:** AWS Glue is a serverless ETL service that charges only for resources used during the job run. EventBridge Scheduler triggers it daily at midnight. No infrastructure to manage. For a 30-minute job, Glue is cost-effective and purpose-built for ETL. Lambda's 15-minute limit would be a concern.

---

**Q50.** A Solutions Architect needs to design a system that processes financial transactions and requires exactly-once execution with strict ordering. Which combination is correct?

- A. SQS Standard + Lambda
- B. SQS FIFO + Lambda with `MessageGroupId`
- C. SNS + SQS Standard + Lambda
- D. Kinesis Data Streams + Lambda

**Answer: B**  
**Explanation:** SQS FIFO queues guarantee exactly-once processing and strict ordering within a message group (defined by `MessageGroupId`). Financial transactions must be ordered and not duplicated. SQS Standard offers at-least-once delivery with best-effort ordering, which is unsuitable for financial transactions.

---

## Additional Key Concept Questions

---

**Q51.** What is the primary difference between a Security Group and a Network ACL in AWS?

- A. Security Groups are stateless; NACLs are stateful
- B. Security Groups are stateful; NACLs are stateless
- C. Security Groups apply to VPCs; NACLs apply to subnets
- D. NACLs only support Allow rules

**Answer: B**  
**Explanation:** Security Groups are stateful — return traffic is automatically allowed. NACLs are stateless — both inbound and outbound rules must explicitly allow return traffic. Security Groups act at the instance level; NACLs act at the subnet level.

---

**Q52.** An EC2 instance needs to access S3 without going through the internet. The instance is in a private subnet. What is the most cost-effective solution?

- A. NAT Gateway
- B. VPC Gateway Endpoint for S3
- C. VPC Interface Endpoint for S3
- D. AWS Direct Connect

**Answer: B**  
**Explanation:** VPC Gateway Endpoints for S3 and DynamoDB are free. They route traffic from private subnets to S3 through the AWS network without NAT Gateway data processing charges. Interface Endpoints (PrivateLink) cost per hour and per GB.

---

**Q53.** Which AWS storage service is most appropriate for storing and sharing NFS-based files across multiple Linux EC2 instances in the same VPC?

- A. Amazon S3
- B. Amazon EBS
- C. Amazon EFS
- D. Amazon FSx for Windows File Server

**Answer: C**  
**Explanation:** Amazon EFS provides a POSIX-compliant NFS file system that can be mounted on multiple Linux instances simultaneously across AZs. EBS can only be attached to one instance at a time (with limited Multi-Attach). FSx for Windows is SMB-based.

---

**Q54.** A company wants to migrate a large volume of data from on-premises NAS to Amazon S3 on a regular weekly schedule. Which service is most suitable?

- A. AWS Snowball
- B. AWS DataSync
- C. AWS Transfer Family
- D. S3 Transfer Acceleration

**Answer: B**  
**Explanation:** AWS DataSync automates and accelerates online data transfers between on-premises storage and AWS services. It supports NFS and SMB sources, handles scheduling, encryption, and integrity verification. Snowball is for one-time large offline migrations.

---

**Q55.** An application stores data in S3. The compliance team requires that deleted objects can be recovered for up to 30 days. What is the simplest solution?

- A. Enable S3 CRR (Cross-Region Replication)
- B. Enable S3 Versioning and configure a lifecycle rule to expire noncurrent versions after 30 days
- C. Enable MFA Delete
- D. Enable S3 Intelligent-Tiering

**Answer: B**  
**Explanation:** S3 Versioning preserves all versions of an object including delete markers. A lifecycle policy can automatically expire (delete) noncurrent versions after 30 days to manage storage costs. This enables recovery of accidentally deleted objects within the 30-day window.

---

**Q56.** A company uses CloudFormation to deploy infrastructure. They want to reuse template components across multiple stacks. Which CloudFormation feature enables this?

- A. CloudFormation Mappings
- B. CloudFormation Nested Stacks
- C. CloudFormation StackSets
- D. CloudFormation Parameters

**Answer: B**  
**Explanation:** CloudFormation Nested Stacks allow modular, reusable templates. A parent stack references child stacks (e.g., VPC stack, security group stack), enabling DRY (Don't Repeat Yourself) design. StackSets deploy the same stack to multiple accounts/regions.

---

**Q57.** An application on EC2 needs to connect to an RDS database. The connection string should not be stored in the application code or environment variables. What is the recommended approach?

- A. Store the connection string in S3 and read it at startup
- B. Use AWS Secrets Manager and retrieve the secret at runtime via SDK
- C. Pass the connection string through EC2 user data
- D. Store the connection string in CloudFormation outputs

**Answer: B**  
**Explanation:** AWS Secrets Manager stores credentials encrypted and provides automatic rotation. The EC2 instance (via IAM role) retrieves the secret at runtime using the AWS SDK. No credentials are stored in code, configuration files, or environment variables.

---

**Q58.** Which CloudWatch metric can be used to scale an EC2 Auto Scaling Group based on SQS queue depth?

- A. `AWS/SQS:ApproximateNumberOfMessagesVisible`
- B. `AWS/EC2:CPUUtilization`
- C. `AWS/ApplicationELB:RequestCount`
- D. `AWS/SQS:NumberOfMessagesSent`

**Answer: A**  
**Explanation:** `ApproximateNumberOfMessagesVisible` is the number of messages available in the queue. A custom CloudWatch alarm or target tracking policy using this metric triggers Auto Scaling to add EC2 instances when the queue grows, processing backlog faster.

---

**Q59.** A company has two VPCs in different regions that need to communicate privately. What is the most appropriate solution?

- A. VPC Peering (inter-region)
- B. AWS Transit Gateway with inter-region peering
- C. Site-to-Site VPN between VPCs
- D. Both A and B are valid, Transit Gateway is better for multiple VPCs

**Answer: D**  
**Explanation:** Both work for two VPCs. VPC Peering is simpler and cheaper for two VPCs. AWS Transit Gateway Inter-Region Peering scales better when connecting multiple VPCs because it avoids the complexity of a full mesh peering topology.

---

**Q60.** After a failover, a Route 53 health-check-based failover routing policy should switch DNS to the secondary endpoint. The primary endpoint has recovered. How does Route 53 handle the return to primary?

- A. Manual intervention is required to switch back
- B. Route 53 automatically detects the healthy primary via health checks and switches back
- C. DNS TTL must expire and be manually updated
- D. CloudWatch alarm manually triggers the failover back

**Answer: B**  
**Explanation:** Route 53 health checks continuously monitor the primary endpoint. Once it becomes healthy again, Route 53 automatically resumes routing to the primary record without manual intervention. TTL affects how quickly DNS propagates, but the routing decision is automatic.

---

## Quick Reference — Key AWS Service Comparisons

| Scenario | Best Service |
|---|---|
| Object storage | S3 |
| Block storage for single EC2 | EBS |
| Shared NFS across EC2 | EFS |
| Shared Windows file system | FSx for Windows |
| Serverless containers | Fargate |
| Container orchestration | ECS / EKS |
| Serverless functions | Lambda |
| Relational DB managed | RDS / Aurora |
| NoSQL managed | DynamoDB |
| In-memory cache | ElastiCache (Redis / Memcached) |
| Time-series data | Timestream |
| Data warehouse / analytics | Redshift |
| Serverless SQL on S3 | Athena |
| Search | OpenSearch Service |
| Streaming data ingest | Kinesis Data Streams |
| Stream → S3 delivery | Kinesis Data Firehose |
| Async message queue | SQS |
| Pub/Sub notifications | SNS |
| Event bus | EventBridge |
| CDN / Edge caching | CloudFront |
| DNS management | Route 53 |
| Private dedicated line | Direct Connect |
| Encrypted VPN over internet | Site-to-Site VPN |
| API management | API Gateway |
| Infrastructure as Code | CloudFormation / CDK |
| Config compliance | AWS Config |
| API audit log | CloudTrail |
| Metrics & alarms | CloudWatch |
| App performance tracing | X-Ray |
| Secret storage | Secrets Manager |
| Config parameters | SSM Parameter Store |
| Encryption keys | KMS |
| DDoS protection | Shield (Standard/Advanced) |
| WAF (L7 protection) | AWS WAF |
| CSPM / security findings | Security Hub |
| Threat detection | GuardDuty |
| Sensitive data detection | Macie |
| Centralized multi-account | AWS Organizations + SCPs |
| Cost analysis | Cost Explorer |
| Right-sizing recommendations | Compute Optimizer |
| Cost anomaly detection | AWS Cost Anomaly Detection |

---

## AWS CLI Command Reference — aws-operation-command.md Coverage

| # | Service | Key Commands Covered |
|---|---|---|
| 0 | Prerequisites | Install CLI v2, configure profiles, verify identity |
| 1 | IAM | Users, groups, policies, roles, instance profiles, password policy |
| 2 | EC2 | Launch, AMIs, security groups, Elastic IPs, EBS volumes, snapshots |
| 3 | VPC | Subnets, IGW, Route Tables, NAT Gateway, Peering, VPC Endpoints |
| 4 | S3 | Buckets, encryption, versioning, sync, presigned URLs, lifecycle |
| 5 | RDS | Multi-AZ, snapshots, restore, modify, deletion protection |
| 6 | ECS | Clusters, task definitions, services, Fargate deployments |
| 7 | EKS | Create cluster, node groups, kubeconfig, version updates |
| 8 | Lambda | Deploy, invoke, versions, aliases, triggers |
| 9 | ALB/NLB | Target groups, listeners, health checks, HTTPS |
| 10 | Auto Scaling | Launch templates, ASG, target tracking, maintenance |
| 11 | Route 53 | Hosted zones, A records, Alias, health checks |
| 12 | CloudFront | Distributions, cache invalidation |
| 13 | SQS | Standard, FIFO, DLQ, purge |
| 14 | SNS | Topics, subscriptions, fan-out |
| 15 | CloudWatch | Alarms, metrics, log groups, tail, filter |
| 16 | CloudFormation | Deploy, change sets, diff, wait, outputs |
| 17 | Secrets Manager + SSM | Create, retrieve, rotate, decode |
| 18 | KMS | Create keys, encrypt/decrypt data |
| 19 | ACM | Request, import, DNS validation |
| 20 | Systems Manager | Session Manager, run commands on fleets |
| 21 | Cost & Billing | Usage reports, budget alerts |

---

## Exam Domain Coverage Summary

| Domain | Questions | Topics Covered |
|---|---|---|
| Domain 1 – Secure Architectures (30%) | Q1–Q10 | Encryption, IAM roles, SCPs, WAF, CloudTrail, permission boundaries, VPC endpoints |
| Domain 2 – Resilient Architectures (26%) | Q11–Q20 | DR strategies, Multi-AZ, session state, SQS FIFO, DynamoDB On-Demand, Global Accelerator |
| Domain 3 – High-Performing Architectures (24%) | Q21–Q30 | Read replicas, ElastiCache, CloudFront, Kinesis, Redshift, WebSocket API |
| Domain 4 – Cost-Optimized Architectures (20%) | Q31–Q40 | Spot, Reserved, Savings Plans, Glacier, VPC Endpoints, Athena, Compute Optimizer |
| Mixed Architecture Scenarios | Q41–Q50 | Three-tier design, EFS, ETL with Glue, FIFO transactions, Fargate |
| Key Concepts | Q51–Q60 | SGs vs NACLs, S3 versioning, Nested Stacks, Secrets Manager, Route 53 failover |
| Quick Reference Table | — | 40+ service-to-use-case mappings |