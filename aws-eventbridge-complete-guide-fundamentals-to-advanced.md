---
description: >-
  AWS EventBridge is a serverless event bus service that connects application
  data from your own apps, SaaS apps, and AWS services in real-time.
---

# AWS EventBridge Complete Guide: Fundamentals to Advanced

***

### Table of Contents

1. EventBridge Fundamentals
2. Architecture & Components
3. Event Concepts
4. Rules & Patterns
5. Event Sources & Targets
6. Configuration & Setup
7. Advanced Topics
8. Troubleshooting
9. Best Practices

***

### EventBridge Fundamentals

#### What is AWS EventBridge?

EventBridge is a serverless event bus that receives events from sources and routes them to targets based on rules.

```
Event Source
    │
    ├─ AWS Services (EC2, S3, RDS)
    ├─ Custom Applications
    ├─ SaaS Applications (Datadog, Stripe, etc.)
    └─ Partner Event Sources
         │
         ↓
    EventBridge Event Bus
         │
    Rules & Patterns
         │
    Process → Filter → Route
         │
         ↓
    Event Targets
    ├─ Lambda
    ├─ SNS
    ├─ SQS
    ├─ Kinesis
    ├─ Step Functions
    └─ HTTP Endpoints
```

**Key Characteristics:**

* Serverless: No infrastructure to manage
* Real-time: Millisecond latency
* Scalable: Handles millions of events per second
* Flexible: Supports complex routing rules
* Reliable: At-least-once delivery guarantee
* Pay-per-use: Only pay for events processed

#### Why Use EventBridge?

| Without EventBridge  | With EventBridge          |
| -------------------- | ------------------------- |
| Tightly coupled apps | Loosely coupled apps      |
| Direct API calls     | Event-driven architecture |
| Complex integrations | Simple rule-based routing |
| Manual polling       | Real-time processing      |
| Hard to scale        | Scales automatically      |

***

### Architecture & Components

#### EventBridge Architecture Flow

```
EVENT SOURCES
    │
    ├─ AWS Services
    │  (Events published automatically)
    │
    ├─ Custom Applications
    │  (Send via SDK/API)
    │
    ├─ SaaS & Partner Events
    │  (From AWS Partner Network)
    │
    └─ Scheduled Events (Cron/Rate)
         │
         ↓
    DEFAULT EVENT BUS (or Custom Bus)
         │
    Rules Engine
    ├─ Rule 1: Pattern matching
    ├─ Rule 2: Filtering
    ├─ Rule 3: Transformations
         │
         ↓
    EVENT TARGETS
         │
    ├─ Compute: Lambda, ECS, Fargate
    ├─ Messaging: SNS, SQS
    ├─ Streaming: Kinesis, Data Firehose
    ├─ Orchestration: Step Functions, SFN
    ├─ Destinations: HTTP, API Gateway
    ├─ Storage: S3, DynamoDB
    └─ Monitoring: CloudWatch Logs
```

#### Key Components

**Event Bus**

* Default event bus: Receives AWS service events
* Custom event buses: Isolated event channels
* Cross-account/region: Share events across AWS accounts

**Rules**

* Define which events to process
* Use patterns to match event structure
* Can transform events before sending to targets
* Up to 300 rules per event bus

**Targets**

* Destinations for matched events
* Can invoke multiple targets per rule
* Optional Dead Letter Queue (DLQ)
* Retry policy: Exponential backoff (0-86400 seconds)

**Events**

* JSON format: Structured data
* Event source field: Identifies origin
* Detail type: Categorizes event
* Detail field: Event-specific payload
* Maximum size: 256 KB

***

### Event Concepts

#### Event Structure

```json
{
  "version": "0",
  "id": "abc123def456",
  "detail-type": "EC2 Instance State-change Notification",
  "source": "aws.ec2",
  "account": "123456789012",
  "time": "2024-01-15T12:30:45Z",
  "region": "us-east-1",
  "resources": [
    "arn:aws:ec2:us-east-1:123456789012:instance/i-1234567890abcdef0"
  ],
  "detail": {
    "instance-id": "i-1234567890abcdef0",
    "state": "running",
    "state-transition-reason": "User initiated"
  }
}
```

**Fields:**

* **version**: Event format version (always "0")
* **id**: Unique event identifier
* **detail-type**: Type of event
* **source**: Service/application that sent event
* **account**: AWS account ID
* **time**: Event timestamp (ISO 8601)
* **region**: AWS region
* **resources**: Related resource ARNs
* **detail**: Event-specific data

#### Event Sources

| Source       | Events         | Use Case                |
| ------------ | -------------- | ----------------------- |
| AWS Services | Native events  | Automated workflows     |
| Custom Apps  | PutEvents API  | Application integration |
| SaaS Apps    | Partner events | Third-party tracking    |
| Scheduled    | Cron/Rate      | Periodic tasks          |

**AWS Service Events (Common):**

```
aws.ec2          - Instance, volume, snapshot changes
aws.s3           - Object created, deleted, restored
aws.rds          - DB instance, cluster changes
aws.lambda       - Function execution, errors
aws.iam          - User, role, policy changes
aws.cloudformation - Stack events
aws.autoscaling  - Launch/terminate instances
aws.health       - Account health
```

**Custom Application Event:**

```json
{
  "Source": "myapp.orders",
  "DetailType": "Order Placed",
  "Detail": {
    "orderId": "12345",
    "customerId": "cust-789",
    "amount": 99.99,
    "items": 3
  }
}
```

#### Event Bus Types

**Default Event Bus**

* Receives events from AWS services
* Available in every AWS account
* Cannot be deleted
* Shared by all rules in account

```
AWS Services → Default Event Bus → All Rules
```

**Custom Event Bus**

* Isolated event channels
* For custom applications
* Multi-tenant scenarios
* Cross-account access possible

```
Custom App → Custom Event Bus → Dedicated Rules
```

**Partner Event Bus**

* Receive events from SaaS partners
* Datadog, Zendesk, Auth0, Twilio, etc.
* Partner activates, you receive
* No additional cost

***

### Rules & Patterns

#### Rule Basics

A rule specifies which events to match and which targets to invoke.

**Simple Rule**

```json
{
  "Name": "ProcessOrderRule",
  "EventBusName": "default",
  "EventPattern": {
    "source": ["myapp.orders"],
    "detail-type": ["Order Placed"]
  },
  "State": "ENABLED",
  "Targets": [
    {
      "Arn": "arn:aws:lambda:us-east-1:123456789012:function:ProcessOrder",
      "RoleArn": "arn:aws:iam::123456789012:role/service-role",
      "RetryPolicy": {
        "MaximumEventAge": 3600,
        "MaximumRetryAttempts": 2
      }
    }
  ]
}
```

#### Pattern Matching

**Exact Match**

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"]
}
```

Matches events where source is exactly "aws.ec2"

**Multiple Values (OR)**

```json
{
  "source": ["aws.ec2", "aws.rds"],
  "detail-type": ["EC2 Instance State-change Notification", "RDS DB Instance Event"]
}
```

Matches if source is EITHER "aws.ec2" OR "aws.rds"

**Nested Property Match**

```json
{
  "detail": {
    "state": ["running", "stopped"]
  }
}
```

Matches events where detail.state is "running" or "stopped"

**Wildcard Matching**

```json
{
  "source": ["aws.*"],
  "detail-type": ["*"]
}
```

Matches all events from any AWS service

**Numeric Comparison**

```json
{
  "detail": {
    "amount": [{"numeric": [">", 100]}]
  }
}
```

Matches events where amount > 100

**Prefix Matching**

```json
{
  "detail": {
    "orderId": [{"prefix": "ORD-"}]
  }
}
```

Matches order IDs starting with "ORD-"

**Exists Check**

```json
{
  "detail": {
    "result": [{"exists": true}]
  }
}
```

Matches events that have "result" field

**Complex Pattern Example**

```json
{
  "source": ["myapp.orders"],
  "detail-type": ["Order Placed"],
  "detail": {
    "status": ["pending"],
    "amount": [{"numeric": [">", 50]}],
    "customerId": [{"prefix": "PREMIUM-"}]
  }
}
```

Matches: orders from myapp.orders, status pending, amount > 50, premium customers

#### Scheduled Rules (Cron/Rate)

**Rate Expression (Fixed Interval)**

```
rate(5 minutes)      - Every 5 minutes
rate(1 hour)         - Every 1 hour
rate(7 days)         - Every 7 days
```

**Cron Expression (Specific Time)**

```
cron(0 12 * * ? *)   - Every day at 12:00 PM UTC
cron(0 8 * * MON-FRI *) - Weekdays at 8:00 AM
cron(0 0 1 * ? *)    - First day of month at midnight
```

**Example: Daily Report**

```json
{
  "Name": "DailyReportRule",
  "ScheduleExpression": "cron(0 8 * * ? *)",
  "Targets": [
    {
      "Arn": "arn:aws:lambda:us-east-1:123456789012:function:GenerateReport",
      "RoleArn": "arn:aws:iam::123456789012:role/service-role"
    }
  ]
}
```

***

### Event Sources & Targets

#### Common Event Sources

| Source        | Frequency           | Use Cases                   |
| ------------- | ------------------- | --------------------------- |
| EC2           | Instance changes    | Auto-scaling, monitoring    |
| S3            | Object operations   | Image processing, archiving |
| RDS           | Database events     | Backup, alerts, auditing    |
| Lambda        | Invocations, errors | Error handling, logging     |
| CodePipeline  | Pipeline stages     | CI/CD automation            |
| CloudTrail    | API calls           | Security, compliance, audit |
| Health Events | Account health      | Alerting, incident response |

#### Common Event Targets

| Target          | Use Case                   | Max Capacity             |
| --------------- | -------------------------- | ------------------------ |
| Lambda          | Processing, transformation | 900 concurrent           |
| SNS             | Notifications, fan-out     | Unlimited                |
| SQS             | Queuing, buffering         | 120,000 msgs/min         |
| Kinesis         | Streaming, real-time       | Shard capacity           |
| Step Functions  | Orchestration, workflows   | Account limits           |
| HTTP Endpoint   | External APIs, webhooks    | Depends on target        |
| S3              | Event logging, storage     | Unlimited                |
| DynamoDB        | Store event data           | On-demand or provisioned |
| CloudWatch Logs | Logging, monitoring        | Unlimited                |

#### Lambda Target Example

```json
{
  "Type": "AWS::Events::Rule",
  "Properties": {
    "Name": "ProcessS3EventRule",
    "EventBusName": "default",
    "EventPattern": {
      "source": ["aws.s3"],
      "detail-type": ["Object Created"]
    },
    "State": "ENABLED",
    "Targets": [
      {
        "Arn": "arn:aws:lambda:us-east-1:123456789012:function:ProcessImage",
        "RoleArn": "arn:aws:iam::123456789012:role/service-role",
        "DeadLetterConfig": {
          "Arn": "arn:aws:sqs:us-east-1:123456789012:dlq"
        },
        "RetryPolicy": {
          "MaximumEventAge": 3600,
          "MaximumRetryAttempts": 2
        }
      }
    ]
  }
}
```

#### SNS Target Example

```json
{
  "Type": "AWS::Events::Rule",
  "Properties": {
    "EventPattern": {
      "source": ["aws.ec2"],
      "detail-type": ["EC2 Instance State-change Notification"],
      "detail": {
        "state": ["terminated"]
      }
    },
    "State": "ENABLED",
    "Targets": [
      {
        "Arn": "arn:aws:sns:us-east-1:123456789012:AlertTopic",
        "RoleArn": "arn:aws:iam::123456789012:role/service-role"
      }
    ]
  }
}
```

#### SQS Target Example

```json
{
  "Type": "AWS::Events::Rule",
  "Properties": {
    "EventPattern": {
      "source": ["myapp.orders"]
    },
    "Targets": [
      {
        "Arn": "arn:aws:sqs:us-east-1:123456789012:OrderQueue",
        "RoleArn": "arn:aws:iam::123456789012:role/service-role",
        "SqsParameters": {
          "MessageGroupId": "order-processing"
        }
      }
    ]
  }
}
```

***

### Configuration & Setup

#### AWS CLI Commands

**Create Custom Event Bus**

```bash
aws events create-event-bus \
  --name my-custom-bus \
  --region us-east-1
```

**Put Events**

```bash
aws events put-events \
  --entries file://event.json \
  --region us-east-1
```

**Event JSON File (event.json)**

```json
[
  {
    "Source": "myapp.orders",
    "DetailType": "Order Placed",
    "Detail": "{\"orderId\":\"12345\",\"amount\":99.99}",
    "EventBusName": "default"
  }
]
```

**Create Rule**

```bash
aws events put-rule \
  --name ProcessOrderRule \
  --event-bus-name default \
  --event-pattern '{"source":["myapp.orders"],"detail-type":["Order Placed"]}' \
  --state ENABLED \
  --region us-east-1
```

**Add Target to Rule**

```bash
aws events put-targets \
  --rule ProcessOrderRule \
  --event-bus-name default \
  --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:123456789012:function:ProcessOrder","RoleArn"="arn:aws:iam::123456789012:role/service-role" \
  --region us-east-1
```

**List Rules**

```bash
aws events list-rules --event-bus-name default
```

**Describe Rule**

```bash
aws events describe-rule --name ProcessOrderRule
```

**Delete Rule (and targets)**

```bash
aws events remove-targets --rule ProcessOrderRule --ids "1"
aws events delete-rule --name ProcessOrderRule
```

#### CloudFormation Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'EventBridge Setup for Order Processing'

Resources:
  # Custom Event Bus
  OrderEventBus:
    Type: AWS::Events::EventBus
    Properties:
      Name: order-events

  # IAM Role for EventBridge
  EventBridgeRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: events.amazonaws.com
            Action: sts:AssumeRole
      Policies:
        - PolicyName: LambdaInvoke
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action: lambda:InvokeFunction
                Resource: !GetAtt ProcessOrderFunction.Arn

  # EventBridge Rule
  ProcessOrderRule:
    Type: AWS::Events::Rule
    Properties:
      Name: ProcessOrderRule
      EventBusName: !Ref OrderEventBus
      EventPattern:
        source:
          - myapp.orders
        detail-type:
          - Order Placed
      State: ENABLED
      Targets:
        - Arn: !GetAtt ProcessOrderFunction.Arn
          RoleArn: !GetAtt EventBridgeRole.Arn
          RetryPolicy:
            MaximumEventAge: 3600
            MaximumRetryAttempts: 2
          DeadLetterConfig:
            Arn: !GetAtt DLQ.Arn

  # Lambda Function
  ProcessOrderFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: ProcessOrder
      Runtime: python3.11
      Handler: index.lambda_handler
      Code:
        ZipFile: |
          import json
          def lambda_handler(event, context):
              print(f"Processing order: {json.dumps(event)}")
              return {"statusCode": 200}

  # Dead Letter Queue
  DLQ:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: eventbridge-dlq

Outputs:
  EventBusName:
    Value: !Ref OrderEventBus
  RuleArn:
    Value: !GetAtt ProcessOrderRule.Arn
```

#### Terraform Configuration

```hcl
# EventBridge Rule
resource "aws_cloudwatch_event_rule" "process_order" {
  name           = "ProcessOrderRule"
  event_bus_name = "default"
  
  event_pattern = jsonencode({
    source      = ["myapp.orders"]
    detail-type = ["Order Placed"]
  })
}

# EventBridge Target
resource "aws_cloudwatch_event_target" "lambda" {
  rule       = aws_cloudwatch_event_rule.process_order.name
  target_id  = "ProcessOrderLambda"
  arn        = aws_lambda_function.process_order.arn
  role_arn   = aws_iam_role.eventbridge_role.arn

  retry_policy {
    maximum_event_age       = 3600
    maximum_retry_attempts  = 2
  }

  dead_letter_config {
    arn = aws_sqs_queue.dlq.arn
  }
}

# Lambda Permission
resource "aws_lambda_permission" "eventbridge" {
  statement_id  = "AllowExecutionFromEventBridge"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.process_order.function_name
  principal     = "events.amazonaws.com"
  source_arn    = aws_cloudwatch_event_rule.process_order.arn
}
```

***

### Advanced Topics

#### Event Transformation

Transform event data before sending to target.

```json
{
  "Type": "AWS::Events::Rule",
  "Properties": {
    "EventPattern": {
      "source": ["myapp.orders"]
    },
    "Targets": [
      {
        "Arn": "arn:aws:lambda:us-east-1:123456789012:function:ProcessOrder",
        "RoleArn": "arn:aws:iam::123456789012:role/service-role",
        "InputTransformer": {
          "InputPathsMap": {
            "orderId": "$.detail.orderId",
            "amount": "$.detail.amount"
          },
          "InputTemplate": "{\"id\":\"<orderId>\",\"total\":<amount>,\"timestamp\":\"<aws.events.event.ingestion-time>\"}"
        }
      }
    ]
  }
}
```

#### Cross-Account Events

Allow EventBridge to receive events from other AWS accounts.

**Sender Account (123456789012):**

```json
{
  "Type": "AWS::Events::Rule",
  "Properties": {
    "EventBusName": "default",
    "EventPattern": {
      "source": ["myapp"]
    },
    "Targets": [
      {
        "Arn": "arn:aws:events:us-east-1:987654321098:event-bus/default",
        "RoleArn": "arn:aws:iam::123456789012:role/cross-account-role"
      }
    ]
  }
}
```

**Receiver Account (987654321098) - Event Bus Policy:**

```json
{
  "Statement": [
    {
      "Sid": "AllowCrossAccount",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "events:PutEvents",
      "Resource": "arn:aws:events:us-east-1:987654321098:event-bus/default"
    }
  ]
}
```

#### Cross-Region Events

Replicate events across regions using EventBridge.

```json
{
  "Type": "AWS::Events::Rule",
  "Properties": {
    "EventBusName": "default",
    "EventPattern": {
      "source": ["myapp"]
    },
    "Targets": [
      {
        "Arn": "arn:aws:events:eu-west-1:123456789012:event-bus/default",
        "RoleArn": "arn:aws:iam::123456789012:role/cross-region-role"
      }
    ]
  }
}
```

#### Event Archive & Replay

Archive events and replay them for recovery.

**Create Archive:**

```bash
aws events create-event-source-mapping \
  --event-bus-name default \
  --archive-name my-archive \
  --retention-days 7 \
  --region us-east-1
```

**Replay Events:**

```bash
aws events start-replay \
  --archive-name my-archive \
  --event-start-time 2024-01-15T00:00:00Z \
  --event-end-time 2024-01-16T00:00:00Z \
  --event-source-arn arn:aws:events:us-east-1:123456789012:event-bus/default \
  --region us-east-1
```

#### Event Bus Policy

Control who can access event bus.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": [
        "events:PutEvents",
        "events:PutRule"
      ],
      "Resource": "arn:aws:events:us-east-1:123456789012:event-bus/custom-bus"
    }
  ]
}
```

***

### Troubleshooting

#### Rule Not Triggering

**Check 1: Event Pattern Matches**

```bash
# Test event pattern
aws events test-event-pattern \
  --event-pattern '{"source":["myapp.orders"]}' \
  --event '{"source":"myapp.orders","detail-type":"Order Placed"}'
```

**Check 2: Rule is Enabled**

```bash
aws events describe-rule --name ProcessOrderRule
# Look for "State": "ENABLED"
```

**Check 3: Target has Permission**

```bash
# For Lambda targets
aws lambda get-policy --function-name ProcessOrder
# Should show events.amazonaws.com principal
```

**Check 4: Event Bus is Correct**

```bash
aws events list-rules --event-bus-name default
# Verify rule is listed
```

#### Events Not Reaching Targets

**Check CloudWatch Logs:**

```bash
# View EventBridge invocations
aws logs tail /aws/events/my-rule --follow

# Lambda logs
aws logs tail /aws/lambda/ProcessOrder --follow
```

**Check DLQ:**

```bash
# Receive messages from DLQ
aws sqs receive-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/eventbridge-dlq

# Shows which events failed
```

**Common Issues:**

| Issue                  | Cause               | Solution                                           |
| ---------------------- | ------------------- | -------------------------------------------------- |
| No events received     | Wrong event pattern | Verify pattern matches event structure             |
| Timeout errors         | Lambda slow         | Increase timeout, optimize code                    |
| Permission denied      | Missing IAM role    | Add lambda:InvokeFunction permission               |
| No output              | Rule not enabled    | Enable rule: `aws events put-rule --state ENABLED` |
| DLQ receiving messages | Target failing      | Check target logs, fix code                        |

#### Debugging Commands

```bash
# See recent rule invocations
aws cloudwatch list-metrics \
  --namespace AWS/Events \
  --metric-name Invocations

# Check rule statistics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Events \
  --metric-name Invocations \
  --dimensions Name=Rule,Value=ProcessOrderRule \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-16T00:00:00Z \
  --period 300 \
  --statistics Sum

# Test rule locally (without sending to target)
aws events put-events \
  --entries file://test-event.json \
  --region us-east-1
```

***

### Best Practices

#### Design Patterns

**Fan-out Pattern**

```
Single Event → Multiple Targets
   Order Event
      ├─→ Lambda (Process)
      ├─→ SNS (Notify)
      ├─→ SQS (Queue)
      └─→ DynamoDB (Log)
```

**Filter & Route Pattern**

```
Events with Filters
   Order Event
      ├─→ Amount > $100 → Premium Processing
      ├─→ Amount $10-100 → Standard Processing
      └─→ Amount < $10 → Auto-approve
```

**Aggregation Pattern**

```
Multiple Events → Aggregation → Action
   Order Events (collect 10)
      ↓
   Batch Process
      ↓
   Send to Data Warehouse
```

#### Error Handling

**Use Dead Letter Queues**

```json
{
  "DeadLetterConfig": {
    "Arn": "arn:aws:sqs:us-east-1:123456789012:dlq"
  }
}
```

**Configure Retry Policy**

```json
{
  "RetryPolicy": {
    "MaximumEventAge": 3600,
    "MaximumRetryAttempts": 2
  }
}
```

**Exponential Backoff Retries:**

```
Attempt 1: Immediate
Attempt 2: 1-2 seconds
Attempt 3: 2-4 seconds
...
After final attempt → DLQ
```

#### Performance Optimization

**Batch Processing**

```json
{
  "BatchParameters": {
    "BatchSize": 10,
    "MaximumBatchingWindowInSeconds": 30
  }
}
```

**Reduce Payload Size**

```json
{
  "InputTransformer": {
    "InputPathsMap": {
      "orderId": "$.detail.orderId"
    },
    "InputTemplate": "{\"id\":\"<orderId>\"}"
  }
}
```

**Parallel Processing**

```
Multiple SQS Queues
   Event → Queue 1 → Worker 1
        → Queue 2 → Worker 2
        → Queue 3 → Worker 3
```

#### Monitoring & Alerting

**CloudWatch Metrics to Monitor**

| Metric            | What to Watch    |
| ----------------- | ---------------- |
| Invocations       | Events processed |
| FailedInvocations | Failed targets   |
| ThrottledRules    | Rate limiting    |
| TriggeredRules    | Rule executions  |

**Create Alarms:**

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name EventBridgeFailures \
  --alarm-description "Alert on EventBridge failures" \
  --metric-name FailedInvocations \
  --namespace AWS/Events \
  --statistic Sum \
  --period 300 \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1
```

#### Naming Conventions

```
Rules:
  ProcessOrderRule
  SendNotificationRule
  DailyReportRule
  
Event Buses:
  order-events
  payment-events
  notification-events
  
Lambda Functions:
  process-order
  send-notification
  generate-report
```

#### Security Best Practices

**IAM Role (Least Privilege)**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "lambda:InvokeFunction"
      ],
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ProcessOrder"
    }
  ]
}
```

**Event Bus Policy (Restrict Access)**

```json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::123456789012:root"
  },
  "Action": "events:PutEvents",
  "Resource": "arn:aws:events:us-east-1:123456789012:event-bus/custom-bus",
  "Condition": {
    "StringEquals": {
      "events:source": "myapp.orders"
    }
  }
}
```

**Encrypt Events (Optional)**

* Use VPC endpoints for private EventBridge
* Encrypt payloads at application level
* Use Secrets Manager for sensitive data

#### Capacity Planning

**EventBridge Limits:**

| Metric                  | Limit          |
| ----------------------- | -------------- |
| Rules per event bus     | 300            |
| Targets per rule        | 5 (soft limit) |
| Events per second       | Unlimited      |
| Payload size            | 256 KB         |
| Event archive retention | 0-3650 days    |
| Rule name length        | 64 characters  |

**Scaling Considerations:**

* EventBridge scales automatically
* No provisioning needed
* Costs increase with event volume
* Monitor invocation metrics

#### Cost Optimization

**Pay only for:**

* Events published to custom event buses
* Events published via PutEvents API
* Ingestion of partner events

**Not charged for:**

* Events from AWS services (default bus)
* Rule evaluations
* Managed rules

**Tips:**

* Use event filtering to reduce costs
* Archive and replay instead of reprocessing
* Batch events when possible
* Consider SNS/SQS for high-volume low-priority events

***

### Quick Reference

#### Common AWS CLI Commands

```bash
# Create rule
aws events put-rule --name MyRule --event-pattern '{...}'

# Add target
aws events put-targets --rule MyRule --targets Id=1,Arn=arn:...

# List rules
aws events list-rules --event-bus-name default

# Delete rule
aws events remove-targets --rule MyRule --ids 1
aws events delete-rule --name MyRule

# Test pattern
aws events test-event-pattern --event-pattern '{...}' --event '{...}'

# Put events
aws events put-events --entries '[{...}]'

# List targets
aws events list-targets-by-rule --rule MyRule
```

#### Event Pattern Operators

| Operator     | Example                           |
| ------------ | --------------------------------- |
| Exact match  | `"state": ["running"]`            |
| OR logic     | `"state": ["running", "stopped"]` |
| Wildcard     | `"source": ["aws.*"]`             |
| Numeric      | `[{"numeric": [">", 100]}]`       |
| Prefix       | `[{"prefix": "ORD-"}]`            |
| Exists       | `[{"exists": true}]`              |
| Anything-but | `[{"anything-but": "error"}]`     |

#### Common Event Source Formats

**S3:**

```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": { "name": ["my-bucket"] }
  }
}
```

**EC2:**

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["running", "stopped"]
  }
}
```

**Custom:**

```json
{
  "source": ["myapp.orders"],
  "detail-type": ["Order Placed"],
  "detail": {
    "status": ["pending"]
  }
}
```

***

### Summary

**EventBridge Key Points:**

* Serverless event bus for event-driven architectures
* Routes events from sources to targets based on rules
* Supports AWS services, custom apps, and SaaS partners
* Pattern matching for flexible filtering
* Built-in reliability with retries and DLQs
* No infrastructure to manage

**Typical Flow:**

```
Event Source → Event Bus → Rule (Filter) → Target (Action)
```

**Use Cases:**

* Application integration
* Microservices orchestration
* Real-time monitoring
* Automated workflows
* Cross-account/region event routing

**Best Practices:**

1. Use DLQ for error handling
2. Filter events to reduce costs
3. Implement retry policies
4. Monitor with CloudWatch
5. Use least privilege IAM roles
6. Archive events for replay
7. Batch process when possible
8. Test patterns before deploying

***

### Resources

* [AWS EventBridge Documentation](https://docs.aws.amazon.com/eventbridge/)
* [EventBridge Event Pattern Guide](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-and-event-patterns.html)
* [EventBridge Service Integrations](https://docs.aws.amazon.com/eventbridge/latest/userguide/what-is-amazon-eventbridge.html)
* [AWS EventBridge Samples](https://github.com/aws-samples/amazon-eventbridge-samples)
* [EventBridge Pricing](https://aws.amazon.com/eventbridge/pricing/)
