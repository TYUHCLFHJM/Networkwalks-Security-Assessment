# Network Reconnaissance & Local Network Discovery

**Networkwalks Cybersecurity Internship · Week 2 | Technical Assessment Portfolio**

![Status](https://img.shields.io/badge/Status-Completed-2ea44f) ![Platform](https://img.shields.io/badge/Platform-Kali%20Linux%20%7C%20Windows-2563eb) ![Scope](https://img.shields.io/badge/Scope-Reconnaissance%20%26%20Host%20Discovery-64748b)

> **Assessment type:** Public-information reconnaissance and authorized local-network host discovery. This project documents observed configurations, **not** a demonstrated compromise or full penetration test.

## Executive summary

This project documents two Week 2 practical modules completed during the Networkwalks cybersecurity internship:

1. **W2-PM1 — Footprinting & Reconnaissance:** Six tools used to gather publicly exposed domain, DNS, website technology, HTTP-header and web application firewall information for `networkwalks.com`.
2. **W2-PM5 — Network Scanning with Zenmap:** An Nmap ping discovery scan of an authorized local Wi-Fi subnet (`192.168.1.0/24`) using the Windows Zenmap GUI, with IP/MAC observations and topology export.

The domain resolved to **192.232.216.135** during testing; HTTP responses identified **Apache**, WhatWeb fingerprinted **WordPress** and related technologies, and Wafw00f reported **ModSecurity (SpiderLabs)**. The latest documented local discovery scan reported **7 active hosts out of 256 addresses checked**. A separate earlier Zenmap view/topology contained **9 host entries**; the counts are retained separately because they represent different observations.

**Key conclusion:** The evidence supports technology and network visibility findings only. No exploitation, authenticated review, vulnerability validation, or proof of security impact was performed.

## Contents

- [Scope and ethics](#scope-and-ethics)
- [Environment and methodology](#environment-and-methodology)
- [Module 1 — Reconnaissance](#module-1--reconnaissance)
- [Module 5 — Local network discovery](#module-5--local-network-discovery)
- [Security observations and recommendations](#security-observations-and-recommendations)
- [Limitations](#limitations)
- [Evidence index](#evidence-index)

## Scope and ethics

| Item | Scope |
|---|---|
| Public website | `networkwalks.com` — public WHOIS/DNS, basic web fingerprinting and header inspection |
| Local network | Personal/authorized Wi-Fi subnet `192.168.1.0/24` — host discovery only |
| Techniques | Non-exploitative reconnaissance; `nmap -sn` host discovery |
| Excluded | Exploitation, credential testing, brute force, intrusive vulnerability scanning, persistence, data extraction |

All results are point-in-time observations. Network ownership and authorization should be confirmed before repeating scans on another environment.

## Environment and methodology

| Component | Environment |
|---|---|
| Reconnaissance | Kali Linux in Oracle VirtualBox |
| Local network discovery | Windows host running Zenmap / Nmap 7.993 (as shown in screenshot) |
| Tools | WHOIS, WhatWeb, Nslookup, Curl, Wafw00f, DNSRecon, Zenmap/Nmap |
| Evidence date | November 2026 (timestamps visible in screenshots) |
| Local Wi-Fi IP | `192.168.1.10` |
| Local subnet / gateway | `192.168.1.0/24` / `192.168.1.1` |

Workflow: run each command → review output → preserve a screenshot and (where applicable) text output → summarize only observable findings.

## Module 1 — Reconnaissance

### 1. WHOIS — Domain registration

```bash
whois networkwalks.com
whois networkwalks.com > ~/Desktop/Task1_WHOIS.txt
```

| Attribute | Observed result |
|---|---|
| Registrar | GoDaddy.com, LLC |
| Creation | 2019-11-06 |
| Last updated | 2025-11-12 |
| Registry expiry | 2027-11-06 |
| Name servers | `ns6135.hostgator.com`, `ns6136.hostgator.com` |
| DNSSEC | Unsigned in WHOIS output |

**Interpretation:** Public registration data identifies the registrar and delegated DNS providers. An unsigned DNSSEC indication is not, by itself, evidence of an exploitable weakness.

![WHOIS evidence](evidence/01-whois-output.png)

### 2. WhatWeb — Website fingerprinting

```bash
whatweb networkwalks.com
whatweb https://networkwalks.com
whatweb https://networkwalks.com > ~/Desktop/Task2_WhatWeb.txt
```

| Attribute | Observed result |
|---|---|
| HTTP behavior | `301 Moved Permanently` to HTTPS |
| HTTPS behavior | `200 OK` |
| Server | Apache |
| IP | `192.232.216.135` |
| CMS fingerprint | WordPress `7.1.4` (tool-reported; unverified) |
| Plugin fingerprint | WP Download Manager `3.3.76` (tool-reported) |
| JavaScript | jQuery `3.7.1` |
| Other indicators | Bootstrap, Google Tag Manager, HTML5, `_wpdm_client` cookie |

**Interpretation:** Technology fingerprints are useful for an asset inventory but may be inaccurate or influenced by caching, themes or detection heuristics. No CVE or vulnerable configuration is established from these results alone.

![WhatWeb HTTPS evidence](evidence/04-whatweb-https.png)

### 3. Nslookup — DNS resolution

```bash
nslookup networkwalks.com
nslookup networkwalks.com > ~/Desktop/Task3_Nslookup.txt
```

- Resolver queried: `8.8.8.8` (Google Public DNS).
- Resolved IPv4 address: `192.232.216.135`.
- Response type: non-authoritative answer from the recursive resolver.

![Nslookup evidence](evidence/05-nslookup.png)

### 4. Curl — HTTP response headers

```bash
curl -I https://networkwalks.com
curl -I https://networkwalks.com > ~/Desktop/Task4_Curl.txt
```

Observed: `HTTP/2 200`, `server: Apache`, `content-type: text/html; charset=UTF-8`, WordPress API links (`/wp-json/`), and a `_wpdm_client` cookie marked `Secure` and `HttpOnly`. The response also included `referrer-policy: no-referrer-when-downgrade` and a permissions policy.

**Interpretation:** `Secure` and `HttpOnly` are positive cookie safeguards. The referrer policy could be reviewed against the organization's privacy requirements; its presence does not establish a vulnerability.

![Curl header evidence](evidence/06-curl-headers.png)

### 5. Wafw00f — WAF identification

```bash
wafw00f https://networkwalks.com
wafw00f https://networkwalks.com > ~/Desktop/Task5_Wafw00f.txt
```

The tool reported **ModSecurity (SpiderLabs)** as the detected WAF after **2 requests**. This is a fingerprinting result, not a test of the WAF's effectiveness or coverage.

![Wafw00f evidence](evidence/07-wafw00f.png)

### 6. DNSRecon — DNS record enumeration

```bash
dnsrecon -d networkwalks.com -t std
dnsrecon -d networkwalks.com -t std > ~/Desktop/Task6_DNSRecon.txt
```

| Record / feature | Observed result |
|---|---|
| A | `networkwalks.com` → `192.232.216.135` |
| NS | `ns6135.hostgator.com`, `ns6136.hostgator.com` |
| MX | `mail.networkwalks.com` |
| TXT | SPF policy and Google site-verification token |
| SRV | Autodiscover service records associated with cPanel email discovery |
| DNS software | BIND `9.16.23-RH` (tool-reported fingerprint) |
| DNSSEC | No answer to DNSSEC query |
| Tool summary | `8 Records Found` (DNSRecon's own count) |

**Interpretation:** DNS records disclose public service dependencies and email-routing configuration. A missing DNSSEC answer alone does not prove a security failure. The BIND version is a reported fingerprint and was not independently verified.

![DNSRecon evidence](evidence/08-dnsrecon-output.png)

## Module 5 — Local network discovery

### Network configuration

Windows `ipconfig` showed:

| Property | Value |
|---|---|
| Active adapter | Wi-Fi |
| Host IP | `192.168.1.10` |
| Subnet mask | `255.255.255.0` |
| Gateway | `192.168.1.1` |
| Target CIDR | `192.168.1.0/24` |

The VMware VMnet interfaces were excluded from this Wi-Fi discovery exercise.

![Windows ipconfig evidence](evidence/10-windows-ipconfig.png)

### Host discovery

Zenmap target: `192.168.1.0/24`  
Nmap command:

```bash
nmap -sn 192.168.1.0/24
```

**Latest completed scan:** `256 IP addresses (7 hosts up) scanned in 4.58 seconds`.

| IP | MAC (where reported) | Reported vendor / notes |
|---|---|---|
| `192.168.1.1` | `C0:91:20:E2:EF:B8` | Sichuan Tianyi; gateway address |
| `192.168.1.4` | `04:B9:E3:61:D2:9C` | Samsung Electronics |
| `192.168.1.8` | `1E:60:C3:F6:52:23` | Unknown |
| `192.168.1.9` | `84:A1:B7:D3:63:32` | Honor Device |
| `192.168.1.10` | Not reported | Scanning Windows host |
| `192.168.1.11` | `12:AF:F1:86:76:BA` | Unknown |
| `192.168.1.12` | `0A:84:23:8C:76:E6` | Unknown |

**Important reconciliation:** A different earlier screenshot and exported topology displayed **nine** entries, including `192.168.1.2` and `192.168.1.14`. The later scan explicitly reports **seven** hosts. These are not combined into a single scan result; hosts may change availability between scans, and Zenmap can retain previously discovered hosts in its host list. The seven-host count is authoritative **for the latest displayed scan only**.

![Latest completed Zenmap scan](evidence/13-zenmap-seven-host-scan.png)

### Network topology

Zenmap's topology view visualizes discovered hosts relative to the scanning machine. Because this was a ping discovery scan, **dotted connections and positions are not evidence of physical cabling, exact routing, or network segmentation**.

![Zenmap topology](evidence/12-zenmap-topology.png)

**[Open the exported network topology PDF](evidence/network-topology.pdf)**

## Security observations and recommendations

| Observation | Evidence-based interpretation | Recommended follow-up |
|---|---|---|
| Public CMS/plugin fingerprints | Versions were reported by WhatWeb, not validated as vulnerable | Verify installed versions internally and apply vendor-supported updates |
| Public DNS and email records | Expected for functioning DNS/email infrastructure | Periodically audit DNS records and SPF/DKIM/DMARC configuration with authorized access |
| WAF fingerprint | ModSecurity detected; protection efficacy not tested | Review WAF rules, monitoring, and alerting with the site administrator |
| HTTP headers and cookie attributes | `Secure`/`HttpOnly` present; referrer policy observed | Review overall HTTP security-header baseline in a separate authorized assessment |
| Multiple active local hosts | Seven active hosts in the latest scan; device ownership not confirmed | Compare against the router's authorized device inventory and investigate unfamiliar entries |

**Risk classification:** *Informational / requires validation*. No confirmed exploitable vulnerabilities, privilege escalation, sensitive-data exposure, or compromise were demonstrated. Assigning Critical/High findings or CVSS scores would be unsupported.

## Limitations

- Passive/public-information and low-impact discovery activities only; no exploit testing.
- Domain data and host availability can change over time.
- Tool fingerprints, manufacturer names and DNS version strings are not independently verified.
- Ping discovery does not enumerate open ports, vulnerabilities, or all possible hosts.
- The topology PDF and latest scan show different host counts; they are separate observations.
- The screenshots contain private LAN addresses and MAC identifiers. Review/redact before public publication if the network is not solely yours.

## Evidence index

| Evidence | File |
|---|---|
| WHOIS output and save command | [01](evidence/01-whois-output.png) · [02](evidence/02-whois-save-command.png) |
| WhatWeb HTTP and HTTPS | [03](evidence/03-whatweb-http.png) · [04](evidence/04-whatweb-https.png) |
| Nslookup | [05](evidence/05-nslookup.png) |
| Curl | [06](evidence/06-curl-headers.png) |
| Wafw00f | [07](evidence/07-wafw00f.png) |
| DNSRecon | [08](evidence/08-dnsrecon-output.png) · [09](evidence/09-dnsrecon-saved.png) |
| Windows IP configuration | [10](evidence/10-windows-ipconfig.png) |
| Initial Zenmap host discovery | [11](evidence/11-zenmap-initial-discovery.png) |
| Topology image and PDF | [12](evidence/12-zenmap-topology.png) · [PDF](evidence/network-topology.pdf) |
| Latest seven-host scan | [13](evidence/13-zenmap-seven-host-scan.png) |

---

*Prepared as a technical learning portfolio for the Networkwalks Cybersecurity Internship — Week 2. Results are limited to the supplied screenshots and exported topology.*
