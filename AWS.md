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
