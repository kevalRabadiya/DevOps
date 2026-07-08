---
description: >-
  A complete, practical guide to setting up free SSL/TLS certificates using
  Let's Encrypt + Certbot on an Ubuntu server, including manual setup, automatic
  renewal, and testing.
---

# SSL/TLS Certificate Guide (Ubuntu Server)

> ***
>
> ### Table of Contents
>
> * 1\. Overview
> * 2\. Prerequisites
> * 3\. How It Works (Diagram)
> * 4\. Install Certbot
> * 5\. Issue a Certificate
>   * 5.1 Nginx
>   * 5.2 Apache
>   * 5.3 Standalone (no web server / custom setup)
> * 6\. Verify Certificate Installation
> * 7\. Manual Renewal
> * 8\. Automatic Renewal
>   * 8.1 Renewal Flow (Diagram)
>   * 8.2 Systemd Timer (default)
>   * 8.3 Cron Job (alternative)
> * 9\. Testing Everything
> * 10\. Firewall Rules
> * 11\. Common Errors & Fixes
> * 12\. Useful Commands Cheat Sheet
>
> ***
>
> ### 1. Overview
>
> ***
>
> ### 2. Prerequisites
>
> Before starting, make sure you have:
>
> * ✅ A registered **domain name** (e.g. `example.com`) pointing to your server's public IP (A/AAAA record)
> * ✅ Root or `sudo` access to the Ubuntu server
> * ✅ Port **80** (HTTP) and **443** (HTTPS) open in your firewall/security group
> * ✅ A running web server (Nginx or Apache) — optional if using standalone mode
>
> Check your DNS is pointing correctly:
>
> ```bash
> dig +short example.com
> ```
>
> This should return your server's public IP address.
>
> ***
>
> ### 3. How It Works (Diagram)
>
> ```mermaid
> flowchart LR
>     A[Your Domain] -->|DNS A Record| B[Ubuntu Server]
>     B -->|Certbot requests cert| C[Let's Encrypt CA]
>     C -->|Domain validation<br/>HTTP-01 / TLS-ALPN-01| B
>     C -->|Issues Certificate| B
>     B -->|Installs cert| D[Nginx / Apache]
>     D -->|Serves HTTPS| E[Visitor Browser]
> ```
>
> ***
>
> ### 4. Install Certbot
>
> Update packages and install Certbot via **snap** (officially recommended method):
>
> ```bash
> sudo apt update && sudo apt upgrade -y
>
> # Remove any old certbot installed via apt (avoids conflicts)
> sudo apt remove certbot -y
>
> # Install snapd if not present
> sudo apt install snapd -y
>
> # Install core snap and certbot
> sudo snap install core
> sudo snap refresh core
> sudo snap install --classic certbot
>
> # Create a symlink so 'certbot' command works globally
> sudo ln -s /snap/bin/certbot /usr/bin/certbot
> ```
>
> Verify installation:
>
> ```bash
> certbot --version
> ```
>
> ***
>
> ### 5. Issue a Certificate
>
> #### 5.1 Nginx (recommended)
>
> Install the Nginx plugin and issue + auto-configure the certificate in one command:
>
> ```bash
> sudo apt install python3-certbot-nginx -y
>
> sudo certbot --nginx -d example.com -d www.example.com
> ```
>
> Certbot will:
>
> 1. Verify domain ownership
> 2. Obtain the certificate
> 3. Automatically edit your Nginx config to enable HTTPS
> 4. Ask if you want to redirect HTTP → HTTPS (choose **yes**)
>
> #### 5.2 Apache
>
> ```bash
> sudo apt install python3-certbot-apache -y
>
> sudo certbot --apache -d example.com -d www.example.com
> ```
>
> #### 5.3 Standalone (no web server / custom setup)
>
> Use this if no web server is running on port 80, or you manage certs manually (e.g., for a custom app, mail server, etc.):
>
> ```bash
> sudo systemctl stop nginx   # stop whatever uses port 80 first
> sudo certbot certonly --standalone -d example.com -d www.example.com
> sudo systemctl start nginx
> ```
>
> Certificates are stored at:
>
> ```
> /etc/letsencrypt/live/example.com/fullchain.pem   # certificate + chain
> /etc/letsencrypt/live/example.com/privkey.pem     # private key
> ```
>
> ***
>
> ### 6. Verify Certificate Installation
>
> Check that Nginx/Apache config is valid and reload:
>
> ```bash
> # Nginx
> sudo nginx -t && sudo systemctl reload nginx
>
> # Apache
> sudo apachectl configtest && sudo systemctl reload apache2
> ```
>
> View certificate details:
>
> ```bash
> sudo certbot certificates
> ```
>
> Check expiry date directly from the cert file:
>
> ```bash
> sudo openssl x509 -enddate -noout -in /etc/letsencrypt/live/example.com/fullchain.pem
> ```
>
> ***
>
> ### 7. Manual Renewal
>
> To renew all certificates manually at any time:
>
> ```bash
> sudo certbot renew
> ```
>
> To force-renew even if not close to expiry (useful for testing config changes):
>
> ```bash
> sudo certbot renew --force-renewal
> ```
>
> To renew only one specific domain's certificate:
>
> ```bash
> sudo certbot certonly --cert-name example.com --force-renewal
> ```
>
> After manual renewal, reload the web server so it picks up the new cert:
>
> ```bash
> sudo systemctl reload nginx
> ```
>
> ***
>
> ### 8. Automatic Renewal
>
> Let's Encrypt certificates are valid for **90 days**. Certbot's auto-renewal system checks twice a day and renews any certificate within **30 days of expiry**.
>
> #### 8.1 Renewal Flow (Diagram)
>
> ```mermaid
> sequenceDiagram
>     participant Timer as systemd Timer (2x/day)
>     participant Certbot
>     participant CA as Let's Encrypt
>     participant Web as Nginx/Apache
>
>     Timer->>Certbot: Trigger renewal check
>     Certbot->>Certbot: Is cert < 30 days from expiry?
>     alt Yes, renew needed
>         Certbot->>CA: Request new certificate
>         CA-->>Certbot: Issue renewed certificate
>         Certbot->>Web: Reload service (deploy hook)
>     else No, not needed yet
>         Certbot-->>Timer: Skip, nothing to do
>     end
> ```
>
> #### 8.2 Systemd Timer (default)
>
> When installed via snap, Certbot **automatically** sets up a systemd timer. Check it's active:
>
> ```bash
> systemctl list-timers | grep certbot
> ```
>
> You should see `snap.certbot.renew.service` scheduled to run.
>
> Check the timer status directly:
>
> ```bash
> sudo systemctl status snap.certbot.renew.service
> ```
>
> **Add a deploy hook** so your web server always reloads after a real renewal (only runs when a cert actually renews):
>
> ```bash
> sudo mkdir -p /etc/letsencrypt/renewal-hooks/deploy
>
> sudo tee /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh > /dev/null <<'EOF'
> #!/bin/bash
> systemctl reload nginx
> EOF
>
> sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh
> ```
>
> #### 8.3 Cron Job (alternative)
>
> If you prefer cron over systemd (e.g., certbot installed via apt instead of snap):
>
> ```bash
> sudo crontab -e
> ```
>
> Add this line to run renewal check twice daily at random-ish times (avoids load spikes on Let's Encrypt servers):
>
> ```bash
> 0 3,15 * * * root certbot renew --quiet --deploy-hook "systemctl reload nginx"
> ```
>
> Verify the cron entry:
>
> ```bash
> sudo crontab -l
> ```
>
> ***
>
> ### 9. Testing Everything
>
> #### 9.1 Dry-run renewal test (safe, no actual renewal)
>
> This is the **most important test** — it simulates the entire renewal process without hitting rate limits or changing anything:
>
> ```bash
> sudo certbot renew --dry-run
> ```
>
> Look for: `Congratulations, all simulated renewals succeeded`
>
> #### 9.2 Test HTTPS is live
>
> ```bash
> curl -I https://example.com
> ```
>
> Look for `HTTP/2 200` or `HTTP/1.1 200 OK`.
>
> #### 9.3 Check certificate chain from the server
>
> ```bash
> echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates -subject -issuer
> ```
>
> #### 9.4 Online SSL configuration test
>
> Run a full grading test (checks protocol versions, cipher strength, chain, HSTS, etc.):
>
> ```
> https://www.ssllabs.com/ssltest/analyze.html?d=example.com
> ```
>
> Aim for grade **A** or **A+**.
>
> #### 9.5 Test HTTP → HTTPS redirect
>
> ```bash
> curl -I http://example.com
> ```
>
> Should return `301` or `302` redirecting to `https://`.
>
> ***
>
> ### 10. Firewall Rules
>
> If using `ufw`:
>
> ```bash
> sudo ufw allow 'Nginx Full'      # allows both 80 and 443
> sudo ufw allow OpenSSH
> sudo ufw enable
> sudo ufw status
> ```
>
> Or manually:
>
> ```bash
> sudo ufw allow 80/tcp
> sudo ufw allow 443/tcp
> ```
>
> ***
>
> ### 11. Common Errors & Fixes
>
> ***
>
> ### 12. Useful Commands Cheat Sheet
>
> ```bash
> # Install certbot
> sudo snap install --classic certbot
> sudo ln -s /snap/bin/certbot /usr/bin/certbot
>
> # Issue certificate (Nginx)
> sudo certbot --nginx -d example.com -d www.example.com
>
> # List all certificates
> sudo certbot certificates
>
> # Manual renewal (all certs)
> sudo certbot renew
>
> # Force renew a specific domain
> sudo certbot certonly --cert-name example.com --force-renewal
>
> # Dry-run test (safe)
> sudo certbot renew --dry-run
>
> # Check renewal timer status
> systemctl list-timers | grep certbot
>
> # Delete a certificate
> sudo certbot delete --cert-name example.com
>
> # Revoke a certificate
> sudo certbot revoke --cert-path /etc/letsencrypt/live/example.com/fullchain.pem
> ```
>
> ***
>
> #### Summary
>
> * **Issue**: `certbot --nginx -d yourdomain.com`
> * **Renew (manual)**: `certbot renew`
> * **Renew (auto)**: systemd timer runs twice daily — no action needed
> * **Test**: `certbot renew --dry-run` + SSL Labs test
>
> Your site should now auto-renew forever without manual intervention. ✅

| Error                                              | Likely Cause                                     | Fix                                                |
| -------------------------------------------------- | ------------------------------------------------ | -------------------------------------------------- |
| `Timeout during connect (likely firewall problem)` | Port 80/443 blocked                              | Open ports in `ufw` / cloud security group         |
| `DNS problem: NXDOMAIN`                            | Domain not pointing to server                    | Fix A record, wait for DNS propagation             |
| `Too many certificates already issued`             | Hit Let's Encrypt rate limit (5/week per domain) | Wait 7 days, or use `--dry-run` for testing        |
| `Certificate not yet due for renewal`              | Cert still valid > 30 days                       | Use `--force-renewal` to override for testing      |
| Nginx fails to reload after renewal                | Syntax error or missing reload hook              | Run `sudo nginx -t`, add deploy hook (Section 8.2) |

| Item                  | Detail                                         |
| --------------------- | ---------------------------------------------- |
| Certificate Authority | Let's Encrypt (free, 90-day validity)          |
| Tool                  | Certbot                                        |
| OS                    | Ubuntu 20.04 / 22.04 / 24.04                   |
| Web servers supported | Nginx, Apache, or standalone                   |
| Renewal               | Automatic (every 60 days, checked twice daily) |
