---
description: From Beginner to Advanced
---

# Virtual Private Cloud (VPC) - Complete Guide

***

### Table of Contents

1. [Beginner Level](https://claude.ai/chat/f0e01f2b-75a0-46dd-9ef6-780d945e1251#beginner-level)
   * What is VPC?
   * Why VPC Matters
   * Core Components
   * Getting Started
2. [Intermediate Level](https://claude.ai/chat/f0e01f2b-75a0-46dd-9ef6-780d945e1251#intermediate-level)
   * Subnets Deep Dive
   * Routing & Route Tables
   * Security Groups & NACLs
   * Internet Connectivity
3. [Advanced Level](https://claude.ai/chat/f0e01f2b-75a0-46dd-9ef6-780d945e1251#advanced-level)
   * VPC Peering
   * VPN & Hybrid Connectivity
   * Multi-Account Architecture
   * Advanced Security Patterns
4. [Real-World Scenarios](https://claude.ai/chat/f0e01f2b-75a0-46dd-9ef6-780d945e1251#real-world-scenarios)

***

### BEGINNER LEVEL

#### What is VPC?

A **Virtual Private Cloud (VPC)** is a virtual network environment in a cloud provider (like AWS, Azure, or GCP) that allows you to launch cloud resources in an isolated, logically separated network. Think of it as your own private section of the cloud where you have complete control over networking.

**Key Analogy:** If the cloud is like a large apartment building, your VPC is like having your own private floor with your own apartment, hallways, and security rules.

#### Why VPC Matters

| Aspect                | Benefit                                                         |
| --------------------- | --------------------------------------------------------------- |
| **Security**          | Isolate your resources from others; control who can access what |
| **Control**           | Define your own IP address ranges, subnets, and routing rules   |
| **Performance**       | Low-latency communication between your resources                |
| **Compliance**        | Meet regulatory requirements for data isolation and security    |
| **Cost Optimization** | Efficient resource allocation and traffic management            |

#### Core Components of VPC

**1. IP Address Space (CIDR Block)**

The range of IP addresses your VPC can use. Example: `10.0.0.0/16` means you have 65,536 available IP addresses.

```
VPC CIDR: 10.0.0.0/16 (Main network)
├── Subnet 1: 10.0.1.0/24 (256 addresses)
├── Subnet 2: 10.0.2.0/24 (256 addresses)
└── Subnet 3: 10.0.3.0/24 (256 addresses)
```

**2. Subnets**

Subdivisions of your VPC where you place actual resources (EC2 instances, databases, etc.). Each subnet must be in a specific Availability Zone.

* **Public Subnet:** Instances can communicate with the internet
* **Private Subnet:** Instances cannot directly reach the internet

**3. Internet Gateway (IGW)**

The door between your VPC and the internet. Without it, your VPC is completely isolated.

**4. Route Tables**

Rules that determine where network traffic goes. Example: "If destination is 0.0.0.0/0 (anywhere on internet), send through Internet Gateway."

**5. NAT Gateway**

Allows instances in private subnets to initiate outbound internet connections while remaining private.

***

### Simple VPC Architecture Diagram (Beginner)

```
┌─────────────────────────────────────────────────────┐
│              AWS Region (us-east-1)                 │
│                                                     │
│  ┌───────────────────────────────────────────────┐  │
│  │    VPC (10.0.0.0/16)                         │  │
│  │                                               │  │
│  │  ┌──────────────────┐  ┌──────────────────┐  │  │
│  │  │  Public Subnet   │  │  Private Subnet  │  │  │
│  │  │  (10.0.1.0/24)  │  │  (10.0.2.0/24)  │  │  │
│  │  │                  │  │                  │  │  │
│  │  │ [Web Server]     │  │ [Database]       │  │  │
│  │  │ [App Server]     │  │                  │  │  │
│  │  └──────────────────┘  └──────────────────┘  │  │
│  │          ↓                      ↓             │  │
│  │    [Internet Gateway]    [NAT Gateway]       │  │
│  │          ↓                      ↓             │  │
│  └──────────────────────────────────────────────┘  │
│                      ↓                              │
│            [Internet / AWS Services]                │
└─────────────────────────────────────────────────────┘
```

***

#### Getting Started: Basic VPC Setup

**Step 1: Create a VPC**

Define your IP address space:

```yaml
VPC Name: my-production-vpc
CIDR Block: 10.0.0.0/16
```

**Step 2: Create Subnets**

```yaml
Public Subnet:
  Name: public-subnet-1a
  CIDR: 10.0.1.0/24
  Availability Zone: us-east-1a

Private Subnet:
  Name: private-subnet-1a
  CIDR: 10.0.2.0/24
  Availability Zone: us-east-1a
```

**Step 3: Add Internet Gateway**

```yaml
Name: my-igw
Attach to: my-production-vpc
```

**Step 4: Configure Route Tables**

```yaml
Public Route Table:
  - Destination: 0.0.0.0/0 → Target: Internet Gateway
  - Associated Subnets: public-subnet-1a

Private Route Table:
  - Destination: 0.0.0.0/0 → Target: NAT Gateway
  - Associated Subnets: private-subnet-1a
```

***

### INTERMEDIATE LEVEL

#### Subnets Deep Dive

**Public vs Private Subnets: Detailed Comparison**

| Feature                   | Public Subnet                     | Private Subnet                    |
| ------------------------- | --------------------------------- | --------------------------------- |
| **Route to Internet**     | Via Internet Gateway              | Via NAT Gateway (optional)        |
| **Inbound from Internet** | Can receive (via security groups) | Cannot receive                    |
| **Auto-assign Public IP** | Can be enabled                    | Disabled by default               |
| **Use Cases**             | Web servers, Load balancers       | Databases, Cache, Background jobs |
| **Typical Cost**          | Lower                             | Lower (NAT costs extra)           |

**Multi-AZ Subnet Strategy**

```
Region: us-east-1
│
├─ Availability Zone 1a
│  ├─ Public Subnet (10.0.1.0/24)
│  └─ Private Subnet (10.0.2.0/24)
│
├─ Availability Zone 1b
│  ├─ Public Subnet (10.0.11.0/24)
│  └─ Private Subnet (10.0.12.0/24)
│
└─ Availability Zone 1c
   ├─ Public Subnet (10.0.21.0/24)
   └─ Private Subnet (10.0.22.0/24)
```

**Benefits:**

* **High Availability:** If one AZ fails, others continue working
* **Load Distribution:** Spread traffic across AZs
* **Disaster Recovery:** Automatic failover capabilities

***

#### Routing & Route Tables

**Understanding Route Tables**

A route table is a set of rules (routes) that determine where network traffic is directed.

**Basic Route Table Example:**

```
Route Table: public-routes

Priority | Destination | Target | Status
---------|-------------|--------|--------
1        | 10.0.0.0/16 | Local  | Active (local VPC traffic)
2        | 0.0.0.0/0   | igw-   | Active (internet traffic)
         |             | xxx    |
```

**How it works:**

1. Packet arrives with destination IP 8.8.8.8
2. Checks route 1: Does 8.8.8.8 match 10.0.0.0/16? NO
3. Checks route 2: Does 8.8.8.8 match 0.0.0.0/0? YES → Send to Internet Gateway

**Advanced Routing Patterns**

**Pattern 1: Gateway Prefix Lists**

```
Route: Destination traffic to S3
  Destination: pl-12345678 (S3 prefix list)
  Target: S3 Gateway Endpoint (No NAT cost!)
```

**Pattern 2: VPN Routing**

```
Route: Corporate network traffic
  Destination: 172.16.0.0/12
  Target: Virtual Private Gateway
```

***

#### Security Groups & Network ACLs

**Security Groups (Stateful Firewall)**

Think of security groups as per-instance firewalls.

**Characteristics:**

* Stateful: If you allow outbound, return traffic is automatically allowed
* Applied at instance level
* Default: Deny all inbound, allow all outbound

**Example Security Group Rules:**

```yaml
SecurityGroup: web-server-sg

Inbound Rules:
  - Type: HTTP
    Protocol: TCP
    Port: 80
    Source: 0.0.0.0/0 (Anyone)
    
  - Type: HTTPS
    Protocol: TCP
    Port: 443
    Source: 0.0.0.0/0 (Anyone)
    
  - Type: SSH
    Protocol: TCP
    Port: 22
    Source: 203.0.113.0/24 (Admin network)

Outbound Rules:
  - Type: All traffic
    Protocol: All
    Destination: 0.0.0.0/0
```

**Network ACLs (Stateless Firewall)**

NACLs operate at subnet level and are stateless.

**Characteristics:**

* Stateless: Must explicitly allow return traffic
* Applied at subnet level
* Processed in rule number order
* Default: Allow all

**Example NACL Rules:**

```yaml
NACL: public-subnet-nacl

Inbound Rules (processed in order):
  100 | HTTP   | TCP | 80    | 0.0.0.0/0     | ALLOW
  110 | HTTPS  | TCP | 443   | 0.0.0.0/0     | ALLOW
  120 | SSH    | TCP | 22    | 203.0.113.0/24| ALLOW
  140 | Ephemeral| TCP | 1024-65535 | 0.0.0.0/0 | ALLOW
  *   | All    | All | All   | All           | DENY

Outbound Rules:
  100 | HTTP   | TCP | 80    | 0.0.0.0/0     | ALLOW
  110 | HTTPS  | TCP | 443   | 0.0.0.0/0     | ALLOW
  120 | Ephemeral| TCP | 1024-65535 | 0.0.0.0/0 | ALLOW
  *   | All    | All | All   | All           | DENY
```

**Security Groups vs NACLs**

| Aspect          | Security Groups              | NACLs                          |
| --------------- | ---------------------------- | ------------------------------ |
| **Level**       | Instance                     | Subnet                         |
| **Stateful**    | Yes                          | No                             |
| **Processing**  | All rules (implicit deny)    | Order matters (explicit rules) |
| **Default**     | Deny inbound, allow outbound | Allow all                      |
| **Association** | Multiple per instance        | One per subnet                 |
| **Performance** | Slightly slower              | Slightly faster                |

***

#### Internet Connectivity Options

**Option 1: Internet Gateway (IGW)**

For instances with public IP addresses in public subnets.

```
Instance (Public IP: 54.123.45.67)
         ↓
    [Security Group: Allow HTTP]
         ↓
  [Route Table: 0.0.0.0/0 → IGW]
         ↓
    [Internet Gateway]
         ↓
    [Internet]
```

**Costs:** Free **Use Case:** Web servers, load balancers **Security:** Instances are directly exposed

**Option 2: NAT Gateway**

For instances in private subnets to initiate outbound connections.

```
Instance (Private IP: 10.0.2.5)
         ↓
    [Security Group: Allow Outbound]
         ↓
  [Route Table: 0.0.0.0/0 → NAT Gateway]
         ↓
    [NAT Gateway] (Public IP: 54.123.45.68)
         ↓
    [Internet]
```

**Costs:** $0.045/hour + data processing charges **Use Case:** Private instances needing outbound internet **Security:** Instances remain private; external traffic cannot initiate connections

**Option 3: VPC Endpoints**

Direct connection to AWS services without internet traffic.

**Gateway Endpoints (Free):**

* S3
* DynamoDB

**Interface Endpoints (Low cost):**

* API Gateway
* Lambda
* SNS, SQS
* Systems Manager
* And many more

**Benefits:**

* No internet traffic
* Lower latency
* Better security
* Reduced NAT costs

***

### ADVANCED LEVEL

#### VPC Peering

Connect two VPCs together as if they were a single network.

**One-to-One Peering Architecture**

```
┌──────────────────────────────────────────────┐
│         AWS Region (us-east-1)              │
│                                             │
│  ┌──────────────┐      ┌──────────────┐    │
│  │ VPC-A        │      │ VPC-B        │    │
│  │ 10.0.0.0/16  │      │ 10.1.0.0/16  │    │
│  │              │      │              │    │
│  │  EC2: A1     │      │  EC2: B1     │    │
│  │  EC2: A2     │◄────►│  EC2: B2     │    │
│  │              │ Peering Connection│     │
│  │  RDS-A       │      │  RDS-B       │    │
│  └──────────────┘      └──────────────┘    │
│                                             │
└──────────────────────────────────────────────┘
```

**Route Table Configuration for Peering**

**VPC-A Route Table:**

```yaml
Destination: 10.1.0.0/16 (VPC-B traffic)
Target: pcx-12345678 (Peering Connection)
Status: Active
```

**VPC-B Route Table:**

```yaml
Destination: 10.0.0.0/16 (VPC-A traffic)
Target: pcx-12345678 (Peering Connection)
Status: Active
```

**VPC Peering Considerations**

| Aspect          | Details                                         |
| --------------- | ----------------------------------------------- |
| **CIDR Blocks** | Must not overlap                                |
| **Cost**        | Free (no data transfer costs)                   |
| **Region**      | Same or different region (cross-region peering) |
| **Account**     | Same or different AWS account                   |
| **Transitive**  | NOT transitive (A→B and B→C doesn't mean A→C)   |
| **DNS**         | Can enable DNS hostname resolution              |

**Mesh Peering (Advanced)**

Multiple VPCs all connected to each other:

```
        VPC-A
         / \
        /   \
      VPC-B-VPC-C
        \   /
         \ /
        VPC-D

Note: Requires (n²-n)/2 connections
For 4 VPCs: 6 peering connections
```

**Challenge:** This becomes difficult to manage. Solution: Use **Transit Gateway**

***

#### Transit Gateway: Multi-VPC Hub

Centralized connection point for multiple VPCs, on-premises networks, and remote users.

```
                    ┌──────────────────┐
                    │ Transit Gateway  │
                    │    (Hub)         │
                    └────────┬─────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────▼────┐         ┌────▼────┐        ┌────▼────┐
    │ VPC-A   │         │ VPC-B   │        │ VPC-C   │
    │10.0.0/16│         │10.1.0/16│        │10.2.0/16│
    └─────────┘         └─────────┘        └─────────┘
    
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
                    ┌────────▼─────────┐
                    │ VPN Connection   │
                    │ (On-Premises)    │
                    └──────────────────┘
```

**Benefits:**

* Simplified routing
* Easy to add new VPCs
* Supports multiple connection types
* Enables network segmentation via route tables

***

#### VPN & Hybrid Connectivity

**Site-to-Site VPN**

Securely connect your on-premises data center to AWS.

```
┌──────────────────────────────────────────────────┐
│          AWS Account (Cloud)                     │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │ VPC (10.0.0.0/16)                         │ │
│  │                                            │ │
│  │ ┌──────────────────────────────────────┐  │ │
│  │ │ Virtual Private Gateway               │  │ │
│  │ │ (VGW) - Customer side                │  │ │
│  │ └────────────────┬─────────────────────┘  │ │
│  └────────────────┼────────────────────────┘ │
│                   │                           │
│                   │ IPSec VPN Tunnel          │
│                   │ (Encrypted)              │
│                   │                           │
│  ┌────────────────▼────────────────────────┐ │
│  │ Customer Gateway                        │ │
│  │ (Your on-premises side)                 │ │
│  └────────────────┬────────────────────────┘ │
└───────────────────┼──────────────────────────┘
                    │
         ┌──────────▼──────────┐
         │ Your Data Center    │
         │ (On-Premises)       │
         │ 172.16.0.0/12       │
         └─────────────────────┘
```

**Connection Details:**

```yaml
Site-to-Site VPN:
  Customer Gateway IP: 203.0.113.0 (Public IP)
  VPN Endpoint IP: 54.123.45.67 (AWS Public IP)
  Tunnel 1: Active
  Tunnel 2: Backup
  Encryption: AES-128
  Authentication: Pre-shared key
  Status: Connected
```

**AWS Direct Connect**

Dedicated network connection for consistent high performance.

```
┌────────────────────────────────────────────┐
│       Your Data Center                     │
│       172.16.0.0/12                        │
└──────────────────┬─────────────────────────┘
                   │
        AWS Direct Connect
        (1 Gbps, 10 Gbps, or 100 Gbps)
        Dedicated Connection
                   │
┌──────────────────▼─────────────────────────┐
│          AWS Region                        │
│                                            │
│  ┌────────────────────────────────────┐   │
│  │ Virtual Private Gateway (VGW)      │   │
│  │ Connected to VPC (10.0.0.0/16)     │   │
│  └────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

**Comparison:**

| Feature         | VPN                 | Direct Connect              |
| --------------- | ------------------- | --------------------------- |
| **Speed**       | Up to 1.25 Gbps     | 1-100 Gbps                  |
| **Consistency** | Variable            | Consistent                  |
| **Latency**     | Variable            | Predictable                 |
| **Cost**        | Low                 | Higher                      |
| **Setup Time**  | Days                | Weeks                       |
| **Use Case**    | Remote workers, SMB | Enterprise, high-throughput |

***

#### Multi-Account VPC Strategy

**Architecture for Large Organizations**

```
                  ┌──────────────────────┐
                  │ AWS Organizations    │
                  │ Root Account         │
                  └──────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   ┌────▼─────┐         ┌────▼─────┐        ┌────▼─────┐
   │ Prod OUs  │         │ Dev OUs   │        │ Sec OUs  │
   └────┬──────┘         └────┬──────┘        └────┬─────┘
        │                     │                    │
   ┌────▼──────┐        ┌────▼──────┐        ┌────▼─────┐
   │ Prod Acct │        │ Dev Acct  │        │ Sec Acct │
   │ VPC: Prod │        │ VPC: Dev  │        │VPC: Logs │
   └───────────┘        └───────────┘        └──────────┘
```

**Network Architecture in Multi-Account Setup**

**Option 1: Hub-and-Spoke (Recommended)**

```
                    ┌──────────────────┐
                    │ Network Hub Acct │
                    │ Transit Gateway  │
                    └────────┬─────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   ┌────▼──────┐        ┌────▼──────┐       ┌────▼──────┐
   │ Prod Acct │        │ Dev Acct  │       │ Sec Acct  │
   │ (Spoke)   │        │ (Spoke)   │       │ (Spoke)   │
   └───────────┘        └───────────┘       └───────────┘
```

**Benefits:**

* Centralized routing
* Easy to add new accounts
* Consistent security policies
* Shared services

**Option 2: Flat Peering**

```
Every account connected to every other account
(Not recommended for more than 3-4 accounts)
```

**Cross-Account Resource Access**

**Scenario:** EC2 in Account A needs to access RDS in Account B

```yaml
Step 1: RDS Security Group (Account B)
  Inbound: Allow port 3306 from sg-xxx (Account A)
  
Step 2: EC2 Security Group (Account A)
  Rule: Allow outbound to 10.1.2.0/24 (Account B CIDR)
  
Step 3: Route Tables
  Both accounts must route traffic through
  Peering Connection or Transit Gateway
  
Step 4: Permissions
  Role in Account A needs permission:
  - ec2:DescribeSecurityGroups (for Account B)
  - ec2:DescribeNetworkInterfaces
```

***

#### Advanced Security Patterns

**Defense in Depth: Layered Security**

```
┌──────────────────────────────────────────────────┐
│ Layer 1: Perimeter Security                      │
│ - VPC Flow Logs                                  │
│ - Network Firewall                               │
└──────────────────────────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────────┐
│ Layer 2: Network Level Security                  │
│ - NACLs (Subnet firewall)                        │
│ - Route Tables (Traffic control)                 │
└──────────────────────────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────────┐
│ Layer 3: Instance Level Security                 │
│ - Security Groups (Per-instance firewall)        │
│ - OS-level firewalls (iptables, Windows)         │
└──────────────────────────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────────┐
│ Layer 4: Application Level Security              │
│ - API authentication                             │
│ - Encryption in transit (TLS)                    │
│ - Input validation                               │
└──────────────────────────────────────────────────┘
```

**VPC Flow Logs: Network Monitoring**

Monitor all network traffic in your VPC.

```yaml
VPC Flow Log Format:
version account-id interface-id srcaddr dstaddr
srcport dstport protocol packets bytes start end 
action log-status

Example Log Entry:
2 123456789012 eni-1a2b3c4d 10.0.1.10 10.0.2.5 
49152 3306 6 342 36028 1435969670 1435969728 
ACCEPT OK
```

**Interpretation:**

* Source: 10.0.1.10 (EC2 instance)
* Destination: 10.0.2.5 (RDS database)
* Port: 3306 (MySQL)
* Status: ACCEPT (traffic allowed)

**Endpoint Protection Patterns**

```
Scenario: Database accessed only from specific subnet

Public Subnet
  ├─ Web Tier
  └─ NAT Gateway ───────┐
                        │
                   Database Endpoint
                   (No Internet Access)
                        │
Private Subnet         │
  └─ App Tier ────────┘
     (Uses VPC Endpoint)

Result: Database never touches public internet
```

***

### REAL-WORLD SCENARIOS

#### Scenario 1: E-Commerce Platform

**Requirements:**

* High availability (multiple AZs)
* PCI-DSS compliance
* Thousands of concurrent users
* Real-time inventory updates

**Architecture:**

```
┌────────────────────────────────────────────────────────┐
│           AWS Region: us-east-1                        │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Availability Zone 1a          Availability Zone 1b   │
│  ┌─────────────────────────┐   ┌─────────────────────┐│
│  │ Public Subnet (10.0.1/24)│   │Public Subnet(10.0.11│
│  │ ┌──────────────────────┐│   │ ┌──────────────────┐││
│  │ │ Load Balancer        ││   │ │Load Balancer     │││
│  │ └──────────────────────┘│   │ └──────────────────┘││
│  │ ┌──────────────────────┐│   │ ┌──────────────────┐││
│  │ │ NAT Gateway          ││   │ │NAT Gateway       │││
│  │ └──────────────────────┘│   │ └──────────────────┘││
│  └─────────────────────────┘   └─────────────────────┘│
│                                                        │
│  ┌─────────────────────────┐   ┌─────────────────────┐│
│  │ Private Subnet (10.0.2) │   │ Private Subnet(10.0 ││
│  │ ┌──────────────────────┐│   │ ┌──────────────────┐││
│  │ │ Web Tier             ││   │ │Web Tier          │││
│  │ │ (Auto Scaling)       ││   │ │(Auto Scaling)    │││
│  │ └──────────────────────┘│   │ └──────────────────┘││
│  └─────────────────────────┘   └─────────────────────┘│
│                                                        │
│  ┌─────────────────────────┐   ┌─────────────────────┐│
│  │ Private Subnet (10.0.3) │   │ Private Subnet(10.0 ││
│  │ ┌──────────────────────┐│   │ ┌──────────────────┐││
│  │ │ Application Tier     ││   │ │Application Tier  │││
│  │ │ (Business Logic)     ││   │ │(Business Logic)  │││
│  │ └──────────────────────┘│   │ └──────────────────┘││
│  └─────────────────────────┘   └─────────────────────┘│
│                                                        │
│  ┌─────────────────────────┐   ┌─────────────────────┐│
│  │ Private Subnet (10.0.4) │   │ Private Subnet(10.0 ││
│  │ ┌──────────────────────┐│   │ ┌──────────────────┐││
│  │ │ RDS Database (Master)││   │ │RDS Database(Slave││
│  │ │ ElastiCache Redis    ││   │ │Replication       │││
│  │ └──────────────────────┘│   │ └──────────────────┘││
│  └─────────────────────────┘   └─────────────────────┘│
│                                                        │
└────────────────────────────────────────────────────────┘
                      Internet Gateway
                            ↓
                      (Internet Traffic)
```

**VPC Configuration:**

```yaml
VPC: ecommerce-prod
CIDR: 10.0.0.0/16

Subnets:
  Public (Web Tier):
    - 10.0.1.0/24 (AZ 1a)
    - 10.0.11.0/24 (AZ 1b)
    
  Private (App Tier):
    - 10.0.2.0/24 (AZ 1a)
    - 10.0.12.0/24 (AZ 1b)
    
  Private (Database):
    - 10.0.4.0/24 (AZ 1a)
    - 10.0.14.0/24 (AZ 1b)

Internet Gateway: ecom-igw
NAT Gateways: 2 (one per AZ for HA)

Route Tables:
  Public:
    0.0.0.0/0 → IGW
    
  Private:
    0.0.0.0/0 → NAT Gateway (same AZ)

Security Groups:
  ALB-SG:
    Inbound: 80, 443 from 0.0.0.0/0
    Outbound: All
    
  Web-SG:
    Inbound: 80, 443 from ALB-SG
    Outbound: All
    
  App-SG:
    Inbound: 8080 from Web-SG
    Outbound: 3306 to DB-SG
    
  DB-SG:
    Inbound: 3306 from App-SG
    Outbound: None (internal only)

Endpoints:
  S3 Gateway: For object storage
  CloudWatch: For monitoring
```

***

#### Scenario 2: Enterprise Multi-Account Setup

**Requirements:**

* Separate production and development
* Compliance and audit logging
* Network isolation
* Centralized security monitoring

**Architecture:**

```
┌────────────────────────────────────────────────────────┐
│          AWS Organizations                             │
│         Root Account (Management)                       │
│     ┌──────────────────────────┐                       │
│     │ CloudTrail Aggregation   │                       │
│     │ Organization Trail       │                       │
│     └──────────────────────────┘                       │
└────────────────────────────────────────────────────────┘
              │              │              │
     ┌────────▼──┐    ┌─────▼─────┐    ┌──▼───────┐
     │ Prod OU   │    │  Dev OU   │    │Security OU
     │           │    │           │    │
     │ Prod Acct │    │ Dev Acct  │    │ Logging &
     │ VPC: Prod │    │ VPC: Dev  │    │ Monitoring
     └────┬──────┘    └─────┬─────┘    │ Acct
          │                 │          └──────┬───┘
          │    ┌────────────┴───────────┐     │
          │    │                        │     │
     ┌────▼────▼────────────────────────▼──┐  │
     │  Network Account                    │  │
     │  ┌────────────────────────────────┐ │  │
     │  │  Transit Gateway (Central Hub) │ │  │
     │  └────────────────────────────────┘ │  │
     │  ┌────────────────────────────────┐ │  │
     │  │  VPN Connection to HQ          │ │  │
     │  └────────────────────────────────┘ │  │
     │  ┌────────────────────────────────┐ │  │
     │  │  Route 53 (DNS)                │ │  │
     │  └────────────────────────────────┘ │  │
     └────────────────────────────────────┘  │
                     │                       │
                     │◄──────────────────────┘
                     │
          VPC Flow Logs Analysis
          Security Hub
          Config Rules
```

**Account Structure:**

```yaml
Production Account:
  VPC: prod-vpc (10.0.0.0/16)
  Subnets:
    - Public: 10.0.1.0/24, 10.0.2.0/24
    - Private: 10.0.11.0/24, 10.0.12.0/24
  Attachment: TGW prod-attachment
  
Development Account:
  VPC: dev-vpc (10.1.0.0/16)
  Subnets:
    - Public: 10.1.1.0/24
    - Private: 10.1.11.0/24
  Attachment: TGW dev-attachment

Network Account:
  VPC: network-vpc (10.2.0.0/16)
  Transit Gateway: central-tgw
  VPN Gateway: hq-vpn-gw
  Route 53 Private Zone: internal.company.com

Logging Account:
  VPC: logging-vpc (10.3.0.0/16)
  S3: Centralized VPC Flow Logs
  CloudWatch: Centralized Logs
  Security Hub: Aggregated findings
```

**Data Flow Example:**

```
EC2 in Prod Account (10.0.1.5)
wants to reach RDS in Dev Account (10.1.11.5)

1. Traffic leaves EC2
2. Passes through Security Group (prod-app-sg)
3. Matches Route Table: 10.1.0.0/16 → TGW
4. Crosses Transit Gateway (prod-attachment)
5. Arrives at dev-attachment
6. Matches Dev Route Table: 10.0.0.0/16 → TGW
7. Passes through RDS Security Group (dev-db-sg)
8. Reaches RDS Instance

Return traffic follows same path in reverse.
```

***

#### Scenario 3: Hybrid Cloud Setup

**Requirements:**

* Connect on-premises data center
* Low-latency access to cloud resources
* Disaster recovery capabilities
* Maintain on-premises workloads

**Architecture:**

```
┌─────────────────────────────────────┐     ┌──────────────────────────┐
│      On-Premises Data Center        │     │      AWS Region          │
│                                     │     │                          │
│  ┌──────────────────────────────┐   │     │  ┌────────────────────┐  │
│  │ Production Applications      │   │     │  │ VPC (10.0.0.0/16)  │  │
│  │ (172.16.0.0/12)            │   │     │  │                    │  │
│  └──────┬───────────────────────┘   │     │  │ ┌──────────────┐   │  │
│         │                           │     │  │ │Public Subnet │   │  │
│  ┌──────▼───────────────────────┐   │     │  │ └──────────────┘   │  │
│  │ Corporate Network            │   │     │  │ ┌──────────────┐   │  │
│  │ (10.0.0.0/8)                │   │     │  │ │App Tier      │   │  │
│  └──────┬───────────────────────┘   │     │  │ │(Auto-scaled) │   │  │
│         │                           │     │  │ └──────────────┘   │  │
│         │                           │     │  │ ┌──────────────┐   │  │
│  ┌──────▼───────────────────────┐   │     │  │ │DB Tier       │   │  │
│  │ Direct Connect Router         │   │     │  │ │(RDS)         │   │  │
│  │ (Hardware VPN backup)         │   │     │  │ └──────────────┘   │  │
│  └──────┬────────┬───────────────┘   │     │  └────────────────────┘  │
│         │        │                   │     └──────────────────────────┘
│  AWS DX │ IPSec  │                   │              ↑        ↑
│  1Gbps  │ Backup │                   │              │        │
└─────────┼────────┼───────────────────┘              │        │
          │        │                         AWS DX + Backup VPN
          │        │                         (Active + Standby)
          │        └─────────────────────────────────┤
          └──────────────────────────────────────────┤
                                                     ↓
                                          ┌──────────────────────┐
                                          │ AWS Region (Disaster │
                                          │ Recovery)            │
                                          │ VPC (10.1.0.0/16)    │
                                          │ (Warm standby)       │
                                          └──────────────────────┘
```

**Connection Details:**

```yaml
Primary Connection: AWS Direct Connect
  Type: 1 Gbps connection
  Location: Your nearest AWS DX location
  Lead Time: 4-6 weeks
  Latency: ~5-20ms
  Cost: $0.30/hour + data transfer
  
Backup Connection: VPN
  Type: IPSec VPN
  Setup: Days
  Latency: ~50-100ms (variable)
  Cost: $0.05/hour + data transfer
  
Failover Logic:
  Primary: DX (Active)
  Secondary: VPN (Standby)
  BGP (Border Gateway Protocol) handles failover
  
On-Premises Router Configuration:
  BGP ASN: 65000
  Route advertisements: 172.16.0.0/12, 10.0.0.0/8
  
AWS Router Configuration:
  BGP ASN: 64512
  Route advertisements: 10.0.0.0/16, 10.1.0.0/16
  
Failover Time:
  DX failure detected: <1 second
  Traffic rerouted to VPN: <5 seconds
  VPN fully active: <10 seconds
```

**Disaster Recovery Setup:**

```yaml
Production Region (us-east-1):
  Tier 1: Online active production
  RTO: 0 minutes
  RPO: 0 minutes
  
DR Region (us-west-2):
  Tier 2: Warm standby (1/4 capacity)
  RTO: 5-10 minutes
  RPO: 1 hour (via daily snapshots)
  
Data Replication:
  RDS: Read replica in DR region
  S3: Cross-region replication
  EBS: Daily snapshots to DR region
  
Testing: Monthly DR drills
```

***

### Advanced Configuration Examples

#### Example 1: Zero Trust Network Access

```yaml
# Principles:
# 1. Verify every access request
# 2. Use least privilege
# 3. Microsegment the network

VPC with Microsegmentation:

Tier 1 - Public:
  CIDR: 10.0.0.0/24
  Purpose: Load balancers only
  
Tier 2 - API Gateway:
  CIDR: 10.0.1.0/24
  Purpose: API endpoints
  Security Group: Only from Tier 1 LB
  
Tier 3 - Application:
  CIDR: 10.0.2.0/24
  Purpose: Business logic
  Security Group: Only from Tier 2 API
  
Tier 4 - Database:
  CIDR: 10.0.3.0/24
  Purpose: Data storage
  Security Group: Only from Tier 3 App
  NACL: Deny all except 3306 from 10.0.2.0/24

Inter-Tier Communication:
  Tier 1 → 2: Port 443 only
  Tier 2 → 3: Port 8443 only
  Tier 3 → 4: Port 3306 only
  
Denied by Default:
  Any unlisted communication is blocked
  Requires explicit security group rules
  Requires matching NACL rules
```

#### Example 2: VPC Endpoint for S3

Save on NAT Gateway costs:

```yaml
# Without VPC Endpoint (with NAT):
EC2 → Security Group → Route Table → NAT Gateway
→ Internet Gateway → S3 Public Endpoint
Cost: ~$0.045/hour per NAT + data processing

# With VPC Endpoint (Gateway):
EC2 → Security Group → Route Table → S3 Endpoint
→ S3 (Private connection)
Cost: Free (for S3 and DynamoDB)

Configuration:

VPC Endpoint: s3-endpoint
Type: Gateway
Service: com.amazonaws.us-east-1.s3
Route Table Association:
  - private-subnet-1 route table
  - private-subnet-2 route table
  
New Route Added Automatically:
  Destination: 0.0.0.0/0 (S3 prefix list)
  Target: s3-endpoint
  
Policy (Example - Restrict to one bucket):
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalAccount": "123456789012"
        }
      }
    }
  ]
}

Result:
  - No NAT Gateway costs
  - Faster access to S3
  - Data never crosses public internet
  - Better security posture
```

***

### Key Takeaways

#### Beginner Essentials

✓ VPC provides network isolation\
✓ Subnets divide VPC into smaller networks\
✓ Internet Gateway enables internet connectivity\
✓ Security Groups control traffic at instance level\
✓ Route Tables determine traffic paths

#### Intermediate Must-Knows

✓ Multiple subnets across multiple AZs for HA\
✓ Public vs Private subnets serve different purposes\
✓ NACLs provide subnet-level stateless filtering\
✓ NAT Gateways enable outbound internet from private subnets\
✓ VPC Endpoints reduce costs and improve security

#### Advanced Patterns

✓ VPC Peering for direct VPC-to-VPC communication\
✓ Transit Gateway for multi-VPC hub-and-spoke architecture\
✓ Site-to-Site VPN for on-premises connectivity\
✓ Direct Connect for dedicated, high-performance connectivity\
✓ Multi-account strategies for enterprise scale\
✓ VPC Flow Logs for network monitoring and compliance\
✓ Layered security with defense in depth

***

### Best Practices Checklist

* \[ ] Design subnets with future growth in mind
* \[ ] Use multiple AZs for high availability
* \[ ] Implement both public and private subnets
* \[ ] Enable VPC Flow Logs for all environments
* \[ ] Use security groups as primary firewall
* \[ ] Use NACLs for DDoS mitigation
* \[ ] Implement least privilege access
* \[ ] Document your network topology
* \[ ] Regular audit of security groups
* \[ ] Plan CIDR blocks to avoid overlaps
* \[ ] Use VPC Endpoints to reduce NAT costs
* \[ ] Enable DNS hostnames in VPC
* \[ ] Implement network segmentation
* \[ ] Use CloudWatch for VPC monitoring
* \[ ] Regular disaster recovery testing

***

### Conclusion

VPC is the foundation of secure, scalable cloud architecture. From simple single-subnet setups to complex multi-account, multi-region enterprise networks, understanding VPC at all levels—beginner, intermediate, and advanced—is crucial for cloud architects and engineers.

The key is to start simple, understand the fundamentals deeply, and gradually build toward enterprise patterns as your needs grow.

**Happy networking!**&#x20;
