---
description: >-
  Amazon Elastic Container Service (ECS) is a fully managed container
  orchestration service that simplifies deploying, managing, and scaling
  containerized applications on AWS
---

# AWS ECS Complete Guide: Fundamentals to Advanced

***

### Table of Contents

1. ECS Fundamentals
2. Architecture & Components
3. Task Definitions
4. ECS Clusters
5. ECS Services
6. Networking & Load Balancing
7. Deployment & Scaling
8. Advanced Topics
9. Troubleshooting
10. Best Practices

***

### ECS Fundamentals

#### What is Amazon ECS?

ECS is a container orchestration platform for running, stopping, and managing containers at scale without managing EC2 instances or Kubernetes.

```
Docker Container
    │
    ├─ Package application
    ├─ Include dependencies
    ├─ Specify resources
    └─ Define environment
         │
         ↓
    Task Definition (Container Blueprint)
         │
    Upload to ECR (Amazon Container Registry)
         │
         ↓
    ECS Cluster (Compute Environment)
    ├─ EC2 Launch Type (Your EC2 instances)
    ├─ Fargate Launch Type (Serverless containers)
    └─ Hybrid Mode (Both)
         │
         ↓
    ECS Service (Manage & Scale)
    ├─ Desired Count
    ├─ Load Balancing
    ├─ Auto-scaling
    └─ Health Checks
         │
         ↓
    Running Container Instances
```

**Key Characteristics:**

* Managed container orchestration (not Kubernetes)
* Supports Docker containers
* Integrates with AWS services
* Two launch types: EC2 and Fargate
* Auto-scaling built-in
* Easy CI/CD integration

#### ECS vs Alternatives

| Feature        | ECS EC2       | ECS Fargate     | EKS           | EC2 Direct   |
| -------------- | ------------- | --------------- | ------------- | ------------ |
| Infrastructure | You manage    | AWS manages     | You manage    | You manage   |
| Learning curve | Low           | Low             | High          | Low          |
| Cost           | Moderate      | Higher          | Higher        | Low          |
| Flexibility    | High          | Limited         | Very high     | Very high    |
| Scaling        | Manual/Auto   | Auto            | Manual/Auto   | Manual       |
| Best for       | Standard apps | Serverless feel | Complex orch. | Full control |

#### Common Use Cases

| Use Case               | Solution                        |
| ---------------------- | ------------------------------- |
| Web applications       | ECS EC2 + ALB                   |
| Microservices          | ECS Fargate + Service Discovery |
| Batch processing       | ECS + CloudWatch Events         |
| APIs                   | ECS Fargate + API Gateway       |
| Real-time applications | ECS EC2 + Kinesis               |
| Data pipelines         | ECS + Step Functions            |

***

### Architecture & Components

#### ECS Architecture Overview

```
AWS Account
    │
    ├─ ECR (Container Registry)
    │  └─ Store Docker images
    │
    ├─ ECS Cluster
    │  └─ Logical grouping
    │
    ├─ EC2 Instances (Optional)
    │  └─ Container hosts
    │
    ├─ ECS Tasks
    │  ├─ Running containers
    │  └─ Based on task definitions
    │
    ├─ ECS Services
    │  ├─ Manage task lifecycle
    │  ├─ Load balancing
    │  └─ Auto-scaling
    │
    ├─ CloudWatch (Monitoring)
    │  ├─ Logs
    │  ├─ Metrics
    │  └─ Alarms
    │
    └─ IAM Roles
       ├─ Task role (Container permissions)
       └─ Execution role (Pull images, logs)
```

#### Core Components

**Cluster**

* Logical grouping of tasks
* Collection of resources
* Can span multiple AZs
* Can mix EC2 and Fargate

**Task Definition**

* Template for running Docker container
* Specifies Docker image, CPU, memory, environment
* Similar to docker-compose file
* Versioned (task-definition:1, task-definition:2)

**Task**

* Instance of a task definition
* Running container
* Has unique IP and hostname
* Can be standalone or part of service

**Service**

* Manages task lifecycle
* Maintains desired number of tasks
* Handles load balancing
* Provides auto-scaling and health checks

**Container Registry (ECR)**

* Store Docker images
* Private repository
* Integrated with ECS
* IAM-based access control

#### Launch Types

**EC2 Launch Type**

```
Your EC2 Instances
    │
    ├─ You provision
    ├─ You manage
    ├─ You scale
    └─ ECS Agent installed
         │
         ↓
    Run ECS Tasks
```

Pros: Full control, cost-effective, high performance Cons: You manage infrastructure, patching, scaling

**Fargate Launch Type**

```
AWS Manages Infrastructure
    │
    ├─ Serverless containers
    ├─ Pay per task
    ├─ Auto-scaling built-in
    └─ No EC2 management
         │
         ↓
    Run ECS Tasks
```

Pros: Simpler, no infrastructure management, easy scaling Cons: Higher cost, less control, limited customization

***

### Task Definitions

#### Task Definition Structure

Task definition is JSON document specifying how to run Docker container.

```json
{
  "family": "my-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "my-container",
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest",
      "cpu": 256,
      "memory": 512,
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8080,
          "hostPort": 8080,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "ENV",
          "value": "production"
        },
        {
          "name": "LOG_LEVEL",
          "value": "info"
        }
      ],
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789012:secret:db-password"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-app",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:8080/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ],
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskRole",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole"
}
```

#### Key Fields

| Field                   | Purpose              | Example                  |
| ----------------------- | -------------------- | ------------------------ |
| family                  | Task definition name | my-app                   |
| cpu                     | CPU units (256-4096) | 256, 512, 1024           |
| memory                  | Memory in MB         | 512, 1024, 2048          |
| containerDefinitions    | Container specs      | Array of containers      |
| networkMode             | Network config       | awsvpc, bridge, host     |
| requiresCompatibilities | Launch types         | \["FARGATE"] or \["EC2"] |

#### Container Definition

| Field            | Purpose                            |
| ---------------- | ---------------------------------- |
| name             | Container name                     |
| image            | Docker image URI                   |
| cpu              | CPU allocation (Fargate: total)    |
| memory           | Memory allocation (Fargate: total) |
| essential        | Stop task if container fails       |
| portMappings     | Port forwarding                    |
| environment      | Environment variables              |
| secrets          | Secrets from Secrets Manager       |
| logConfiguration | CloudWatch logs config             |
| healthCheck      | Container health definition        |

#### IAM Roles

**Task Execution Role**

* Permissions for ECS to manage task
* Pull images from ECR
* Write logs to CloudWatch
* Pull secrets from Secrets Manager

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchGetImage",
        "ecr:GetDownloadUrlForLayer"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:us-east-1:123456789012:log-group:/ecs/*"
    }
  ]
}
```

**Task Role**

* Permissions for application inside container
* Access AWS services (S3, DynamoDB, etc.)
* Database credentials
* API permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:Query",
        "dynamodb:GetItem"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:123456789012:table/MyTable"
    }
  ]
}
```

#### AWS CLI Commands

**Register Task Definition**

```bash
aws ecs register-task-definition \
  --cli-input-json file://task-definition.json \
  --region us-east-1
```

**List Task Definitions**

```bash
aws ecs list-task-definitions --family-prefix my-app
```

**Describe Task Definition**

```bash
aws ecs describe-task-definition \
  --task-definition my-app:1 \
  --region us-east-1
```

**Deregister Task Definition**

```bash
aws ecs deregister-task-definition \
  --task-definition my-app:1 \
  --region us-east-1
```

***

### ECS Clusters

#### Cluster Basics

Cluster is logical grouping of capacity (EC2 instances or Fargate).

```
ECS Cluster
    │
    ├─ Cluster Name: my-cluster
    ├─ Region: us-east-1
    ├─ AZs: us-east-1a, us-east-1b
    │
    ├─ EC2 Instances (Optional)
    │  ├─ Instance 1 (4GB RAM, 2 CPU)
    │  ├─ Instance 2 (4GB RAM, 2 CPU)
    │  └─ Instance 3 (8GB RAM, 4 CPU)
    │
    ├─ Fargate Capacity (Optional)
    │  └─ On-demand, serverless
    │
    ├─ Services
    │  ├─ Web Service
    │  ├─ API Service
    │  └─ Worker Service
    │
    └─ Tasks
       ├─ Running task instances
       └─ Across instances/zones
```

#### Create Cluster (AWS CLI)

**Fargate Cluster**

```bash
aws ecs create-cluster \
  --cluster-name my-fargate-cluster \
  --capacity-providers FARGATE FARGATE_SPOT \
  --default-capacity-provider-strategy capacityProvider=FARGATE,weight=1,base=1 \
  --region us-east-1
```

**EC2 Cluster**

```bash
aws ecs create-cluster \
  --cluster-name my-ec2-cluster \
  --region us-east-1
```

#### CloudFormation Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'ECS Cluster Setup'

Resources:
  # Cluster
  ECSCluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: my-app-cluster
      ClusterSettings:
        - Name: containerInsights
          Value: enabled

  # CloudWatch Log Group
  LogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: /ecs/my-app
      RetentionInDays: 30

  # EC2 Instance (optional)
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c2d06d50ce30b442  # ECS-optimized AMI
      InstanceType: t3.medium
      KeyName: my-key
      IamInstanceProfile: ecsInstanceProfile
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          echo "ECS_CLUSTER=${ECSCluster}" >> /etc/ecs/ecs.config

  # IAM Role for EC2
  ECSInstanceRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceforEC2Role

  ECSInstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Roles:
        - !Ref ECSInstanceRole

Outputs:
  ClusterArn:
    Value: !GetAtt ECSCluster.ClusterArn
  ClusterName:
    Value: !Ref ECSCluster
```

#### Cluster Monitoring

```bash
# List cluster resources
aws ecs describe-clusters \
  --clusters my-app-cluster \
  --include ATTACHMENTS,STATISTICS \
  --region us-east-1

# Get cluster statistics
aws ecs list-container-instances \
  --cluster my-app-cluster \
  --region us-east-1

# Describe container instances
aws ecs describe-container-instances \
  --cluster my-app-cluster \
  --container-instances <instance-arn> \
  --region us-east-1
```

***

### ECS Services

#### Service Basics

Service ensures specified number of tasks always running.

```
ECS Service
    │
    ├─ Task Definition: my-app:1
    ├─ Desired Count: 3
    ├─ Launch Type: FARGATE
    ├─ Network: VPC
    │
    ├─ Scheduling Strategy
    │  ├─ REPLICA (Default): Run desired count
    │  └─ DAEMON: Run one per instance
    │
    ├─ Deployment Configuration
    │  ├─ Minimum healthy: 66%
    │  ├─ Maximum: 150%
    │  └─ Deployment type: Rolling/Blue-Green
    │
    ├─ Load Balancer
    │  └─ ALB/NLB distribution
    │
    ├─ Service Discovery
    │  └─ AWS Cloud Map DNS
    │
    └─ Auto-scaling
       ├─ Min tasks: 1
       ├─ Max tasks: 10
       └─ Scale on CPU/Memory
```

#### Create Service (AWS CLI)

```bash
aws ecs create-service \
  --cluster my-app-cluster \
  --service-name my-app-service \
  --task-definition my-app:1 \
  --desired-count 3 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={
    subnets=[subnet-12345678,subnet-87654321],
    securityGroups=[sg-12345678],
    assignPublicIp=DISABLED
  }" \
  --load-balancers targetGroupArn=arn:aws:elasticloadbalancing:...,containerName=my-container,containerPort=8080 \
  --region us-east-1
```

#### CloudFormation Template

```yaml
Resources:
  ECSService:
    Type: AWS::ECS::Service
    Properties:
      ServiceName: my-app-service
      Cluster: !Ref ECSCluster
      TaskDefinition: !Ref TaskDefinition
      DesiredCount: 3
      LaunchType: FARGATE
      NetworkConfiguration:
        AwsvpcConfiguration:
          AssignPublicIp: DISABLED
          Subnets:
            - !Ref PrivateSubnet1
            - !Ref PrivateSubnet2
          SecurityGroups:
            - !Ref ContainerSecurityGroup
      LoadBalancers:
        - ContainerName: my-container
          ContainerPort: 8080
          TargetGroupArn: !Ref TargetGroup
      DeploymentConfiguration:
        MaximumPercent: 150
        MinimumHealthyPercent: 66
        DeploymentCircuitBreaker:
          Enable: true
          Rollback: true
      HealthCheckGracePeriodSeconds: 60

  # Target Group
  TargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      TargetType: ip
      Protocol: HTTP
      Port: 8080
      VpcId: !Ref VPC
      HealthCheckPath: /health
      HealthCheckIntervalSeconds: 30
      HealthCheckTimeoutSeconds: 5
      HealthyThresholdCount: 2
      UnhealthyThresholdCount: 3
```

#### Service Management Commands

```bash
# List services
aws ecs list-services --cluster my-app-cluster

# Describe service
aws ecs describe-services \
  --cluster my-app-cluster \
  --services my-app-service

# Update service (change desired count)
aws ecs update-service \
  --cluster my-app-cluster \
  --service my-app-service \
  --desired-count 5

# Update task definition
aws ecs update-service \
  --cluster my-app-cluster \
  --service my-app-service \
  --task-definition my-app:2 \
  --force-new-deployment

# Delete service
aws ecs delete-service \
  --cluster my-app-cluster \
  --service my-app-service \
  --force
```

***

### Networking & Load Balancing

#### Network Modes

**awsvpc (Recommended for Fargate)**

```
Each task gets ENI
    │
    ├─ Own security group
    ├─ Own private IP
    ├─ Own hostname
    └─ Easy load balancing
```

**bridge (EC2 Only)**

```
Tasks share host network
    │
    ├─ Port mapping (host:container)
    ├─ NAT-style networking
    └─ Multiple instances per host
```

**host (EC2 Only)**

```
Task uses host network directly
    │
    ├─ Same IP as EC2 instance
    ├─ Direct port access
    └─ No port mapping
```

#### Load Balancing Setup

**Application Load Balancer (ALB)**

```
Internet
    │
    ↓
ALB (Port 80/443)
    │
    ├─→ Target Group 1 (Port 8080) → Service 1
    ├─→ Target Group 2 (Port 8081) → Service 2
    └─→ Target Group 3 (Port 8082) → Service 3
```

**Network Load Balancer (NLB)**

```
Internet
    │
    ↓
NLB (Ultra-low latency)
    │
    ├─→ Target Group 1 → Service 1
    ├─→ Target Group 2 → Service 2
    └─→ Target Group 3 → Service 3
```

#### CloudFormation ALB Setup

```yaml
Resources:
  # Application Load Balancer
  LoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Type: application
      Subnets:
        - !Ref PublicSubnet1
        - !Ref PublicSubnet2
      SecurityGroups:
        - !Ref ALBSecurityGroup

  # Target Group
  TargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      TargetType: ip
      Protocol: HTTP
      Port: 8080
      VpcId: !Ref VPC
      HealthCheckProtocol: HTTP
      HealthCheckPath: /health
      HealthCheckIntervalSeconds: 30
      HealthyThresholdCount: 2

  # Listener (HTTP)
  Listener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !Ref LoadBalancer
      Protocol: HTTP
      Port: 80
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref TargetGroup

  # Security Group
  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: ALB Security Group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
```

#### Service Discovery

**AWS Cloud Map (Service Registry)**

```
Service Name: my-api
    │
    ├─ DNS Name: my-api.service.local
    ├─ Auto-registered on task launch
    └─ Auto-deregistered on task stop
         │
         ↓
    Service-to-Service Communication
    └─ Internal DNS resolution
```

**CloudFormation Service Discovery**

```yaml
Resources:
  # Service Discovery Namespace
  PrivateNamespace:
    Type: AWS::ServiceDiscovery::PrivateDnsNamespace
    Properties:
      Name: service.local
      Vpc: !Ref VPC

  # Service Discovery Service
  DiscoveryService:
    Type: AWS::ServiceDiscovery::Service
    Properties:
      Name: my-api
      NamespaceId: !Ref PrivateNamespace
      DnsConfig:
        RoutingPolicy: MULTIVALUE
        DnsRecords:
          - TTL: 60
            Type: A
          - TTL: 60
            Type: SRV
      HealthCheckCustomConfig:
        FailureThreshold: 1

  # ECS Service with Service Discovery
  ECSService:
    Type: AWS::ECS::Service
    Properties:
      ServiceName: my-api-service
      Cluster: !Ref ECSCluster
      TaskDefinition: !Ref TaskDefinition
      DesiredCount: 3
      LaunchType: FARGATE
      ServiceRegistries:
        - RegistryArn: !GetAtt DiscoveryService.Arn
          ContainerName: my-container
          ContainerPort: 8080
```

***

### Deployment & Scaling

#### Deployment Strategies

**Rolling Deployment (Default)**

```
Old Task (v1)       New Task (v2)
    Running -----→ Stop
    
    ↓               ↓
    
  Stop 1st v1     Start 1st v2
  
         ↓
  Stop 2nd v1
  Start 2nd v2
  
         ↓
  Stop 3rd v1
  Start 3rd v2
  
Result: Zero downtime, gradual transition
```

**Blue-Green Deployment**

```
Blue Environment (v1)      Green Environment (v2)
├─ 3 tasks running          ├─ 3 tasks running
├─ All traffic             └─ Staged testing
└─ Production
    
    ↓
    
Switch traffic from Blue to Green
    
    ↓
    
Remove Blue environment
```

**Canary Deployment**

```
Current: 95% v1, 5% v2
    ↓
Healthy: 95% v1, 5% v2
    ↓
Increase: 90% v1, 10% v2
    ↓
Increase: 50% v1, 50% v2
    ↓
Complete: 0% v1, 100% v2
```

#### Deployment Configuration

```yaml
DeploymentConfiguration:
  MaximumPercent: 150        # Can run 150% of desired (3→4-5 tasks)
  MinimumHealthyPercent: 66  # Keep at least 66% healthy (3→2 running)
  DeploymentCircuitBreaker:
    Enable: true             # Stop deployment if too many fail
    Rollback: true           # Rollback to previous version
```

#### Auto-scaling Setup

**Application Auto Scaling Policy**

```bash
# Register scalable target
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/my-cluster/my-service \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 1 \
  --max-capacity 10

# Create scaling policy (CPU)
aws application-autoscaling put-scaling-policy \
  --policy-name my-scaling-policy \
  --service-namespace ecs \
  --resource-id service/my-cluster/my-service \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
    },
    "ScaleOutCooldown": 60,
    "ScaleInCooldown": 300
  }'

# Create scaling policy (Memory)
aws application-autoscaling put-scaling-policy \
  --policy-name my-memory-scaling \
  --service-namespace ecs \
  --resource-id service/my-cluster/my-service \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 80.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ECSServiceAverageMemoryUtilization"
    }
  }'
```

**CloudFormation Auto-scaling**

```yaml
Resources:
  # Scalable Target
  ScalableTarget:
    Type: AWS::ApplicationAutoScaling::ScalableTarget
    Properties:
      MaxCapacity: 10
      MinCapacity: 1
      ResourceId: !Sub 'service/${ECSCluster}/${ECSService}'
      RoleARN: !GetAtt AutoScalingRole.Arn
      ScalableDimension: ecs:service:DesiredCount
      ServiceNamespace: ecs

  # Scaling Policy (CPU)
  ScalingPolicy:
    Type: AWS::ApplicationAutoScaling::ScalingPolicy
    Properties:
      PolicyName: cpu-scaling
      PolicyType: TargetTrackingScaling
      ScalingTargetId: !Ref ScalableTarget
      TargetTrackingScalingPolicyConfiguration:
        TargetValue: 70.0
        PredefinedMetricSpecification:
          PredefinedMetricType: ECSServiceAverageCPUUtilization
        ScaleOutCooldown: 60
        ScaleInCooldown: 300
```

***

### Advanced Topics

#### Task Placement Strategies (EC2)

**Spread Strategy**

```
Tasks spread evenly across instances
    │
    Instance 1: 1 task
    Instance 2: 1 task
    Instance 3: 1 task
    
Benefit: High availability
```

**Binpack Strategy**

```
Tasks packed on fewest instances
    │
    Instance 1: 3 tasks
    Instance 2: 0 tasks
    Instance 3: 0 tasks
    
Benefit: Cost optimization
```

**Custom Constraint**

```
Tasks placed based on instance attributes
    │
    Constraint: Placement group = web
    
    Instance web-1: Tasks placed
    Instance db-1: Tasks skipped
```

#### Container Insights (Monitoring)

Enable detailed monitoring:

```bash
aws ecs create-cluster \
  --cluster-name my-cluster \
  --cluster-settings name=containerInsights,value=enabled
```

**CloudWatch Metrics Available:**

* CPU utilization (container, service, cluster)
* Memory utilization
* Task count
* Service count
* Network metrics

**View Dashboard:**

```bash
# Open CloudWatch Insights dashboard
# Namespace: ECS/ContainerInsights
# Dimensions: ClusterName, ServiceName, TaskDefinitionFamily
```

#### Capacity Providers (EC2)

Auto-scale EC2 instances based on task needs.

```yaml
Resources:
  CapacityProvider:
    Type: AWS::ECS::CapacityProvider
    Properties:
      Name: my-capacity-provider
      AutoScalingGroupProvider:
        AutoScalingGroupArn: !GetAtt ASG.GroupArn
        ManagedScaling:
          Status: ENABLED
          TargetCapacity: 80
          MinimumScalingStepSize: 1
          MaximumScalingStepSize: 10000
        ManagedTerminationProtection: ENABLED

  ClusterCapacityProviderAssociation:
    Type: AWS::ECS::ClusterCapacityProviderAssociations
    Properties:
      CapacityProviders:
        - !Ref CapacityProvider
      Cluster: !Ref ECSCluster
      DefaultCapacityProviderStrategy:
        - CapacityProvider: !Ref CapacityProvider
          Weight: 1
          Base: 1
```

#### Secrets Management

Store sensitive data securely:

```bash
# Create secret
aws secretsmanager create-secret \
  --name MyDatabasePassword \
  --secret-string '{"username":"admin","password":"secret123"}'
```

**Reference in Task Definition:**

```json
{
  "secrets": [
    {
      "name": "DB_PASSWORD",
      "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789012:secret:MyDatabasePassword:password::"
    }
  ]
}
```

#### Container Logging

**CloudWatch Logs**

```json
{
  "logConfiguration": {
    "logDriver": "awslogs",
    "options": {
      "awslogs-group": "/ecs/my-app",
      "awslogs-region": "us-east-1",
      "awslogs-stream-prefix": "ecs"
    }
  }
}
```

**Splunk**

```json
{
  "logConfiguration": {
    "logDriver": "splunk",
    "options": {
      "splunk-token": "arn:aws:secretsmanager:...",
      "splunk-url": "https://your-instance.splunkcloud.com"
    }
  }
}
```

**Awslogs Options:**

```
awslogs-group           - Log group name
awslogs-region          - AWS region
awslogs-stream-prefix   - Stream prefix
awslogs-datetime-format - Timestamp format
```

***

### Troubleshooting

#### Task Not Starting

**Check Task Definition**

```bash
# Describe task definition
aws ecs describe-task-definition --task-definition my-app:1

# Validate image accessibility
aws ecr describe-images \
  --repository-name my-app \
  --image-ids imageTag=latest
```

**Check Cluster Capacity (EC2)**

```bash
# List container instances
aws ecs list-container-instances --cluster my-cluster

# Describe instance resources
aws ecs describe-container-instances \
  --cluster my-cluster \
  --container-instances <arn>
```

**Check CloudWatch Logs**

```bash
# View ECS agent logs
tail -f /var/log/ecs/ecs-agent.log

# Check container logs
aws logs tail /ecs/my-app --follow
```

#### Common Task Failures

| Error                       | Cause                 | Solution                           |
| --------------------------- | --------------------- | ---------------------------------- |
| CannotPullContainerError    | Image not found       | Check ECR repository, image tag    |
| OutOfMemory                 | Insufficient memory   | Increase task memory in definition |
| CannotInvokeContainerError  | Container crash       | Check container logs               |
| ResourceInitializationError | Task role permission  | Verify IAM role permissions        |
| ServiceNotStable            | Health checks failing | Check health check endpoint        |

#### Debugging Commands

```bash
# List tasks
aws ecs list-tasks --cluster my-cluster --service-name my-service

# Describe task
aws ecs describe-tasks \
  --cluster my-cluster \
  --tasks <task-arn>

# View stopped task reason
aws ecs describe-tasks \
  --cluster my-cluster \
  --tasks <task-arn> \
  --query 'tasks[0].stoppedReason'

# Get logs
aws logs tail /ecs/my-app --follow --since 1h

# Check service events
aws ecs describe-services \
  --cluster my-cluster \
  --services my-service \
  --query 'services[0].events'
```

#### Performance Issues

**High CPU Usage**

```bash
# Check container metrics
aws cloudwatch get-metric-statistics \
  --namespace ECS/ContainerInsights \
  --metric-name ContainerCpuUtilized \
  --dimensions Name=ServiceName,Value=my-service \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-16T00:00:00Z \
  --period 300 \
  --statistics Average,Maximum
```

**High Memory Usage**

```bash
# Check memory metrics
aws cloudwatch get-metric-statistics \
  --namespace ECS/ContainerInsights \
  --metric-name ContainerMemoryUtilized \
  --dimensions Name=ServiceName,Value=my-service \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-16T00:00:00Z \
  --period 300 \
  --statistics Average,Maximum
```

***

### Best Practices

#### Architecture Design

**Microservices Structure**

```
Each service:
├─ Own task definition
├─ Own ECR repository
├─ Own ECS service
├─ Own load balancer target group
└─ Independent scaling
```

**Task Right-Sizing**

```
Production:
- Request: 70% of expected usage
- Limit: 100% or slightly higher
- Monitor and adjust

Example (Web Service):
- CPU Request: 256
- CPU Limit: 512
- Memory Request: 512
- Memory Limit: 1024
```

**Network Architecture**

```
Public Subnets: ALB/NLB
    │
    ↓
Private Subnets: ECS Tasks
    │
    ├─ No internet access by default
    ├─ NAT Gateway for outbound
    └─ Security groups for control
```

#### Container Image Best Practices

**Multi-stage Builds**

```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Runtime
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY app.js .
EXPOSE 8080
CMD ["node", "app.js"]
```

**Image Scanning**

```bash
# Enable ECR image scanning
aws ecr put-image-scanning-configuration \
  --repository-name my-app \
  --image-scanning-configuration scanOnPush=true
```

**Image Tag Strategy**

```
Tagging Convention:
- latest        → Latest stable
- v1.2.3        → Specific version
- dev           → Development build
- staging       → Staging environment
- prod          → Production (never use latest in prod)

Never use 'latest' in production!
```

#### Deployment Best Practices

**Continuous Deployment Pipeline**

```
Push to Git
    ↓
Build Docker image
    ↓
Scan image
    ↓
Push to ECR
    ↓
Update task definition
    ↓
Update ECS service
    ↓
Blue-green deployment
    ↓
Health checks pass
    ↓
Production live
```

**Pre-deployment Checks**

```
✓ Image scanned for vulnerabilities
✓ All env variables set
✓ Database migrations complete
✓ Dependencies available
✓ Health check endpoint working
✓ Monitoring dashboards ready
✓ Rollback plan documented
```

#### Monitoring & Alerting

**Essential Metrics to Monitor**

```
Service Level:
- Desired vs. Running task count
- Task CPU utilization
- Task memory utilization
- Service request count

Container Level:
- Container restart count
- Failed health checks
- Exit codes

Infrastructure Level (EC2):
- Instance CPU utilization
- Instance memory utilization
- Disk usage
- Network I/O
```

**CloudWatch Alarms**

```bash
# Alert on task failures
aws cloudwatch put-metric-alarm \
  --alarm-name ECS-Task-Failures \
  --alarm-description "Alert when tasks fail" \
  --metric-name FailedTasks \
  --namespace ECS/ContainerInsights \
  --statistic Sum \
  --period 300 \
  --threshold 3 \
  --comparison-operator GreaterThanThreshold
```

#### Security Best Practices

**Container Security**

* Run as non-root user in Dockerfile
* Use read-only root filesystem when possible
* Scan images for vulnerabilities
* Keep base images updated

**Network Security**

```
Security Groups:
- ALB: 80/443 from internet
- ECS Tasks: Only from ALB/services
- Database: Only from application tasks
```

**Secrets Management**

```
✓ Store secrets in Secrets Manager
✓ Reference in task definition
✓ Use IAM roles for access
✓ Rotate credentials regularly
✗ Never hardcode in images
✗ Never commit to git
```

**IAM Least Privilege**

```
Task Role:
- Only necessary AWS service permissions
- Restrict to specific resources
- Use resource-based policies

Execution Role:
- Pull from specific ECR repos
- Write to specific log groups
- Read specific Secrets Manager secrets
```

#### Cost Optimization

**Choose Right Launch Type**

```
Use Fargate if:
- Variable load
- Serverless preference
- Small team

Use EC2 if:
- Stable baseline load
- Cost-sensitive
- Specific customization
```

**Optimize Task Right-sizing**

```
CPU/Memory combinations for Fargate:
256 CPU   → 512-2048 MB
512 CPU   → 1024-3840 MB
1024 CPU  → 2048-7680 MB
2048 CPU  → 4096-15360 MB
4096 CPU  → 8192-30720 MB
```

**Use Spot Instances (EC2)**

```
Capacity Provider Strategy:
- 70% On-demand (baseline)
- 30% Spot (savings)

Spot Savings: 70-90% cheaper
```

**Reserved Capacity**

```
For predictable workloads:
- ECS EC2 RIs
- Fargate RIs
- Potential 40-50% savings
```

#### Naming Conventions

```
Task Definitions:
  my-app
  my-api
  my-worker

Services:
  my-app-service
  my-api-service
  my-worker-service

Clusters:
  production
  staging
  development

Log Groups:
  /ecs/my-app
  /ecs/my-api
  /ecs/my-worker
```

***

### Quick Reference

#### Essential AWS CLI Commands

```bash
# Cluster Operations
aws ecs create-cluster --cluster-name my-cluster
aws ecs list-clusters
aws ecs describe-clusters --clusters my-cluster
aws ecs delete-cluster --cluster my-cluster

# Task Definitions
aws ecs register-task-definition --cli-input-json file://task-def.json
aws ecs list-task-definitions
aws ecs describe-task-definition --task-definition my-app:1

# Services
aws ecs create-service --cluster my-cluster --service-name my-service ...
aws ecs list-services --cluster my-cluster
aws ecs update-service --cluster my-cluster --service my-service --desired-count 5
aws ecs delete-service --cluster my-cluster --service my-service --force

# Tasks
aws ecs list-tasks --cluster my-cluster
aws ecs describe-tasks --cluster my-cluster --tasks <arn>
aws ecs run-task --cluster my-cluster --task-definition my-app:1 ...
aws ecs stop-task --cluster my-cluster --task <arn>

# Logs
aws logs tail /ecs/my-app --follow
aws logs tail /ecs/my-app --since 1h
```

#### Common Task Definition CPU/Memory Combinations

| Scenario           | CPU  | Memory |
| ------------------ | ---- | ------ |
| Light web service  | 256  | 512    |
| Medium web service | 512  | 1024   |
| Heavy web service  | 1024 | 2048   |
| API service        | 512  | 1024   |
| Worker/batch       | 1024 | 2048   |
| Cache service      | 2048 | 4096   |

#### Deployment Comparison

| Strategy    | Downtime | Risk     | Speed     |
| ----------- | -------- | -------- | --------- |
| Rolling     | None     | Low      | Slow      |
| Blue-Green  | None     | Low      | Fast      |
| Canary      | None     | Very Low | Medium    |
| All-at-once | Yes      | High     | Very Fast |

***

### Summary

**ECS Key Points:**

* Managed container orchestration service
* Two launch types: EC2 (control) and Fargate (serverless)
* Task definitions define container specifications
* Services manage task lifecycle and scaling
* Integrates with ALB/NLB for load balancing
* Auto-scaling based on metrics
* Comprehensive monitoring with CloudWatch

**Basic Workflow:**

```
Create Task Definition → Create Cluster → Create Service → Deploy Tasks → Monitor
```

**Use ECS When:**

* Running containerized applications
* Want AWS-managed orchestration
* Need simple deployment pipeline
* Integration with AWS services important

**Avoid ECS If:**

* Need complex multi-cluster orchestration → Use EKS
* Require full Kubernetes features → Use EKS
* Simple single container → Use Lightsail

***

### Resources

* [AWS ECS Documentation](https://docs.aws.amazon.com/ecs/)
* [ECS Best Practices Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/best_practices.html)
* [ECS Task Definition Reference](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html)
* [AWS ECS Samples & Demos](https://github.com/aws-samples/amazon-ecs-samples)
* [ECS Pricing](https://aws.amazon.com/ecs/pricing/)
