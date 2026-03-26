# ☠ WelchSec C2 Domain Monitor

A free, browser-based threat intelligence tool for monitoring newly registered and known malicious domains for Command & Control (C2) infrastructure indicators. Helps security teams, SOC analysts, and threat hunters identify emerging C2 domains before they are used in attacks against their organization.

**Live Tool → [welchsec.github.io/C2-Monitor](https://welchsec.github.io/C2-Monitor/)**

---

## ⚠️ Legal & Ethical Disclaimer

> This tool is for **defensive security research and threat intelligence only**.
> Use it to protect your organization by identifying domains that may target you or your sector.
> Do not use this tool to interact with, attack, or disrupt live infrastructure.
> All data sources used are publicly available threat intelligence feeds.

---

## 🧰 What It Does

The tool has three modes accessible from the top navigation bar:

---

### 📡 Mode 1 — Live Feed Monitor

Pulls domains from four free, publicly available threat intelligence feeds and scores each one for C2 indicators using a weighted scoring engine.

**Data Sources:**

| Source | What It Provides | URL |
|--------|-----------------|-----|
| ThreatFox (abuse.ch) | Today's C2 IOC domains | threatfox.abuse.ch |
| URLhaus (abuse.ch) | Active malware hosting domains | urlhaus.abuse.ch |
| C2-Tracker (GitHub) | Community-tracked C2 domains | github.com/montysecurity/C2-Tracker |
| OpenPhish | Live phishing domains | openphish.com |

**Features:**
- Configurable minimum risk score threshold (show all, suspicious only, high risk, critical only)
- Adjustable domain limit (100, 250, or 500 domains per fetch)
- Live progress bar and running stats
- Filter results by CRITICAL / HIGH / MEDIUM risk level
- CSV export of all flagged domains

---

### 🔎 Mode 2 — Check a Domain

Paste any suspicious domain for a full C2 risk analysis. Combines client-side scoring with server-side DNS and WHOIS checks for a complete picture.

**Checks performed:**

| Check | What It Detects |
|-------|----------------|
| DGA Scoring | Domain Generation Algorithm patterns — high entropy, consonant ratio, random character sequences |
| TLD Classification | 20+ TLDs commonly abused for C2 (.xyz, .top, .tk, .gq, .ml etc.) |
| Brand Impersonation | Domain contains major brand names (Microsoft, Google, PayPal, Amazon etc.) in a non-official context |
| Entropy Analysis | Statistical randomness of the domain name |
| C2 Keywords | Words like cdn, gate, beacon, loader, payload, relay, tunnel in domain names |
| Hyphen/Number Patterns | Excessive hyphens and digits common in C2 naming |
| DNS Analysis | Missing MX records (throwaway domain), low TTL (fast-flux detection) |
| Domain Age | Registration date via RDAP — freshly registered domains score higher |
| Subdomain Depth | Deeply nested subdomains used to make C2 URLs look legitimate |

**Output:**
- Risk score 0–100 with color-coded risk level
- Full breakdown of every indicator triggered
- DNS record table
- Plain-English verdict and recommended action

---

### ☠ Mode 3 — Known C2 Feeds

Pulls live IOC data directly from three abuse.ch public threat intelligence feeds. All data is current — pulled at time of request.

**Feeds:**

| Feed | What It Contains |
|------|-----------------|
| Feodo Tracker | Active botnet C2 IP addresses (Emotet, QakBot, TrickBot etc.) |
| URLhaus | Currently active malware hosting URLs and domains |
| ThreatFox | Today's C2 IOCs across multiple malware families |

**Features:**
- Searchable by domain, IP, or malware family name
- Filterable by type (domains vs IP addresses)
- Live IOC count by source
- Ready to copy into firewall rules, DNS blocklists, or SIEM

---

## 📊 C2 Risk Scoring System

Every domain is scored 0–100 based on weighted indicators:

| Indicator | Score Impact |
|-----------|-------------|
| Known C2 TLD (.xyz, .top, .tk, .gq, .ml etc.) | +35 |
| DGA pattern detected | +30 |
| Brand impersonation | +25 |
| High entropy domain name | +20 |
| Excessive hyphens or numbers | +15 |
| No MX records (throwaway domain) | +15 |
| Domain registered < 7 days ago | +10 |
| C2-related keywords in name | +10 |
| Legitimate TLD (.com, .org, .net) | -10 |

| Score Range | Risk Level | Recommended Action |
|-------------|-----------|-------------------|
| 70–100 | CRITICAL | Block immediately, investigate all internal connections |
| 50–69 | HIGH | Add to watchlist, alert on internal DNS queries |
| 30–49 | MEDIUM | Monitor, investigate if seen in network logs |
| 0–29 | LOW | Low risk, no action required |

---

## 🚀 Setup Guide

### Step 1 — Deploy the Cloudflare Worker

The tool requires a Cloudflare Worker to fetch from external threat intelligence feeds. This is necessary because browsers block direct cross-origin requests from static GitHub Pages sites.

1. Go to [workers.cloudflare.com](https://workers.cloudflare.com) and sign up — no credit card needed

2. Click **Workers & Pages → Create Application → Create Worker**

3. Name it something like `welchsec-proxy` and click **Deploy**

4. Click **Edit Code**, select all the default code and delete it

5. Copy the contents of `cloudflare-worker.js` from the [WelchSec Recon Tool repo](https://github.com/welchsec/Recon-Tool) and paste it in

6. Click **Deploy**

7. Copy your Worker URL:
   ```
   https://welchsec-proxy.yourname.workers.dev
   ```

> **Already have a Worker from another WelchSec tool?**
> The same worker handles everything — just paste your existing Worker URL. No changes needed.

---

### Step 2 — Use the Tool

1. Open [welchsec.github.io/C2-Monitor](https://welchsec.github.io/C2-Monitor/)
2. Paste your **Cloudflare Worker URL** into the Worker URL field
3. Select a mode from the top navigation bar
4. Configure options and click the action button

No API keys are required for any mode.

---

## 💡 Usage Tips

### Live Feed Monitor
- Run this daily as part of your morning threat intelligence routine
- Set the minimum score to **50+** to focus on genuinely suspicious domains
- Export to CSV and compare against your DNS logs to check if any internal hosts have resolved flagged domains
- CRITICAL-scored domains from ThreatFox and URLhaus are confirmed active malware infrastructure — consider blocking immediately at your DNS resolver

### Check a Domain
- Use this when you receive a suspicious email, see an unknown domain in your firewall logs, or want to quickly assess a URL before clicking
- Provide your Worker URL for the most complete analysis — it enables DNS and domain age checks which significantly improve accuracy
- A LOW score does not guarantee a domain is safe — sophisticated C2 can avoid all automated detection

### Known C2 Feeds
- Use the search box to quickly check if a specific domain or IP appears in known C2 lists
- Export relevant IOCs and add them to your firewall blocklist or SIEM detection rules
- Check regularly — abuse.ch feeds update continuously as new malware infrastructure is identified

### Integrating with Your Security Stack
Findings from this tool can be used to:
- Create DNS blocklist entries in Pi-hole, Unbound, or corporate DNS resolvers
- Add Suricata/Snort rules for network-level detection
- Create SIEM alerts for internal hosts resolving flagged domains
- Populate threat intelligence platforms like MISP or OpenCTI

---

## 💰 Cost

| Component | Cost |
|-----------|------|
| GitHub Pages hosting | Free |
| Cloudflare Worker | Free (100k requests/day) |
| ThreatFox API | Free, no key required |
| URLhaus API | Free, no key required |
| C2-Tracker feed | Free, public GitHub repo |
| OpenPhish feed | Free |
| Feodo Tracker | Free, no key required |

---

## 🔧 Troubleshooting

**Live Feed Monitor returns no results**
Check that your Cloudflare Worker URL is correct and the worker has been deployed with the latest `cloudflare-worker.js`. Open browser DevTools (F12) → Console for specific error messages.

**Domain check shows no DNS data**
DNS and WHOIS lookups require the Worker URL to be filled in. The client-side scoring (entropy, TLD, DGA) still runs without it.

**Known C2 feeds return errors**
The abuse.ch APIs are occasionally rate-limited or temporarily unavailable. Try again after a few minutes.

**GitHub Pages shows README instead of the tool**
The HTML file must be named exactly `index.html`. Rename it in your GitHub repo if needed.

---

## 🔗 Related Tools

| Tool | Description | Link |
|------|-------------|------|
| Recon Suite | 9-module recon toolkit for bug bounty | [welchsec.github.io/Recon-Tool](https://welchsec.github.io/Recon-Tool/) |
| Phishing Analyzer | URL analysis for phishing indicators | [welchsec.github.io/Phishing-Analyzer](https://welchsec.github.io/Phishing-Analyzer/) |
| OSINT Dashboard | Domain intelligence aggregator | [welchsec.github.io/OSINT-Dashboard](https://welchsec.github.io/OSINT-Dashboard/) |
| Intrusion Report | DFIR-style reports from live threat intel | [welchsec.github.io/Intrusion-Report](https://welchsec.github.io/Intrusion-Report/) |
| TTP Report | AI-powered MITRE ATT&CK threat intelligence | [welchsec.github.io/TTP-Threat-Report](https://welchsec.github.io/TTP-Threat-Report/) |

All tools share the same Cloudflare Worker — one deployment handles everything.

---

## 📬 Contact

Built by [@welchsec](https://github.com/welchsec)
