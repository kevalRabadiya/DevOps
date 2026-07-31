---
description: >-
  Explore the architecture of IAM systems. Understand Identity Providers,
  Directory Services, Access Control Systems, Audit & Logging, and Token
  Services that work together to secure your organization.
---

# IAM Service: Basic to Advanced Guide

### Table of Contents

1. Introduction
2. Basic Concepts
3. Core Components
4. Authentication
5. Authorization
6. IAM Policies
7. Roles and Permissions
8. Best Practices
9. Advanced Topics
10. Implementation Examples

***

### Introduction

#### What is IAM?

Identity and Access Management (IAM) is a comprehensive security framework that manages user identities and controls their access to resources within an organization. IAM systems ensure that the right people have access to the right resources at the right time, while maintaining security and compliance.

#### Why IAM is Important

* **Security**: Protects sensitive resources from unauthorized access
* **Compliance**: Helps meet regulatory requirements (GDPR, HIPAA, SOC 2)
* **Efficiency**: Streamlines user provisioning and access management
* **Auditability**: Tracks who accessed what and when
* **Cost Optimization**: Reduces unnecessary access and associated risks

#### IAM Pillars

```
┌─────────────────────────────────┐
│    Identity & Access Management │
├─────────────────────────────────┤
│ Authentication │ Authorization  │
│ (Who are you?) │ (What can you?) │
└─────────────────────────────────┘
```

***

### Basic Concepts

#### Identity

An identity represents a person, application, or service that needs access to resources.

**Types of Identities:**

* **Users**: Individual people within an organization
* **Service Accounts**: Non-human accounts for applications/services
* **Federated Identities**: Users from external identity providers
* **Machine Identities**: Devices or APIs requiring access

#### Authentication

The process of verifying that someone is who they claim to be.

**Authentication Methods:**

* **Password-based**: Username and password combination
* **Multi-Factor Authentication (MFA)**: Multiple verification methods
* **SSO (Single Sign-On)**: One login for multiple applications
* **Biometric**: Fingerprint, facial recognition
* **Certificate-based**: Digital certificates for verification

#### Authorization

The process of determining what authenticated users/services are allowed to do.

**Authorization Models:**

* **Role-Based Access Control (RBAC)**: Access based on roles
* **Attribute-Based Access Control (ABAC)**: Access based on attributes
* **Scope-Based**: Limited access to specific resources

#### Sessions

A session represents an authenticated user's active connection.

```
Session Lifecycle:
1. Authentication → User logs in
2. Session Created → Token issued
3. Resource Access → User performs actions
4. Session Maintenance → Token refreshed if needed
5. Logout → Session terminated
```

***

### Core Components

#### 1. Identity Provider (IdP)

Manages user identities and authentication. Examples:

* Azure AD
* Okta
* Auth0
* AWS IAM
* Google Cloud Identity

#### 2. Directory Services

Centralized repository of user information.

**Features:**

* User attributes (name, email, department)
* Group memberships
* Contact information
* Password policies

#### 3. Access Control System

Enforces authorization rules based on policies.

```
Access Control Flow:
┌─────────────┐
│   Resource  │
└──────┬──────┘
       │
       ▼
┌──────────────────────┐
│ Policy Evaluation    │
├──────────────────────┤
│ Check User/Role      │
│ Check Resource       │
│ Check Conditions     │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│ Allow or Deny        │
└──────────────────────┘
```

#### 4. Audit & Logging

Tracks all access and changes for compliance.

**What to Log:**

* Login/logout events
* Permission changes
* Failed access attempts
* Administrative actions
* Resource access

#### 5. Token Service

Issues and manages authentication/authorization tokens.

**Token Types:**

* **Access Tokens**: Short-lived, used for API calls
* **Refresh Tokens**: Long-lived, used to obtain new access tokens
* **ID Tokens**: Contains user identity information (OpenID Connect)

***

### Authentication

#### Authentication Factors

```
┌──────────────────────────────────────┐
│  Something You Know (Knowledge)      │
│  - Password                          │
│  - PIN                               │
│  - Security Question                 │
├──────────────────────────────────────┤
│  Something You Have (Possession)     │
│  - Hardware token                    │
│  - Mobile phone (SMS/App)            │
│  - Certificate                       │
├──────────────────────────────────────┤
│  Something You Are (Inherence)       │
│  - Fingerprint                       │
│  - Facial Recognition                │
│  - Iris Scan                         │
├──────────────────────────────────────┤
│  Where You Are (Location)            │
│  - IP Address                        │
│  - Geographic Location               │
└──────────────────────────────────────┘
```

#### Authentication Methods

**Single Factor Authentication (SFA)**

Simple but less secure. Only one verification method.

```bash
Username: john.doe@company.com
Password: SecurePassword123
# Authentication complete
```

**Multi-Factor Authentication (MFA)**

Multiple verification methods provide better security.

```
Step 1: Enter Username & Password
        ▼
Step 2: Receive OTP via Email/SMS
        ▼
Step 3: Enter OTP
        ▼
Step 4: Access Granted
```

**Passwordless Authentication**

Modern approach eliminating passwords.

**Passwordless Methods:**

* Windows Hello (facial recognition/PIN)
* FIDO2 Security Keys
* Mobile app notifications
* Biometric authentication

#### Token-Based Authentication

Modern applications use tokens instead of sessions.

```
┌────────────────────────────────────────┐
│ User provides credentials              │
└────────────────────────────────────────┘
              ▼
┌────────────────────────────────────────┐
│ Server validates and creates JWT       │
└────────────────────────────────────────┘
              ▼
┌────────────────────────────────────────┐
│ Client stores token (localStorage)     │
└────────────────────────────────────────┘
              ▼
┌────────────────────────────────────────┐
│ Each request includes token in header  │
│ Authorization: Bearer <token>          │
└────────────────────────────────────────┘
```

**JWT (JSON Web Token) Structure**

```
Header.Payload.Signature

Header:
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload:
{
  "sub": "user123",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516242622
}

Signature:
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

***

### Authorization

#### Authorization Models

**1. Role-Based Access Control (RBAC)**

Simplest model - permissions assigned to roles.

```
User → Role → Permissions → Resources

Example:
John (User)
  └─ Manager (Role)
      ├─ Read Reports
      ├─ Create Reports
      ├─ Approve Requests
      └─ Manage Team
```

**Advantages:**

* Simple to understand
* Easy to manage
* Scalable for small-medium organizations

**Disadvantages:**

* Limited granularity
* Role explosion problem
* Difficult to manage complex scenarios

**2. Attribute-Based Access Control (ABAC)**

Permissions based on attributes (context).

```
Access Decision:
┌─────────────────────────────────────┐
│ User Attributes                     │
│ - Department: Engineering           │
│ - Level: Senior                     │
│ - Location: New York                │
├─────────────────────────────────────┤
│ Resource Attributes                 │
│ - Type: Financial Document          │
│ - Classification: Confidential       │
│ - Owner: Finance Team               │
├─────────────────────────────────────┤
│ Environment Attributes              │
│ - Time: Business Hours              │
│ - IP Range: Corporate Network       │
│ - Date: Weekday                     │
├─────────────────────────────────────┤
│ Policy: User.Department = "Finance" │
│         AND Resource.Type = "Report"│
│         AND Env.Time = "BusinessHrs"│
└─────────────────────────────────────┘
```

**Advantages:**

* Fine-grained control
* Flexible policies
* Handles complex scenarios

**Disadvantages:**

* Complex to implement
* Performance overhead
* Steep learning curve

**3. Scope-Based Access Control**

Permissions limited to specific resources or scopes (common in OAuth 2.0).

```
Example OAuth 2.0 Scopes:
- read:profile
- write:profile
- read:reports
- delete:documents
- admin:all

Request:
GET /api/profile
Authorization: Bearer <token>
Token Scopes: [read:profile, write:profile]

Result: ALLOWED (read:profile is in token scopes)
```

***

### IAM Policies

#### What are Policies?

Policies are rules that define who can do what on which resources under what conditions.

#### Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadDatabase",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/john"
      },
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:123456789012:table/Users",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "us-east-1"
        }
      }
    }
  ]
}
```

#### Policy Components

**1. Effect**

Determines whether the policy allows or denies access.

```
"Effect": "Allow"    # Grants permission
"Effect": "Deny"     # Explicitly denies access
```

**2. Principal**

Specifies who the policy applies to.

```json
"Principal": {
  "AWS": "arn:aws:iam::123456789012:user/john",
  "Service": "lambda.amazonaws.com",
  "Federated": "arn:aws:iam::123456789012:saml-provider/ExampleProvider"
}
```

**3. Action**

Specifies what operations are allowed/denied.

```json
"Action": [
  "s3:GetObject",
  "s3:PutObject",
  "s3:*"  // Wildcard - all S3 actions
]
```

**4. Resource**

Specifies which resources the policy applies to.

```json
"Resource": [
  "arn:aws:s3:::my-bucket",
  "arn:aws:s3:::my-bucket/*",
  "*"  // All resources
]
```

**5. Condition**

Optional - specifies when the policy applies.

```json
"Condition": {
  "StringEquals": {
    "aws:username": "john"
  },
  "IpAddress": {
    "aws:SourceIp": ["192.168.1.0/24"]
  },
  "DateGreaterThan": {
    "aws:CurrentTime": "2024-01-01T00:00:00Z"
  },
  "StringLike": {
    "aws:userid": "*:session-name"
  }
}
```

#### Policy Types

**1. Identity-Based Policies**

Attached to users, groups, or roles.

```
┌─────────────┐
│ User/Group  │◄──── Policy
│    /Role    │
└─────────────┘
```

**2. Resource-Based Policies**

Attached to resources.

```
┌──────────────┐
│ S3 Bucket    │
│              │◄──── Policy
│ (Resource)   │
└──────────────┘
```

**3. Boundary Policies**

Define maximum permissions (limits, doesn't grant).

**4. Session Policies**

Temporary permissions for federated users.

#### Policy Evaluation Logic

```
1. Explicit Deny Present?
   ├─ YES → DENY (Final)
   └─ NO → Continue

2. Explicit Allow Present?
   ├─ YES → ALLOW (Default Allow)
   └─ NO → DENY (Default Deny)

Note: One explicit deny overrides all allows
```

***

### Roles and Permissions

#### What are Roles?

Roles are collections of permissions that can be assigned to users, groups, or applications.

#### Role vs Group

| Aspect     | Role               | Group             |
| ---------- | ------------------ | ----------------- |
| Purpose    | Define permissions | Organize users    |
| Assignment | To users/services  | Users only        |
| Scope      | Application/system | Directory service |
| Temporary  | Yes (sessions)     | No (persistent)   |
| Best For   | Permissions        | User organization |

#### Common Role Hierarchies

```
Admin
├─ Super Admin (All permissions)
└─ Limited Admin (Specific areas)

Manager
├─ Department Manager
└─ Team Lead

User
├─ Power User (Extended permissions)
├─ Standard User (Basic permissions)
└─ Guest (Minimal permissions)
```

#### Permission Granularity

```
Course Level:           Granular Level:
┌──────────────────┐   ┌─────────────────────────┐
│ read             │   │ read:profile            │
│ write            │   │ read:reports            │
│ delete           │   │ write:profile           │
│ admin            │   │ write:reports           │
└──────────────────┘   │ delete:profile          │
                       │ delete:reports          │
                       │ admin:all               │
                       │ admin:users             │
                       └─────────────────────────┘
```

#### Dynamic Role Assignment

Roles assigned based on conditions/attributes.

```
IF user.department = "Finance" 
   AND user.level = "Senior"
   AND request.time = "business_hours"
THEN assign role "FinanceAnalyst"

IF user.project = "ProjectX"
   AND NOT user.terminated
THEN assign role "ProjectX_Developer"
```

***

### Best Practices

#### 1. Principle of Least Privilege

Grant minimum permissions required for job function.

```
❌ Wrong:
  role: Admin (Full access to everything)
  
✅ Correct:
  role: ReportReader (Read-only access to reports)
       + DatabaseUser (Query access to specific tables)
```

#### 2. Separation of Duties

Critical operations require multiple people/approvals.

```
Database Access:
- Developer: Read/Query only
- DBA: Modify/Create only
- Auditor: Read-only (cannot modify)
- Approval: Manager must approve changes
```

#### 3. Regular Access Reviews

Periodically verify user access is still needed.

```
Quarterly Review Process:
1. Generate access report per user
2. Manager reviews each user's access
3. Remove unnecessary permissions
4. Document justification
5. Audit trail maintained
```

#### 4. Strong Authentication

```
✅ Use Multi-Factor Authentication (MFA)
✅ Enforce strong password policies
✅ Implement passwordless authentication
✅ Use SSO for multiple applications
✅ Regularly rotate credentials
```

#### 5. Secure Credential Management

```
✅ Store secrets in vault (HashiCorp, AWS Secrets Manager)
✅ Never hardcode credentials
✅ Rotate API keys regularly
✅ Use short-lived tokens
✅ Implement key rotation policies
```

#### 6. Implement Audit Logging

```
Log and Monitor:
✓ All authentication attempts (success/failure)
✓ Permission changes
✓ Sensitive data access
✓ Administrative actions
✓ Unusual patterns (multiple failed logins)
```

#### 7. Just-In-Time (JIT) Access

Grant temporary elevated permissions when needed.

```
User requests access:
  ↓
Approval workflow
  ↓
Temporary credentials issued (time-limited)
  ↓
User performs task
  ↓
Credentials automatically revoked
```

#### 8. Immutable Audit Trails

Prevent modification of access logs.

```
Audit Log:
- Append-only
- Encrypted storage
- Separate storage from live systems
- Retention policy (e.g., 7 years)
- Regular integrity verification
```

***

### Advanced Topics

#### 1. Federated Identity Management

Allow external users to access resources using their own credentials.

```
External User → External IdP → Federation → Internal Application
                (Okta)        (SAML/OAuth)

Flow:
1. User tries to access internal app
2. App redirects to external IdP
3. User authenticates with external IdP
4. IdP returns assertion/token
5. App verifies and creates session
```

**SAML (Security Assertion Markup Language)**

```xml
<!-- SAML Assertion -->
<saml:Assertion>
  <saml:Subject>
    <saml:NameID>john.doe@example.com</saml:NameID>
  </saml:Subject>
  <saml:AuthnStatement>
    <saml:AuthnContext>
      <saml:AuthnContextClassRef>
        urn:oasis:names:tc:SAML:2.0:ac:classes:Password
      </saml:AuthnContextClassRef>
    </saml:AuthnContext>
  </saml:AuthnStatement>
  <saml:AttributeStatement>
    <saml:Attribute Name="Department" Value="Engineering"/>
    <saml:Attribute Name="Role" Value="Manager"/>
  </saml:AttributeStatement>
</saml:Assertion>
```

**OAuth 2.0 / OpenID Connect**

```
OAuth 2.0 (Authorization):
┌──────────┐                    ┌──────────┐
│  Client  │ ──── auth code ───→│   IdP    │
│          │◄─── access token ──│          │
└──────────┘                    └──────────┘

OpenID Connect (Authentication + Authorization):
Adds ID Token containing user information to OAuth 2.0
```

#### 2. Privileged Access Management (PAM)

Controls and monitors administrative access.

```
PAM Architecture:
┌─────────────────────────────────────┐
│ PAM Solution                        │
├─────────────────────────────────────┤
│ ✓ Password Vault (encrypted storage)│
│ ✓ Session Recording (video audit)   │
│ ✓ Approval Workflows                │
│ ✓ Temporary Elevation               │
│ ✓ Activity Monitoring               │
└─────────────────────────────────────┘

User Request → Approval → Temporary Credentials → Task Execution → Audit
```

#### 3. Zero Trust Architecture

Never trust, always verify - regardless of location.

```
Traditional Perimeter Security:
┌─────────────────────────────────┐
│ Firewall                        │
│ ┌─────────────────────────────┐ │
│ │ Trust Everything Inside     │ │
│ └─────────────────────────────┘ │
└─────────────────────────────────┘

Zero Trust Model:
┌────────────────────────────┐
│ Every Request              │
│ ✓ Authenticate            │
│ ✓ Authorize               │
│ ✓ Encrypt                 │
│ ✓ Validate                │
└────────────────────────────┘
```

#### 4. Risk-Based Adaptive Authentication

Adjust authentication requirements based on risk score.

```
Risk Scoring:
┌──────────────────────────┐
│ User Factors:            │
│ - IP location changed    │
│ - New device             │
│ - Time anomaly           │
│ - Unusual activity       │
│ - Failed logins          │
└──────────────────────────┘
        ↓
┌──────────────────────────┐
│ Risk Score Calculation   │
│ Low (1-33%)              │
│ Medium (34-66%)          │
│ High (67-100%)           │
└──────────────────────────┘
        ↓
┌──────────────────────────┐
│ Authentication:          │
│ Low:    Password only    │
│ Medium: MFA required     │
│ High:   Enhanced MFA +   │
│         Manual approval  │
└──────────────────────────┘
```

#### 5. Attribute-Based Access Control (ABAC) Advanced

```
Complex ABAC Policy:
{
  "Version": "1.0",
  "Rules": [
    {
      "Name": "FinanceReportAccess",
      "Effect": "Allow",
      "Conditions": {
        "user": {
          "department": ["Finance", "Executive"],
          "clearance_level": {"gte": 3}
        },
        "resource": {
          "type": "report",
          "classification": ["Confidential", "Internal"]
        },
        "environment": {
          "time_of_day": "business_hours",
          "location": "office",
          "network": "corporate"
        },
        "action": ["read", "export"]
      }
    }
  ]
}
```

#### 6. Identity Orchestration

Automate identity processes across systems.

```
Onboarding Workflow:
┌──────────────┐
│ New Employee │
└──────┬───────┘
       │ Trigger
       ▼
┌───────────────────┐
│ HR System         │
│ (Name, Dept, Mgr) │
└────────┬──────────┘
         │
         ▼
┌───────────────────────────┐
│ Identity Orchestration    │
│ - Create user in AD       │
│ - Create email            │
│ - Provision access        │
│ - Configure groups        │
│ - Set permissions         │
└────────┬──────────────────┘
         │
         ▼
┌──────────────────────┐
│ Employee Ready       │
│ (All systems set up) │
└──────────────────────┘
```

#### 7. Blockchain-Based Identity

Decentralized identity management.

```
Blockchain Identity:
┌─────────────────────────────────┐
│ User Maintains Control          │
│ - Own private key               │
│ - Own credentials               │
│ - Selective disclosure          │
└─────────────────────────────────┘
         ↓
┌─────────────────────────────────┐
│ Verifiable Credentials          │
│ - Issued by trusted entities    │
│ - Cryptographically signed      │
│ - Self-sovereign               │
└─────────────────────────────────┘
         ↓
┌─────────────────────────────────┐
│ On Blockchain                   │
│ - Immutable record              │
│ - Decentralized verification    │
│ - User-controlled sharing       │
└─────────────────────────────────┘
```

***

### Implementation Examples

#### Example 1: Simple RBAC Implementation

```python
# Simple RBAC System
class User:
    def __init__(self, username, role):
        self.username = username
        self.role = role

class Role:
    def __init__(self, name, permissions):
        self.name = name
        self.permissions = permissions  # List of permission strings

class Resource:
    def __init__(self, resource_id, required_permission):
        self.resource_id = resource_id
        self.required_permission = required_permission

class AccessControl:
    def __init__(self):
        self.roles = {}
        self.users = {}
        
    def create_role(self, name, permissions):
        self.roles[name] = Role(name, permissions)
    
    def create_user(self, username, role):
        self.users[username] = User(username, role)
    
    def check_access(self, username, resource):
        user = self.users.get(username)
        if not user:
            return False
        
        role = self.roles.get(user.role)
        if not role:
            return False
        
        return resource.required_permission in role.permissions

# Usage
ac = AccessControl()
ac.create_role("Admin", ["read", "write", "delete"])
ac.create_role("User", ["read"])
ac.create_user("john", "Admin")
ac.create_user("jane", "User")

admin_resource = Resource("report", "write")
print(ac.check_access("john", admin_resource))  # True
print(ac.check_access("jane", admin_resource))  # False
```

#### Example 2: JWT-Based Authentication

```python
import jwt
from datetime import datetime, timedelta

class JWTAuth:
    def __init__(self, secret_key):
        self.secret_key = secret_key
    
    def create_token(self, user_id, role, expires_in_hours=1):
        payload = {
            'user_id': user_id,
            'role': role,
            'iat': datetime.utcnow(),
            'exp': datetime.utcnow() + timedelta(hours=expires_in_hours)
        }
        token = jwt.encode(payload, self.secret_key, algorithm='HS256')
        return token
    
    def verify_token(self, token):
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=['HS256'])
            return payload
        except jwt.ExpiredSignatureError:
            return None  # Token expired
        except jwt.InvalidTokenError:
            return None  # Invalid token

# Usage
auth = JWTAuth('your-secret-key')
token = auth.create_token('user123', 'admin')
print(f"Token: {token}")

payload = auth.verify_token(token)
if payload:
    print(f"User: {payload['user_id']}, Role: {payload['role']}")
else:
    print("Token invalid or expired")
```

#### Example 3: OAuth 2.0 Flow

```python
# Simplified OAuth 2.0 Authorization Code Flow
import requests
from urllib.parse import urlencode, parse_qs
from urllib.request import urlopen

class OAuth2Client:
    def __init__(self, client_id, client_secret, auth_url, token_url, redirect_uri):
        self.client_id = client_id
        self.client_secret = client_secret
        self.auth_url = auth_url
        self.token_url = token_url
        self.redirect_uri = redirect_uri
    
    def get_authorization_url(self, scopes):
        params = {
            'client_id': self.client_id,
            'response_type': 'code',
            'scope': ' '.join(scopes),
            'redirect_uri': self.redirect_uri
        }
        return f"{self.auth_url}?{urlencode(params)}"
    
    def exchange_code_for_token(self, code):
        data = {
            'grant_type': 'authorization_code',
            'code': code,
            'client_id': self.client_id,
            'client_secret': self.client_secret,
            'redirect_uri': self.redirect_uri
        }
        response = requests.post(self.token_url, data=data)
        return response.json()  # Contains access_token, refresh_token, etc.

# Usage
oauth = OAuth2Client(
    client_id='your-client-id',
    client_secret='your-client-secret',
    auth_url='https://auth.example.com/authorize',
    token_url='https://auth.example.com/token',
    redirect_uri='https://yourapp.com/callback'
)

# Step 1: Redirect user to authorization URL
auth_url = oauth.get_authorization_url(['read:profile', 'write:profile'])
print(f"Redirect user to: {auth_url}")

# Step 2: After user approves, receive auth code
# (In real app, this comes from redirect)
auth_code = 'received-from-redirect'

# Step 3: Exchange code for access token
token_response = oauth.exchange_code_for_token(auth_code)
print(f"Access Token: {token_response['access_token']}")
```

#### Example 4: ABAC Policy Engine

```python
class ABACPolicyEngine:
    def __init__(self):
        self.policies = []
    
    def add_policy(self, name, conditions, effect="Allow"):
        self.policies.append({
            'name': name,
            'conditions': conditions,
            'effect': effect
        })
    
    def evaluate_request(self, user_attrs, resource_attrs, environment_attrs):
        # Check explicit denies first
        for policy in self.policies:
            if policy['effect'] == "Deny":
                if self._match_conditions(user_attrs, resource_attrs, 
                                         environment_attrs, policy['conditions']):
                    return False
        
        # Check for allows
        for policy in self.policies:
            if policy['effect'] == "Allow":
                if self._match_conditions(user_attrs, resource_attrs, 
                                         environment_attrs, policy['conditions']):
                    return True
        
        # Default deny
        return False
    
    def _match_conditions(self, user_attrs, resource_attrs, 
                         environment_attrs, conditions):
        for key, value in conditions.items():
            if key.startswith('user.'):
                attr = key.replace('user.', '')
                if user_attrs.get(attr) not in value:
                    return False
            elif key.startswith('resource.'):
                attr = key.replace('resource.', '')
                if resource_attrs.get(attr) not in value:
                    return False
            elif key.startswith('env.'):
                attr = key.replace('env.', '')
                if environment_attrs.get(attr) not in value:
                    return False
        return True

# Usage
engine = ABACPolicyEngine()

# Add policy: Finance team can read confidential reports during business hours
engine.add_policy(
    'FinanceReportAccess',
    {
        'user.department': ['Finance'],
        'user.clearance': ['3', '4', '5'],
        'resource.type': ['report'],
        'resource.classification': ['Confidential'],
        'env.time': ['business_hours'],
    },
    effect="Allow"
)

# Evaluate access
user = {'department': 'Finance', 'clearance': '3'}
resource = {'type': 'report', 'classification': 'Confidential'}
environment = {'time': 'business_hours'}

access_granted = engine.evaluate_request(user, resource, environment)
print(f"Access: {access_granted}")  # True
```

#### Example 5: MFA Implementation

```python
import pyotp
import qrcode
from io import BytesIO

class MFAService:
    def __init__(self):
        self.user_secrets = {}
    
    def generate_secret(self, user_id):
        """Generate a new TOTP secret for user"""
        secret = pyotp.random_base32()
        self.user_secrets[user_id] = secret
        return secret
    
    def get_qr_code(self, user_id, user_email):
        """Generate QR code for authenticator app"""
        secret = self.user_secrets.get(user_id)
        if not secret:
            secret = self.generate_secret(user_id)
        
        totp = pyotp.TOTP(secret)
        uri = totp.provisioning_uri(
            name=user_email,
            issuer_name='YourApp'
        )
        
        qr = qrcode.QRCode()
        qr.add_data(uri)
        qr.make()
        
        return qr
    
    def verify_totp(self, user_id, token):
        """Verify TOTP token"""
        secret = self.user_secrets.get(user_id)
        if not secret:
            return False
        
        totp = pyotp.TOTP(secret)
        return totp.verify(token)
    
    def verify_mfa(self, user_id, totp_token, backup_code=None):
        """Complete MFA verification"""
        # Check TOTP
        if self.verify_totp(user_id, totp_token):
            return True
        
        # Check backup code (simplified)
        if backup_code and backup_code in self._get_backup_codes(user_id):
            self._revoke_backup_code(user_id, backup_code)
            return True
        
        return False
    
    def _get_backup_codes(self, user_id):
        # Retrieve stored backup codes
        pass
    
    def _revoke_backup_code(self, user_id, code):
        # Remove used backup code
        pass

# Usage
mfa = MFAService()
user_id = 'user123'

# 1. Generate secret
secret = mfa.generate_secret(user_id)
print(f"Secret: {secret}")

# 2. Get QR code for authenticator app
qr = mfa.get_qr_code(user_id, 'user@example.com')
qr.show()

# 3. User scans QR code and provides TOTP token
totp_token = '123456'  # From authenticator app
if mfa.verify_totp(user_id, totp_token):
    print("MFA verified!")
else:
    print("MFA failed!")
```

***

### Conclusion

IAM is a critical component of modern security infrastructure. Starting with basic concepts of authentication and authorization, organizations can build sophisticated identity systems using advanced techniques like ABAC, PAM, and zero trust architecture.

#### Key Takeaways

1. **Authentication** verifies identity; **Authorization** controls access
2. **RBAC** is simple but **ABAC** provides finer control
3. **Least Privilege** and **Separation of Duties** are foundational principles
4. **MFA** significantly improves security posture
5. **Audit Logging** is essential for compliance and forensics
6. **Zero Trust** is the modern security paradigm
7. Implement **Just-In-Time** access for privileged operations



***
