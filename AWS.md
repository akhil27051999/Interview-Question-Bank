# DevOps Interview Preparation Guide — AWS (Reformatted)

A comprehensive collection of real-time and technical interview questions focused on AWS for DevOps roles.  
Each question is shown as a numbered heading (Q#). Answers are placed under a bold **Answer** subheading. Command examples, JSON snippets, and short tables are included in fenced code blocks for clarity.

---

## Table of Contents
- [Category 1: AWS Fundamentals & Global Infrastructure](#category-1-aws-fundamentals--global-infrastructure)
- [Category 2: Compute Services](#category-2-compute-services)
- [Category 3: Storage Services](#category-3-storage-services)
- [Category 4: Database Services](#category-4-database-services)
- [Category 5: Networking & Content Delivery](#category-5-networking--content-delivery)
- [Category 6: Security & Identity](#category-6-security--identity)
- [Category 7: Monitoring & Management](#category-7-monitoring--management)
- [Category 8: Application Services](#category-8-application-services)
- [Category 9: Cost Optimization](#category-9-cost-optimization)
- [Category 10: Migration & Hybrid Cloud](#category-10-migration--hybrid-cloud)
- [Category 11: Real-time Troubleshooting Scenarios](#category-11-real-time-troubleshooting-scenarios)

---

## AWS

---

### Category 1: AWS Fundamentals & Global Infrastructure

#### Q1 — What is AWS and explain its key services?
**Answer**

AWS (Amazon Web Services) is a comprehensive cloud computing platform offering over 200 services globally.

Key Services:
- Compute: EC2, Lambda, ECS  
- Storage: S3, EBS, EFS  
- Database: RDS, DynamoDB, Redshift  
- Networking: VPC, CloudFront, Route 53  
- Security: IAM, KMS, Shield

---

#### Q2 — Explain AWS Global Infrastructure components.
**Answer**

- **Regions:** Geographically separate areas (e.g., `us-east-1`, `eu-west-1`)  
- **Availability Zones (AZs):** Isolated data centers within regions  
- **Edge Locations:** Points of presence for CloudFront and Route 53  
- **Regional Edge Caches:** Between origin and edge locations

---

#### Q3 — What is the difference between Regions and Availability Zones?
**Answer**

- **Regions:** Separate geographical areas, completely isolated  
- **Availability Zones:** Multiple isolated locations (data centers) within a region

Best Practice: Deploy across multiple AZs for high availability.

---

#### Q4 — What is the AWS Shared Responsibility Model?
**Answer**

- **AWS responsibility:** Security *OF* the cloud (infrastructure, hardware, host OS)  
- **Customer responsibility:** Security *IN* the cloud (data, applications, IAM, guest OS)  
- **Shared:** Some services and controls are shared depending on the service model (IaaS, PaaS, SaaS)

---

### Category 2: Compute Services

#### Q5 — Explain EC2 instance types and use cases.
**Answer**

Examples of instance families and typical use cases:
- **General Purpose (M5, T3):** Web servers, small databases  
- **Compute Optimized (C5):** Batch processing, gaming servers  
- **Memory Optimized (R5, X1):** In-memory databases, real-time analytics  
- **Storage Optimized (I3, D2):** NoSQL databases, data warehousing  
- **Accelerated Computing (P3, G4):** Machine learning, graphics rendering

---

#### Q6 — What is AWS Lambda and when to use it?
**Answer**

Serverless compute service that runs code in response to events.

Use Cases:
- Real-time file processing  
- Backends for mobile/web applications  
- Scheduled tasks (cron jobs)  
- IoT data processing

---

#### Q7 — Difference between EC2, ECS, and EKS?
**Answer**

- **EC2:** Virtual servers, full control, manual management  
- **ECS:** AWS-managed container orchestration  
- **EKS:** Managed Kubernetes service (Kubernetes ecosystem)

Choose EC2 for traditional apps, ECS for simple container workloads, EKS for Kubernetes workloads.

---

#### Q8 — What are Auto Scaling Groups and how do they work?
**Answer**

Automatically adjusts the number of EC2 instances based on demand.

Components:
- Launch configuration/template  
- Minimum, maximum, desired capacity  
- Scaling policies (target tracking, step scaling)  
- Health checks

---

### Category 3: Storage Services

#### Q9 — Compare S3, EBS, and EFS storage services.
**Answer**

- **S3:** Object storage, virtually unlimited, highly durable (99.999999999%)  
- **EBS:** Block storage for EC2, persistent per-instance volumes  
- **EFS:** Managed NFS file system, shared across instances

Use Cases: S3 for backups/static websites, EBS for EC2-attached databases, EFS for shared file systems.

---

#### Q10 — What are S3 storage classes and when to use each?
**Answer**

- **Standard:** Frequently accessed data  
- **Intelligent-Tiering:** Unknown access patterns  
- **Standard-IA:** Infrequently accessed, quick retrieval  
- **One Zone-IA:** Infrequent, non-critical data in a single AZ  
- **Glacier:** Archive, retrieval in minutes to hours  
- **Glacier Deep Archive:** Lowest cost, retrieval in ~12 hours

---

#### Q11 — Explain S3 Cross-Region Replication (CRR).
**Answer**

Automatically replicates objects across AWS regions.

Use Cases:
- Compliance and regulatory requirements  
- Lower latency for global users  
- Disaster recovery strategy

---

#### Q12 — What is AWS Snow Family and when to use it?
**Answer**

Physical devices for data transfer when internet is impractical.

Use Cases:
- Petabyte-scale data migration  
- Offline data processing  
- Disaster recovery  
- Edge computing in remote locations

---

### Category 4: Database Services

#### Q13 — Compare RDS, DynamoDB, and Aurora.
**Answer**

- **RDS:** Managed relational DBs (MySQL, PostgreSQL, etc.)  
- **DynamoDB:** Managed NoSQL, serverless, high scale  
- **Aurora:** MySQL/PostgreSQL-compatible, high performance and availability

Choose RDS for traditional relational needs, DynamoDB for serverless/noSQL, Aurora for high-performance relational workloads.

---

#### Q14 — What is DynamoDB and explain its key features?
**Answer**

Fully managed NoSQL database service.

Key Features:
- Automatic scaling  
- Single-digit millisecond latency  
- ACID transactions  
- Global tables (multi-region)  
- On-demand capacity mode

---

#### Q15 — Explain RDS Multi-AZ and Read Replicas.
**Answer**

- **Multi-AZ:** Synchronous replication to standby in another AZ for failover (HA).  
- **Read Replicas:** Asynchronous replication for read scaling; can be promoted to standalone.

Use Multi-AZ for availability; Read Replicas for read-heavy scaling.

---

#### Q16 — What is Amazon Redshift and when to use it?
**Answer**

Fully managed data warehouse for analytics.

Use Cases:
- Business intelligence and analytics  
- Large-scale data processing  
- Complex queries on structured data  
- Integration with BI tools (Tableau, QuickSight)

---

### Category 5: Networking & Content Delivery

#### Q17 — What is VPC and explain its components?
**Answer**

Virtual Private Cloud — isolated virtual network.

Core components:
- **Subnets:** IP ranges within an AZ  
- **Route Tables:** Traffic routing rules  
- **Internet Gateway:** Internet connectivity for public subnets  
- **NAT Gateway:** Outbound internet access for private subnets  
- **Security Groups & NACLs:** Stateful (SG) and stateless (NACL) firewalls

---

#### Q18 — Difference between Security Groups and NACLs.
**Answer**

- **Security Groups:** Stateful, instance-level, allow rules only (default deny).  
- **NACLs:** Stateless, subnet-level, support allow and deny rules.

Best Practice: Use both for defense in depth.

---

#### Q19 — What is AWS CloudFront and how does it work?
**Answer**

CDN service that caches content at edge locations to reduce latency. Integrates with S3, EC2, and load balancers; supports both static and dynamic content.

---

#### Q20 — Explain Route 53 routing policies.
**Answer**

Routing policy types:
- **Simple:** Single resource routing  
- **Weighted:** Distribute traffic by weight  
- **Latency:** Route to region with lowest latency  
- **Failover:** Active-passive routing  
- **Geolocation:** Route based on user location  
- **Multi-value:** Return multiple healthy records

---

### Category 6: Security & Identity

#### Q21 — What is IAM and explain its core components?
**Answer**

Identity and Access Management service.

Core components:
- **Users:** Individual identities  
- **Groups:** Collections of users  
- **Roles:** For AWS services and cross-account access  
- **Policies:** JSON documents defining permissions

---

#### Q22 — Explain IAM Policy types and structure.
**Answer**

Policy types:
- **Identity-based policies:** Attached to users, groups, roles  
- **Resource-based policies:** Attached to resources (e.g., S3 bucket policy)

Policy structure example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::bucket/*"
    }
  ]
}

```

#### Q23 — What is AWS KMS and how is it used?
**Answer**

AWS Key Management Service (KMS) is a managed service for creating and controlling cryptographic keys.

Common uses:
- Encrypting EBS volumes, S3 objects, RDS databases.  
- Managing customer-managed keys (CMKs) and AWS-managed keys.  
- Integrating with services (S3, EBS, RDS) and custom applications via the KMS API.  
- Audit key usage via AWS CloudTrail.

Example (KMS key policy snippet):
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "AWS": "arn:aws:iam::123456789012:role/MyRole" },
    "Action": "kms:Decrypt",
    "Resource": "*"
  }]
}
```

#### Q24 — Explain AWS WAF and Shield.
**Answer**

- **AWS WAF (Web Application Firewall):**
  - Protects web applications from common exploits (e.g., SQL injection, XSS).
  - Attach to CloudFront, Application Load Balancer (ALB), API Gateway, AppSync.
  - Supports custom rules, managed rule groups, rate-based rules, IP sets.
  - Useful for blocking malicious request patterns at the edge.

- **AWS Shield:**
  - DDoS protection service.
  - **Shield Standard:** Automatic baseline protection for all AWS customers (no extra charge).
  - **Shield Advanced:** Paid tier with enhanced detection/mitigation, cost protection, and 24/7 DDoS Response Team (DRT) access.
  - Often used together with CloudFront/ALB and WAF for layered protection.

---

#### Q25 — What is CloudWatch and what does it monitor?
**Answer**

Amazon CloudWatch is an observability service for monitoring AWS resources and applications.

Monitors:
- **Metrics:** Built-in (EC2 CPU, EBS I/O) and custom application metrics.
- **Logs:** Application/system logs (CloudWatch Logs), log-based metrics.
- **Alarms:** Trigger SNS notifications, Auto Scaling actions, or automated remediation.
- **Events:** Schedule tasks or respond to state changes (EventBridge integration).
- **Dashboards:** Visualize key metrics and logs.

Common uses: performance monitoring, alerting, log retention/search, and automated remediation.

---

#### Q26 — Compare CloudWatch and CloudTrail.
**Answer**

- **CloudWatch**
  - Focus: Operational monitoring (metrics, logs, alarms, dashboards).
  - Use: Track performance, health, and operational telemetry.
- **CloudTrail**
  - Focus: Audit and governance (API call history).
  - Use: Security auditing, compliance, who/what/when of control-plane actions.

Use both: CloudWatch for ops; CloudTrail for security/audit trails.

---

#### Q27 — What is AWS Config and why is it important?
**Answer**

AWS Config continuously records and evaluates AWS resource configurations.

Benefits:
- Historical configuration tracking (who changed what and when).
- Compliance assessment via managed/custom rules.
- Resource inventory and relationships mapping.
- Useful for security audits, troubleshooting configuration drift, and compliance reporting.

---

#### Q28 — Explain AWS Systems Manager.
**Answer**

AWS Systems Manager (SSM) provides operational tooling for managing and operating fleets (EC2 and on-prem).

Key features:
- **Run Command:** Execute commands at scale without SSH.
- **Session Manager:** Secure shell access without opening inbound ports or managing SSH keys.
- **Patch Manager:** Automated patching for fleets.
- **Parameter Store:** Hierarchical, secure storage for configuration and secrets (integrates with KMS).
- **State Manager / Automation / Inventory:** Enforce configs, automate runbooks, and collect inventory.

Use for central management, compliance, and secure access.

---

#### Q29 — What is SQS and SNS? When to use each?
**Answer**

- **SQS (Simple Queue Service)**
  - Message queuing service (standard and FIFO queues).
  - Use for decoupling services, buffering, retryable asynchronous processing.
  - Guarantees: At-least-once delivery (standard) and ordering/ exactly-once semantics (FIFO with appropriate configuration).

- **SNS (Simple Notification Service)**
  - Pub/Sub (fan-out) messaging and notifications.
  - Pushes messages to subscribers: SQS, Lambda, HTTP/S endpoints, email, SMS.
  - Use for broadcasting events to multiple consumers, notifications, fan-out patterns.

Pattern: Publish to SNS and fan-out to multiple SQS queues or Lambda consumers.

---

#### Q30 — Explain AWS Step Functions and use cases.
**Answer**

AWS Step Functions is a serverless workflow orchestration service that composes Lambda functions and other AWS services into state machines.

Features:
- Visual workflow definitions and execution history.
- Built-in error handling, retries, timeouts, and branching (Choice state).
- Parallel, Map (for-each) and Wait states for complex flows.

Use cases:
- ETL pipelines and data processing orchestrations.
- Long-running business processes with human approvals.
- Microservice orchestration and retries with backoff.
- Complex workflows requiring observability and error handling.

---

#### Q31 — What are AWS cost optimization strategies?
**Answer**

Strategies:
- Buy **Reserved Instances** or **Savings Plans** for stable workloads.
- Use **Auto Scaling** to match capacity to demand.
- Shift interruptible workloads to **Spot Instances** to cut costs.
- Right-size instances and storage regularly (use Trusted Advisor, Compute Optimizer).
- Use **S3 lifecycle policies** to tier infrequently accessed data to IA/Glacier.
- Tag resources for cost allocation and use **Cost Explorer** / budgets / alerts.
- Consolidated billing via AWS Organizations to maximize discounts.

---

#### Q32 — Explain AWS Organizations and Service Control Policies.
**Answer**

- **AWS Organizations:** Centralized management of multiple AWS accounts, consolidated billing, and organizational units (OUs).
- **Service Control Policies (SCPs):** Organization-level guardrails that restrict the maximum available permissions in member accounts (deny overrides).

Use SCPs to enforce security and compliance boundaries across accounts.

---

#### Q33 — What is AWS Direct Connect?
**Answer**

A dedicated, private network connection from on-premises locations to AWS.

Benefits:
- Lower and more consistent latency compared to internet.
- Increased bandwidth and predictable performance.
- Private connectivity (bypasses the public internet).
- Useful for hybrid cloud architectures and high-throughput workloads.

---

#### Q34 — Explain AWS Storage Gateway.
**Answer**

Hybrid storage service connecting on-premises environments with AWS storage.

Gateway types:
- **File Gateway:** Presents NFS/SMB, stores objects in S3 (file workloads).
- **Volume Gateway:** Exposes iSCSI block storage backed by EBS/S3 (cached or stored volumes).
- **Tape Gateway:** Virtual tape library backed by S3/Glacier for backup workloads.

Use for incremental migration, backups to cloud, and local caching for cloud storage.

---

#### Q35 — EC2 instance cannot connect to the Internet. How to troubleshoot?
**Answer**

Troubleshoot checklist:
1. **Security Group:** Ensure outbound rules allow required traffic/ports.  
2. **Subnet Route Table:** Confirm route to Internet Gateway (`0.0.0.0/0 -> igw`) for public subnet.  
3. **Public IP / Elastic IP:** Verify instance has a public IP or is in subnet routed via NAT for private subnets.  
4. **NACLs:** Check subnet NACLs allow outbound and inbound return traffic.  
5. **Instance OS Firewall:** Inspect `iptables`/`ufw` for blocking rules.  
6. **NAT Gateway/Instance:** For private subnets, ensure NAT is properly configured and reachable.  
7. **DNS:** Verify resolver settings (`/etc/resolv.conf`) and test `nslookup`/`dig`.

---

#### Q36 — S3 bucket access denied errors. How to resolve?
**Answer**

Steps:
1. **Check IAM permissions:** Ensure the caller principal has required `s3:*` actions.  
2. **Inspect bucket policy:** Look for denies or principal restrictions.  
3. **Object ACLs:** Verify object-level ACLs if used (modern practice favors bucket policies/IAM).  
4. **S3 Block Public Access:** Confirm global block settings are not preventing access.  
5. **KMS permissions:** If object encrypted with KMS CMK, ensure `kms:Decrypt` is allowed.  
6. **Service Control Policies (SCPs):** Ensure organization-level SCPs aren't blocking actions.  
7. **Requester Pays:** If enabled, requester must be configured to pay.

---

#### Q37 — High latency in application. How to investigate?
**Answer**

Investigation steps:
- **CloudWatch Metrics:** Check EC2, ELB/ALB, RDS, and Lambda metrics for CPU, memory, I/O, and latency.  
- **Load Balancer:** Inspect target response times and healthy/unhealthy hosts.  
- **Database:** Look for slow queries, high connections, locks, or IO saturation.  
- **Network:** Check cross-AZ traffic, VPC endpoints, NACLs, and routing issues.  
- **CDN:** Use CloudFront to cache static assets; check cache hit ratios.  
- **Tracing:** Use AWS X-Ray to track distributed latency across services.  
- **Scaling:** Ensure Auto Scaling and connection pools are sized properly.

---

#### Q38 — RDS instance running out of storage. How to handle?
**Answer**

Remediation steps:
- **Monitor:** Configure CloudWatch alarm for free storage space.  
- **Increase storage:** Modify RDS to allocate more storage (some engines allow online resize).  
- **Enable autoscaling:** Use storage autoscaling where supported.  
- **Archive/purge:** Move old data to S3 or archival store and purge unnecessary data.  
- **Optimize schema:** Archive or prune large tables and optimize indexes.  
- **Scale:** Consider vertical scaling (larger instance/IO) or read-replicas for read load separation.

---

#### Q39 — Lambda function timing out. How to debug?
**Answer**

Debug approach:
- **CloudWatch Logs:** Inspect logs for stack traces and where execution stalls.  
- **Increase timeout:** Adjust function timeout setting if workload legitimately needs more time.  
- **Code optimization:** Reduce cold-start and initialization overhead; shorten synchronous external calls.  
- **External dependencies:** Check downstream service latencies (DB, APIs); add retries/backoff or asynchronous processing.  
- **X-Ray:** Enable tracing to pinpoint slow segments.  
- **Provisioned Concurrency:** Use to reduce cold-start latency for performance-sensitive functions.

---

#### Q40 — Auto Scaling not working as expected. How to troubleshoot?
**Answer**

Troubleshooting checklist:
- **CloudWatch alarms:** Verify the metric, period, and thresholds are correct and alarms transition to ALARM state.  
- **Scaling policies:** Confirm correct policy type (target tracking, step scaling) and cooldown periods.  
- **Health checks:** Ensure ELB/EC2 health checks are set correctly and instances are reporting healthy.  
- **Capacity/quotas:** Check account quotas or AZ capacity limits that may prevent launching instances.  
- **Activity History:** Review Auto Scaling group's activity history to see failure reasons.  
- **IAM permissions:** Ensure Auto Scaling role can create/attach instances and resources.

---

If you want, I can:
- Export these formatted Q24–Q40 entries into a separate `aws/README.md`.  
- Continue formatting other AWS Q&A items or produce printable/flashcard versions.

Which would you like next?
```
