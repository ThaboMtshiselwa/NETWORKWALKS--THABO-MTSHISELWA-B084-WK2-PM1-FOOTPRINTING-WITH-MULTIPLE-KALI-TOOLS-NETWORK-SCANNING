# NETWORKWALKS--THABO-MTSHISELWA-B084-WK2-PM1-FOOTPRINTING-WITH-MULTIPLE-KALI-TOOLS-NETWORK-SCANNING
# 🛡️ W2-PM-B084 — Penetration Testing Report

**Footprinting with Multiple Kali Tools & Network Scanning**

| Pentester (Cybersecurity Proffessional) | Thabo Mtshiselwa |
|---|---|
| Program / Batch | B084 — Networkwalks |
| Date | 14 November 2026 |
| Modules | W2-PM1 (Multiple Kali Tools) · W2-PM5 (Zenmap Scanning) |
| Phases | Phase 1: Reconnaissance & Footprinting · Phase 2: Scanning & Network Discovery |
| Authorization | Written permission secured — see report disclaimer |

---
## 1. Liability Disclaimer

>I have performed these activities only on the systems & devices where I had secured written permission or the devices/systems that I own myself.

>All these materials are for education and research purpose only. Do not use anything from here to break the law. 

>The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. 

>Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries unauthorised access is a crime even when nothing is damaged.

---

## 2. Introduction

- This report covers footprinting the networkwalks.com domain using multiple Kali Linux tools as part of my (W2-PM1) and scanning my own local network with Zenmap (W2-PM5).
- The footprinting phase and the other covers the scanning phase, so together they show how an attacker moves from gathering public information to mapping live hosts on a network. It is the Week 2 part of my ongoing internship program at Networkwalks.
- All commands were run in Kali Linux (footprinting) and on a Windows PC with Zenmap installed (scanning). 
- Every step below includes the exact command used, the result I observed, a screenshot as evidence, and a short note on why the finding matters from an attacker's point of view.

---

## 3.📌 Overview

This repository contains the Week 2 penetration testing report covering:

- **Footprinting** of `networkwalks.com` using six Kali Linux tools
- **Network scanning** of a local LAN using Zenmap (Nmap GUI)
- **Risk analysis**, **recommendations**, and **evidence screenshots**

> 🚨 **Ethical Use Only** — All activities were performed with written permission
> or on systems owned by the author. This material is for education and research.
> Do not use any of it to break the law.

---

## 🧰 Tools Used

| Tool | Purpose |
|---|---|
| WHOIS | Domain registration details (registrar, name servers, dates) |
| WhatWeb | Web technology fingerprinting (WordPress, plugins, IP) |
| nslookup | DNS resolution (domain → IP) |
| curl -I | HTTP response header inspection |
| wafw00f | Web Application Firewall detection |
| dnsrecon | DNS record enumeration (NS, MX, SPF, TXT, SRV) |
| Zenmap (Nmap GUI) | Local subnet host discovery & topology |
| Windows CMD | Local IP / MAC identification (`ipconfig /all`) |

---

## 📋 Key Findings

| # | Finding | Evidence | Risk |
|---|---|---|---|
| 1 | Web technology/version exposed | WhatWeb → WordPress 7.1.4, WP Download Manager 3.3.76 | Medium |
| 2 | Server IP identifiable | nslookup → 192.232.216.135 | Low |
| 3 | HTTP technical info exposed | curl -I → `/wp-json/`, X-Redirect-By, X-XNC-Engine | Low |
| 4 | WAF technology identifiable | wafw00f → ModSecurity (SpiderLabs) | Low |
| 5 | DNS infrastructure exposed | dnsrecon → BIND 9.16.23-RH, mail/SPF/TXT/SRV records | Medium |
| 6 | Live hosts visible on LAN | Zenmap → 1 live host (192.168.56.1) | Medium |

> These are **observations**, not confirmed vulnerabilities. No exploitation was performed.

---

## 4. Activities Performed
## 4.1 Footprinting & Reconnaissance
I performed reconnaissance against the networkwalks.com domain using six Kali Linux tools: **WHOIS**, **WhatWeb**, **Nslookup**, **Curl -I**, **Wafw00f** and **DNSRecon**. Each tool was used to collect a different type of information about the target.

## 4.1.1. WHOIS – Public domain registration record
First, I used WHOIS to obtain publicly available domain registration information and identify the domain's name servers.

**Command:** $ whois networkwalks.com

**Result:** The output revealed the domain registration details including the registrar **(GoDaddy.com, LLC)**, creation date (2019-11-06), update date (2025-11-12), name servers **(NS6135.HOSTGATOR.COM and NS6136.HOSTGATOR.COM)**, and registrant contact information. The domain is registered through a privacy protection service (Domains By Proxy, LLC) with contact details masked for privacy. The web server is hosted on Apache.

**Why it matters:** WHOIS provides valuable information about the domain ownership, registration timeline, and hosting infrastructure. The name servers reveal that the site is hosted on HostGator. The creation date (2019) indicates an established domain, and the privacy protection suggests the owner wants to keep registration details confidential.

---

## 4.1.2. WhatWeb – Technology fingerprinting
I then used WhatWeb to identify technologies used by the website.

**Command:** $ whatweb networkwalks.com

**Result:** WhatWeb identified **WordPress 7.1.4** and **WP Download Manager 3.3.76**, along with other technologies including Apache, Bootstrap 7.1.4, jQuery 3.7.1, and various HTTP headers. The tool also detected the site uses cookies (wpdm_client), email addresses (info@networkwalks.com), and the IP address **192.232.216.135**.

**Why it matters:** Exposing exact software and versions allows an attacker to search for known vulnerabilities in those specific versions. WordPress 7.1.4 and WP Download Manager 3.3.76 are specific versions that could be checked against vulnerability databases.

---

## 4.1.3. Nslookup – DNS resolution
Using **Nslookup**, I resolved the domain name to its IP address.

**Command:** $ nslookup networkwalks.com

**Result:** The output showed:
- Server: 10.0.0.1 (Google DNS)
- Non-authoritative answer: networkwalks.com resolves to **192.232.216.135**

**Why it matters:** Knowing the IP address lets an attacker scan the server directly, look up other sites on the same IP, and map the target's infrastructure. This is the first step in active reconnaissance.

---

## 4.1.4. Curl -I – HTTP response headers
I used **Curl** with the -I option to inspect the HTTP response headers.

**Command:** $ curl -I https://networkwalks.com

**Result:** The response headers revealed:
- HTTP/1.1 200 (successful response)
- Server: Apache (web server technology)
- X-Redirect-By: WordPress (confirms WordPress CMS)
- **X-XNC-Engine:** WordPress (additional WordPress confirmation)
- **Set-Cookie: wpdm_client=...** (cookie for WP Download Manager)
- **Link headers** pointing to the WordPress REST API at /wp-json/ endpoint
- The response also exposed caching headers and security policies

**Why it matters:** HTTP headers leak the web server type (Apache), the CMS (WordPress), and hidden endpoints (the WordPress REST API at /wp-json/). This information helps an attacker understand the application architecture and potential attack surfaces.

---

## 4.1.5. Wafw00f – Web Application Firewall detection

Next, I used **Wafw00f** to determine whether a Web Application Firewall was protecting the website.

**Command:** $ wafw00f networkwalks.com

**Result:** The tool identified that networkwalks.com is behind ModSecurity (SpiderLabs) WAF. The output showed that ModSecurity is actively blocking requests and the site has a WAF deployed.

**Why it matters:** Knowing a WAF (ModSecurity) is present shapes the whole attack strategy. ModSecurity is a well-known open-source WAF that can block common attack patterns. An attacker would need to find bypass techniques or focus on other vulnerabilities. The presence of a WAF also indicates the site owners are security conscious.

---

## 4.1.6. DNSRecon – DNS record enumeration

Finally, I used **DNSRecon** to enumerate DNS records.

**Command:** $ dnsrecon -d networkwalks.com

**Result:** The tool enumerated:
- **Name Servers (NS):** ns6135.hostgator.com, ns61356.hostgator.com
- **Mail Servers (MX):** mail.networkwalks.com (192.232.216.135), with SPF and TXT records
- **SPF Record:** v=spf1 a mx include:websitewelcome.com ~all
- **TXT Records:** Google site verification, cPanel records
- **SRV Records:** cpanel, autodiscover, autoconfig, cpcontacts, webdisk, whm, etc.
- **DNS Software Version:** BIND 9.16.23-RH

**Why it matters:** DNSRecon maps the target's entire DNS footprint. Each record is a potential foothold. The mail server (mail.networkwalks.com) is a potential target for email-based attacks. The SPF policy reveals which servers are authorized to send email on behalf of the domain. The BIND 9.16.23 version is a specific DNS server software version that could be checked for vulnerabilities. The cPanel service records reveal the hosting management interface.

---

## 4.2 Network Scanning with Zenmap

For the second activity, I used **Zenmap** to perform network discovery on my local network. The practical required me to identify my local IP address and subnet, discover live hosts, identify their IP and MAC addresses, and generate a network topology.

## Step 1: Find local IP address and LAN subnet

Before moving forward, I downloaded and installed Zenmap from official website https://nmap.org/download.html on my Windows PC. (You can also find other OS like MacOS, Linux RPM and other source code distribution operating systems.

I first used the **Windows ipconfig** command to identify my local IP address and LAN subnet.

**Command:** C:\> ipconfig

**Result:** The output showed:
- IPv4 Address: 192.168.56.1
- Subnet Mask: 255.255.255.0
- Default Gateway: (blank)
  
**Why it matters:** Knowing your local IP and subnet is the starting point for any network scan. The subnet (192.168.56.0/24) defines the range of IP addresses to scan.

---

## Step 2: Ping Scan with Zenmap

I then opened Zenmap and entered the local subnet (192.168.56.0/24) and selected Ping Scan to identify active hosts.

**Command in Zenmap:** nmap -sn 192.168.56.0/24
**Result:** The scan identified 1 live host:
- 192.168.56.1 (my own PC)
- MAC Address: 98:59:7A:DB:28:97
  
The scan took 4.25 seconds and completed successfully.

**Why it matters:** This confirms which devices are alive on the network. In this case, only my own PC was found, which is expected for a minimal lab setup. In a real network, this would reveal all active devices.

---

## Step 3: Topology generation
After completing the scan, I opened the **Topology** section in Zenmap, enabled the **legend**, and saved the network topology in **PDF format** as required by the practical task.

**Result:** The topology showed:
- **DESKTOP-E06SS4E (192.168.8.159)** – my desktop PC
- **192.168.56.1** – the localhost/loopback interface
- The topology legend explained the symbols for hosts, routers, firewalls, and traceable connections
  
**Why it matters:** The topology provides a visual representation of the network. It helps identify the relationship between hosts and can be used in the final report to document the network layout.

---

## 5. Risk Analysis / Impact
Based on the information collected during the footprinting and network scanning activities, I identified the following potential risks.

## Risk level key: Critical • Medium • Low
The risks above are observations from the footprinting and scanning exercises, not confirmed vulnerabilities.

The practical exercises primarily involved information gathering and host discovery. No exploitation or vulnerability validation was performed as part of these two modules.

Therefore, the presence of information such as a software version, IP address or DNS record does not by itself mean that the system is vulnerable. Further authorized security testing would be required to confirm any actual vulnerability.

---

## 6. Recommendations
Based on the observations from these activities I’ve learned, I recommend the following security improvements:

1. Review publicly exposed technology information: Organizations should regularly review what information about their web technologies, CMS and plugins is publicly visible. Consider using a security plugin to hide WordPress version information.
2. Keep software updated: CMS platforms, plugins and other web technologies should be regularly updated and reviewed against current security advisories. WordPress 7.1.4 and WP Download Manager 3.3.76 should be checked for known vulnerabilities.
3. Review HTTP headers: HTTP response headers should be reviewed to determine whether unnecessary technical information is being exposed. Consider removing or obfuscating the X-Redirect-By and X-XNC-Engine headers.
4. Review DNS records regularly: DNS records should be checked periodically to ensure that only required information and services are publicly exposed. The cPanel service records (autodiscover, autoconfig, etc.) should be reviewed to ensure they are necessary.
5. Properly configure and monitor the WAF: Keep the WAF (ModSecurity) enabled and tuned, since it already blocks naive attacks. Regular log reviews and rule updates are recommended.
6. Perform regular internal network discovery: Organizations should periodically scan their own networks to identify active devices. This helps detect unauthorized devices.
7. Investigate unknown devices: Any unexpected device discovered during network scanning should be investigated and verified.
8. Maintain network documentation: Network topology and device information should be documented and updated regularly.
9. Perform security testing with authorization: Reconnaissance and scanning should only be performed against systems and networks where appropriate authorization has been provided.

---

## 7. Conclusion
During Week 2 of my Cybersecurity & Ethical Hacking internship, I completed practical activities covering footprinting, reconnaissance and network scanning.

In the footprinting activity, I used six Kali Linux tools to collect information about the target domain. I learned how **WHOIS** can provide domain information (registrar, creation date, name servers), **WhatWeb** can identify web technologies (WordPress 7.1.4 and WP Download Manager 3.3.76), **Nslookup** can resolve domain names (192.232.216.135), **Curl** can inspect HTTP headers (exposing /wp-json/), **Wafw00f** can identify a WAF (ModSecurity/SpiderLabs), and **DNSRecon** can provide additional DNS information (BIND 9.16.23, mail servers, SPF records).

In the network scanning activity, I used **Zenmap** to identify my local network configuration (192.168.56.1/24) and discover active hosts. I also collected IP and MAC address information (98:59:7A:DB:28:97) and created a network topology.

The exercises showed me that information gathering is an important part of cybersecurity. Even before attempting to exploit a system, a security professional can learn a significant amount about an environment by carefully analyzing publicly available information and network responses.

I also learned that technical findings should be documented clearly. A good cybersecurity report should explain what was performed, what was discovered, what the observation means, what risk it may create, and what can be done to reduce that risk.

Finally, I learned that reconnaissance and scanning must always be performed within an authorized scope. These activities were completed as part of the assigned educational cybersecurity lab.

## 8. Evidence Collected
1.	WHOIS Output - Domain Registration Details
   <img width="940" height="564" alt="image" src="https://github.com/user-attachments/assets/9d87a722-6455-4d12-9b7b-a701b0444a4a" />
   
2.	WHOIS Output - Continued Registration Details
   <img width="940" height="559" alt="image" src="https://github.com/user-attachments/assets/a4c8f722-2118-46c5-b80c-55bae1a4d6f5" />
   
3.	WHOIS Output - Registrant and Name Server Details
   <img width="940" height="561" alt="image" src="https://github.com/user-attachments/assets/ab187375-af91-4530-890a-86cad4973f16" />
   
4.	WhatWeb Output - Technology Fingerprinting
   <img width="940" height="449" alt="image" src="https://github.com/user-attachments/assets/66362996-8b32-4563-b426-347bb5fca5ed" />
   
5.	Nslookup Output - Domain to IP Resolution
   <img width="940" height="449" alt="image" src="https://github.com/user-attachments/assets/eb7bdb95-e09c-42ec-b56d-f8c8f3279cca" />
   
6.	Curl Output - HTTP Response Headers
   <img width="940" height="448" alt="image" src="https://github.com/user-attachments/assets/38d30d87-dd74-4bf3-9129-cb2b14b572a7" />
   
7.	Wafw00f Output - WAF Detection
   <img width="940" height="448" alt="image" src="https://github.com/user-attachments/assets/bb2a4d78-1e9a-46e2-9aa7-e31de29bd855" />
   
8.	DNSRecon Output - DNS Record Enumeration
   <img width="940" height="445" alt="image" src="https://github.com/user-attachments/assets/c6a457d0-77b8-4c1e-865a-2c04b0bd69f4" />
   
9.	Windows ipconfig Output - Local IP and Subnet
    <img width="940" height="497" alt="image" src="https://github.com/user-attachments/assets/2f5b7fc0-1305-46de-b6a0-92bc4c557b7d" />
    
10.	Zenmap Ping Scan Results - Live Hosts
    <img width="940" height="497" alt="image" src="https://github.com/user-attachments/assets/3949c628-3bc1-4314-9e3f-2f62a27842f5" />
    
11.	Zenmap Topology View - Network Map
    <img width="940" height="498" alt="image" src="https://github.com/user-attachments/assets/c93935b3-f3f0-4bc2-9dcb-b902ea6ca35c" />
    
12.	Zenmap Topology View
    [Network Topology Scanning.pdf](https://github.com/user-attachments/files/34852033/Network.Topology.Scanning.pdf)

---

-End-
**Author:** Thabo Mtshiselwa

**Program:** Cybersecurity program at Networkwalks | Week: 02 | Cybersecurity Professional B084

**Date:** 14 November 2026













