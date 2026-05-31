# SOC Lab — teluguseemalo.in

Industry-level Security Operations Center (SOC) lab built on a 
live production React web application, monitoring real-world 
attacks using Cloudflare WAF, Elastic SIEM, and OWASP ZAP.

![GitHub last commit](https://img.shields.io/github/last-commit/rangumallikarjun/soc-lab-teluguseemalo)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## Architecture
Internet Traffic
↓
teluguseemalo.in (React/Netlify)
↓
Cloudflare WAF + CDN
├── Block: Scanners, Path Traversal, Admin Probing
├── Rate Limiting: 20 req/10sec
└── Worker: soc-log-forwarder
↓
ngrok Tunnel (oppressor-yonder-calm.ngrok-free.dev)
↓
Logstash (port 8080) → JSON Parse → GeoIP Lookup
↓
Elasticsearch (port 9200) → cloudflare-logs-*
↓
Kibana (port 5601) → SOC Dashboard

---

## Tech Stack

| Component | Tool | Version |
|---|---|---|
| Website | React / Netlify | — |
| Domain | teluguseemalo.in (GoDaddy) | — |
| WAF | Cloudflare Free | — |
| Log Forwarder | Cloudflare Worker | — |
| Tunnel | ngrok | 3.39.5 |
| SIEM | Elasticsearch | 8.11.0 |
| Dashboard | Kibana | 8.11.0 |
| Log Pipeline | Logstash | 8.11.0 |
| Scanner | OWASP ZAP | 2.17.0 |
| Incident Mgmt | Jira | Free |

---

## Phases Completed

### Phase 1 — Cloudflare WAF
- Configured custom WAF rules blocking scanners (sqlmap, nikto, nmap, masscan)
- Blocked path traversal attempts (/.git, /etc/passwd, /.env)
- Blocked admin probing (/wp-admin, /phpmyadmin)
- Rate limiting: 20 requests per 10 seconds per IP
- Deployed Cloudflare Worker (soc-log-forwarder) to forward logs

### Phase 2 — Elastic SIEM + Kibana
- Self-hosted Elastic Stack 8.11 via Docker
- Real-time log ingestion via ngrok → Logstash → Elasticsearch
- Built 7-panel Kibana SOC Dashboard:
  - World map with GeoIP visualization
  - Requests over time
  - Top source IPs
  - Top URLs
  - Request methods
  - Top user agents
  - Requests by country

### Phase 3 — OWASP ZAP
- Ran Spider + Active Scan against live site
- Generated 200+ attack requests visible in Kibana
- Found 12 alerts including missing security headers
- Identified OWASP Top 10 vulnerabilities

### Phase 4 — Incident Management (Jira)
- Created Jira project: SOC Lab — teluguseemalo.in
- Triaged 3 real security incidents:
  - INC-001: Path Traversal — Poland IP (Remediated)
  - INC-002: WordPress Login Probe (Remediated)
  - INC-003: High Volume Reconnaissance (Investigating)
- MITRE ATT&CK mapping for each incident

### Phase 5 — Security Remediation
- Implemented security headers via netlify.toml
- Reduced ZAP alerts from 12 to 8
- Iteratively tuned CSP for Tawk.to, Google Fonts, Cloudflare Beacon

---

## Real Incidents Detected

### INC-001 — Path Traversal Attack (Poland)
| Field | Value |
|---|---|
| IP | 94.26.88.32 |
| Country | Poland |
| ASN | AS201814 MEVSPACE sp. z o.o. |
| Path | /.git/config |
| Time | 2026-05-30 20:11:00 EDT |
| Total Requests | 169+ |
| Action | Blocked by WAF |

**MITRE ATT&CK:**
- T1190 — Exploit Public-Facing Application
- T1083 — File and Directory Discovery
- T1036 — Masquerading (spoofed Firefox user agent)
- TA0043 — Reconnaissance

### INC-002 — WordPress Probe
- Automated scanner probing /wp-login.php on non-WordPress site
- Detected via Kibana Top URLs panel
- Remediated by adding path to WAF block rule

### INC-003 — High Volume Reconnaissance
- IP 68.197.65.7 generated 87 requests in 24 hours
- Detected via Kibana Top Source IPs panel
- Under investigation

---

## Security Headers Implemented

```toml
X-Frame-Options = "DENY"
X-Content-Type-Options = "nosniff"
X-XSS-Protection = "1; mode=block"
Referrer-Policy = "strict-origin-when-cross-origin"
Strict-Transport-Security = "max-age=31536000; includeSubDomains"
Content-Security-Policy = "default-src 'self'; ..."
```

---

## Startup Sequence

```bash
# 1. Start Docker
cd C:\soc-lab
docker-compose up -d

# 2. Start ngrok
ngrok http --domain=oppressor-yonder-calm.ngrok-free.dev 8080

# 3. Open Kibana
http://localhost:5601

# 4. Open ngrok inspector
http://localhost:4040
```

---

## Screenshots

| Screenshot | Description |
|---|---|
| ![WAF Rules](screenshots/phase1-cloudflare/cloudflare-waf-rules.png) | Cloudflare WAF rules — 2 custom rules + rate limiting active |
| ![Poland IP](screenshots/phase1-cloudflare/cloudflare-poland-ip.png) | Real attack blocked — Poland IP 94.26.88.32 attempting /.git/config |
| ![Docker](screenshots/phase2-kibana/docker-containers.png) | Docker Desktop — all 3 containers running |
| ![Kibana](screenshots/phase2-kibana/kibana-dashboard.png) | Kibana SOC Dashboard — world map + all panels |
| ![ZAP Before](screenshots/phase3-zap/zap-before-fix.png) | OWASP ZAP — 12 alerts before security headers |
| ![ZAP After](screenshots/phase3-zap/zap-after-fix.png) | OWASP ZAP — 8 alerts after netlify.toml fix |
| ![Jira](screenshots/phase4-jira/jira-board.png) | Jira board — 3 real incidents triaged |
| ![Headers](screenshots/phase5-remediation/response-headers.png) | Security headers live — CSP, HSTS, X-Frame-Options confirmed |
---

## Resume Bullets

- Designed and deployed hybrid SOC lab monitoring live React web application using Cloudflare WAF, Elastic SIEM, and ngrok for end-to-end threat detection
- Detected and blocked real-world attack — Poland IP (AS201814 MEVSPACE) attempting path traversal to /.git/config, mapped to MITRE ATT&CK T1190, T1083, T1036
- Implemented Cloudflare Workers to forward real-time HTTP logs to self-hosted Elastic/Logstash/Kibana stack with world map visualization
- Triaged 3 real security incidents in Jira with MITRE ATT&CK mapping and remediation tracking
- Identified 12 vulnerabilities via OWASP ZAP active scan, implemented security headers via netlify.toml reducing alerts by 33%
- Implemented and tuned Content Security Policy iteratively whitelisting legitimate services while blocking unauthorized scripts

---

## Author

Mallikarjun Rangu  
GitHub: [@rangumallikarjun](https://github.com/rangumallikarjun)
