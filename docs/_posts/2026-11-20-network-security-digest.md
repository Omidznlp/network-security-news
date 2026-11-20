---
layout: post
title: "Network Device Security Digest - 2026-11-20"
date: 2026-11-20 15:14:04 +0000
edition: daily
critical_count: 2
item_count: 8
kev: "CVE-2026-109618, CVE-2026-109619"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-11-20

*daily edition · window 24h · 23 feeds · 8 items · 2 critical · generated 2026-11-20 15:14 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — Cisco Expressway CVE-2026-76466 exploited in the wild days after PoC release; four devices breached

Threat actors have exploited CVE-2026-76466 (CVSS 9.8), a critical flaw in Cisco Expressway, against a Russian organisation shortly after a proof-of-concept exploit was published. Reporting states four devices were breached and the attackers reached the internal network. The vulnerability allows an unauthenticated remote attacker with network access to the Expressway to compromise it.

Expressway deployments sit in the DMZ and terminate traversal and remote-access traffic from the internet, so a pre-authentication compromise is an inbound foothold at the perimeter with a path toward internal systems - exactly what the reported intrusion demonstrates. The window between PoC publication and exploitation was short, which means an unpatched, internet-facing Expressway should be treated as a live incident risk rather than a scheduled patch item.

The collected reporting does not state a fixed release for CVE-2026-76466, so confirm the fixed build against Cisco's advisory for your train before scheduling the upgrade. Treat any device that was internet-facing and unpatched as potentially compromised: patching alone does not evict an attacker who already has credentials, tokens or a modified configuration.

The same cycle brought a second large patching workload: F5 released emergency fixes for five critical BIG-IP vulnerabilities (RCE and DoS) and five critical NGINX vulnerabilities, two of them HTTP/2 flaws. Neither F5 set has reported in-the-wild exploitation yet, but the BIG-IP and NGINX install base is wide enough that these should be queued immediately behind the Expressway work.

**Do this first**

- Inventory every Cisco Expressway (Expressway-C and Expressway-E) and record the running version; check it against Cisco's fixed-release table for CVE-2026-76466 - the public reporting does not state a fixed build, so use the vendor advisory.
- Until patched, restrict inbound access to the Expressway at the upstream firewall to the source ranges that genuinely need it, and remove any administrative access (HTTPS admin, SSH, API) reachable from the internet.
- Treat unpatched internet-facing units as suspect: review admin accounts, scheduled tasks, certificates and configuration for unexpected changes, and check logs for authentication anomalies around and after the PoC release date.
- After upgrading, rotate local admin credentials, API tokens, certificates and any service-account secrets stored on the appliance, and terminate active sessions.
- Queue the F5 work next: apply the BIG-IP hotfixes across every unit in each device group and verify config sync, and patch NGINX/NGINX Plus instances where HTTP/2 is enabled.
- Ship Expressway and BIG-IP logs to the SIEM and alert on admin logins, configuration changes and firmware changes.

---

> **On the CISA KEV catalog in this edition:** CVE-2026-109618, CVE-2026-109619 — treat these as confirmed-exploited and patch on an emergency change.

## Executive summary

- Cisco Expressway CVE-2026-76466 (CVSS 9.8) is being exploited in the wild after a public PoC; four devices were breached and the attackers reached the internal network. Patch per Cisco's advisory and treat exposed units as suspect.
- F5 released emergency patches for five critical BIG-IP vulnerabilities (RCE and DoS) and five critical NGINX vulnerabilities, including two HTTP/2 flaws (CVE-2026-123007, CVE-2026-123633) fixed on 18 November 2026. No in-the-wild exploitation reported for either set.
- SonicWall Generation 6 firewalls will receive no further security patches, including for critical flaws; the vendor says to retire them and get them off the internet edge immediately.
- The OpenJDK flaw CVE-2026-114938 has produced portfolio advisories from both Cisco and Palo Alto Networks; both are rated at the top of their scales and neither collected advisory enumerates affected products yet, so track the vendor pages for product lists and fixed releases.
- Two PHP RCE flaws (CVE-2026-109618, CVE-2026-109619) are in CISA KEV; they are not network-device bugs, but gateway IPS coverage is the stopgap for web stacks you cannot patch today.
- No new defensive AI capability was announced by a network vendor in this window.

---

## Items by vendor

## F5

*3 item(s) — 1 critical, 2 high*

### Load balancers

#### 🔴 CRITICAL — F5 releases emergency patches for critical BIG-IP RCE flaws

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/f5-releases-emergency-patches-for-critical-big-ip-rce-flaws/) · 2026-11-20* 

> F5 issued emergency patches for five critical BIG-IP vulnerabilities that can be exploited for denial of service or remote code execution.

**Affected:** F5 BIG-IP. The collected report does not name the CVEs, affected branches or fixed versions - check the vendor advisory for the fixed release.  

**What happened.** F5 warned customers to patch five critical BIG-IP vulnerabilities that attackers can exploit to trigger denial of service or gain remote code execution, releasing the fixes as emergency patches. No exploitation is reported in the collected item.

**Why it matters.** BIG-IP units terminate TLS for the applications behind them and hold certificates, session state and administrative credentials, so an RCE on the appliance is a compromise of everything it fronts. A DoS path on the same device is an outage of the whole application tier. Emergency release cadence means F5 does not expect customers to wait for the next maintenance window.

**Recommended actions**

- Pull the CVE list and fixed-version table from F5's advisory and compare against the running software version of every BIG-IP unit and vCMP guest.
- Apply the hotfix to all units in each device group and verify configuration sync afterwards; upgrade the standby member first and fail over.
- Remove the management interface and TMUI from public exposure entirely, and restrict administrative access to the management network.
- Rotate device certificates, admin credentials and API tokens after patching.
- Review iRules, policies and virtual-server configuration for unauthorised changes before declaring the device clean.

**Legacy / unpatchable gear.** BIG-IP hardware or software branches past end of technical support will not receive these fixes: move the affected virtual servers to a supported unit, or isolate the legacy appliance behind a supported inspecting device with inbound restricted to required sources, management access only from a jump host, and a dated replacement plan.

**Detection.** Alert on TMUI and iControl REST access from outside the management network, on unexpected restarts or core dumps of BIG-IP daemons (a DoS attempt usually shows there first), and on configuration or iRule changes outside change windows. Ship audit logs to the SIEM.

**AI angle.** None.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/f5-releases-emergency-patches-for-critical-big-ip-rce-flaws/)**

#### 🟠 HIGH — NGINX2Root: F5 patches five critical NGINX vulnerabilities

*[Help Net Security](https://www.helpnetsecurity.com/2026/11/19/nginx2root-nginx-vulnerabilities/) · 2026-11-19* 

> F5 patched five critical NGINX flaws dubbed NGINX2Root that can reach remote code execution under certain conditions; no exploitation reported so far.

**Affected:** NGINX (open-source web server); two of the five flaws are CVE-2026-123007 and CVE-2026-123633. The collected report does not give affected or fixed version numbers - check F5's advisory for the fixed release.  
**CVEs:** [CVE-2026-123007](https://nvd.nist.gov/vuln/detail/CVE-2026-123007), [CVE-2026-123633](https://nvd.nist.gov/vuln/detail/CVE-2026-123633)  

**What happened.** F5 published fixes for five critical NGINX vulnerabilities, named NGINX2Root by Xint researchers, that can allow remote code execution under certain conditions. Two of them, CVE-2026-123633 and CVE-2026-123007, were independently reported by researchers at Rapid7, Wiz, Ikotas Labs, Palo Alto Networks and Metnir. The report states there is no evidence any of them has been exploited in the wild and none are on CISA's KEV catalog.

**Why it matters.** NGINX fronts a very large share of web applications and is also embedded in appliances and ingress layers, so the fleet to be inventoried is usually larger than the one the web team owns. Multiple independent discoveries of the same flaws means exploit knowledge is already distributed.

**Recommended actions**

- Inventory NGINX and NGINX Plus instances, including those embedded in appliances and container ingress, and record the running version against F5's fixed-release table.
- Prioritise instances that terminate internet-facing traffic and have HTTP/2 enabled; where patching has to wait, disabling HTTP/2 removes the exposure path described for the two HTTP/2 CVEs.
- Apply the vendor fix to the whole device group or ingress tier and verify configuration consistency afterwards.
- Remove any administrative or status endpoint of the proxy tier from public exposure, and restrict it to the management network.
- Record the decision (patched / mitigated / accepted risk) for each instance group in the change record.

**Legacy / unpatchable gear.** Appliances and vendor products with an embedded, unpatchable NGINX build will not get a fix on your schedule: put them behind a supported inspecting proxy or WAF, restrict inbound to required sources, and raise a vendor case asking for the patched build and an interim workaround.

**Detection.** Alert on nginx worker process crashes, segfaults and unexpected restarts in error.log - memory-corruption exploitation attempts tend to leave crashes before they succeed. Review access logs for anomalous HTTP/2 traffic and look for any process spawned by the nginx user.

**AI angle.** None.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/11/19/nginx2root-nginx-vulnerabilities/)**

#### 🟠 HIGH — Two Critical NGINX HTTP/2 Flaws: What to Patch First

*[Check Point Blog](https://blog.checkpoint.com/security/two-critical-nginx-http-2-flaws-what-to-patch-first/) · 2026-11-19* 

> Two critical NGINX HTTP/2 flaws, CVE-2026-123007 and CVE-2026-123633, allow takeover of the proxy without credentials when HTTP/2 is enabled; fixes shipped 18 November 2026.

**Affected:** NGINX with HTTP/2 enabled. Fixes were released by F5 and the NGINX project on 18 November 2026; Check Point says its own advisory carries the version tables, and that Check Point WAF as a Service was assessed as not vulnerable.  
**CVEs:** [CVE-2026-123007](https://nvd.nist.gov/vuln/detail/CVE-2026-123007), [CVE-2026-123633](https://nvd.nist.gov/vuln/detail/CVE-2026-123633)  

**What happened.** Check Point published guidance on CVE-2026-123007 and CVE-2026-123633, two critical NGINX flaws that allow takeover of the web server by an unauthenticated attacker when HTTP/2 is enabled, with no involvement of application code. F5 and the NGINX project released fixes on 18 November 2026. Check Point's R&D assessed its WAF as a Service as not vulnerable and published an advisory with version tables, mitigations and hunting guidance.

**Why it matters.** These are the two flaws from the NGINX2Root set with the clearest exploitation condition - HTTP/2 enabled, no authentication, no dependency on the application - which makes them the ones to patch first and the ones with a usable interim mitigation. HTTP/2 is on by default in many ingress and proxy deployments.

**Recommended actions**

- Identify which NGINX instances have HTTP/2 enabled on internet-facing listeners; those are the ones to patch first.
- Apply the fixed version from F5's and the NGINX project's tables (released 18 November 2026) - verify against the vendor table rather than assuming your distribution package is current.
- Where a maintenance window is not available, disable HTTP/2 on exposed listeners as the interim mitigation and record it as a temporary change to be reverted after patching.
- If you front NGINX with a WAF, confirm with your vendor whether coverage exists and in which mode, and do not treat that as a substitute for the upgrade.
- Re-scan your public ranges afterwards to confirm no exposed listener is still on a vulnerable build.

**Legacy / unpatchable gear.** Embedded NGINX inside appliances and unsupported distributions cannot be patched by you: disable HTTP/2 at that tier if the configuration is reachable, otherwise front the device with a supported proxy or WAF, deny inbound from untrusted zones, and open a vendor case for the patched build.

**Detection.** Review access and error logs for anomalous HTTP/2 stream patterns and for worker process crashes or restarts, and use the hunting guidance in the vendor advisories. Alert on any process spawned by the nginx service account and on new outbound connections from proxy hosts.

**AI angle.** None.

📄 **[Read the full report at Check Point Blog →](https://blog.checkpoint.com/security/two-critical-nginx-http-2-flaws-what-to-patch-first/)**

---

## Cisco

*2 item(s) — 1 critical, 1 high*

### VPN gateways

#### 🔴 CRITICAL — Hackers Exploit Critical Cisco Expressway Flaw, Breach Four Devices and Reach Internal Network

*[The Hacker News](https://thehackernews.com/2026/11/hackers-exploit-critical-cisco.html) · 2026-11-20* 

> Attackers exploited Cisco Expressway flaw CVE-2026-76466 (CVSS 9.8) shortly after a PoC appeared, breaching four devices and reaching the internal network.

**Affected:** Cisco Expressway. CVE-2026-76466, CVSS 9.8, exploitable by an unauthenticated remote attacker with network access to the Expressway. The collected report does not state a fixed release - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-76466](https://nvd.nist.gov/vuln/detail/CVE-2026-76466)  

**What happened.** Threat actors were observed exploiting CVE-2026-76466 in Cisco Expressway to breach a Russian organisation shortly after public release of a proof-of-concept exploit. Four devices were breached and the attackers reached the internal network.

**Why it matters.** Confirmed exploitation of a pre-authentication flaw on a DMZ device that exists to carry traffic from the internet to internal services. The PoC-to-exploitation interval was days, so every unpatched internet-facing Expressway is exposed now rather than at some future point, and the reported outcome - internal network access - is the worst case for this device class.

**Recommended actions**

- Locate every Expressway-C and Expressway-E, record running versions, and check them against Cisco's fixed-release table for CVE-2026-76466; schedule the upgrade immediately rather than in the next window.
- Before patching, limit inbound access at the upstream firewall to the source ranges that require traversal, and confirm no administrative service (HTTPS admin, SSH, API) is reachable from the internet.
- Treat each exposed unpatched unit as potentially compromised: review admin accounts, certificates, scheduled tasks and configuration diffs, and look for signs of pivoting from the DMZ into internal subnets.
- After upgrade, rotate admin credentials, API tokens, certificates and service-account secrets on the appliance and terminate all active sessions.
- Verify the DMZ segmentation the attackers in this case got past: Expressway should not be able to reach internal management networks, and its outbound flows should be restricted to the specific internal services it proxies.

**Legacy / unpatchable gear.** Expressway versions past end of software maintenance will not get a fixed build: take the device off the public IP, terminate external traffic on a supported appliance instead, or isolate it behind a supported inspecting firewall with strict inbound ACLs, management only from a jump host with session logging, and a dated replacement.

**Detection.** Hunt for unauthenticated requests to Expressway web endpoints from unexpected ASNs around and after the PoC release, new or modified local admin accounts, and outbound connections from the Expressway to internal hosts it does not normally reach. Alert on configuration changes and on any administrative session not originating from the management network.

**AI angle.** None.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/11/hackers-exploit-critical-cisco.html)**

### Other

#### 🟠 HIGH — Vulnerability in Oracle OpenJDK (Java4Shell) Affecting Cisco Products: November 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nov2026-java-rce-eCa6ss9z) · 2026-11-19* `vendor-advisory`

> Cisco opened a portfolio advisory for the OpenJDK flaw CVE-2026-114938 (Java4Shell), an RCE triggered when processing Base64 content; affected products are not yet listed.

**Affected:** Cisco products that bundle OpenJDK. The underlying flaw affects OpenJDK Java 17.0.6 and later on x86-64 platforms when processing Base64 encoded content. The advisory is marked as pending update and does not enumerate affected Cisco products or fixed releases in this text - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-114938](https://nvd.nist.gov/vuln/detail/CVE-2026-114938)  

**What happened.** Oracle announced fixes on 17 November 2026 for CVE-2026-114938, an OpenJDK vulnerability that can lead to remote code execution. Cisco published advisory cisco-sa-nov2026-java-rce-eCa6ss9z with a Security Impact Rating of High, pointing to the Oracle Critical Patch Update advisory, the OpenJDK advisory of 2026/11/17 and a Talos threat advisory, and says it will be updated as more information becomes available.

**Why it matters.** Portfolio advisories of this shape typically resolve into a long product list over the following days, and Java components inside network management and collaboration products are often reachable from the management network. Starting the inventory before the product list lands shortens the eventual patch cycle.

**Recommended actions**

- Subscribe to or poll the advisory URL and recheck daily until the affected-product table is published; do not conclude you are unaffected from the current empty list.
- Build the candidate list now from your asset inventory: Cisco management, collaboration and monitoring products rather than switch and router NOS images, which are where bundled Java usually sits.
- Restrict management-plane access to those products to a dedicated management VLAN or out-of-band network, with ACLs permitting only administrative sources.
- When the fixed releases are published, verify running versions against the table and schedule upgrades, HA secondary first.
- Ship management-platform logs to the SIEM and alert on admin logins and configuration changes in the meantime.

**Legacy / unpatchable gear.** Cisco products past end of software maintenance will not receive a Java fix: isolate them behind a supported inspecting firewall, allow management only from a dedicated jump host with full session logging, and set a replacement date with a named budget owner.

**Detection.** Alert on the Java process on affected appliances spawning child processes or making outbound connections, and on unexpected administrative sessions to management platforms. Watch for Talos signature coverage for CVE-2026-114938 in IPS logs.

**AI angle.** None.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nov2026-java-rce-eCa6ss9z)**

---

## Palo Alto Networks

*1 item(s) — 1 high*

### Other

#### 🟠 HIGH — CVE-2026-114938 Portfolio Status: Impact of Java Vulnerability (CVE-2026-114938) on Palo Alto Networks Products (Severity: CRITICAL)

*[Palo Alto Networks PSIRT](https://security.paloaltonetworks.com/CVE-2026-114938) · 2026-11-19* `vendor-advisory`

> Palo Alto Networks opened a critical-severity portfolio status page for the Java flaw CVE-2026-114938, with product impact still being determined.

**Affected:** Palo Alto Networks products, pending the portfolio assessment. The collected PSIRT entry carries no body text, so no affected products, versions or fixed releases are stated - check the vendor advisory for the product list and fixed release.  
**CVEs:** [CVE-2026-114938](https://nvd.nist.gov/vuln/detail/CVE-2026-114938)  

**What happened.** Palo Alto Networks PSIRT published a portfolio status page for CVE-2026-114938, the OpenJDK vulnerability also covered by Cisco's advisory in this cycle, with a severity of CRITICAL. The collected entry contains only the title and severity.

**Why it matters.** A critical-rated portfolio page from a firewall vendor for a Java RCE usually resolves toward management and cloud-delivered components rather than PAN-OS data plane, but until the page is populated the exposure is unknown. Two major network vendors opening advisories for the same OpenJDK flaw in one day indicates a broad bundled-component problem worth tracking actively.

**Recommended actions**

- Poll the PSIRT page for CVE-2026-114938 daily until the affected-product list and fixed releases are published; the current empty entry is not an all-clear.
- Pre-build the candidate inventory: Panorama and other management or cloud-delivered components first, with firewall platforms and versions recorded for comparison against the eventual table.
- Restrict management access to Panorama and related platforms to a dedicated management VLAN or out-of-band network, and enforce MFA on all administrative accounts.
- When fixed releases land, verify versions against the vendor table and schedule the upgrade - management platform before managed firewalls.
- Record the decision for each product group in the change record once the assessment is published.

**Legacy / unpatchable gear.** Products on PAN-OS releases past end of life will not receive a fix for this flaw: isolate their management access behind a jump host with full session logging, deny management reachability from user and untrusted zones, and set a dated upgrade to a supported release.

**Detection.** Alert on administrative logins and configuration pushes from Panorama or other management platforms outside expected hours and sources, and on outbound connections initiated by the management platform itself. Correlate with IPS coverage for CVE-2026-114938 once signatures are available.

**AI angle.** None.

📄 **[Read the full report at Palo Alto Networks PSIRT →](https://security.paloaltonetworks.com/CVE-2026-114938)**

---

## SonicWall

*1 item(s) — 1 high*

### Firewalls

#### 🟠 HIGH — SonicWall Warns Gen 6 Firewalls Won't Get Security Patches, Even for Critical Flaws

*[The Hacker News](https://thehackernews.com/2026/11/sonicwall-warns-gen-6-firewalls-wont.html) · 2026-11-20* `AI`

> SonicWall says Generation 6 firewalls will get no security patches even for critical flaws, and tells customers to retire them and pull them off the internet edge now.

**Affected:** SonicWall Generation 6 firewalls, all internet-facing deployments in particular. No specific CVE is involved.  

**What happened.** SonicWall urged all customers still running Generation 6 firewalls to retire and replace them immediately and to stop exposing the devices at the internet edge, citing security risk from AI-assisted attacks. The vendor said attackers now use automation and AI to find and exploit unsupported edge devices faster than ever, and that it expects internet-facing Gen 6 firewalls to be actively targeted.

**Why it matters.** This is the vendor withdrawing the one control that justified keeping the hardware - the prospect of a patch for a critical flaw. An internet-facing Gen 6 unit is now a permanently unpatched perimeter device, and SonicWall is explicitly saying it expects targeting.

**Recommended actions**

- Enumerate every Gen 6 chassis by model and serial from the inventory and management console, and flag each one that holds a public IP or terminates VPN.
- This week, remove Gen 6 units from the internet edge: move the public IP and VPN termination to a supported appliance, or place the Gen 6 behind a supported inspecting firewall with inbound denied from untrusted zones.
- Remove management access (HTTPS/SSH/API/SNMP) from all untrusted interfaces and restrict it to a dedicated management VLAN reachable only from a jump host.
- Rotate local admin credentials, VPN pre-shared keys, certificates and API keys on every Gen 6 device, and enforce MFA on remaining administrative and VPN accounts.
- Export and review each configuration for unexpected admin users, port forwards, static routes, tunnels or modified login pages before you trust the device for another day.
- Raise the replacement as a funded project with a date and a budget owner, and record the accepted risk for every unit that stays in service past it.

**Legacy / unpatchable gear.** This item is entirely about unpatchable gear: assume no fix will ever arrive for Gen 6, treat each remaining unit as a high-risk asset behind a supported firewall, keep a NetFlow baseline and alert on any new outbound flow, and document the accepted risk with an owner and an end date rather than leaving it implicit.

**Detection.** Alert on administrative logins to Gen 6 devices from outside the management network, on configuration changes outside change windows, and on any new outbound flow from the appliance itself. Scan your own public ranges to confirm no Gen 6 management or SSL-VPN port is still answering.

**AI angle.** SonicWall's stated reason is AI and automation shortening the time between a device becoming unsupported and being found and exploited - the vendor's own argument that the risk-acceptance window for end-of-life edge gear is now days, not budget cycles. It is a threat assertion, not a defensive product.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/11/sonicwall-warns-gen-6-firewalls-wont.html)**

---

## Check Point

*1 item(s) — 1 medium*

### Firewalls

#### 🟡 MEDIUM — New PHP Vulnerabilities: What Organizations Need to Know and How Check Point Customers Are Protected

*[Check Point Blog](https://blog.checkpoint.com/security/new-php-vulnerabilities-what-organizations-need-to-know-and-how-check-point-customers-are-protected/) · 2026-11-19* `KEV`

> Check Point describes gateway coverage for three PHP flaws, two of which (CVE-2026-109618, CVE-2026-109619) are unauthenticated RCE and now in CISA KEV.

**Affected:** PHP installations, not network devices themselves. CVE-2026-109618 (CVSS 4.0 score 9.2, in CISA KEV), CVE-2026-109619 (in CISA KEV) and CVE-2026-115560, which the post says involves a more specific PHP configuration and should be considered separately from the two original "PHP2Shell" vulnerabilities.  
**CVEs:** [CVE-2026-109618](https://nvd.nist.gov/vuln/detail/CVE-2026-109618) **[KEV]**, [CVE-2026-109619](https://nvd.nist.gov/vuln/detail/CVE-2026-109619) **[KEV]**, [CVE-2026-115560](https://nvd.nist.gov/vuln/detail/CVE-2026-115560)  

**What happened.** On 19 November 2026 Check Point published guidance on recently disclosed PHP vulnerabilities and the protections it says cover its customers. CVE-2026-109618 and CVE-2026-109619 are critical unauthenticated remote code execution flaws; CISA has added both to the Known Exploited Vulnerabilities catalog, with a stated due date of 29 October 2026.

**Why it matters.** The bug is in a web technology rather than a network device, so it belongs to the application owners - but the network team usually owns the only control that can be deployed within hours. KEV listing means the clock is a compliance clock as well as a risk one for federal and contract-bound estates.

**Recommended actions**

- Confirm IPS/threat-prevention protections for these CVEs are enabled in prevent mode (not detect) on gateways in front of PHP-based applications, and that the signature package is current.
- Give the application owners the KEV entries and the CISA due date, and ask for the PHP version inventory; do not substitute gateway coverage for patching.
- Restrict inbound paths to PHP applications to the proxy or gateway tier only, so no host is reachable around the inspection point.
- Check CVE-2026-115560 separately against your PHP configuration rather than assuming the same exposure as the two PHP2Shell flaws.
- Record which application groups are protected by virtual patching versus actually upgraded, with a date for the real fix.

**Legacy / unpatchable gear.** For PHP applications on unsupportable platforms that cannot be upgraded, the gateway is the only control: enforce strict inbound ACLs, keep IPS in prevent mode, isolate the host in its own segment with no lateral access to management or database networks, and set a decommission or rebuild date.

**Detection.** Alert on IPS prevent/detect hits for these CVE signatures at the gateway, and correlate with outbound connections initiated by web server hosts - a PHP RCE usually shows up as the web user making an unexpected egress connection or spawning a shell.

**AI angle.** None.

📄 **[Read the full report at Check Point Blog →](https://blog.checkpoint.com/security/new-php-vulnerabilities-what-organizations-need-to-know-and-how-check-point-customers-are-protected/)**

---

## 🧠 AI & network devices — SonicWall cites AI-assisted attacks as the reason Gen 6 firewalls must come off the edge now

SonicWall told customers to retire and replace Generation 6 firewalls immediately and to stop exposing them at the internet edge, saying that "attackers now use automation and AI to find and exploit unsupported edge devices faster than ever" and that it expects internet-facing Gen 6 firewalls to be actively targeted. The vendor is making an exposure argument rather than disclosing a specific vulnerability: Gen 6 will not receive security patches, even for critical flaws.

For network teams the practical content is a timeline change. If discovery and exploitation of unsupported edge devices are accelerating, the risk-acceptance window for end-of-life gear on a public IP shrinks to the time it takes to put a supported inspecting device in front of it - not the next budget cycle. The same logic applies to any unsupported edge appliance, not only SonicWall's.

**Vendors with an AI angle in this edition:** Check Point, Cisco, F5, Palo Alto Networks, SonicWall

**Preventing it on current systems**

- Shorten the patch SLA for internet-facing network devices to days: assume exploit development and mass scanning now move faster than your historical change cadence.
- Treat any AI/LLM integration on network devices (assistants, AIOps, MCP or agent connectors) as a privileged admin path: scoped read-only credentials, no unattended config write, full audit trail.
- Test AI-facing management surfaces for prompt injection from attacker-controlled data (device logs, hostnames, ticket text) before letting an agent act on them.
- Baseline NetFlow and syslog and alert on deviation, rather than relying on signature updates alone to catch fast-moving edge exploitation.
- Require callback verification for any out-of-band request to change firewall or VPN configuration, to blunt AI-enabled social engineering of the network team.

**Preventing it on legacy / end-of-life systems**

- Unsupported edge devices such as SonicWall Gen 6 will not get a fix: take them off the internet edge and place them behind a supported inspecting firewall, denying inbound access from untrusted zones.
- Apply compensating controls now - strict management ACLs, no direct management access, a dedicated jump host and full session logging.
- Set a replacement date and a named budget owner, and document the accepted risk until the hardware is gone.
- Monitor remaining legacy units as high-risk assets: NetFlow baseline and alerting on any new outbound flow.

---

<!--index
{
 "url": "/2026/11/20/network-security-digest/",
 "date": "2026-11-20",
 "entries": [
  {
   "title": "F5 releases emergency patches for critical BIG-IP RCE flaws",
   "link": "https://www.bleepingcomputer.com/news/security/f5-releases-emergency-patches-for-critical-big-ip-rce-flaws/",
   "source": "BleepingComputer",
   "relevance": "critical",
   "device_types": [
    "load-balancer"
   ],
   "vendors": [
    "F5"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "None.",
   "summary": "F5 issued emergency patches for five critical BIG-IP vulnerabilities that can be exploited for denial of service or remote code execution.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Hackers Exploit Critical Cisco Expressway Flaw, Breach Four Devices and Reach Internal Network",
   "link": "https://thehackernews.com/2026/11/hackers-exploit-critical-cisco.html",
   "source": "The Hacker News",
   "relevance": "critical",
   "device_types": [
    "vpn-gateway",
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [
    "CVE-2026-76466"
   ],
   "kev": false,
   "ai": true,
   "ai_angle": "None.",
   "summary": "Attackers exploited Cisco Expressway flaw CVE-2026-76466 (CVSS 9.8) shortly after a PoC appeared, breaching four devices and reaching the internal network.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "NGINX2Root: F5 patches five critical NGINX vulnerabilities",
   "link": "https://www.helpnetsecurity.com/2026/11/19/nginx2root-nginx-vulnerabilities/",
   "source": "Help Net Security",
   "relevance": "high",
   "device_types": [
    "load-balancer",
    "other"
   ],
   "vendors": [
    "F5"
   ],
   "cves": [
    "CVE-2026-123007",
    "CVE-2026-123633"
   ],
   "kev": false,
   "ai": true,
   "ai_angle": "None.",
   "summary": "F5 patched five critical NGINX flaws dubbed NGINX2Root that can reach remote code execution under certain conditions; no exploitation reported so far.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Vulnerability in Oracle OpenJDK (Java4Shell) Affecting Cisco Products: November 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nov2026-java-rce-eCa6ss9z",
   "source": "Cisco PSIRT",
   "relevance": "high",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [
    "CVE-2026-114938"
   ],
   "kev": false,
   "ai": true,
   "ai_angle": "None.",
   "summary": "Cisco opened a portfolio advisory for the OpenJDK flaw CVE-2026-114938 (Java4Shell), an RCE triggered when processing Base64 content; affected products are not yet listed.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "SonicWall Warns Gen 6 Firewalls Won't Get Security Patches, Even for Critical Flaws",
   "link": "https://thehackernews.com/2026/11/sonicwall-warns-gen-6-firewalls-wont.html",
   "source": "The Hacker News",
   "relevance": "high",
   "device_types": [
    "firewall"
   ],
   "vendors": [
    "SonicWall"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "SonicWall's stated reason is AI and automation shortening the time between a device becoming unsupported and being found and exploited - the vendor's own argument that the risk-acceptance window for end-of-life edge gear is now days, not budget cycles. It is a threat assertion, not a defensive product.",
   "summary": "SonicWall says Generation 6 firewalls will get no security patches even for critical flaws, and tells customers to retire them and pull them off the internet edge now.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Two Critical NGINX HTTP/2 Flaws: What to Patch First",
   "link": "https://blog.checkpoint.com/security/two-critical-nginx-http-2-flaws-what-to-patch-first/",
   "source": "Check Point Blog",
   "relevance": "high",
   "device_types": [
    "load-balancer",
    "other"
   ],
   "vendors": [
    "F5"
   ],
   "cves": [
    "CVE-2026-123007",
    "CVE-2026-123633"
   ],
   "kev": false,
   "ai": true,
   "ai_angle": "None.",
   "summary": "Two critical NGINX HTTP/2 flaws, CVE-2026-123007 and CVE-2026-123633, allow takeover of the proxy without credentials when HTTP/2 is enabled; fixes shipped 18 November 2026.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "CVE-2026-114938 Portfolio Status: Impact of Java Vulnerability (CVE-2026-114938) on Palo Alto Networks Products (Severity: CRITICAL)",
   "link": "https://security.paloaltonetworks.com/CVE-2026-114938",
   "source": "Palo Alto Networks PSIRT",
   "relevance": "high",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Palo Alto Networks"
   ],
   "cves": [
    "CVE-2026-114938"
   ],
   "kev": false,
   "ai": true,
   "ai_angle": "None.",
   "summary": "Palo Alto Networks opened a critical-severity portfolio status page for the Java flaw CVE-2026-114938, with product impact still being determined.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "New PHP Vulnerabilities: What Organizations Need to Know and How Check Point Customers Are Protected",
   "link": "https://blog.checkpoint.com/security/new-php-vulnerabilities-what-organizations-need-to-know-and-how-check-point-customers-are-protected/",
   "source": "Check Point Blog",
   "relevance": "medium",
   "device_types": [
    "firewall"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [
    "CVE-2026-109618",
    "CVE-2026-109619",
    "CVE-2026-115560"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None.",
   "summary": "Check Point describes gateway coverage for three PHP flaws, two of which (CVE-2026-109618, CVE-2026-109619) are unauthenticated RCE and now in CISA KEV.",
   "ai_defense": false,
   "defense": null
  }
 ]
}
-->

## How this was produced

- Feeds polled: 23 ok, 0 failed
- Raw items: 739 → in window: 58 → network-device relevant: 11 → published: 8
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
