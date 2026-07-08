---
description: DNS Resolution Process
---

# DNS Resolution Process - Step by Step

```
    <div class="visualization">
        <!-- Step 1 -->
        <div class="step">
            <div class="step-number">1</div>
            <div class="step-content">
                <div class="step-title">User Initiates Query</div>
                <div class="step-description">
                    User types a domain name (www.example.com) in their browser or pings a domain.
                </div>
                <div class="code-block">Browser: example.com</div>
            </div>
        </div>
        
        <div class="arrow-down">↓</div>
        
        <!-- Step 2 -->
        <div class="step">
            <div class="step-number">2</div>
            <div class="step-content">
                <div class="step-title">Check Local Caches</div>
                <div class="step-description">
                    System checks if the IP address is already cached locally. This is the fastest path.
                </div>
                <div class="cache-layers">
                    <div class="cache-layer">
                        <div class="cache-layer-name">Browser Cache</div>
                        <div class="cache-layer-desc">Remembers sites you visit</div>
                        <div class="cache-layer-time">~0.001ms</div>
                    </div>
                    <div class="cache-layer">
                        <div class="cache-layer-name">OS Cache</div>
                        <div class="cache-layer-desc">System resolver cache</div>
                        <div class="cache-layer-time">~0.5ms</div>
                    </div>
                    <div class="cache-layer">
                        <div class="cache-layer-name">ISP Cache</div>
                        <div class="cache-layer-desc">Resolver cache (if found → return)</div>
                        <div class="cache-layer-time">~10ms</div>
                    </div>
                </div>
            </div>
        </div>
        
        <div class="arrow-down">↓ (If not cached)</div>
        
        <!-- Step 3 -->
        <div class="step">
            <div class="step-number">3</div>
            <div class="step-content">
                <div class="step-title">Query Recursive Resolver</div>
                <div class="step-description">
                    Typically operated by your ISP or Google DNS (8.8.8.8). This resolver will do the work of finding the answer for you.
                </div>
                <div class="code-block">Query: "Where is example.com?"
```

Recursive Resolver: "I'll find out"

```
        <div class="arrow-down">↓</div>
        
        <!-- Step 4 -->
        <div class="step">
            <div class="step-number">4</div>
            <div class="step-content">
                <div class="step-title">Query Root Nameserver</div>
                <div class="step-description">
                    Root server is asked which TLD server has the answer. One of 13 root server clusters responds.
                </div>
                <div class="code-block">Query: "Where is .com zone?"
```

Root: "Ask this TLD server" → IP of TLD

```
        <div class="arrow-down">↓</div>
        
        <!-- Step 5 -->
        <div class="step">
            <div class="step-number">5</div>
            <div class="step-content">
                <div class="step-title">Query TLD Nameserver</div>
                <div class="step-description">
                    TLD server knows which authoritative nameserver has the zone data for example.com.
                </div>
                <div class="code-block">Query: "Where is example.com?"
```

TLD: "Ask this authoritative server" → IP of Auth NS

```
        <div class="arrow-down">↓</div>
        
        <!-- Step 6 -->
        <div class="step">
            <div class="step-number">6</div>
            <div class="step-content">
                <div class="step-title">Query Authoritative Nameserver</div>
                <div class="step-description">
                    Authoritative server for example.com returns the actual DNS record with the IP address.
                </div>
                <div class="code-block">Query: "What is A record for example.com?"
```

Auth NS: "93.184.216.34" → TTL: 3600 seconds

```
        <div class="arrow-down">↓</div>
        
        <!-- Step 7 -->
        <div class="step">
            <div class="step-number">7</div>
            <div class="step-content">
                <div class="step-title">Return IP Through Chain</div>
                <div class="step-description">
                    IP address is returned back through each resolver, caching at each level along the way.
                </div>
                <div class="client-server">
                    <div class="server-box">
                        <div class="box-title">🔗 Response Path:</div>
                        Auth NS → TLD → Root → Recursive → OS → Browser
                    </div>
                    <div class="client-box">
                        <div class="box-title">💾 Caching:</div>
                        All components cache the result
                    </div>
                </div>
            </div>
        </div>
        
        <div class="arrow-down">↓</div>
        
        <!-- Step 8 -->
        <div class="step">
            <div class="step-number">8</div>
            <div class="step-content">
                <div class="step-title">Browser Connects to Server</div>
                <div class="step-description">
                    Browser now connects to the IP address (93.184.216.34) and loads the website.
                </div>
                <div class="code-block">Browser connects to: 93.184.216.34:80 (or :443 for HTTPS)
```

Website loads successfully!

```
        <!-- Timeline -->
        <div class="timeline">
            <div class="timeline-title">📊 Typical Resolution Timeline</div>
            <div class="timeline-item">
                <div class="timeline-dot"></div>
                <div class="timeline-text">
                    <strong>Cache Hit</strong> - IP found locally
                    <span class="timing">1-50ms</span>
                </div>
            </div>
            <div class="timeline-item">
                <div class="timeline-dot"></div>
                <div class="timeline-text">
                    <strong>ISP Resolver Hit</strong> - Found in recursive resolver
                    <span class="timing">50-150ms</span>
                </div>
            </div>
            <div class="timeline-item">
                <div class="timeline-dot"></div>
                <div class="timeline-text">
                    <strong>Full Resolution</strong> - Had to contact root/TLD/auth
                    <span class="timing">150-500ms</span>
                </div>
            </div>
            <div class="timeline-item">
                <div class="timeline-dot"></div>
                <div class="timeline-text">
                    <strong>Slow Resolution</strong> - Network issues, slow servers
                    <span class="timing">500ms+</span>
                </div>
            </div>
        </div>
        
        <!-- Comparison -->
        <div style="margin-top: 40px; padding: 20px; background: #f8f9fa; border-radius: 8px;">
            <div style="font-size: 16px; font-weight: bold; margin-bottom: 20px; color: #333;">⚡ Query Type Comparison</div>
            <div class="comparison">
                <div class="comparison-box" style="border-left: 4px solid #2196f3;">
                    <h4>✓ Recursive Query (What you use)</h4>
                    <ul>
                        <li>Client asks resolver</li>
                        <li>Resolver does all the work</li>
                        <li>Resolver guarantees answer or error</li>
                        <li>Most common query type</li>
                    </ul>
                </div>
                <div class="comparison-box" style="border-left: 4px solid #ff9800;">
                    <h4>✓ Iterative Query (What resolvers use)</h4>
                    <ul>
                        <li>Server asked to provide best answer</li>
                        <li>If it doesn't know, refers elsewhere</li>
                        <li>No guarantee of final answer</li>
                        <li>Used between DNS servers</li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</div>
```
