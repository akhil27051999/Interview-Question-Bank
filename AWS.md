# DevOps Interview Preparation Guide — AWS Section

A comprehensive collection of real-time and technical interview questions focused on AWS for DevOps roles.

Technologies Covered
- AWS

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

### Category 1: AWS Fundamentals & Global Infrastructure

1. **What is AWS and explain its key services?**  
Answer:  
AWS (Amazon Web Services) is a comprehensive cloud computing platform offering over 200 services globally.

Key Services:
- Compute: EC2, Lambda, ECS
- Storage: S3, EBS, EFS
- Database: RDS, DynamoDB, Redshift
- Networking: VPC, CloudFront, Route 53
- Security: IAM, KMS, Shield

2. **Explain AWS Global Infrastructure components.**  
Answer:
- Regions: Geographically separate areas (e.g., us-east-1, eu-west-1)
- Availability Zones (AZs): Isolated data centers within regions
- Edge Locations: Points of presence for CloudFront and Route 53
- Regional Edge Caches: Between origin and edge locations

3. **What is the difference between Regions and Availability Zones?**  
Answer:
- Regions: Separate geographical areas, completely isolated
- Availability Zones: Multiple isolated locations within a region

Best Practice: Deploy across multiple AZs for high availability

4. **What is the AWS Shared Responsibility Model?**  
Answer:
- AWS Responsibility: Security OF the cloud (infrastructure, hardware, software)
- Customer Responsibility: Security IN the cloud (data, platform, applications, IAM)
- Shared: Some services have shared controls

---

### Category 2: Compute Services

5. **Explain EC2 instance types and use cases.**  
Answer:
- General Purpose (M5, T3): Web servers, small databases
- Compute Optimized (C5): Batch processing, gaming servers
- Memory Optimized (R5, X1): In-memory databases, real-time analytics
- Storage Optimized (I3, D2): NoSQL databases, data warehousing
- Accelerated Computing (P3, G4): Machine learning, graphics rendering

6. **What is AWS Lambda and when to use it?**  
Answer:  
Serverless compute service that runs code in response to events.

Use Cases:
- Real-time file processing
- Backends for mobile/web applications
- Scheduled tasks (cron jobs)
- IoT data processing

7. **Difference between EC2, ECS, and EKS?**  
Answer:
- EC2: Virtual servers, full control, manual management
- ECS: Container orchestration service, AWS managed
- EKS: Managed Kubernetes service, CNCF compliant

Choice: EC2 for traditional apps, ECS for simple containers, EKS for Kubernetes ecosystem

8. **What are Auto Scaling Groups and how do they work?**  
Answer:  
Automatically adjusts the number of EC2 instances based on demand.

Components:
- Launch configuration/template
- Minimum, maximum, desired capacity
- Scaling policies (target tracking, step scaling)
- Health checks

---

### Category 3: Storage Services

9. **Compare S3, EBS, and EFS storage services.**  
Answer:
- S3: Object storage, unlimited scalability, 99.999999999% durability
- EBS: Block storage for EC2, persistent, regional
- EFS: File storage, shared across instances, NFS protocol

Use Cases: S3 for backups/static websites, EBS for databases, EFS for shared files

10. **What are S3 storage classes and when to use each?**  
Answer:
- Standard: Frequently accessed data
- Intelligent-Tiering: Unknown access patterns
- Standard-IA: Infrequently accessed, rapid access needed
- One Zone-IA: Infrequently accessed, non-critical data
- Glacier: Archives, retrieval in minutes to hours
- Glacier Deep Archive: Lowest cost, retrieval in 12 hours

11. **Explain S3 Cross-Region Replication (CRR).**  
Answer:  
Automatically replicates objects across AWS regions.

Use Cases:
- Compliance requirements
- Lower latency for global users
- Disaster recovery strategy

12. **What is AWS Snow Family and when to use it?**  
Answer:  
Physical devices for data transfer when internet is impractical.

Use Cases:
- Petabyte-scale data migration
- Offline data processing
- Disaster recovery
- Edge computing in remote locations

---

### Category 4: Database Services

13. **Compare RDS, DynamoDB, and Aurora.**  
Answer:
- RDS: Managed relational databases (MySQL, PostgreSQL, etc.)
- DynamoDB: NoSQL database, serverless, unlimited scale
- Aurora: MySQL/PostgreSQL compatible, high performance, auto-scaling

Choice: RDS for traditional apps, DynamoDB for serverless, Aurora for high performance

14. **What is DynamoDB and explain its key features?**  
Answer:  
Fully managed NoSQL database service.

Key Features:
- Automatic scaling
- Single-digit millisecond latency
- ACID transactions
- Global tables for multi-region deployment
- On-demand capacity mode

15. **Explain RDS Multi-AZ and Read Replicas.**  
Answer:
- Multi-AZ: Synchronous replication to standby instance in different AZ for failover
- Read Replicas: Asynchronous replication for read scaling, can be promoted to standalone

Use Cases: Multi-AZ for high availability, Read Replicas for read-heavy workloads

16. **What is Amazon Redshift and when to use it?**  
Answer:  
Fully managed data warehouse service.

Use Cases:
- Business intelligence and analytics
- Large-scale data processing
- Complex queries on structured data
- Integration with BI tools (Tableau, QuickSight)

---

### Category 5: Networking & Content Delivery

17. **What is VPC and explain its components?**  
Answer:  
Virtual Private Cloud - isolated network in AWS.

Components:
- Subnets: Range of IP addresses in an AZ
- Route Tables: Define traffic routing
- Internet Gateway: Connect to internet
- NAT Gateway: Outbound internet for private subnets
- Security Groups & NACLs: Firewall rules

18. **Difference between Security Groups and NACLs.**  
Answer:
- Security Groups: Stateful, operate at instance level, allow rules only
- NACLs: Stateless, operate at subnet level, allow/deny rules

Best Practice: Use both for defense in depth

19. **What is AWS CloudFront and how does it work?**  
Answer:  
Content Delivery Network (CDN) service.

How it works:
- Caches content at edge locations
- Reduces latency for end users
- Integrates with S3, EC2, Load Balancers
- Supports dynamic and static content

20. **Explain Route 53 routing policies.**  
Answer:
- Simple: Basic routing to resources
- Weighted: Distribute traffic by weight
- Latency: Route to region with lowest latency
- Failover: Active-passive setup
- Geolocation: Route based on user location
- Multi-value: Multiple healthy records

---

### Category 6: Security & Identity

21. **What is IAM and explain its core components?**  
Answer:  
Identity and Access Management service.

Components:
- Users: Individual accounts for people
- Groups: Collection of users with same permissions
- Roles: For AWS services and cross-account access
- Policies: JSON documents defining permissions

22. **Explain IAM Policy types and structure.**  
Answer:  
Policy Types:
- Identity-based policies (attached to users/groups/roles)
- Resource-based policies (attached to resources)

Policy Structure:
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

23. **What is AWS KMS and how is it used?**  
Answer:  
Key Management Service for creating and controlling encryption keys.

Use Cases:
- Encrypt EBS volumes
- Encrypt S3 objects
- Database encryption
- Secrets management

24. **Explain AWS WAF and Shield.**  
Answer:
- WAF: Web Application Firewall, protects against web exploits
- Shield: DDoS protection service

Tiers:
- Standard: Free, automatic protection
- Advanced: Paid, additional features and support

---

### Category 7: Monitoring & Management

25. **What is CloudWatch and what does it monitor?**  
Answer:  
Monitoring and observability service.

Monitors:
- Metrics: Performance data (CPU, memory, disk I/O)
- Logs: Application and system logs
- Events: Schedule automated actions
- Alarms: Notifications and auto-scaling triggers

26. **Compare CloudWatch and CloudTrail.**  
Answer:
- CloudWatch: Performance monitoring, metrics, logs
- CloudTrail: API activity logging, security auditing, compliance

Use Both: CloudWatch for performance, CloudTrail for security

27. **What is AWS Config and why is it important?**  
Answer:  
Service for assessing, auditing, and evaluating AWS resource configurations.

Importance:
- Compliance monitoring
- Change management
- Security analysis
- Resource inventory

28. **Explain AWS Systems Manager.**  
Answer:  
Management service for operational insights and actions.

Features:
- Run Command: Execute commands on instances
- Patch Manager: Automated patching
- Parameter Store: Secure storage for configuration
- Session Manager: Secure instance access

---

### Category 8: Application Services

29. **What is SQS and SNS? When to use each?**  
Answer:
- SQS: Simple Queue Service, message queuing for decoupling
- SNS: Simple Notification Service, pub/sub messaging

Use SQS: For asynchronous processing, task queues  
Use SNS: For fan-out patterns, notifications

30. **Explain AWS Step Functions and use cases.**  
Answer:  
Serverless orchestration service for workflows.

Use Cases:
- ETL pipelines
- Order processing workflows
- Microservices coordination
- Long-running business processes

---

### Category 9: Cost Optimization

31. **What are AWS cost optimization strategies?**  
Answer:
- Use Reserved Instances for predictable workloads
- Implement Auto Scaling
- Use Spot Instances for fault-tolerant workloads
- Monitor with Cost Explorer and Budgets
- Right-size instances regularly
- Use S3 lifecycle policies

32. **Explain AWS Organizations and Service Control Policies.**  
Answer:
- Organizations: Manage multiple AWS accounts
- SCPs: Central control over maximum permissions

Benefits: Consolidated billing, security compliance, resource sharing

---

### Category 10: Migration & Hybrid Cloud

33. **What is AWS Direct Connect?**  
Answer:  
Dedicated network connection from on-premises to AWS.

Benefits:
- Consistent network performance
- Reduced bandwidth costs
- Private connectivity
- Hybrid cloud architectures

34. **Explain AWS Storage Gateway.**  
Answer:  
Hybrid storage service connecting on-premises with cloud storage.

Types:
- File Gateway: S3 integration via NFS/SMB
- Volume Gateway: iSCSI block storage
- Tape Gateway: Virtual tape library for backups

---

### Category 11: Real-time Troubleshooting Scenarios

35. **EC2 instance cannot connect to the internet. How to troubleshoot?**  
Answer:
- Check Security Groups - allow outbound traffic
- Verify route table has internet gateway route
- Check if instance is in public subnet
- Verify NACL allows outbound traffic
- Check instance-level firewall rules

36. **S3 bucket access denied errors. How to resolve?**  
Answer:
- Check IAM permissions for user/role
- Verify bucket policy allows access
- Check if object is encrypted and user has KMS permissions
- Verify S3 Block Public Access settings
- Check for explicit denies in policies

37. **High latency in application. How to investigate?**  
Answer:
- Check CloudWatch metrics for EC2, RDS, ELB
- Use CloudFront for content delivery
- Check for regional latency with Global Accelerator
- Monitor database performance
- Use X-Ray for distributed tracing

38. **RDS instance running out of storage. How to handle?**  
Answer:
- Monitor with CloudWatch alarms
- Enable storage auto-scaling
- Clean up unused data
- Archive old data to S3
- Scale up instance storage (may require downtime)

39. **Lambda function timing out. How to debug?**  
Answer:
- Check CloudWatch logs for errors
- Increase timeout configuration
- Optimize code and dependencies
- Check for external API calls
- Use X-Ray for performance insights

40. **Auto Scaling not working as expected. How to troubleshoot?**  
Answer:
- Verify CloudWatch alarms are triggering
- Check scaling policy configurations
- Verify instance health checks
- Check for sufficient capacity in AZ
- Review Activity History for scaling events
