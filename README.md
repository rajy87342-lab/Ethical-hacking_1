
# ðŸ•µï¸ Ethical Hacking Task 01: Information Gathering & Reconnaissance

**Date:** June 14, 2026
**Intern:** Akash Yadav 


**Program:** Cybersecurity Internship â€” Ethical Hacking Track

> âš ï¸ **Disclaimer:** All reconnaissance performed in this task was **passive only**, using publicly available tools and information. No systems were accessed, exploited, or modified. This task was conducted purely for educational purposes.

---

## ðŸ“Œ Task Objectives

- Understand the first phase of ethical hacking â€” Reconnaissance
- Perform passive information gathering on a public target
- Use WHOIS, DNS tools, header analysis, and public sources
- Document findings in a structured reconnaissance report

---

## âœ… Part A: Target Selection

| Field | Details |
|-------|---------|
| **Website Name** | GitHub |
| **URL** | https://github.com |
| **Reason for Selection** | GitHub is a globally recognized, publicly accessible platform. It is an ideal passive reconnaissance target as it has rich publicly available DNS, WHOIS, and header information, making it excellent for learning without any legal or ethical concerns. |

---

## âœ… Part B: WHOIS Lookup

**Tool Used:** `whois` command on Kali Linux + who.is website

| Field | Value |
|-------|-------|
| Domain Name | GITHUB.COM |
| Registrar | MarkMonitor Inc. |
| Registration Date | 2007-10-09 |
| Expiry Date | 2026-10-09 |
| Last Updated | 2024-09-07 |
| Name Servers | dns1.p08.nsone.net, ns-1283.awsdns-32.org |
| DNSSEC | Unsigned |
| Domain Status | clientDeleteProhibited, clientTransferProhibited, serverUpdateProhibited |

**Key Observations:**
- GitHub was registered in **October 2007**, consistent with its known founding date
- The domain is protected by **MarkMonitor** â€” a registrar commonly used by major enterprises to prevent domain hijacking
- Multiple status flags like `clientTransferProhibited` indicate the domain is heavily locked and cannot be accidentally transferred
- Both **NS1** and **AWS Route 53** name servers are used, indicating a redundant, enterprise-grade DNS infrastructure
- DNSSEC is not enabled â€” this is a minor observation but could theoretically allow DNS spoofing if combined with other vulnerabilities

> âœ… Screenshots included in `/screenshots/part_b/`

---

## âœ… Part C: DNS Enumeration

**Tools Used:** `dig`, `nslookup` on Kali Linux + mxtoolbox.com

### A Record (IP Address)
```
github.com    A    140.82.121.4    TTL: 60s
github.com    A    140.82.121.3    TTL: 60s
```

### MX Record (Mail Servers)
```
github.com    MX    aspmx.l.google.com       Priority: 1
github.com    MX    alt1.aspmx.l.google.com  Priority: 5
github.com    MX    alt2.aspmx.l.google.com  Priority: 5
github.com    MX    alt3.aspmx.l.google.com  Priority: 10
```

### NS Record (Name Servers)
```
github.com    NS    dns1.p08.nsone.net
github.com    NS    dns2.p08.nsone.net
github.com    NS    ns-1283.awsdns-32.org
github.com    NS    ns-1707.awsdns-21.co.uk
```

### TXT Record (SPF / Verification)
```
"v=spf1 ip4:192.30.252.0/22 include:_netblocks.google.com ~all"
"MS=ms44452932"
"docusign=087098e3-3d46-47b7-bb8e-e732b72e4c08"
```

**Key Observations:**
- The **low TTL (60 seconds)** on A records suggests GitHub uses dynamic routing or load balancing
- MX records point to **Google Workspace** â€” GitHub uses Google for its corporate email
- TXT records reveal **SPF configuration** protecting against email spoofing, and third-party service verifications (Microsoft, DocuSign)
- Multiple NS records across NS1 and AWS Route 53 confirm a **highly redundant DNS setup**

> âœ… Screenshots included in `/screenshots/part_c/`

---

## âœ… Part D: Website Technology Identification

**Tools Used:** `curl -I`, Wappalyzer browser extension, BuiltWith

| Category | Technology | Evidence |
|----------|-----------|----------|
| Web Server | GitHub.com (custom) | `server: GitHub.com` response header |
| CDN | Fastly | x-cache and x-served-by headers |
| Backend Language | Ruby on Rails | GitHub's known architecture; Turbo/Hotwire headers |
| JS Framework | React + Hotwire/Turbo | `Turbo-Visit`, `Turbo-Frame` request headers in vary |
| Protocol | HTTP/2 | `HTTP/2 200` response status |
| DNS Provider | NS1 + AWS Route 53 | NS records from dig output |
| CMS | Custom (none) | No WordPress/Drupal indicators found |

**Summary:** GitHub runs a fully custom platform with no off-the-shelf CMS. It uses Fastly as its CDN for global content delivery with very low latency, and its backend infrastructure is built on Ruby on Rails with a modern React + Hotwire frontend. The use of HTTP/2 ensures fast, multiplexed connections. From a security standpoint, the lack of a standard CMS reduces the attack surface from CMS-specific vulnerabilities.

> âœ… Screenshots included in `/screenshots/part_d/`

---

## âœ… Part E: HTTP Security Headers

**Tool Used:** `curl -I https://github.com`

| Header | Present? | Purpose |
|--------|:--------:|---------|
| Content-Security-Policy (CSP) | âœ… Yes | Controls which content sources the browser may load, preventing XSS attacks |
| X-Frame-Options | âœ… Yes | Set to `deny` â€” prevents the page from being embedded in iframes (anti-clickjacking) |
| X-Content-Type-Options | âœ… Yes | Set to `nosniff` â€” prevents MIME-type sniffing, reduces drive-by download risk |
| Strict-Transport-Security (HSTS) | âœ… Yes | Forces HTTPS for 1 year including subdomains; prevents SSL downgrade attacks |
| Referrer-Policy | âœ… Yes | Set to `no-referrer-when-downgrade` â€” protects user privacy on navigation |
| Permissions-Policy | âŒ No | Would restrict browser feature access (camera, mic, geo) â€” not present |

**Key Observations:**
GitHub implements **5 out of 6** key security headers â€” this is an excellent security posture. The only missing header is `Permissions-Policy`, which is a modern header restricting feature access. Overall, GitHub's headers are well-configured, reflecting the security standards expected of a major tech platform.

> âœ… Screenshots included in `/screenshots/part_e/`

---

## âœ… Part F: Robots.txt & Sitemap Analysis

**URL Visited:** https://github.com/robots.txt and https://github.com/sitemap.xml

**Does the website have a robots.txt file?** âœ… Yes

**Does it have a sitemap?** âœ… Yes â€” referenced inside robots.txt

### robots.txt Key Findings:
```
Disallow: /login
Disallow: /logout
Disallow: /search
Disallow: /auth
Disallow: /settings
Disallow: /*/admin
Disallow: /*/tarball/
Disallow: /*/zipball/
Disallow: /orgs/*/teams

Sitemap: https://github.com/sitemap.xml
```

### sitemap.xml Structure:
```xml
<sitemapindex>
  <sitemap><loc>https://github.com/sitemap-users.xml</loc></sitemap>
  <sitemap><loc>https://github.com/sitemap-repos.xml</loc></sitemap>
  <sitemap><loc>https://github.com/sitemap-gists.xml</loc></sitemap>
</sitemapindex>
```

**What information can be learned from these files?**
The `robots.txt` file is a gold mine for a reconnaissance analyst â€” it explicitly lists paths the organization wants to hide from search engines, which inadvertently reveals the **internal URL structure** of the application. We can see that GitHub has `/admin` paths, `/auth` endpoints, `/settings` and `/notifications` sections, and per-repository paths for tarballs and blame views. While this information is public, it tells an attacker which areas are considered sensitive. The sitemap reveals that GitHub separates its content into **users, repositories, and gists**, giving a clear picture of the platform's architecture.

> âœ… Screenshots included in `/screenshots/part_f/`

---

## âœ… Part G: Reconnaissance Report

### Target Summary

| Field | Value |
|-------|-------|
| Target | github.com |
| IP Addresses | 140.82.121.4, 140.82.121.3 |
| Registrar | MarkMonitor Inc. (registered 2007) |
| Mail Provider | Google Workspace |
| CDN | Fastly |
| Backend Stack | Ruby on Rails + React + Hotwire |
| Security Headers | 5/6 present (excellent) |
| robots.txt | Present â€” reveals internal URL structure |
| Sitemap | Present â€” 3 sub-sitemaps (users, repos, gists) |

### Overall Observations

GitHub presents a very strong security posture from a passive reconnaissance perspective. Its HTTP security headers are well-configured, with 5 out of 6 major headers in place. The use of enterprise-grade DNS providers (NS1 + AWS Route 53) and a major CDN (Fastly) demonstrates mature infrastructure planning with redundancy and performance in mind. The WHOIS data shows heavy domain locking through MarkMonitor, reducing domain hijacking risk.

### Conclusion (180 words)

Performing passive reconnaissance on GitHub revealed just how much information is publicly accessible about a target without ever making a single intrusive request. Through WHOIS alone, I identified the domain age, registrar, and expiry details, all of which paint a picture of a mature, well-maintained organization. DNS enumeration exposed the full mail routing infrastructure, pointing to Google Workspace, and confirmed a highly redundant name server setup. Analyzing HTTP headers showed that GitHub takes security seriously, implementing strict HSTS, CSP, X-Frame-Options, and more. The robots.txt file was particularly instructive â€” while its purpose is to guide web crawlers, it unintentionally maps out sensitive sections of the application architecture including admin, authentication, and settings paths. All of this data was gathered using only publicly available tools without ever touching the target system directly.

This exercise taught me that the reconnaissance phase is far more powerful than beginners typically expect. A thorough passive recon can answer nearly every fundamental question about a target's infrastructure, technologies, and exposure â€” all before a single active probe is made.

---

## ðŸ“ Repository Structure

```
Ethical_Hacking_Task_01_SohamSharma/
â”‚
â”œâ”€â”€ README.md                          â† This file
â””â”€â”€ screenshots/
    â”œâ”€â”€ part_b/                        â† WHOIS lookup (browser + terminal)
    â”œâ”€â”€ part_c/                        â† DNS enumeration (dig/nslookup outputs)
    â”œâ”€â”€ part_d/                        â† Technology identification table
    â”œâ”€â”€ part_e/                        â† HTTP security headers table
    â””â”€â”€ part_f/                        â† robots.txt and sitemap.xml views
```

---

## ðŸ“ Notes

- **Target chosen:** github.com â€” a globally public platform, fully appropriate for passive reconnaissance
- All tools used: `whois`, `nslookup`, `dig`, `curl` (all pre-installed on Kali Linux)
- Online tools used: who.is, mxtoolbox.com, BuiltWith, Wappalyzer
- No active scanning, exploitation, or unauthorized access was performed at any stage

---

## ðŸ”— Submission

Submitted via the **Sunday Submission Form** as part of the Ethical Hacking track of the Cybersecurity Internship Program.

> ðŸ’¡ **Key Takeaway:** A skilled ethical hacker never rushes straight to exploitation. Strong reconnaissance skills reveal more about a target than any scanner ever could â€” and doing it passively means zero legal risk, zero noise, and zero footprint on the target's logs.
