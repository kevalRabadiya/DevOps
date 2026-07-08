---
description: DNS Hierarchy Visualization
---

# DNS Hierarchy & Architecture

```
    <div class="hierarchy">
        <!-- Root Level -->
        <div class="level">
            <div class="node root-node">
                <h3>Root Nameservers</h3>
                <p>13 server clusters worldwide</p>
            </div>
        </div>
        
        <div class="connector">↓</div>
        
        <!-- TLD Level -->
        <div class="level">
            <div class="node tld-node">
                <h3>.COM TLD</h3>
                <p>Verisign</p>
            </div>
            <div class="node tld-node">
                <h3>.ORG TLD</h3>
                <p>PIR</p>
            </div>
            <div class="node tld-node">
                <h3>.NET TLD</h3>
                <p>Verisign</p>
            </div>
            <div class="node tld-node">
                <h3>.IO TLD</h3>
                <p>IO.net</p>
            </div>
        </div>
        
        <div class="connector">↓</div>
        
        <!-- Authoritative Level -->
        <div class="level">
            <div class="node auth-node">
                <h3>example.com</h3>
                <p>NS: ns1, ns2</p>
            </div>
            <div class="node auth-node">
                <h3>google.com</h3>
                <p>NS: ns1, ns2, ns3</p>
            </div>
            <div class="node auth-node">
                <h3>github.com</h3>
                <p>NS: ns1, ns2, ns3</p>
            </div>
            <div class="node auth-node">
                <h3>amazon.com</h3>
                <p>NS: ns1, ns2, ns3</p>
            </div>
        </div>
        
        <div class="connector">↓</div>
        
        <!-- Records Level -->
        <div class="level">
            <div class="node" style="background: linear-gradient(135deg, #fa709a 0%, #fee140 100%); color: #333;">
                <h3>DNS Records</h3>
                <p>A, AAAA, MX, CNAME, TXT, NS, SOA</p>
            </div>
        </div>
    </div>
    
    <div class="description">
        <strong>How it works:</strong> When you type example.com in your browser, the query travels down this hierarchy:
        <br><br>
        1. <strong>Root Servers</strong> tell you which TLD server to ask<br>
        2. <strong>TLD Servers</strong> tell you which authoritative nameserver has the answer<br>
        3. <strong>Authoritative Servers</strong> return the actual DNS record<br>
        4. <strong>Your browser</strong> connects to the IP address in the DNS record
    </div>
    
    <div class="legend">
        <div class="legend-item">
            <div class="legend-color root"></div>
            <span><strong>Root</strong> - Global entry point</span>
        </div>
        <div class="legend-item">
            <div class="legend-color tld"></div>
            <span><strong>TLD</strong> - Top-level domain (.com, .org)</span>
        </div>
        <div class="legend-item">
            <div class="legend-color auth"></div>
            <span><strong>Authoritative</strong> - Zone owner's servers</span>
        </div>
        <div class="legend-item">
            <div class="legend-color" style="background: linear-gradient(135deg, #fa709a 0%, #fee140 100%);"></div>
            <span><strong>Records</strong> - Actual DNS data</span>
        </div>
    </div>
</div>
```
