---
description: >-
  Domain Name System (DNS) is a hierarchical, distributed system that translates
  human-readable domain names into IP addresses.
---

# DNS Complete Guide: Fundamentals to Advanced

***

### Table of Contents

1. DNS Fundamentals
2. DNS Architecture
3. DNS Records
4. DNS Resolution Process
5. DNS Server Configuration
6. Advanced Concepts
7. Troubleshooting
8. Best Practices

***

### DNS Fundamentals

#### What is DNS?

DNS translates domain names into IP addresses.

```
www.example.com  →  93.184.216.34  →  Browser connects
```

**Key Characteristics:**

* **Hierarchical**: Root → TLD → Authoritative servers
* **Distributed**: No single point of failure
* **Recursive**: Queries forwarded through chain
* **Cached**: Results stored at multiple levels
* **Port**: UDP 53 (queries), TCP 53 (zone transfers)

#### Why DNS Matters

Without DNS, you'd need to remember IP addresses. DNS makes the internet user-friendly and scalable.

***

### DNS Architecture

#### Hierarchy Structure

```
                 ROOT SERVERS
                 (13 clusters)
                      │
        ┌─────────────┼─────────────┐
        │             │             │
      .COM           .ORG          .NET
    (TLD Servers)  (TLD Servers)  (TLD Servers)
        │             │             │
    Authoritative  Authoritative  Authoritative
    Nameservers    Nameservers    Nameservers
        │             │             │
    Records      Records      Records
   (A, MX, etc) (A, MX, etc) (A, MX, etc)
```

#### Components

| Component | Role        | Example                 |
| --------- | ----------- | ----------------------- |
| Root NS   | Entry point | Points to TLD           |
| TLD NS    | Zone finder | Points to authoritative |
| Auth NS   | Data holder | Contains DNS records    |
| Resolver  | Questioner  | Client-side or ISP      |

***

### DNS Records

#### Record Types Quick Reference

| Record    | Purpose          | Example                        | TTL   |
| --------- | ---------------- | ------------------------------ | ----- |
| **A**     | IPv4 address     | example.com → 93.184.216.34    | 3600  |
| **AAAA**  | IPv6 address     | example.com → 2001:db8::1      | 3600  |
| **CNAME** | Alias            | www → example.com              | 3600  |
| **MX**    | Mail server      | example.com → mail.example.com | 3600  |
| **TXT**   | Text/Auth        | SPF, DKIM, DMARC, verification | 3600  |
| **NS**    | Nameserver       | Zone delegation                | 86400 |
| **SOA**   | Zone metadata    | Serial, refresh, retry, expire | 3600  |
| **PTR**   | Reverse DNS      | 93.184.216.34 → example.com    | 3600  |
| **SRV**   | Service location | \_sip.\_tcp → server:port      | 3600  |
| **CAA**   | CA authorization | Cert issuing permissions       | 3600  |

#### Detailed Explanations

**A Record - Maps domain to IPv4**

```
example.com    A    93.184.216.34
www.example.com A    93.184.216.34
api.example.com A    93.184.216.35
```

Most fundamental record type. Every domain needs at least one A record.

**AAAA Record - Maps domain to IPv6**

```
example.com    AAAA    2001:db8::1
```

For IPv6-enabled networks and modern infrastructure.

**CNAME Record - Creates domain alias**

```
www.example.com    CNAME    example.com
blog.example.com   CNAME    www.example.com
cdn.example.com    CNAME    d1234.cloudfront.net
```

⚠️ Rules: Cannot point to IP, cannot coexist with other records at same name

**MX Record - Routes emails to mail servers**

```
example.com    MX    10    mail.example.com
example.com    MX    20    mail2.example.com
```

Priority (10, 20, etc): Lower number = higher priority

**TXT Record - Text information storage**

```
example.com    TXT    "v=spf1 include:_spf.google.com ~all"
_dmarc.example.com TXT    "v=DMARC1; p=reject; rua=mailto:admin@example.com"
```

Used for: SPF (email authentication), DKIM (email signing), DMARC (policy), site verification

**NS Record - Specifies nameservers**

```
example.com    NS    ns1.example.com
example.com    NS    ns2.example.com
```

Used for: Zone delegation, subdomain delegation

**SOA Record - Zone authority and replication**

```
example.com SOA ns1.example.com admin@example.com (
    2024010101  ; Serial (must increment on changes)
    3600        ; Refresh (secondary checks primary every X seconds)
    1800        ; Retry (wait X seconds before retry)
    604800      ; Expire (discard data after X seconds)
    86400 )     ; Minimum TTL
```

**PTR Record - Reverse DNS lookup**

```
34.216.184.93.in-addr.arpa    PTR    mail.example.com
```

IP address maps to domain name. Used for email validation.

**SRV Record - Service with port**

```
_sip._tcp.example.com    SRV    10 60 5060 sipserver.example.com
_ldap._tcp.example.com   SRV    10 60 389 ldapserver.example.com
```

Format: \_service.\_protocol.name

**CAA Record - Certificate Authority control**

```
example.com    CAA    0 issue "letsencrypt.org"
example.com    CAA    0 issuewild ";;"
```

Restricts which CAs can issue certificates for domain.

#### Complete Zone File

```
$ORIGIN example.com.
$TTL 3600

; Authority
@    IN    SOA    ns1.example.com. admin.example.com. (
                   2024010101  3600  1800  604800  86400)
@    IN    NS     ns1.example.com.
@    IN    NS     ns2.example.com.

; Web
@          IN    A      93.184.216.34
www        IN    A      93.184.216.34
api        IN    A      93.184.216.35

; Mail
@          IN    MX     10  mail.example.com.
mail       IN    A      93.184.216.36

; Aliases
blog       IN    CNAME  www.example.com.
shop       IN    CNAME  www.example.com.

; Authentication
@          IN    TXT    "v=spf1 ip4:93.184.216.36 -all"
_dmarc     IN    TXT    "v=DMARC1; p=reject;"

; Glue records (NS IP addresses)
ns1        IN    A      192.0.2.1
ns2        IN    A      192.0.2.2
```

***

### DNS Resolution Process

#### Step-by-Step Query Flow

```
USER BROWSER
    │
    ├─ Type: example.com
    │
    ├─→ Check Browser Cache (found? → stop)
    │
    ├─→ Check OS Cache (found? → stop)
    │
    ├─→ Check ISP Cache (found? → stop)
    │
    ├─→ Query Recursive Resolver (8.8.8.8, 1.1.1.1)
    │
    ├─→ Resolver queries ROOT
    │   Root: "Ask TLD for .com"
    │
    ├─→ Resolver queries TLD
    │   TLD: "Ask this authoritative server"
    │
    ├─→ Resolver queries AUTHORITATIVE NS
    │   Auth: "A record is 93.184.216.34"
    │
    ├─ Response flows back:
    │  Auth → TLD → Root → Resolver → OS → Browser
    │  (Each level caches the result)
    │
    └─→ Browser connects to 93.184.216.34
        Website loads!
```

#### Performance Timeline

| Scenario       | Time      | Description                        |
| -------------- | --------- | ---------------------------------- |
| Cache Hit      | 1-50ms    | IP found locally                   |
| Resolver Cache | 50-150ms  | Found in ISP resolver              |
| Full Lookup    | 150-500ms | Full hierarchy traversal           |
| Slow           | 500ms+    | Network issues or misconfiguration |

#### Query Types

**Recursive Query** (What your browser does)

```
Browser asks: "Give me the IP for example.com"
Resolver must find answer or return error
```

**Iterative Query** (What resolvers do between servers)

```
Resolver asks Root: "Do you know example.com?"
Root says: "No, ask this TLD server"
Resolver asks TLD: "Do you know example.com?"
TLD says: "No, ask this authoritative server"
```

***

### DNS Server Configuration

#### Popular DNS Servers

**BIND (Most Common)**

```bash
sudo apt-get install bind9 bind9-utils
# Config: /etc/bind/named.conf
# Zones: /etc/bind/zones/
```

**PowerDNS (Modern)**

```bash
sudo apt-get install pdns pdns-backend-mysql
# Config: /etc/powerdns/pdns.conf
```

**Unbound (Recursive)**

```bash
sudo apt-get install unbound
# Config: /etc/unbound/unbound.conf
```

#### Primary Nameserver Config (BIND)

```
# /etc/bind/named.conf

zone "example.com" {
    type master;
    file "/etc/bind/zones/db.example.com";
    allow-transfer { 192.0.2.2; };  # Secondary NS IP
    notify yes;  # Notify secondary on changes
};
```

#### Secondary Nameserver Config (BIND)

```
# /etc/bind/named.conf

zone "example.com" {
    type slave;
    masters { 192.0.2.1; };  # Primary NS IP
    file "/etc/bind/zones/db.example.com";
};
```

#### Basic BIND Options

```
options {
    directory "/var/cache/bind";
    
    # Queries
    recursion yes;
    allow-recursion { 127.0.0.1; 192.168.1.0/24; };
    
    # Security
    allow-transfer { 192.0.2.2; };  # Only secondary
    query-response-rate-limit { all category 100; };
    
    # Performance
    max-cache-size 1g;
};
```

#### Common Commands

```bash
# Reload zone
rndc reload example.com

# Check zone validity
named-checkzone example.com /etc/bind/zones/db.example.com

# Verify configuration
named-checkconf /etc/bind/named.conf

# View statistics
rndc stats

# Flush cache
rndc flush
```

#### Zone Transfer Process

```
Primary NS                Secondary NS
    │                          │
    ├──→ SOA Query (check serial)
    │                          │
    ├──→ AXFR Request (if serial newer)
    │                          │
    ├──→ All Records           │
    │                          │
    ├──→ SOA (complete)        │
    │                          │
    │← ACK                     │
    
Secondary now has copy of all records
```

***

### Advanced Concepts

#### DNS Load Balancing

**Round Robin**

```
example.com    A    192.0.2.1
example.com    A    192.0.2.2
example.com    A    192.0.2.3

Response 1: 192.0.2.1
Response 2: 192.0.2.2
Response 3: 192.0.2.3
Response 4: 192.0.2.1 (cycle repeats)
```

**Weighted Distribution**

```
example.com    A    192.0.2.1  (70%)
example.com    A    192.0.2.2  (20%)
example.com    A    192.0.2.3  (10%)
```

**Geographic Routing**

```
User in US   → 1.2.3.4 (US Server)
User in EU   → 5.6.7.8 (EU Server)
User in Asia → 9.10.11.12 (Asia Server)
```

#### Subdomain Delegation

```
example.com (Parent Zone)
    │
    └─ api.example.com (Delegated Zone)
       NS    ns1.api.example.com
       NS    ns2.api.example.com

Lookup: api.example.com/path
    1. Query example.com servers
    2. Get NS records for api.example.com
    3. Query api.example.com servers
    4. Get actual records
```

#### Dynamic DNS (DDNS)

When IP changes (home internet, mobile devices):

1. Client detects IP change
2. Sends update to DNS server (RFC 2136)
3. Server updates A/AAAA record
4. New IP propagates to all resolvers

Use cases: Home servers, VPN, mobile apps, edge computing

#### Anycast DNS

```
Single IP (e.g., 8.8.8.8) → Multiple data centers
    │
    ├─ California
    ├─ Europe
    ├─ Asia
    └─ Australia

User query → Nearest DC (lowest latency)
Automatic failover if DC down
```

***

### Troubleshooting

#### Query Tools

```bash
# Full info
dig example.com

# Specific record
dig example.com A
dig example.com MX
dig example.com TXT

# Specific nameserver
dig @ns1.example.com example.com
dig @8.8.8.8 example.com

# Trace resolution path
dig +trace example.com

# Answer only
dig example.com +short
dig example.com +noall +answer

# With statistics
dig example.com +stats
```

#### Problem: DNS Not Resolving

```bash
# Check configuration
cat /etc/resolv.conf

# Test different resolvers
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com

# Check connectivity to nameserver
nc -zv ns1.example.com 53

# Query specific nameserver
dig @ns1.example.com example.com

# Check firewall (UDP 53)
sudo iptables -L -n | grep 53
```

#### Problem: Slow DNS Resolution

```bash
# Measure response time
time dig example.com

# Check TTL
dig example.com +noall +answer

# Compare resolvers
dig @8.8.8.8 example.com +stats
dig @1.1.1.1 example.com +stats

# Check record TTL settings
dig @ns1 example.com +noall +answer
```

#### Problem: Zone Transfer Fails

```bash
# Check SOA serial on both servers
dig @ns1.example.com example.com SOA
dig @ns2.example.com example.com SOA

# Verify secondary config
sudo systemctl status bind9
sudo journalctl -u bind9 -f

# Check allow-transfer rules
cat /etc/bind/named.conf | grep allow-transfer

# Force secondary update
rndc notify example.com
```

#### Real-time Monitoring

```bash
# Watch DNS queries
tcpdump -i eth0 -n 'port 53'

# Watch specific domain
tcpdump -i eth0 -n 'port 53 and host example.com'

# Monitor BIND
tail -f /var/log/syslog | grep named

# Check resolver statistics
cat /var/log/bind/statistics.log
```

***

### Best Practices

#### Redundancy

```
Minimum: 2 nameservers (ICANN requirement)
Recommended: 3-4 nameservers
Location: Different data centers, different networks

Example setup:
ns1.example.com  (Primary DC)
ns2.example.com  (Secondary DC)
ns3.example.com  (Tertiary DC - Different Provider)
```

#### TTL Strategy

```
Static Content:    86400s (24 hours)  - A, AAAA
Applications:       3600s (1 hour)    - Web services
APIs:                300s (5 mins)    - Frequent changes
Mail:               3600s (1 hour)    - MX records
Load Balancers:       60s (1 min)     - Often changes
```

Lower TTL before planned changes, restore to normal after.

#### Security Settings (BIND)

```
# Restrict zone transfers
allow-transfer { 192.0.2.2; };

# Restrict recursive queries
allow-recursion { 
    127.0.0.1;           # Localhost
    192.168.1.0/24;      # Internal only
};

# Rate limiting
query-response-rate-limit { all category 100; };

# Logging
logging {
    channel default_log {
        file "/var/log/bind/queries.log" versions 7 size 100m;
        print-time yes;
    };
    category queries { default_log; };
};
```

#### Operational Procedures

**Before Making Changes:**

1. Lower TTL to 300s
2. Take backup of zone file
3. Document change
4. Test in staging

**After Making Changes:**

1. Monitor propagation
2. Verify from different locations
3. Check all records
4. Restore TTL to 3600s

**Zone File Changes:**

1. Edit zone file
2. Increment SOA serial number
3. Run: `named-checkzone example.com db.example.com`
4. Reload: `rndc reload example.com`
5. Verify: `dig example.com`

**Backup Strategy:**

```
- Daily backups of zone files
- Version control for configurations
- Keep 30 days of backups
- Test restore procedures monthly
```

#### Naming Conventions

```
www         - Web server
mail, smtp  - Mail servers
api         - API endpoints
cdn         - CDN
staging     - Test environment
ns1, ns2    - Nameservers
db          - Database
cache       - Cache servers
```

***

### Quick Reference

#### Common Commands Cheat Sheet

```bash
# Lookup
dig domain.com                      # Full info
dig domain.com +short               # IP only
dig domain.com MX                   # Mail servers
dig domain.com TXT                  # Text records

# Specific server
dig @ns1.domain.com domain.com      # Query nameserver
dig @8.8.8.8 domain.com             # Query Google DNS

# Trace
dig +trace domain.com               # Show full path

# Reverse
dig -x 93.184.216.34                # Reverse lookup

# Server management
rndc reload domain.com              # Reload zone
rndc flush                          # Clear cache
named-checkzone domain.com db.file  # Check zone
```

#### DNS Ports

| Service       | Port | Protocol |
| ------------- | ---- | -------- |
| DNS           | 53   | UDP      |
| Zone Transfer | 53   | TCP      |
| DoH (HTTPS)   | 443  | HTTPS    |
| DoT (TLS)     | 853  | TLS      |

#### Public Resolvers

| Provider   | IP               | Features                |
| ---------- | ---------------- | ----------------------- |
| Google     | 8.8.8.8, 8.8.4.4 | Fast, reliable          |
| Cloudflare | 1.1.1.1, 1.0.0.1 | Privacy-focused         |
| Quad9      | 9.9.9.9          | Security, malware block |

#### Config Paths

```
BIND:
  /etc/bind/named.conf           # Main config
  /etc/bind/zones/               # Zone files
  /var/log/syslog or bind/       # Logs

Unbound:
  /etc/unbound/unbound.conf      # Configuration
  /var/log/unbound.log           # Logs
```

***

### Summary

**DNS Essentials:**

* Hierarchical system with root → TLD → authoritative
* Every DNS lookup follows same basic flow
* Multiple caching layers for performance
* Requires redundancy for reliability
* Common records: A, AAAA, MX, TXT, NS, SOA

**Critical Takeaways:**

1. Always use 2+ nameservers (distributed)
2. Set appropriate TTLs (3600-86400 for stable)
3. Monitor DNS activity and failures
4. Test changes before deployment
5. Keep backups of zone files
6. Document all configurations
7. Plan for failures and rollback
8. Use logging for debugging

**Basic Workflow:**

```
Domain registered → Nameservers set → Zone created → 
Records added → Propagation (24-48h) → Live
```

***

### Resources

* [RFC 1035 - DNS Specification](https://tools.ietf.org/html/rfc1035)
* [BIND Official](https://www.isc.org/bind/)
* [PowerDNS](https://www.powerdns.com/)
* [CoreDNS](https://coredns.io/)
* [ICANN DNS Info](https://www.icann.org/dns)
