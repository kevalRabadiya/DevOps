# AWS Services: Practical Overview & Blog Guide

Quick reference guide for AWS services with practical use cases and blog resources.

***

### Table of Contents

1. [Compute Services](./#table-of-contents)
2. Storage Services
3. Database Services
4. Networking & Content Delivery
5. Security & Identity
6. Developer Tools
7. Application Integration
8. Analytics & Big Data
9. Machine Learning
10. Management & Monitoring

***

### Compute Services

#### EC2 - Elastic Compute Cloud

**What it does:** Virtual servers in the cloud

**Practical Uses:**

* Web servers, application servers
* Batch processing, HPC workloads
* Development/testing environments

**Blog Topics:**

* EC2 instance types and right-sizing
* Auto Scaling groups for high availability
* Security groups and NACLs configuration
* Cost optimization strategies

***

#### Lambda - Serverless Functions

**What it does:** Run code without managing servers

**Practical Uses:**

* Event-driven processing (S3 uploads, API calls)
* Scheduled jobs (cron-like tasks)
* API backends, microservices
* Data transformation pipelines

**Blog Topics:**

* Lambda vs EC2: when to choose
* Building REST APIs with API Gateway + Lambda
* Cost optimization for Lambda
* Cold start problem and solutions

***

#### ECS / EKS - Container Orchestration

**What it does:** Run containerized applications

**Practical Uses:**

* Docker container deployment
* Microservices architecture
* CI/CD pipelines
* Multi-container applications

**Blog Topics:**

* ECS vs EKS comparison
* Task definitions and services
* Container security best practices
* Autoscaling containers

***

#### Elastic Beanstalk

**What it does:** PaaS for deploying web applications

**Practical Uses:**

* Quick app deployment
* Simplified DevOps
* Multi-tier application hosting

**Blog Topics:**

* Deploying Django/Node.js apps
* Environment configuration
* Rolling deployments

***

### Storage Services

#### S3 - Simple Storage Service

**What it does:** Object storage service

**Practical Uses:**

* Static website hosting
* Data backups and archives
* Media files (images, videos)
* Data lakes for analytics

**Blog Topics:**

* S3 storage classes explained
* Lifecycle policies for cost optimization
* S3 access control (bucket policies, IAM)
* Versioning and replication
* Pre-signed URLs for secure sharing

***

#### EBS - Elastic Block Storage

**What it does:** Persistent block storage for EC2

**Practical Uses:**

* Database volumes
* Application data storage
* High-performance computing

**Blog Topics:**

* EBS volume types (gp3, io1, etc.)
* Snapshots and backups
* EBS encryption
* Performance optimization

***

#### EFS - Elastic File System

**What it does:** Managed NFS file system

**Practical Uses:**

* Shared storage for multiple EC2 instances
* Content management systems
* Big data analysis

**Blog Topics:**

* EFS vs EBS vs S3
* Access control with security groups
* Performance modes and throughput

***

#### Glacier / Deep Archive

**What it does:** Long-term data archival

**Practical Uses:**

* Compliance data retention
* Disaster recovery
* Cost-effective backups

**Blog Topics:**

* Storage class transitions
* Retrieval times and costs
* Lifecycle policies

***

### Database Services

#### RDS - Relational Database Service

**What it does:** Managed relational databases

**Practical Uses:**

* Web application databases
* OLTP workloads
* Multi-AZ high availability

**Blog Topics:**

* Choosing RDS engine (MySQL, PostgreSQL, MariaDB)
* Read replicas for scaling
* Backup strategies
* Performance tuning
* Multi-AZ failover explained

***

#### DynamoDB

**What it does:** Managed NoSQL database

**Practical Uses:**

* Real-time applications
* IoT data ingestion
* User sessions, shopping carts
* Time-series data

**Blog Topics:**

* Primary keys design (partition + sort key)
* On-demand vs provisioned capacity
* DynamoDB streams for real-time processing
* Global tables for multi-region
* Query vs Scan operations

***

#### ElastiCache

**What it does:** In-memory data store (Redis/Memcached)

**Practical Uses:**

* Session storage
* Caching frequently accessed data
* Real-time leaderboards
* Message queues

**Blog Topics:**

* Redis vs Memcached
* Cache invalidation strategies
* High availability setup
* Cost optimization

***

#### DocumentDB / MongoDB

**What it does:** Document-oriented NoSQL database

**Practical Uses:**

* Flexible schema applications
* Content management
* Product catalogs

**Blog Topics:**

* MongoDB vs DynamoDB
* Document structure design
* Indexing strategies

***

### Networking & Content Delivery

#### VPC - Virtual Private Cloud

**What it does:** Isolated network environment

**Practical Uses:**

* Network isolation and security
* Multi-tier application architecture
* Hybrid connectivity

**Blog Topics:**

* Subnet design (public/private)
* Route tables and NAT gateways
* Security groups and NACLs
* VPC peering and Transit Gateway

***

#### CloudFront

**What it does:** Content Delivery Network (CDN)

**Practical Uses:**

* Faster content delivery globally
* DDoS protection
* API acceleration
* Live streaming

**Blog Topics:**

* Cache behaviors and TTL
* Origin configurations
* Geographic restrictions
* Invalidation strategies

***

#### ALB / NLB - Load Balancers

**What it does:** Distribute traffic across instances

**Practical Uses:**

* High availability
* Load balancing
* API Gateway alternative
* SSL/TLS termination

**Blog Topics:**

* ALB vs NLB vs CLB
* Health checks configuration
* Sticky sessions
* Target groups

***

#### Route 53

**What it does:** DNS and domain registration

**Practical Uses:**

* Domain name management
* Traffic routing policies
* Health checks and failover
* Alias records for AWS resources

**Blog Topics:**

* Routing policies (simple, weighted, latency)
* Health checks
* Private hosted zones
* Cost optimization

***

### Security & Identity

#### IAM - Identity and Access Management

**What it does:** Manage users, roles, and permissions

**Practical Uses:**

* User access control
* Service-to-service authentication
* Cross-account access
* Temporary credentials

**Blog Topics:**

* IAM policies explained
* Roles vs users
* Least privilege principle
* Service roles for EC2/Lambda
* MFA setup

***

#### Secrets Manager

**What it does:** Securely store and rotate secrets

**Practical Uses:**

* Database passwords
* API keys
* OAuth tokens
* Certificate management

**Blog Topics:**

* Secret rotation
* Cross-service access
* Cost comparison with Parameter Store

***

#### ACM - AWS Certificate Manager

**What it does:** SSL/TLS certificate management

**Practical Uses:**

* HTTPS for websites
* API security
* Domain validation

**Blog Topics:**

* Public vs private certificates
* Auto-renewal
* Certificate pinning

***

#### GuardDuty / Security Hub

**What it does:** Threat detection and security monitoring

**Practical Uses:**

* Detecting unauthorized activity
* Compliance monitoring
* Centralized security view

**Blog Topics:**

* Threat findings interpretation
* Integration with other services
* Response automation

***

### Developer Tools

#### CodeCommit

**What it does:** Git repository service

**Practical Uses:**

* Source code management
* Team collaboration
* Git workflows

**Blog Topics:**

* Branching strategies
* IAM access control
* Pull request reviews

***

#### CodePipeline / CodeBuild / CodeDeploy

**What it does:** CI/CD automation

**Practical Uses:**

* Automated testing and deployment
* Release pipelines
* Blue-green deployments

**Blog Topics:**

* Pipeline stage configuration
* CodeBuild environment setup
* Deployment strategies
* Artifact management

***

#### CloudFormation

**What it does:** Infrastructure as Code (IaC)

**Practical Uses:**

* Automated infrastructure creation
* Stack management
* Environment consistency
* Disaster recovery

**Blog Topics:**

* Template syntax (JSON/YAML)
* Stack creation and updates
* Best practices for IaC
* Rollback strategies

***

#### AWS SAM - Serverless Application Model

**What it does:** Framework for serverless applications

**Practical Uses:**

* Faster Lambda development
* Local testing
* Resource bundling

**Blog Topics:**

* SAM template syntax
* Local debugging
* Deployment automation

***

### Application Integration

#### SNS - Simple Notification Service

**What it does:** Publish-subscribe messaging

**Practical Uses:**

* Email/SMS notifications
* Event distribution
* Alerting systems
* Fan-out patterns

**Blog Topics:**

* Topics and subscriptions
* Message filtering
* SMS delivery
* Email notifications

***

#### SQS - Simple Queue Service

**What it does:** Message queue service

**Practical Uses:**

* Decoupling applications
* Async task processing
* Worker pools
* Rate limiting

**Blog Topics:**

* Standard vs FIFO queues
* Visibility timeout
* Message retention
* Scaling workers

***

#### EventBridge

**What it does:** Event routing and transformation

**Practical Uses:**

* Event-driven architecture
* Cross-service integration
* Scheduled events
* Event filtering and routing

**Blog Topics:**

* Event patterns and rules
* Custom events
* Cross-account events
* Transformation with input paths

***

#### API Gateway

**What it does:** Managed API service

**Practical Uses:**

* REST/WebSocket APIs
* Microservices gateway
* Rate limiting and throttling
* Request/response transformation

**Blog Topics:**

* API stages and deployments
* Authentication (API keys, Cognito)
* CORS configuration
* Caching strategies

***

### Analytics & Big Data

#### Athena

**What it does:** Query data in S3 using SQL

**Practical Uses:**

* Ad-hoc data analysis
* Log analysis
* Cost-effective data querying
* Data exploration

**Blog Topics:**

* Partitioning for performance
* Glue Data Catalog
* Query optimization
* Cost optimization

***

#### Redshift

**What it does:** Data warehouse

**Practical Uses:**

* Business analytics
* OLAP workloads
* Historical data analysis
* BI tool integration

**Blog Topics:**

* Distribution styles
* Sort keys and compression
* Spectrum for S3 querying
* Performance tuning

***

#### Glue

**What it does:** ETL service

**Practical Uses:**

* Data catalog management
* ETL job automation
* Data transformation
* Data discovery

**Blog Topics:**

* Crawlers and catalogs
* Job development
* Data quality checks

***

#### Kinesis

**What it does:** Real-time data streaming

**Practical Uses:**

* Real-time analytics
* Log stream processing
* IoT data ingestion
* Live dashboards

**Blog Topics:**

* Streams vs Firehose
* Shards and scaling
* Data retention
* Consumer applications

***

### Machine Learning

#### SageMaker

**What it does:** Managed ML service

**Practical Uses:**

* Model development and training
* Real-time predictions
* Batch predictions
* AutoML capabilities

**Blog Topics:**

* Built-in algorithms
* Notebook instances
* Training and tuning
* Model deployment

***

#### Rekognition

**What it does:** Image and video analysis

**Practical Uses:**

* Content moderation
* Object detection
* Face recognition
* Text detection (OCR)

**Blog Topics:**

* Image vs video detection
* Custom labels
* Confidence thresholds
* Use case implementations

***

#### Comprehend

**What it does:** Natural language processing

**Practical Uses:**

* Sentiment analysis
* Entity recognition
* Key phrase extraction
* Topic modeling

**Blog Topics:**

* Entity recognition
* Sentiment analysis setup
* Language detection
* Custom classification

***

### Management & Monitoring

#### CloudWatch

**What it does:** Monitoring and observability

**Practical Uses:**

* Metrics and alarms
* Log aggregation
* Custom dashboards
* Application performance monitoring

**Blog Topics:**

* Custom metrics
* Log groups and streams
* Alarms and notifications
* Dashboard creation

***

#### CloudTrail

**What it does:** API audit logging

**Practical Uses:**

* Compliance auditing
* Security investigation
* Change tracking
* Account activity logging

**Blog Topics:**

* CloudTrail configuration
* Log analysis
* Multi-account trails
* Integration with Athena

***

#### Systems Manager

**What it does:** Systems operations and management

**Practical Uses:**

* Parameter management
* Document automation
* Patch management
* Session management

**Blog Topics:**

* Parameter Store
* Automation documents
* Fleet management
* Secure sessions

***

#### Cost Explorer / Budgets

**What it does:** Cost management and optimization

**Practical Uses:**

* Cost analysis
* Budget alerts
* Right-sizing recommendations
* Reserved Instance recommendations

**Blog Topics:**

* Cost allocation tags
* Budget setting
* Cost anomaly detection
* Savings plans vs RIs

***

### Quick Start Paths

#### 🚀 Building a Web Application

1. **Compute:** EC2 or Lambda
2. **Database:** RDS or DynamoDB
3. **Storage:** S3
4. **CDN:** CloudFront
5. **DNS:** Route 53

#### 📊 Data Analytics Pipeline

1. **Ingestion:** Kinesis or S3
2. **Processing:** Glue or Lambda
3. **Storage:** Redshift or S3 Data Lake
4. **Querying:** Athena
5. **Visualization:** QuickSight

#### 🔒 Enterprise Security

1. **Identity:** IAM + Cognito
2. **Secrets:** Secrets Manager
3. **Certificates:** ACM
4. **Monitoring:** GuardDuty + Security Hub
5. **Audit:** CloudTrail

#### 📱 Serverless Application

1. **API:** API Gateway + Lambda
2. **Events:** EventBridge
3. **Queues:** SQS/SNS
4. **Data:** DynamoDB
5. **Storage:** S3

***

### Learning Resources

* **Official AWS Documentation:** https://docs.aws.amazon.com
* **AWS Training & Certification:** https://aws.amazon.com/training
* **AWS Well-Architected Framework:** https://aws.amazon.com/architecture/well-architected
* **AWS Free Tier:** https://aws.amazon.com/free
* **AWS Whitepapers:** https://aws.amazon.com/whitepapers

***

### Key Best Practices

✅ **Security First** - Use IAM roles, encryption, security groups\
✅ **High Availability** - Use multiple AZs, load balancers, failover\
✅ **Cost Optimization** - Right-size resources, use reserved instances, monitor usage\
✅ **Automation** - Use CloudFormation/SAM, CodePipeline, Infrastructure as Code\
✅ **Monitoring** - Enable CloudWatch, CloudTrail, set up alarms\
✅ **Scalability** - Design for growth, use auto-scaling, serverless where possible

***

### Common Service Combinations

| Use Case          | Services                              |
| ----------------- | ------------------------------------- |
| **Web App**       | VPC, ALB, EC2, RDS, S3, CloudFront    |
| **API Backend**   | API Gateway, Lambda, DynamoDB, SQS    |
| **Data Pipeline** | S3, Glue, Redshift, Athena            |
| **Real-time App** | Kinesis, Lambda, DynamoDB, CloudWatch |
| **Mobile App**    | Cognito, API Gateway, Lambda, S3      |
| **Microservices** | ECS/EKS, ALB, SQS/SNS, CloudWatch     |

***

### Tips for Success

1. **Start Small** - Master one service before adding complexity
2. **Use Free Tier** - Experiment without cost
3. **Read Docs** - AWS documentation is comprehensive
4. **Monitor Costs** - Set up budgets and alerts early
5. **Follow Well-Architected Framework** - Best practices guide
6. **Practice IaC** - Use CloudFormation/Terraform from the start
7. **Enable Logging** - CloudTrail, VPC Flow Logs, Application Logs

***

**Last Updated:** 2026\
**AWS Services Covered:** 40+

For detailed implementation guides, check the individual service blogs in this documentation.
