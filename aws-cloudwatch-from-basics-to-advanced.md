---
description: >-
  AWS CloudWatch is a comprehensive monitoring and observability platform that
  provides insights into your AWS resources and applications. It collects and
  tracks metrics, monitors log and sets alarms.
---

# AWS CloudWatch: From Basics to Advanced

### Table of Contents

1. Introduction
2. Core Concepts
3. Getting Started
4. Monitoring AWS Resources
5. Custom Metrics
6. Logs Management
7. CloudWatch Alarms
8. Dashboards
9. Advanced Features
10. Cost Optimization
11. Best Practices

***

#### CloudWatch Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    AWS CloudWatch Platform                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Data Sources              Collection            Storage        │
│  ├─ EC2                    ┌──────────────┐      ┌──────────┐  │
│  ├─ RDS                    │   Metrics    │──┬───│ Database │  │
│  ├─ Lambda          ─────→ │   (Time-   │  │   └──────────┘  │
│  ├─ S3                     │   series)  │  │                  │
│  ├─ Custom Apps   ─────→   └──────────────┘  │   ┌──────────┐  │
│  └─ On-premises            ┌──────────────┐  └───│   Logs   │  │
│                            │     Logs     │      └──────────┘  │
│                            │  (Events)    │                    │
│                            └──────────────┘                    │
│                                    ↓                            │
│                            ┌──────────────┐                    │
│                            │    Alarms    │                    │
│                            │  (Thresholds)│                    │
│                            └──────────────┘                    │
│                                    ↓                            │
│                     ┌──────────────────────────┐               │
│                     │     Automated Actions    │               │
│                     ├──────────────────────────┤               │
│                     │ SNS │ Lambda │ Slack     │               │
│                     └──────────────────────────┘               │
└─────────────────────────────────────────────────────────────────┘
```

#### Why CloudWatch Matters

* **Real-time monitoring** of AWS services
* **Centralized logging** for all your applications
* **Automated responses** through alarms and actions
* **Performance optimization** insights
* **Cost tracking** and resource utilization

***

### Core Concepts

#### 1. Metrics

**Metrics** are the fundamental unit of monitoring in CloudWatch. They represent time-ordered data points that measure specific aspects of your resources.

**Key characteristics:**

* Time-stamped numerical values
* Associated with a namespace
* Can have multiple dimensions
* Resolution (1-minute or 60-second standard; 1-second for high-resolution)

**Example:** CPU utilization of an EC2 instance

#### 2. Namespaces

A **namespace** is a container for metrics. AWS services automatically create namespaces for their metrics.

Common namespaces:

```
AWS/EC2
AWS/RDS
AWS/Lambda
AWS/ECS
AWS/S3
AWS/DynamoDB
Custom/Application
```

#### 3. Dimensions

**Dimensions** are key-value pairs that help identify a specific metric within a namespace.

```
Metrics Hierarchy:
├─ Namespace (AWS/EC2)
│  ├─ Metric (CPUUtilization)
│  │  ├─ Dimension 1: InstanceId = i-123
│  │  ├─ Dimension 2: ImageId = ami-456
│  │  └─ Value: 45.2% (at timestamp)
│  │
│  └─ Metric (NetworkIn)
│     ├─ Dimension 1: InstanceId = i-123
│     └─ Value: 1024 bytes
│
└─ Namespace (AWS/RDS)
   └─ Metric (DatabaseConnections)
      ├─ Dimension 1: DBInstanceIdentifier = mydb
      └─ Value: 52 connections
```

**Example dimensions:**

```
Namespace: AWS/EC2
Dimensions:
  - InstanceId: i-1234567890abcdef0
  - ImageId: ami-0c55b159cbfafe1f0
```

#### 4. Statistics

CloudWatch aggregates metric data into statistics:

| Statistic       | Description                                   |
| --------------- | --------------------------------------------- |
| **Average**     | Mean value over the period                    |
| **Sum**         | Total of all values                           |
| **Minimum**     | Lowest value                                  |
| **Maximum**     | Highest value                                 |
| **SampleCount** | Number of data points                         |
| **Percentile**  | Value at specific percentile (p90, p99, etc.) |

***

### Getting Started

#### Prerequisites

* AWS Account
* AWS CLI configured
* Basic understanding of AWS services
* IAM permissions for CloudWatch

#### Accessing CloudWatch

**Via AWS Console:**

```
AWS Management Console > Services > CloudWatch
```

**Via AWS CLI:**

```bash
aws cloudwatch list-metrics
```

#### Console Layout Overview

```
┌─────────────────────────────────────────────────────┐
│  CloudWatch Dashboard                               │
├─────────────────────────────────────────────────────┤
│ Left Sidebar:                                       │
│ ├── Dashboard                                       │
│ ├── Alarms                                          │
│ ├── Logs                                            │
│ │   ├── Log groups                                  │
│ │   ├── Log streams                                 │
│ │   └── Insights                                    │
│ ├── Metrics                                         │
│ │   ├── All metrics                                 │
│ │   └── Custom namespaces                           │
│ ├── Events                                          │
│ └── Settings                                        │
│                                                     │
│ Main Content Area:                                  │
│ ├── Metric visualization                           │
│ ├── Filter controls                                │
│ └── Time range selector                             │
└─────────────────────────────────────────────────────┘
```

#### Setting Up Your First Monitoring

**Step 1: Navigate to CloudWatch**

1. Open AWS Management Console
2. Search for "CloudWatch" in the services search
3. Select **CloudWatch** from results

**Step 2: Explore Default Metrics**

Navigate to **Metrics** → **All metrics** to see available namespaces:

* **AWS/EC2** - EC2 instance metrics
* **AWS/RDS** - Database metrics
* **AWS/Lambda** - Function execution metrics
* **AWS/S3** - Storage metrics

**Step 3: Create Your First Alarm**

1. Go to **Alarms** → **Create alarm**
2. Select metric (e.g., AWS/EC2 → CPUUtilization)
3. Set threshold (e.g., > 80%)
4. Configure notification (SNS topic)
5. Review and create

***

### Monitoring AWS Resources

#### EC2 Instances

**Default Metrics**

EC2 provides these default metrics:

* **CPUUtilization** - Percentage of CPU usage
* **NetworkIn** - Incoming network traffic
* **NetworkOut** - Outgoing network traffic
* **DiskReadBytes** - Bytes read from disk
* **DiskWriteBytes** - Bytes written to disk

**Console View: EC2 Metrics**

```
CloudWatch > Metrics > AWS/EC2
┌──────────────────────────────────────────┐
│ Namespace: AWS/EC2                       │
├──────────────────────────────────────────┤
│ Instance Details                         │
│ ┌─────────────────────────────────────┐  │
│ │ Instance: i-1234567890abcdef0       │  │
│ │ ✓ CPUUtilization (Current: 45%)     │  │
│ │ ✓ NetworkIn                         │  │
│ │ ✓ NetworkOut                        │  │
│ │ ✓ DiskReadBytes                     │  │
│ │ ✓ DiskWriteBytes                    │  │
│ └─────────────────────────────────────┘  │
│                                          │
│ Graph: Time range [1h ▼] [Last 5min]   │
│ ┌──────────────────────────────────────┐ │
│ │                                      │ │
│ │    CPU Utilization %                │ │
│ │        ╱╲    ╱╲                      │ │
│ │       ╱  ╲  ╱  ╲    45%             │ │
│ │      ╱    ╲╱    ╲                   │ │
│ │     ╱            ╲                  │ │
│ │                                      │ │
│ └──────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

**Enabling Detailed Monitoring**

**Standard Monitoring:** 5-minute intervals (free)\
**Detailed Monitoring:** 1-minute intervals ($3.50/month per instance)

```bash
aws ec2 monitor-instances --instance-ids i-1234567890abcdef0
```

#### RDS Databases

**Common RDS Metrics**

```
AWS/RDS Key Metrics:
├── DatabaseConnections
├── CPUUtilization
├── FreeableMemory
├── ReadLatency / WriteLatency
├── DiskQueueDepth
└── Replication Lag (for Read Replicas)
```

**Console View: RDS Database Monitoring**

```
CloudWatch > Metrics > AWS/RDS > Instances
┌──────────────────────────────────────────┐
│ Database: mydb (MySQL 8.0)               │
├──────────────────────────────────────────┤
│ Performance Metrics                      │
│ ┌─────────────────────────────────────┐  │
│ │ CPU Utilization:      32%           │  │
│ │ Database Connections: 45/100        │  │
│ │ Freeable Memory:      4.2 GB / 8GB  │  │
│ │ Read Latency:         2.3 ms        │  │
│ │ Write Latency:        1.8 ms        │  │
│ │ Disk Queue Depth:     0.5           │  │
│ └─────────────────────────────────────┘  │
│                                          │
│ Add to Dashboard | Create Alarm          │
└──────────────────────────────────────────┘
```

**Getting RDS Metrics via Python**

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

response = cloudwatch.get_metric_statistics(
    Namespace='AWS/RDS',
    MetricName='DatabaseConnections',
    Dimensions=[{'Name': 'DBInstanceIdentifier', 'Value': 'mydb'}],
    StartTime='2024-01-01T00:00:00Z',
    EndTime='2024-01-01T01:00:00Z',
    Period=300,
    Statistics=['Average']
)

for datapoint in response['Datapoints']:
    print(f"Time: {datapoint['Timestamp']}, Connections: {datapoint['Average']}")
```

#### Lambda Functions

**Lambda Metrics**

```
AWS/Lambda Key Metrics:
├── Invocations (total function calls)
├── Duration (execution time in ms)
├── Errors (failed executions)
├── Throttles (rejected due to concurrency)
├── ConcurrentExecutions (simultaneous runs)
└── UnreservedConcurrentExecutions
```

**Console View: Lambda Function Metrics**

```
CloudWatch > Metrics > AWS/Lambda > Functions
┌──────────────────────────────────────────┐
│ Function: myFunction                     │
├──────────────────────────────────────────┤
│ Execution Summary (Last 24 Hours)        │
│ ┌─────────────────────────────────────┐  │
│ │ Invocations:         2,450          │  │
│ │ Errors:              12 (0.49%)     │  │
│ │ Throttles:           0              │  │
│ │ Duration (Avg):      245 ms         │  │
│ │ Max Concurrency:     12/1000        │  │
│ └─────────────────────────────────────┘  │
│                                          │
│ Create Alarm | Add to Dashboard          │
└──────────────────────────────────────────┘
```

**Monitoring Lambda Concurrency**

Lambda has concurrent execution limits. Monitor to avoid throttling:

```python
cloudwatch = boto3.client('cloudwatch')

response = cloudwatch.get_metric_statistics(
    Namespace='AWS/Lambda',
    MetricName='ConcurrentExecutions',
    Dimensions=[{'Name': 'FunctionName', 'Value': 'myFunction'}],
    StartTime='2024-01-01T00:00:00Z',
    EndTime='2024-01-01T01:00:00Z',
    Period=60,
    Statistics=['Maximum']
)
```

#### ECS Containers

**ECS Metrics**

```
AWS/ECS:
- CPUUtilization
- MemoryUtilization
- ServiceCount
- TaskCount
- DesiredTaskCount
- RunningCount
```

***

### Custom Metrics

Custom metrics allow you to monitor application-specific data.

#### Publishing Custom Metrics

**Using Python (Boto3)**

```python
import boto3
from datetime import datetime

cloudwatch = boto3.client('cloudwatch')

response = cloudwatch.put_metric_data(
    Namespace='MyApp/Performance',
    MetricData=[
        {
            'MetricName': 'RequestCount',
            'Value': 100,
            'Unit': 'Count',
            'Timestamp': datetime.now()
        },
        {
            'MetricName': 'ResponseTime',
            'Value': 250.5,
            'Unit': 'Milliseconds',
            'Timestamp': datetime.now(),
            'Dimensions': [
                {
                    'Name': 'Environment',
                    'Value': 'Production'
                },
                {
                    'Name': 'Endpoint',
                    'Value': '/api/users'
                }
            ]
        }
    ]
)
```

#### Metric Units

```
Count, Seconds, Microseconds, Milliseconds
Bytes, Kilobytes, Megabytes, Gigabytes
Bits, Kilobits, Megabits, Gigabits
Percent, Bytes/Second, Kilobytes/Second
None
```

#### High-Resolution Metrics

For metrics requiring sub-minute granularity:

```python
cloudwatch.put_metric_data(
    Namespace='MyApp/HighResolution',
    MetricData=[
        {
            'MetricName': 'ProcessingTime',
            'Value': 50.25,
            'Unit': 'Milliseconds',
            'StorageResolution': 1  # 1-second resolution
        }
    ]
)
```

***

### Logs Management

#### CloudWatch Logs Overview

CloudWatch Logs allows you to centralize logs from AWS resources and applications.

**Log Hierarchy Structure**

```
/aws/lambda/myFunction (Log Group)
├── 2024/01/01/[$LATEST]abc123 (Log Stream)
│   ├── 12:00:01 - "Function started"
│   ├── 12:00:05 - "Processing request"
│   └── 12:00:10 - "Request completed"
│
├── 2024/01/01/[$LATEST]def456 (Log Stream)
│   ├── 12:00:15 - "Function started"
│   ├── 12:00:20 - "Processing request"
│   └── 12:00:25 - "Request completed"
│
└── 2024/01/02/[$LATEST]ghi789 (Log Stream)
    ├── 13:00:01 - "Function started"
    └── 13:00:08 - "Error: Timeout"

Legend:
├─ Log Group: Container for related logs
├─ Log Stream: Sequential logs from single source
└─ Log Event: Individual timestamped message
```

#### Console View: Log Groups

```
CloudWatch > Logs > Log Groups
┌──────────────────────────────────────────────────┐
│ Log Groups                                       │
├──────────────────────────────────────────────────┤
│ ✓ /aws/lambda/myFunction                        │
│   └─ Log Streams: 5 | Retention: 7 days         │
│ ✓ /aws/rds/error                                │
│   └─ Log Streams: 2 | Retention: 30 days        │
│ ✓ /aws/api-gateway/access-logs                  │
│   └─ Log Streams: 12 | Retention: 14 days       │
│                                                 │
│ [Create log group] [Delete] [Edit retention]    │
└──────────────────────────────────────────────────┘
```

#### Creating Log Groups

```python
import boto3

logs = boto3.client('logs')

# Create log group
logs.create_log_group(logGroupName='/aws/lambda/myFunction')

# Set retention policy (7 days)
logs.put_retention_policy(
    logGroupName='/aws/lambda/myFunction',
    retentionInDays=7
)
```

#### Publishing Log Events

**Using Python**

```python
import boto3
import time

logs = boto3.client('logs')

logs.put_log_events(
    logGroupName='/aws/lambda/myFunction',
    logStreamName='2024/01/01/[$LATEST]abcdef123456',
    logEvents=[
        {
            'message': 'Application started',
            'timestamp': int(time.time() * 1000)
        },
        {
            'message': 'Processing request from user: john@example.com',
            'timestamp': int(time.time() * 1000)
        }
    ]
)
```

#### Log Insights Queries

CloudWatch Logs Insights enables complex log analysis through a query language.

**Console View: Log Insights Query**

```
CloudWatch > Logs > Insights
┌────────────────────────────────────────┐
│ Select log group(s): /aws/lambda/      │
│                                        │
│ Query Editor:                          │
│ ┌──────────────────────────────────┐  │
│ │ fields @timestamp, @message      │  │
│ │ | filter @message like /ERROR/   │  │
│ │ | stats count() as ErrorCount    │  │
│ │                                  │  │
│ │ [Run Query] [Save]               │  │
│ └──────────────────────────────────┘  │
│                                        │
│ Results:                               │
│ Time       │ @message              │  │
│ ───────────┼──────────────────────┤  │
│ 12:05:23   │ ERROR: Connection... │  │
│ 12:08:45   │ ERROR: Timeout...    │  │
│ 12:12:19   │ ERROR: Auth failed   │  │
└────────────────────────────────────────┘
```

**Common Queries**

```
# Count log events
fields @timestamp, @message
| stats count()

# Filter by error level
fields @timestamp, @message
| filter @message like /ERROR/
| stats count() as ErrorCount

# Parse JSON and analyze by user
fields @timestamp, @message.userId, @message.responseTime
| stats avg(@message.responseTime) by @message.userId
```

#### Log Group Subscriptions

Automatically send logs to another AWS service (Lambda, Kinesis, S3):

```python
logs.put_subscription_filter(
    logGroupName='/aws/lambda/myFunction',
    filterName='ErrorOnlyFilter',
    filterPattern='ERROR',
    destinationArn='arn:aws:lambda:us-east-1:123456789012:function:ProcessErrors'
)
```

**Subscription Flow:**

```
Log Events → Log Filter → Subscription → Lambda/Kinesis/S3
   (all)     (ERROR only)    (filter)     (process)
```

#### Metric Filters

Create metrics from logs:

```bash
aws logs put-metric-filter \
  --log-group-name /aws/lambda/myFunction \
  --filter-name ErrorCount \
  --filter-pattern "[ERROR]" \
  --metric-transformations \
    metricName=ErrorCount,metricNamespace=MyApp,metricValue=1
```

***

### CloudWatch Alarms

#### Alarm Concepts

**Alarms** monitor metrics and trigger actions when thresholds are breached.

**Alarm States**

```
                    ┌─────────────────┐
                    │ INSUFFICIENT    │
                    │ _DATA (new)     │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ↓                    ↓                    ↓
    ┌──────┐           ┌──────────┐         ┌─────────┐
    │  OK  │←──────┐   │ ALARM    │←────┐   │TESTING  │
    └──┬───┘       │   └──┬───────┘     │   └─────────┘
       │           │      │            │
       └───────────┴──────┼────────────┘
       (metric in range)  │
                    (threshold exceeded)

Actions triggered:
┌─────────────────────────────────────┐
│ State → OK    │ Trigger OKActions    │
│ State → ALARM │ Trigger AlarmActions │
│ State → --    │ Trigger --           │
└─────────────────────────────────────┘
```

| State                  | Meaning                     |
| ---------------------- | --------------------------- |
| **OK**                 | Metric is within threshold  |
| **ALARM**              | Metric exceeded threshold   |
| **INSUFFICIENT\_DATA** | Not enough data to evaluate |

#### Console View: Alarms

```
CloudWatch > Alarms > All Alarms
┌─────────────────────────────────────────────────┐
│ Alarms                                          │
├─────────────────────────────────────────────────┤
│ Status  │ Name              │ Metric   │ Action │
├─────────┼───────────────────┼──────────┼────────┤
│ 🟢 OK   │ high-cpu-alarm    │ CPU > 80 │ SNS    │
│ 🔴 ALRM │ high-mem-alarm    │ Mem > 90 │ SNS    │
│ 🟡 WARN │ disk-space-alarm  │ Disk > 85│ Email  │
│ ⚪ --   │ api-latency-check │ Anomaly  │ Slack  │
│         │                   │          │        │
│ [Create Alarm] [Delete] [Edit] [Test]          │
└─────────────────────────────────────────────────┘
```

#### Creating Alarms

**Simple Alarm with Python**

```python
cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_alarm(
    AlarmName='high-cpu-alarm',
    AlarmDescription='Alert when CPU > 80%',
    MetricName='CPUUtilization',
    Namespace='AWS/EC2',
    Statistic='Average',
    Period=300,
    Threshold=80,
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=2,
    AlarmActions=['arn:aws:sns:us-east-1:123456789012:MySNSTopic']
)
```

**Anomaly Detection Alarm**

Automatically detect anomalies using machine learning:

```python
cloudwatch.put_metric_alarm(
    AlarmName='lambda-duration-anomaly',
    ComparisonOperator='LessThanLowerOrGreaterThanUpperThreshold',
    EvaluationPeriods=2,
    Metrics=[
        {
            'Id': 'm1',
            'ReturnData': True,
            'MetricStat': {
                'Metric': {
                    'Namespace': 'AWS/Lambda',
                    'MetricName': 'Duration',
                    'Dimensions': [
                        {
                            'Name': 'FunctionName',
                            'Value': 'myFunction'
                        }
                    ]
                },
                'Period': 300,
                'Stat': 'Average'
            }
        },
        {
            'Id': 'ad1',
            'Expression': 'ANOMALY_DETECTION_BAND(m1, 2)',
            'ReturnData': True
        }
    ],
    AlarmActions=['arn:aws:sns:us-east-1:123456789012:MySNSTopic']
)
```

#### Composite Alarms

Combine multiple alarms with logic gates:

```python
cloudwatch.put_composite_alarm(
    AlarmName='critical-system-alarm',
    AlarmDescription='Alert if CPU OR Memory are critical',
    AlarmRule='(ALARM(cpu-alarm) OR ALARM(memory-alarm)) AND NOT ALARM(maintenance)',
    AlarmActions=['arn:aws:sns:us-east-1:123456789012:Critical']
)
```

**Examples:**

* `ALARM(a) AND ALARM(b)` - Both must be alarming
* `ALARM(a) OR ALARM(b)` - Either can be alarming
* `NOT ALARM(maintenance)` - Exclude during maintenance windows

#### Alarm Actions

**Supported Actions:**

| Action Type      | Example Use Case            | Target             |
| ---------------- | --------------------------- | ------------------ |
| **SNS**          | Email/SMS alerts            | SNS Topic          |
| **Auto Scaling** | Scale EC2/ECS automatically | Auto Scaling Group |
| **EC2 Actions**  | Stop/terminate instances    | EC2 Instance       |
| **Lambda**       | Trigger custom function     | Lambda Function    |
| **Appconfig**    | Update application config   | AppConfig          |

```python
cloudwatch.put_metric_alarm(
    AlarmName='db-scale-alarm',
    MetricName='DatabaseConnections',
    Namespace='AWS/RDS',
    Threshold=100,
    ComparisonOperator='GreaterThanThreshold',
    AlarmActions=[
        'arn:aws:sns:us-east-1:123456789012:DBAlerts',
        'arn:aws:autoscaling:us-east-1:123456789012:scalingPolicy:xyz'
    ],
    OKActions=['arn:aws:sns:us-east-1:123456789012:DBAlerts']
)
```

***

### Dashboards

#### Console View: CloudWatch Dashboards

```
CloudWatch > Dashboards > MyApplicationDashboard
┌────────────────────────────────────────────────┐
│ MyApplicationDashboard                         │
│ Last updated 5 minutes ago                     │
├────────────────────────────────────────────────┤
│ ┌──────────────────┬──────────────────────┐   │
│ │  Lambda Duration │   Error Count        │   │
│ │       245ms      │       12 errors      │   │
│ │  ╱╲  ╱╲  ╱╲     │  ▁▂▁▂▄▂▁▂▁▃▅▂▁▂▁  │   │
│ └──────────────────┴──────────────────────┘   │
│ ┌──────────────────┬──────────────────────┐   │
│ │   Requests/sec   │   API Response Time  │   │
│ │       450        │       125ms          │   │
│ │  ▄▅▆▇█▇▆▅▄▃▂▁   │   ╱──╲  ╱──╲        │   │
│ └──────────────────┴──────────────────────┘   │
│                                                │
│ [Edit] [Add Widget] [Refresh] [Share]         │
└────────────────────────────────────────────────┘
```

#### Creating Dashboards via Python

```python
import boto3
import json

cloudwatch = boto3.client('cloudwatch')

dashboard_body = {
    'widgets': [
        {
            'type': 'metric',
            'properties': {
                'metrics': [
                    ['AWS/Lambda', 'Duration', {'stat': 'Average'}],
                    ['.', 'Errors', {'stat': 'Sum'}],
                    ['.', 'Throttles', {'stat': 'Sum'}]
                ],
                'period': 300,
                'stat': 'Average',
                'region': 'us-east-1',
                'title': 'Lambda Function Metrics'
            }
        }
    ]
}

cloudwatch.put_dashboard(
    DashboardName='MyApplicationDashboard',
    DashboardBody=json.dumps(dashboard_body)
)
```

#### Widget Types

| Widget Type    | Purpose                                              |
| -------------- | ---------------------------------------------------- |
| **metric**     | Display metrics in line, area, bar, or number format |
| **log**        | Display CloudWatch Logs Insights query results       |
| **alarm**      | Show alarm status                                    |
| **number**     | Display a single metric value                        |
| **annotation** | Add vertical line for events                         |

***

### Advanced Features

#### Anomaly Detection

CloudWatch uses machine learning to detect abnormal behavior automatically.

**Setting Up Anomaly Detection**

```python
cloudwatch.put_metric_alarm(
    AlarmName='api-latency-anomaly',
    ComparisonOperator='LessThanLowerOrGreaterThanUpperThreshold',
    EvaluationPeriods=2,
    Metrics=[
        {
            'Id': 'm1',
            'ReturnData': True,
            'MetricStat': {
                'Metric': {
                    'Namespace': 'MyApp',
                    'MetricName': 'APILatency',
                    'Dimensions': [{'Name': 'Endpoint', 'Value': '/api/v1'}]
                },
                'Period': 300,
                'Stat': 'Average'
            }
        },
        {
            'Id': 'ad1',
            'Expression': 'ANOMALY_DETECTION_BAND(m1, 2)',
            'ReturnData': True
        }
    ]
)
```

#### Metric Math

Perform calculations across metrics:

```python
cloudwatch.put_metric_alarm(
    AlarmName='error-rate-alarm',
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=1,
    Threshold=5,
    Metrics=[
        {
            'Id': 'm1',
            'ReturnData': False,
            'MetricStat': {
                'Metric': {
                    'Namespace': 'MyApp',
                    'MetricName': 'Errors'
                },
                'Period': 300,
                'Stat': 'Sum'
            }
        },
        {
            'Id': 'm2',
            'ReturnData': False,
            'MetricStat': {
                'Metric': {
                    'Namespace': 'MyApp',
                    'MetricName': 'Requests'
                },
                'Period': 300,
                'Stat': 'Sum'
            }
        },
        {
            'Id': 'e1',
            'Expression': '(m1 / m2) * 100',
            'ReturnData': True
        }
    ]
)
```

#### Event-Based Monitoring

Trigger actions based on CloudWatch Events:

```python
events = boto3.client('events')

# Create rule for EC2 state changes
events.put_rule(
    Name='ec2-state-change-rule',
    EventPattern={
        'source': ['aws.ec2'],
        'detail-type': ['EC2 Instance State-change Notification'],
        'detail': {'state': ['stopped', 'terminated']}
    },
    State='ENABLED'
)

# Add Lambda as target
events.put_targets(
    Rule='ec2-state-change-rule',
    Targets=[{'Id': '1', 'Arn': 'arn:aws:lambda:region:account:function:HandleEC2Change'}]
)
```

#### Cross-Account Monitoring

Monitor resources across multiple AWS accounts using a centralized monitoring account. Metrics are aggregated through CloudWatch in the primary account with appropriate permissions granted to member accounts.

***

### Cost Optimization

#### Understanding CloudWatch Costs

CloudWatch costs vary by:

* **Metrics:** $0.10 per custom metric per month
* **Logs:** $0.50 per GB ingested, $0.03 per GB stored
* **API requests:** $0.01 per 1,000 requests
* **Alarms:** $0.10 per alarm per month

#### Cost Optimization Strategies

**1. Set Appropriate Log Retention**

```python
logs = boto3.client('logs')

# Set retention to 7 days instead of indefinite
logs.put_retention_policy(
    logGroupName='/aws/lambda/myFunction',
    retentionInDays=7  # Default is indefinite (expensive!)
)
```

**2. Use Log Filters Efficiently**

```python
# Instead of storing all logs, filter out noise
logs.put_metric_filter(
    logGroupName='/aws/lambda/myFunction',
    filterName='ErrorsOnly',
    filterPattern='[ERROR]',  # Only capture errors
    metricTransformations=[
        {
            'metricName': 'ErrorCount',
            'metricNamespace': 'MyApp',
            'metricValue': '1'
        }
    ]
)
```

**3. Use S3 Export for Long-Term Storage**

```bash
aws logs create-export-task \
  --log-group-name /aws/lambda/myFunction \
  --from 1609459200000 \
  --to 1609545600000 \
  --destination my-bucket \
  --destination-prefix logs/
```

**4. Aggregate Metrics Appropriately**

Use 5-minute aggregation (`StorageResolution: 300`) instead of 1-minute where possible to reduce API calls and costs.

**5. Use Sampling for High-Volume Metrics**

For high-frequency metrics (e.g., every request), sample at 10-20% to reduce costs while maintaining trend visibility.

```python
import random

# Only publish 10% of metrics
if random.random() < 0.1:
    cloudwatch.put_metric_data(Namespace='MyApp', MetricData=[...])
```

***

### Best Practices

#### 1. Establish a Monitoring Strategy

Define what to monitor:

* Business metrics (conversion rate, revenue)
* Application metrics (latency, error rate)
* Infrastructure metrics (CPU, memory, disk)
* Availability metrics (uptime, health checks)

#### 2. Use Consistent Naming Conventions

```
Namespace: Company/Service/Layer
MetricName: DescriptiveAction
Dimensions: Environment, Region, Instance
```

Example:

```
Namespace: Acme/PaymentService/API
MetricName: ProcessingDuration
Dimensions: Environment=Production, Region=us-east-1
```

#### 3. Implement Structured Logging

```json
{
    "timestamp": "2024-01-01T12:00:00Z",
    "level": "INFO",
    "service": "payment-api",
    "requestId": "req-123456",
    "userId": "user-789",
    "action": "ProcessPayment",
    "duration": 245,
    "status": "SUCCESS",
    "amount": 99.99
}
```

#### 4. Use Tags for Organization

```python
cloudwatch.tag_resource(
    ResourceARN='arn:aws:logs:us-east-1:123456789012:log-group:/aws/lambda/myFunction',
    Tags={
        'Environment': 'Production',
        'Application': 'PaymentService',
        'Owner': 'DevOps',
        'CostCenter': 'Engineering'
    }
)
```

#### 5. Implement Tiered Alerting

**Tier 1 - Critical:**

```python
# Immediate notification, page on-call
AlarmActions=['arn:aws:sns:us-east-1:123456789012:Critical']
```

**Tier 2 - Warning:**

```python
# Daily digest
AlarmActions=['arn:aws:sns:us-east-1:123456789012:Warnings']
```

**Tier 3 - Informational:**

```python
# Slack notification, no immediate action
AlarmActions=['arn:aws:sns:us-east-1:123456789012:Info']
```

#### 6. Test Your Alarms Regularly

```bash
# Manually trigger an alarm to test SNS notification
aws cloudwatch set-alarm-state \
  --alarm-name my-test-alarm \
  --state-value ALARM \
  --state-reason "Testing alarm configuration"
```

#### 7. Use Dashboards for Visualization

**Dashboard Best Practices:**

* Group related metrics together
* Use appropriate visualization types
* Include business and technical metrics
* Update dashboards regularly
* Create separate dashboards for different audiences

#### 8. Implement Log Retention Policies

```python
# Define retention based on log type
retention_policies = {
    '/aws/lambda/critical': 30,  # 30 days for critical functions
    '/aws/lambda/standard': 14,  # 14 days for standard functions
    '/aws/lambda/debug': 7,      # 7 days for debug logs
    '/aws/api-gateway': 30,
    '/aws/rds/error': 30,
    '/aws/rds/slowquery': 7
}

for log_group, days in retention_policies.items():
    logs.put_retention_policy(
        logGroupName=log_group,
        retentionInDays=days
    )
```

#### 9. Monitor CloudWatch Itself

```python
# Set up alarms for high log ingestion costs
cloudwatch.put_metric_alarm(
    AlarmName='high-log-volume-alarm',
    MetricName='IncomingBytes',
    Namespace='AWS/Logs',
    Statistic='Sum',
    Period=3600,
    Threshold=1073741824,  # 1 GB
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=1
)
```

#### 10. Use Infrastructure as Code

Define your CloudWatch resources (log groups, alarms, dashboards) using CloudFormation or Terraform for consistency and easy replication across environments.

**CloudFormation approach:**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyLogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: /aws/lambda/myFunction
      RetentionInDays: 7
  HighCPUAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: high-cpu-alarm
      MetricName: CPUUtilization
      Threshold: 80
```

***

### Troubleshooting Common Issues

#### Metrics Not Appearing

**Checklist:**

1. Verify IAM permissions allow `cloudwatch:PutMetricData`
2. Check metric namespace and dimensions are correctly spelled
3. Ensure application has network connectivity to CloudWatch API
4. Look for errors in application logs
5. Wait up to 1 minute for metric to appear in console

**Console Check:**

```
Metrics > All Metrics > Search for namespace
├── If empty: Verify PutMetricData permissions
├── If present but no data: Check metric dimensions
└── If error: Review IAM policy
```

#### High Log Costs

1. Review log retention policies
2. Implement sampling
3. Filter unnecessary logs
4. Use VPC Flow Logs wisely
5. Set up cost alarms

#### Alarms Not Triggering

1. Verify alarm configuration
2. Check metric data is being published
3. Ensure SNS topic is properly configured
4. Test alarm manually
5. Review alarm history

***

### Conclusion

AWS CloudWatch is a powerful monitoring platform that scales with your infrastructure. By implementing proper monitoring strategies, setting up meaningful alarms, and following best practices, you can maintain visibility into your AWS environment and respond quickly to issues.

**Key Takeaways:**

* Start with default metrics and expand to custom metrics
* Use logs for detailed insights and troubleshooting
* Set up tiered alerting for effective incident response
* Monitor costs and optimize retention policies
* Use dashboards to communicate system health
* Implement monitoring from day one of your projects

***

### Additional Resources

* [AWS CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)
* [CloudWatch Logs Insights Query Syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
* [CloudWatch API Reference](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/)
* [CloudWatch Pricing](https://aws.amazon.com/cloudwatch/pricing/)
* [AWS Observability Best Practices](https://aws-observability.github.io/observability-best-practices/)

***
