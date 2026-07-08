---
description: Protecting DNS from threats and ensuring data integrity
---

# DNS Security & DNSSEC

```
    <!-- DNS Threats Section -->
    <div class="section">
        <div class="section-title">
            ⚠️ Common DNS Threats
        </div>
        
        <div class="threat-box">
            <div class="threat-card">
                <div class="threat-title">
                    🎯 DNS Spoofing / Cache Poisoning
                </div>
                <div class="threat-description">
                    Attacker intercepts DNS queries and responds with fake IP addresses, redirecting users to malicious websites.
                </div>
                <ul class="protection-list">
                    <li>DNSSEC signatures</li>
                    <li>Query ID randomization</li>
                    <li>Port randomization</li>
                    <li>Rate limiting</li>
                </ul>
            </div>
            
            <div class="threat-card">
                <div class="threat-title">
                    💥 DNS Amplification DDoS
                </div>
                <div class="threat-description">
                    Attacker spoofs victim's IP and sends queries to open resolvers, amplifying traffic to target.
                </div>
                <ul class="protection-list">
                    <li>Query filtering</li>
                    <li>Rate limiting</li>
                    <li>BCP 38 implementation</li>
                    <li>Access lists</li>
                </ul>
            </div>
            
            <div class="threat-card">
                <div class="threat-title">
                    🔍 DNS Enumeration
                </div>
                <div class="threat-description">
                    Attacker discovers infrastructure by performing zone transfers (AXFR) from nameservers.
                </div>
                <ul class="protection-list">
                    <li>Restrict zone transfers</li>
                    <li>Query filtering</li>
                    <li>TSIG authentication</li>
                    <li>Access control lists</li>
                </ul>
            </div>
            
            <div class="threat-card">
                <div class="threat-title">
                    🚪 DNS Hijacking
                </div>
                <div class="threat-description">
                    Attacker gains control of registrar account and changes nameserver addresses.
                </div>
                <ul class="protection-list">
                    <li>Two-factor auth</li>
                    <li>DNS locking</li>
                    <li>Email alerts</li>
                    <li>Regular audits</li>
                </ul>
            </div>
        </div>
    </div>
    
    <!-- DNSSEC Section -->
    <div class="section">
        <div class="section-title">
            ✨ DNSSEC (DNS Security Extensions)
        </div>
        
        <div class="security-flow">
            <div class="security-column">
                <h4>🔑 DNSSEC Process</h4>
                <div class="security-item">
                    <div class="security-item-icon">1</div>
                    <div>
                        <strong>Key Generation</strong>
                        <div style="font-size: 11px; margin-top: 2px;">Domain owner creates public/private key pair</div>
                    </div>
                </div>
                <div class="security-item">
                    <div class="security-item-icon">2</div>
                    <div>
                        <strong>Zone Signing</strong>
                        <div style="font-size: 11px; margin-top: 2px;">Signs all records with private key</div>
                    </div>
                </div>
                <div class="security-item">
                    <div class="security-item-icon">3</div>
                    <div>
                        <strong>Publish Keys</strong>
                        <div style="font-size: 11px; margin-top: 2px;">DNSKEY record published in zone</div>
                    </div>
                </div>
                <div class="security-item">
                    <div class="security-item-icon">4</div>
                    <div>
                        <strong>DS Record</strong>
                        <div style="font-size: 11px; margin-top: 2px;">Hash submitted to parent zone</div>
                    </div>
                </div>
            </div>
            
            <div class="security-column">
                <h4>✅ Resolver Validation</h4>
                <div class="security-item">
                    <div class="security-item-icon">1</div>
                    <div>
                        <strong>Receive Response</strong>
                        <div style="font-size: 11px; margin-top: 2px;">Gets response with RRSIG signature</div>
                    </div>
                </div>
                <div class="security-item">
                    <div class="security-item-icon">2</div>
                    <div>
                        <strong>Get Public Key</strong>
                        <div style="font-size: 11px; margin-top: 2px;">Retrieves DNSKEY record</div>
                    </div>
                </div>
                <div class="security-item">
                    <div class="security-item-icon">3</div>
                    <div>
                        <strong>Verify Signature</strong>
                        <div style="font-size: 11px; margin-top: 2px;">Validates response authenticity</div>
                    </div>
                </div>
                <div class="security-item">
                    <div class="security-item-icon">4</div>
                    <div>
                        <strong>Result</strong>
                        <div style="font-size: 11px; margin-top: 2px;">✓ Authentic or ✗ Tampered</div>
                    </div>
                </div>
            </div>
        </div>
        
        <div style="margin-top: 20px; padding: 15px; background: #f0f4ff; border-radius: 8px; border-left: 4px solid #667eea;">
            <strong>🔐 DNSSEC Record Types:</strong>
            <div style="margin-top: 10px; font-size: 13px; color: #666; line-height: 1.8;">
                • <strong>DNSKEY:</strong> Public key for zone signing<br>
                • <strong>RRSIG:</strong> Digital signature on DNS records<br>
                • <strong>DS:</strong> Delegation Signer (hash of DNSKEY)<br>
                • <strong>NSEC/NSEC3:</strong> Proves non-existence of records<br>
                • <strong>DLVANCHOR:</strong> Trust anchor for DNSSEC validation
            </div>
        </div>
    </div>
    
    <!-- Implementation Section -->
    <div class="section">
        <div class="section-title">
            🛠️ DNSSEC Implementation Examples
        </div>
        
        <div style="margin-bottom: 20px;">
            <strong>Step 1: Generate Zone Signing Keys</strong>
            <div class="code-block">dnssec-keygen -a RSASHA256 -b 2048 -n ZONE example.com
```

## Generates Kexample.com.+008+XXXXX.key (KSK)

## Generates Kexample.com.+008+YYYYY.key (ZSK)

```
        </div>
        
        <div style="margin-bottom: 20px;">
            <strong>Step 2: Sign the Zone</strong>
            <div class="code-block">dnssec-signzone -A -3 $(head -c 1000 /dev/urandom | md5sum | cut -b 1-16) \
           -N INCREMENT -o example.com db.example.com \
           Kexample.com.+008+*.key
```

## Creates db.example.com.signed with signatures

```
        </div>
        
        <div>
            <strong>Step 3: Verify Signature</strong>
            <div class="code-block">delv @ns1.example.com example.com A
```

## Shows: example.com 3600 IN A 93.184.216.34 ;{ad}

## {ad} flag = authenticated data

```
        </div>
    </div>
    
    <!-- Best Practices Section -->
    <div class="section">
        <div class="section-title">
            ✅ DNS Security Best Practices
        </div>
        
        <div class="comparison-grid">
            <div class="comparison-box good">
                <h4>✓ Good Security Practices</h4>
                <ul>
                    <li>Enable DNSSEC signing</li>
                    <li>Rotate keys regularly</li>
                    <li>Restrict zone transfers</li>
                    <li>Monitor validation failures</li>
                    <li>Use strong authentication</li>
                    <li>Implement rate limiting</li>
                    <li>Enable query logging</li>
                    <li>Audit DNS changes</li>
                    <li>Use reputable DNS providers</li>
                    <li>Test resolver validation</li>
                </ul>
            </div>
            
            <div class="comparison-box bad">
                <h4>✗ Security Mistakes to Avoid</h4>
                <ul>
                    <li>Allowing open recursion</li>
                    <li>Allowing zone transfers from anyone</li>
                    <li>Storing private keys insecurely</li>
                    <li>Not monitoring DNS queries</li>
                    <li>Weak registrar passwords</li>
                    <li>No email alerts on changes</li>
                    <li>Using default configurations</li>
                    <li>Not backing up zone files</li>
                    <li>Ignoring DNSSEC validation</li>
                    <li>Exposed nameserver details</li>
                </ul>
            </div>
        </div>
    </div>
    
    <!-- DNSSEC Keys Section -->
    <div class="section">
        <div class="section-title">
            🔑 DNSSEC Key Types
        </div>
        
        <div class="dnssec-keys">
            <div class="key-box">
                <h4>🔐 KSK (Key Signing Key)</h4>
                <p>
                    <strong>Purpose:</strong> Signs the zone signing keys (ZSK)
                </p>
                <p style="margin-top: 8px;">
                    <strong>Characteristics:</strong>
                    <ul style="margin-left: 15px; margin-top: 5px; font-size: 12px; line-height: 1.6;">
                        <li>Longer key length (2048-4096 bits)</li>
                        <li>Changed less frequently</li>
                        <li>Published in DNSKEY record</li>
                        <li>DS record in parent zone</li>
                    </ul>
                </p>
            </div>
            
            <div class="key-box">
                <h4>🔑 ZSK (Zone Signing Key)</h4>
                <p>
                    <strong>Purpose:</strong> Signs all DNS records in the zone
                </p>
                <p style="margin-top: 8px;">
                    <strong>Characteristics:</strong>
                    <ul style="margin-left: 15px; margin-top: 5px; font-size: 12px; line-height: 1.6;">
                        <li>Shorter key length (1024-2048 bits)</li>
                        <li>Changed more frequently</li>
                        <li>Signs actual DNS records</li>
                        <li>Performance critical</li>
                    </ul>
                </p>
            </div>
        </div>
    </div>
    
    <!-- Troubleshooting Section -->
    <div class="section">
        <div class="section-title">
            🔧 DNSSEC Troubleshooting
        </div>
        
        <div class="process-flow">
            <div class="process-step">
                <div class="step-number">1</div>
                <div class="step-content">
                    <h3>Check DNSSEC Status</h3>
                    <p><code>dig example.com +dnssec</code> should show RRSIG records</p>
                    <p style="margin-top: 5px;">Look for ad (authenticated data) flag in response</p>
                </div>
            </div>
            
            <div class="process-step">
                <div class="step-number">2</div>
                <div class="step-content">
                    <h3>Validate DS Records</h3>
                    <p>Verify DS record in parent zone matches local DNSKEY</p>
                    <p style="margin-top: 5px;"><code>dig DS example.com</code></p>
                </div>
            </div>
            
            <div class="process-step">
                <div class="step-number">3</div>
                <div class="step-content">
                    <h3>Check Key Expiration</h3>
                    <p>Ensure signing keys haven't expired</p>
                    <p style="margin-top: 5px;"><code>dnssec-dsfromkey Kexample.com.+008+XXXXX.key</code></p>
                </div>
            </div>
            
            <div class="process-step">
                <div class="step-number">4</div>
                <div class="step-content">
                    <h3>Monitor Validation Failures</h3>
                    <p>Check resolver logs for DNSSEC validation errors</p>
                    <p style="margin-top: 5px;">Look for BOGUS or SERVFAIL responses</p>
                </div>
            </div>
        </div>
    </div>
</div>
```
