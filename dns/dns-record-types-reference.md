---
description: DNS Record Types Reference
---

# DNS Record Types Reference

Comprehensive guide to all common DNS record types and their use cases

```
    <!-- Common Zone File -->
    <div class="common-records">
        <div class="common-title">📝 Complete Zone File Example</div>
        <div class="zone-file">
```

; Zone file for example.com $ORIGIN example.com. $TTL 3600

; Authority @ SOA ns1.example.com. admin.example.com. (2024010101 3600 1800 604800 86400) @ NS ns1.example.com. @ NS ns2.example.com.

; Web Servers @ A 93.184.216.34 www A 93.184.216.34 api A 93.184.216.35

; Mail @ MX 10 mail.example.com. mail A 93.184.216.36

; Aliases blog CNAME www.example.com. shop CNAME www.example.com.

; Security @ TXT "v=spf1 include:\_spf.google.com \~all" \_dmarc TXT "v=DMARC1; p=reject;"

```
    <!-- Filter Buttons -->
    <div class="filter-buttons">
        <button class="filter-btn active" onclick="filterRecords('all')">All Records</button>
        <button class="filter-btn" onclick="filterRecords('web')">🌐 Web</button>
        <button class="filter-btn" onclick="filterRecords('mail')">📧 Mail</button>
        <button class="filter-btn" onclick="filterRecords('security')">🔒 Security</button>
        <button class="filter-btn" onclick="filterRecords('technical')">⚙️ Technical</button>
    </div>
    
    <!-- Records Grid -->
    <div class="records-grid" id="recordsGrid">
        <!-- Records will be inserted here by JavaScript -->
    </div>
    
    <!-- Comparison Section -->
    <div class="comparison-section">
        <div class="comparison-title">📊 Quick Comparison Table</div>
        <table class="comparison-table">
            <thead>
                <tr>
                    <th>Record Type</th>
                    <th>Points To</th>
                    <th>Purpose</th>
                    <th>Typical TTL</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td><strong>A</strong></td>
                    <td>IPv4 Address</td>
                    <td>Web/Server hosting</td>
                    <td>3600s (1h)</td>
                </tr>
                <tr>
                    <td><strong>AAAA</strong></td>
                    <td>IPv6 Address</td>
                    <td>IPv6 hosting</td>
                    <td>3600s (1h)</td>
                </tr>
                <tr>
                    <td><strong>CNAME</strong></td>
                    <td>Another Domain</td>
                    <td>Aliases/CDN</td>
                    <td>3600s (1h)</td>
                </tr>
                <tr>
                    <td><strong>MX</strong></td>
                    <td>Mail Server</td>
                    <td>Email routing</td>
                    <td>3600s (1h)</td>
                </tr>
                <tr>
                    <td><strong>TXT</strong></td>
                    <td>Text Data</td>
                    <td>Verification/Auth</td>
                    <td>3600s (1h)</td>
                </tr>
                <tr>
                    <td><strong>NS</strong></td>
                    <td>Nameserver</td>
                    <td>Zone delegation</td>
                    <td>86400s (24h)</td>
                </tr>
                <tr>
                    <td><strong>SOA</strong></td>
                    <td>Zone Info</td>
                    <td>Zone authority</td>
                    <td>3600s (1h)</td>
                </tr>
                <tr>
                    <td><strong>PTR</strong></td>
                    <td>Domain</td>
                    <td>Reverse DNS</td>
                    <td>3600s (1h)</td>
                </tr>
                <tr>
                    <td><strong>SRV</strong></td>
                    <td>Server:Port</td>
                    <td>Service location</td>
                    <td>3600s (1h)</td>
                </tr>
                <tr>
                    <td><strong>CAA</strong></td>
                    <td>Certificate CA</td>
                    <td>Certificate auth</td>
                    <td>3600s (1h)</td>
                </tr>
            </tbody>
        </table>
    </div>
</div>

<script>
    const records = [
        {
            name: 'A',
            category: 'web',
            badge: 'Web',
            description: 'Maps domain name to IPv4 address. Most fundamental DNS record type.',
            syntax: 'example.com    A    93.184.216.34',
            ttl: '3600 (1 hour)',
            useCases: [
                'Point domain to web server',
                'Direct email to mail server',
                'Route API requests',
                'Redirect subdomains'
            ]
        },
        {
            name: 'AAAA',
            category: 'web',
            badge: 'Web',
            description: 'Maps domain name to IPv6 address. Used for modern IPv6-enabled networks.',
            syntax: 'example.com    AAAA    2001:db8::1',
            ttl: '3600 (1 hour)',
            useCases: [
                'IPv6 web server hosting',
                'Dual-stack support',
                'Future-proof DNS',
                'Modern network support'
            ]
        },
        {
            name: 'CNAME',
            category: 'web',
            badge: 'Web',
            description: 'Creates an alias for another domain. Cannot point to IP addresses.',
            syntax: 'www.example.com    CNAME    example.com',
            ttl: '3600 (1 hour)',
            useCases: [
                'WWW subdomain aliasing',
                'CDN configuration',
                'Service migration',
                'Multiple domain names'
            ]
        },
        {
            name: 'MX',
            category: 'mail',
            badge: 'Mail',
            description: 'Directs email to mail servers. Multiple records with priority values.',
            syntax: 'example.com    MX    10    mail.example.com',
            ttl: '3600 (1 hour)',
            useCases: [
                'Email routing',
                'Mail server failover',
                'Backup mail server',
                'Mail service configuration'
            ]
        },
        {
            name: 'TXT',
            category: 'security',
            badge: 'Security',
            description: 'Stores text information used for email authentication and verification.',
            syntax: 'example.com    TXT    "v=spf1 include:_spf.google.com ~all"',
            ttl: '3600 (1 hour)',
            useCases: [
                'SPF (Sender Policy Framework)',
                'DKIM (DomainKeys Identified Mail)',
                'DMARC (Domain-based Message Authentication)',
                'Site verification (Google, Facebook)'
            ]
        },
        {
            name: 'NS',
            category: 'technical',
            badge: 'Technical',
            description: 'Specifies authoritative nameservers for the domain or subdomain.',
            syntax: 'example.com    NS    ns1.example.com',
            ttl: '86400 (24 hours)',
            useCases: [
                'Zone delegation',
                'Subdomain nameservers',
                'DNS service definition',
                'Authority indication'
            ]
        },
        {
            name: 'SOA',
            category: 'technical',
            badge: 'Technical',
            description: 'Start of Authority record containing zone metadata and master-slave parameters.',
            syntax: 'example.com    SOA    ns1.example.com    admin@example.com    (2024010101 3600 1800 604800 86400)',
            ttl: '3600 (1 hour)',
            useCases: [
                'Zone master identification',
                'Serial number management',
                'Refresh/retry timing',
                'Primary-secondary sync'
            ]
        },
        {
            name: 'PTR',
            category: 'technical',
            badge: 'Technical',
            description: 'Reverse DNS lookup. Maps IP address to domain name.',
            syntax: '34.216.184.93.in-addr.arpa    PTR    example.com',
            ttl: '3600 (1 hour)',
            useCases: [
                'Email authentication',
                'Server logging',
                'Reverse DNS lookups',
                'Access logging'
            ]
        },
        {
            name: 'SRV',
            category: 'technical',
            badge: 'Technical',
            description: 'Specifies location of service with port information.',
            syntax: '_sip._tcp.example.com    SRV    10 60 5060 sipserver.example.com',
            ttl: '3600 (1 hour)',
            useCases: [
                'SIP/VoIP services',
                'LDAP directory',
                'Kerberos authentication',
                'Service discovery'
            ]
        },
        {
            name: 'CAA',
            category: 'security',
            badge: 'Security',
            description: 'Specifies which Certificate Authorities can issue certificates for the domain.',
            syntax: 'example.com    CAA    0 issue "letsencrypt.org"',
            ttl: '3600 (1 hour)',
            useCases: [
                'SSL certificate control',
                'CA authorization',
                'Wildcard restrictions',
                'Certificate issuance policy'
            ]
        },
        {
            name: 'DKIM',
            category: 'security',
            badge: 'Security',
            description: 'DomainKeys Identified Mail record for email signing.',
            syntax: 'default._domainkey.example.com    TXT    "v=DKIM1; p=MIGfMA0..."',
            ttl: '3600 (1 hour)',
            useCases: [
                'Email authentication',
                'Anti-spoofing protection',
                'Email signature verification',
                'Gmail/Office365 setup'
            ]
        },
        {
            name: 'DMARC',
            category: 'security',
            badge: 'Security',
            description: 'Domain-based Message Authentication, Reporting and Conformance.',
            syntax: '_dmarc.example.com    TXT    "v=DMARC1; p=reject;"',
            ttl: '3600 (1 hour)',
            useCases: [
                'Email authentication policy',
                'Quarantine/reject rules',
                'Monitoring reports',
                'Brand protection'
            ]
        }
    ];
    
    function renderRecords(filterCategory = 'all') {
        const grid = document.getElementById('recordsGrid');
        grid.innerHTML = '';
        
        const filtered = filterCategory === 'all' 
            ? records 
            : records.filter(r => r.category === filterCategory);
        
        filtered.forEach(record => {
            const card = document.createElement('div');
            card.className = `record-card ${record.category}`;
            card.innerHTML = `
                <div class="record-header">
                    <div class="record-name">${record.name}</div>
                    <div class="record-badge badge-${record.category}">${record.badge}</div>
                </div>
                <div class="record-description">${record.description}</div>
                
                <div class="record-section">
                    <div class="section-title">Syntax</div>
                    <div class="syntax">${record.syntax}</div>
                </div>
                
                <div class="record-section">
                    <div class="section-title">TTL</div>
                    <div class="ttl-info">${record.ttl}</div>
                </div>
                
                <div class="record-section">
                    <div class="section-title">Use Cases</div>
                    <div class="use-cases">
                        ${record.useCases.map(uc => `<div class="use-case">${uc}</div>`).join('')}
                    </div>
                </div>
            `;
            grid.appendChild(card);
        });
    }
    
    function filterRecords(category) {
        // Update button states
        document.querySelectorAll('.filter-btn').forEach(btn => {
            btn.classList.remove('active');
        });
        event.target.classList.add('active');
        
        // Render filtered records
        renderRecords(category);
    }
    
    // Initial render
    renderRecords();
</script>
```
