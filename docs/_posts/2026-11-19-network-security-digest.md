---
layout: post
title: "Network Device Security Digest - 2026-11-19"
date: 2026-11-19 15:35:48 +0000
edition: daily
critical_count: 2
item_count: 10
kev: "CVE-2026-109618, CVE-2026-109619"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-11-19

*daily edition · window 24h · 23 feeds · 10 items · 2 critical · generated 2026-11-19 15:35 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — PAN-OS management interfaces exploited through the PHP2Shell pair, both now in KEV

The two PHP vulnerabilities tracked as CVE-2026-109618 and CVE-2026-109619 (reported as PHP2Shell) are being used against firewalls. Palo Alto Networks confirmed on October 28 that attackers were exploiting both against PAN-OS management interfaces exposed to the internet. Both flaws allow code execution on the web server with no authentication, and both are in CISA's Known Exploited Vulnerabilities catalog - added 2026-10-26 with a 2026-10-29 due date, catalogued against PHP Group's PHP rather than under a network vendor's name, which is why they are easy to miss in a vendor-filtered patch process.

This is the management plane of a perimeter firewall reached pre-authentication, so it outranks everything else in this cycle. A foothold there yields the running configuration, the credential and certificate material stored on the device, VPN pre-shared keys, and a trusted position for pivoting inward. The underlying point from the reporting is that a firewall's administrative surface is a web application like any other, and it inherits the vulnerabilities of its web stack.

The source does not state a fixed PAN-OS release for this issue, so check the vendor advisory for the fixed release rather than assuming a build. The control that matters first is exposure: a management interface reachable from the internet is the precondition for this attack, and removing that reachability is faster than any upgrade window.

Treat any internet-exposed PAN-OS management interface as potentially compromised prior to remediation, not merely as vulnerable. Credential rotation after patching is not optional here - a pre-patch foothold survives the upgrade.

**Do this first**

- Enumerate every PAN-OS device whose management interface (HTTPS/SSH/API, including Panorama) answers from the internet, and remove that exposure now - management on a dedicated management VLAN or out-of-band network only.
- Where exposure cannot be removed immediately, restrict the management interface with permitted-IP lists to administrator source ranges and the jump hosts only.
- Check the Palo Alto Networks advisory for the fixed release covering CVE-2026-109618 and CVE-2026-109619 and schedule the upgrade in the change window, HA pair secondary first.
- After patching, rotate local admin credentials, API keys, device certificates and VPN pre-shared keys; terminate active admin and VPN sessions.
- Export the configuration and review it for unexpected admin users, modified login pages, new scripts, static routes or tunnels.
- Search KEV by CVE, not by vendor name - these entries are filed under PHP Group and will not surface in a Palo Alto Networks-only query.

---

> **On the CISA KEV catalog in this edition:** CVE-2026-109618, CVE-2026-109619 — treat these as confirmed-exploited and patch on an emergency change.

## Executive summary

- Firewalls are the live target: CVE-2026-109618 and CVE-2026-109619 (PHP2Shell) are in KEV and were confirmed by Palo Alto Networks on October 28 as being exploited against internet-exposed PAN-OS management interfaces.
- F5 has two separate problems in one cycle - five NGINX2Root remote code execution flaws with public proof-of-concept exploits, and six critical BIG-IP flaws that let an unauthenticated attacker crash the Traffic Management Microkernel on virtual servers with specific iRule or APM configuration.
- Cisco Expressway is being exploited in the wild shortly after a proof of concept went public, against a clustered deployment, with tooling staged for lateral movement.
- Java4Shell (CVE-2026-114938 in OpenJDK) has moved to active exploit attempts; Cisco has opened an advisory for affected Cisco products but has not yet published the product list, so this needs tracking rather than a one-time read.
- SonicWall Gen 6 and Gen 6.5 on SonicOS 6.x is out of support with no further patches regardless of severity - that fleet is now a replacement project, not a patching one.
- No fixed release numbers are quoted in this edition where the sources did not state them; confirm every build against the vendor advisory before opening a change.

---

## Items by vendor

## F5

*2 item(s) — 1 critical, 1 high*

### Load balancers

#### 🔴 CRITICAL — F5 Patches Critical NGINX2Root Flaws as PoC Exploits Go Public

*[SecurityWeek](https://www.securityweek.com/f5-patches-critical-nginx2root-flaws-as-poc-exploits-go-public/) · 2026-11-19* 

> F5 patched five NGINX2Root flaws that let remote unauthenticated attackers run code, with proof-of-concept exploits now public.

**Affected:** NGINX, in the set of five vulnerabilities reported as NGINX2Root. The source does not state the affected or fixed version numbers, or which NGINX builds and F5 NGINX products are in scope.  

**What happened.** F5 released patches for five NGINX2Root vulnerabilities that could allow remote, unauthenticated attackers to execute arbitrary code. Proof-of-concept exploit code is public.

**Why it matters.** Public proof-of-concept plus unauthenticated remote code execution on a reverse proxy is the shortest path from scan to shell, and these services are internet-facing by design. NGINX frequently terminates TLS in front of applications, so code execution there exposes certificate material and the traffic it fronts, and it is often deployed outside the network team's formal device inventory.

**Recommended actions**

- Check the F5 advisory for the fixed releases and the exact affected versions for NGINX2Root; do not assume a build number from the reporting.
- Inventory every NGINX and F5 NGINX instance, including ones deployed as containers or inside application stacks, and record the running version - this is where incomplete inventories bite.
- Patch internet-facing instances first, then internal ones; verify configuration after upgrade.
- Rotate TLS private keys, admin credentials and API tokens on any instance that was internet-exposed before patching.
- Review proxy configuration for unauthorized changes - added upstreams, new locations, altered access rules.
- Keep management and status endpoints off the internet and restricted by ACL to the management network.

**Legacy / unpatchable gear.** Unsupported or abandoned NGINX deployments with no patch owner should be fronted by a supported inspecting proxy or WAF and denied direct inbound access, with a decommission or migration date recorded.

**Detection.** Reverse proxy and WAF logs showing requests matching the published proof-of-concept patterns; unexpected child processes or outbound connections from the NGINX worker user; new files in web-served directories; GreyNoise-style scanner traffic hitting proxy endpoints.

**AI angle.** None stated.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/f5-patches-critical-nginx2root-flaws-as-poc-exploits-go-public/)**

#### 🟠 HIGH — F5 Warns 6 Critical BIG-IP Flaws Can Crash Traffic in Specific Setups, May Allow RCE

*[The Hacker News](https://thehackernews.com/2026/11/f5-warns-6-critical-big-ip-flaws-can.html) · 2026-11-18* 

> Six critical F5 BIG-IP flaws let an unauthenticated attacker crash the Traffic Management Microkernel and drop traffic on virtual servers with specific iRule or APM config.

**Affected:** F5 BIG-IP. Exploitation requires a specific configuration on a virtual server: an iRule that uses a particular command, or an APM policy that includes two particular kinds of agent. The source does not state the CVE identifiers, the affected version ranges or the fixed releases.  

**What happened.** F5 disclosed six critical BIG-IP vulnerabilities that allow an attacker to crash the Traffic Management Microkernel (TMM) without authenticating, disrupting traffic until TMM restarts. Each requires one of the specific virtual server configurations described above. The reporting also notes the flaws may allow remote code execution.

**Why it matters.** TMM is the data plane: when it goes down, every application behind that BIG-IP goes down with it, and no authentication is needed to trigger it. The saving grace is that it only applies to virtual servers configured in specific ways, so this is a configuration audit before it is a patching exercise - but APM policies and iRules are common enough that 'we have BIG-IP' is not an answer either way. The possibility of code execution means a crash should not be assumed to be the worst case.

**Recommended actions**

- Read the F5 advisories to identify the exact iRule command and the two APM agent types named, then audit virtual server configuration across all units to find which ones match.
- Check the vendor advisory for the fixed releases; no fixed BIG-IP versions are stated in the source.
- Apply the F5 fix to all units in the device group and verify configuration sync after upgrade.
- For matching virtual servers that cannot be patched immediately, evaluate removing or replacing the implicated iRule command or APM agent as an interim measure, after validating application impact.
- Remove the management/TMUI interface from public exposure entirely and restrict it to the management network.
- Confirm HA failover and monitoring will actually detect and survive a TMM restart, and alert on TMM crash events rather than on application symptoms.

**Legacy / unpatchable gear.** BIG-IP units past end of technical support will not receive these fixes; remove the implicated iRule or APM configuration from them, place them behind a device that can absorb the traffic, and schedule migration to a supported platform.

**Detection.** TMM restart and core-dump events in /var/log/ltm, traffic-group failovers with no corresponding change activity, and short repeated outages on specific virtual servers; alert on tmm crash signatures forwarded to the SIEM.

**AI angle.** None stated.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/11/f5-warns-6-critical-big-ip-flaws-can.html)**

---

## Palo Alto Networks

*1 item(s) — 1 critical*

### Firewalls

#### 🔴 CRITICAL — Everything Is a Web Application, Including Your Firewall

*[The Hacker News](https://thehackernews.com/2026/11/everything-is-web-application-including.html) · 2026-11-19* `KEV`

> Palo Alto Networks confirmed attackers exploiting two unauthenticated PHP code-execution flaws against internet-exposed PAN-OS management interfaces.

**Affected:** PAN-OS management interfaces exposed to the internet, via CVE-2026-109618 and CVE-2026-109619 in PHP. The source does not state affected or fixed PAN-OS versions.  
**CVEs:** [CVE-2026-109618](https://nvd.nist.gov/vuln/detail/CVE-2026-109618) **[KEV]**, [CVE-2026-109619](https://nvd.nist.gov/vuln/detail/CVE-2026-109619) **[KEV]**  

**What happened.** One of the early targets of PHP2Shell exploitation was the firewall. On October 28 Palo Alto Networks confirmed that attackers were exploiting CVE-2026-109618 and CVE-2026-109619 against internet-exposed PAN-OS management interfaces. Both flaws allow code execution on the web server without authentication. Both are in CISA's KEV catalogue, added 2026-10-26 with a 2026-10-29 due date, listed against PHP Group's PHP.

**Why it matters.** This is pre-authentication code execution on the administrative plane of a perimeter firewall. It yields the configuration, stored credentials and certificates, VPN pre-shared keys and a trusted pivot point into the internal network. Because KEV files the CVEs under PHP rather than under Palo Alto Networks, a vendor-name-driven patch process can miss them entirely.

**Recommended actions**

- Identify every PAN-OS device and Panorama instance with a management interface reachable from the internet and remove that reachability; management belongs on a dedicated management VLAN or out-of-band network.
- Apply permitted-IP restrictions to the management interface for administrator source ranges and jump hosts where exposure cannot be removed immediately.
- Check the Palo Alto Networks advisory for the fixed release covering CVE-2026-109618 and CVE-2026-109619 and schedule the upgrade in the change window, HA pair secondary first.
- Rotate local admin credentials, API keys, certificates and VPN pre-shared keys after patching - a pre-patch compromise survives the upgrade.
- Enforce MFA on all administrative and VPN accounts and remove shared local admin accounts.
- Export and review the configuration for unexpected admin users, scripts, static routes, port forwards or modified login pages.

**Legacy / unpatchable gear.** Appliances past end of support will not receive a PAN-OS fix for this. Take them off the internet edge entirely, place them behind a supported inspecting firewall with inbound denied from untrusted zones, allow management only from a dedicated jump host with full session logging, and set a replacement date with a budget owner.

**Detection.** Web server access logs on the management interface showing unauthenticated POSTs to PHP endpoints; new or unexpected processes, cron entries, files or admin accounts on the device; outbound connections from the management interface to unfamiliar destinations; configuration commits outside change windows.

**AI angle.** None stated.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/11/everything-is-web-application-including.html)**

---

## Cisco

*3 item(s) — 2 high, 1 watch*

### Other

#### 🟠 HIGH — Vulnerability in Oracle OpenJDK (Java4Shell) Affecting Cisco Products: November 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nov2026-java-rce-eCa6ss9z) · 2026-11-18* `vendor-advisory`

> Cisco is tracking CVE-2026-114938 in OpenJDK (Java4Shell), an RCE in Java 17.0.6 and later on x86-64 that affects an as-yet unlisted set of Cisco products.

**Affected:** OpenJDK Java 17.0.6 and later on x86-64 platforms when processing Base64 encoded content, as embedded in Cisco products. Cisco's advisory states it will be updated as additional information becomes available and does not yet enumerate affected products or fixed releases.  
**CVEs:** [CVE-2026-114938](https://nvd.nist.gov/vuln/detail/CVE-2026-114938)  

**What happened.** Oracle announced fixes on November 17, 2026 for CVE-2026-114938, an OpenJDK vulnerability that can lead to remote code execution, triggered on x86-64 when Base64 encoded content is processed. Cisco opened an advisory for Cisco products affected by it, rated Security Impact High, and points to the Oracle Critical Patch Update advisory, the OpenJDK advisory and a Talos threat advisory for detail.

**Why it matters.** Java underpins a large amount of Cisco management and operations software, which is exactly the class of system that holds credentials for the whole estate. The product list is not published yet, so this is an advisory to watch on a schedule rather than act on once: the affected-product table is where the work will appear.

**Recommended actions**

- Subscribe to or re-check the Cisco advisory (cisco-sa-nov2026-java-rce) daily until the affected-product and fixed-release tables are populated - the source states it is incomplete.
- Check the vendor advisory for the fixed release; no fixed Cisco versions are stated in the source and none should be assumed.
- Build the target list now from your inventory: Cisco software components that bundle a Java runtime, with their running versions, so the upgrade scope is ready when Cisco publishes it.
- Restrict management-plane access to Java-based Cisco management systems to the management VLAN or out-of-band network, and keep them off the internet.
- Ship their logs to the SIEM and alert on admin logins, configuration changes and unexpected child-process execution from the application service account.

**Legacy / unpatchable gear.** Cisco products that are past end of software maintenance will not get a Java4Shell fix. Isolate them behind a supported inspecting firewall, restrict management access to a dedicated jump host with session logging, and open a replacement record.

**Detection.** Unexpected process execution or outbound connections from the Java service account on Cisco management hosts; inbound requests carrying large or malformed Base64 payloads to their web services; Talos coverage referenced in the advisory for signature detail.

**AI angle.** None stated.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nov2026-java-rce-eCa6ss9z)**

#### 🟠 HIGH — Hackers Exploit Cisco Expressway Vulnerability Soon After PoC Release

*[SecurityWeek](https://www.securityweek.com/hackers-exploit-cisco-expressway-vulnerability-soon-after-poc-release/) · 2026-11-19* 

> Cisco Expressway devices were exploited shortly after a proof of concept went public, with attackers hitting a clustered deployment and staging lateral movement.

**Affected:** Cisco Expressway, including clustered deployments. The source does not name the CVE, the affected versions or the fixed release.  

**What happened.** Attackers exploited a Cisco Expressway vulnerability soon after proof-of-concept code was released, compromising clustered Expressway devices at a Russian organization and preparing tooling for lateral movement.

**Why it matters.** Expressway sits at the network edge to traverse the firewall for collaboration traffic, so it is both internet-reachable and trusted inward - the shape of device that turns into a pivot. Confirmed in-the-wild exploitation following a public proof of concept means the window between disclosure and attack is closed; the lateral-movement tooling reported here says the attackers treated it as an entry point rather than as a target in itself.

**Recommended actions**

- Identify all Expressway-E and Expressway-C nodes and clusters, with their running versions, and check the Cisco advisory for the fixed release.
- Upgrade to the Cisco fixed release on the stated maintenance path, cluster peers in sequence to preserve service.
- Restrict inbound exposure to the specific ports and source ranges collaboration traffic actually needs, and remove administrative access from the internet-facing interface.
- Rotate administrative credentials, certificates and any service-account credentials held on Expressway nodes after upgrade, and terminate active sessions.
- Hunt inward from Expressway: review east-west connections from these nodes to internal systems, new local accounts, scheduled tasks and modified web resources.
- Verify segmentation between the collaboration edge and the server and management networks, since lateral movement was the attacker's stated objective.

**Legacy / unpatchable gear.** Expressway nodes on versions no longer receiving maintenance should be taken out of the internet path, fronted by a supported inspecting firewall with inbound limited to required ports, managed only from a jump host with session logging, and replaced on a dated plan.

**Detection.** Unexpected outbound or east-west connections from Expressway nodes, new local accounts or SSH keys, unfamiliar processes and files, administrative logins from new sources, and cluster configuration changes outside the change window.

**AI angle.** None stated.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/hackers-exploit-cisco-expressway-vulnerability-soon-after-poc-release/)**

#### ⚪ WATCH — ClamAV Vulnerabilities Affecting Cisco Products: November 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-clamav-multi-e93M3vdA) · 2026-11-18* `vendor-advisory`

> Cisco shipped updates for ClamAV flaws that let a remote attacker bypass scan limits and stall scanning, rated High only on Windows platforms.

**Affected:** Cisco platforms that embed ClamAV. The source states the Security Impact Rating is High only on Windows-based platforms, where the scanning process runs in a privileged context, with Cisco Secure Endpoint Connector for Windows named among the highly impacted platforms, and Medium on other platforms including Linux. Specific fixed versions are not stated in the collected item.  

**What happened.** Multiple ClamAV vulnerabilities could let a remote attacker bypass configured file and scan time limits and cause a denial of service that interrupts scanning operations. Cisco has released software updates for affected Cisco platforms; the advisory states there are no workarounds.

**Why it matters.** Low direct impact on network devices - the named high-impact platform is an endpoint connector, not an appliance. The relevance for network teams is that ClamAV is a shared scanning component across Cisco product lines, so appliance-side scanning on affected platforms can be stalled by crafted content, turning a content-inspection control into a gap while the scanner is down. There is no workaround, so the update is the only remedy.

**Recommended actions**

- Check the Cisco advisory (cisco-sa-clamav-multi) for the affected-platform list and the fixed releases that apply to the Cisco products you run.
- Schedule the update on content-inspection platforms in the normal maintenance window; no workaround exists, per the advisory.
- Alert on scanning-service failures or restarts on affected platforms so a scanner outage does not pass silently as 'clean' traffic.
- Confirm fail-closed or fail-open behaviour is what you intend when the scanning engine is unavailable.

**Legacy / unpatchable gear.** Platforms past end of software maintenance will not receive the ClamAV update; do not rely on their scanning verdicts as a control, and place inspection on a supported device in the path instead.

**Detection.** Scanning engine crash, restart or timeout events in platform logs; a drop in scanned-object counts without a corresponding drop in traffic volume.

**AI angle.** None stated.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-clamav-multi-e93M3vdA)**

---

## Multiple

*2 item(s) — 1 high, 1 watch*

### Other

#### 🟠 HIGH — Java4Shell Exploitation Attempts Begin as Vendors Rush Out Advisories

*[SecurityWeek](https://www.securityweek.com/java4shell-exploitation-attempts-begin-as-vendors-rush-out-advisories/) · 2026-11-19* 

> Exploit attempts against the OpenJDK Java4Shell flaw have started, with Cisco among the vendors publishing impact advisories.

**Affected:** Products embedding affected OpenJDK versions. Cisco, Elastic, Red Hat and Siemens have detailed impact; the source does not enumerate affected network device models or fixed releases.  

**What happened.** GreyNoise, watchTowr and Cloudflare report probing and exploit attempts against the Java4Shell vulnerability, while Cisco, Elastic, Red Hat and Siemens have published advisories detailing their impact.

**Why it matters.** This moves CVE-2026-114938 from 'vendors are assessing' to 'traffic is arriving', which shortens the available window for the Cisco advisory in item 1 to resolve into a product list. Java4Shell is a supply-chain-shaped exposure: the affected code is a runtime bundled inside products, so the asset question is which of your network management systems embed Java, not which vendor is named in the headline.

**Recommended actions**

- Track vendor advisories for every management and infrastructure product you run that bundles a Java runtime, and record fixed releases as they are published - check the vendor advisory for the fixed release rather than inferring one.
- Prioritise internet-reachable Java-based services; exploit attempts are already being observed against them.
- Place a WAF or reverse proxy rule in front of exposed Java web services if the vendor publishes a detection or mitigation pattern, as an interim control pending patches.
- Keep management-platform web interfaces off the internet and restricted to the management VLAN or out-of-band network.
- Alert in the SIEM on scanner and exploit patterns reported by GreyNoise and Cloudflare against your edge.

**Legacy / unpatchable gear.** Products past end of support that bundle an affected Java runtime will not be fixed: remove their internet exposure, place them behind a supported inspecting firewall, restrict management to a jump host, and plan replacement.

**Detection.** Edge and WAF logs showing probing against Java web endpoints, unusual Base64-heavy request payloads, and unexpected process execution or outbound connections from Java service accounts on management hosts.

**AI angle.** None stated.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/java4shell-exploitation-attempts-begin-as-vendors-rush-out-advisories/)**

#### ⚪ WATCH — CISA Warns of 4 Exploited Flaws Linked to Ransomware, Cryptomining, and LLMjacking Attacks

*[The Hacker News](https://thehackernews.com/2026/11/cisa-warns-of-4-exploited-flaws-linked.html) · 2026-11-19* `AI`

> CISA added four actively exploited flaws to KEV, including Synology DiskStation Manager and the LiteLLM proxy, tied to ransomware, cryptomining and LLMjacking.

**Affected:** Synology DiskStation Manager (DSM), BerriAI LiteLLM, Apache Axis, and Vanderbilt University REDCap. No network device platforms are in this batch; specific affected versions are not stated in the source.  

**What happened.** CISA added four vulnerabilities to the Known Exploited Vulnerabilities catalog citing evidence of active exploitation, reported as linked to ransomware, cryptomining and LLMjacking activity. CVE identifiers for the four are not given in the collected item.

**Why it matters.** None of the four is network infrastructure, so this is not a device patching action. It matters to network teams because all four are typically internet-reachable or internet-connected services that sit on segments the network team owns: a Synology NAS and an LLM proxy are common shadow infrastructure, and cryptomining and LLMjacking are both outbound-traffic abuse that shows up in NetFlow before it shows up in an asset report.

**Recommended actions**

- Query the perimeter for inbound exposure of Synology DSM, Apache Axis services, REDCap and LiteLLM instances and remove any that are internet-reachable without a documented need.
- Check the CISA KEV catalogue entries for the four products and apply the remediation they specify within the stated due dates for federal scope, or on your own edge SLA otherwise.
- Alert on outbound traffic from these hosts to mining pools and to LLM provider APIs from systems that have no business calling them.
- Segment NAS and AI-stack hosts away from the network management VLAN so a compromise there does not reach device administration.

**Legacy / unpatchable gear.** Unsupported NAS models will not receive a DSM fix. Deny them inbound access from untrusted zones entirely, restrict them to a server VLAN with no route to management, and set a replacement date.

**Detection.** NetFlow or firewall logs showing new outbound connections from NAS or AI-proxy hosts, especially to mining pools or to model provider endpoints; egress denies from those hosts on the perimeter firewall.

**AI angle.** LLMjacking exploitation of BerriAI LiteLLM is the AI element: an LLM proxy abused for its provider credentials and inference capacity. See the AI section.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/11/cisa-warns-of-4-exploited-flaws-linked.html)**

---

## SonicWall

*1 item(s) — 1 high*

### Firewalls

#### 🟠 HIGH — SonicWall urges customers to replace Gen 6 firewalls now

*[Help Net Security](https://www.helpnetsecurity.com/2026/11/19/sonicwall-gen-6-firewalls-reach-end-of-support/) · 2026-11-19* 

> SonicWall Gen 6 and Gen 6.5 firewalls on SonicOS 6.x are now out of support and will get no further patches; SonicWall says replace them and pull them off the edge.

**Affected:** All SonicWall Gen 6 and Gen 6.5 appliances running SonicOS 6.x.  

**What happened.** In a product advisory published on Wednesday, SonicWall urged customers still running Gen 6 firewalls to replace them immediately and to stop using them at the internet edge. All Gen 6 and Gen 6.5 appliances on SonicOS 6.x have reached end of support, meaning no further firmware updates or security patches 'regardless of severity'. SonicWall calls the Gen 6 fleet a high-risk exposure and expects internet-facing units to be targeted.

**Why it matters.** A perimeter firewall that will never be patched again is a standing pre-authentication exposure with no remediation path - the next SonicOS 6.x vulnerability has no fix by definition. The vendor itself is now recommending removal from the edge rather than mitigation, which is as clear a signal as this class of advisory gets, and it converts a patching task into a procurement and migration project with a lead time.

**Recommended actions**

- List every Gen 6 and Gen 6.5 unit on SonicOS 6.x with its role, site and whether it terminates internet or VPN traffic; this is the replacement backlog.
- Move internet-edge and VPN-termination duties off Gen 6 units first - those are the ones SonicWall expects to be targeted.
- Until replacement, deny all inbound access from untrusted zones, remove management (HTTPS/SSH/API) and SSL VPN portals from any internet-facing interface, and allow administration only from a dedicated jump host on the management VLAN.
- Rotate admin credentials, VPN pre-shared keys and certificates on these devices, and enforce MFA on any remaining remote access they serve.
- Order replacement hardware with a named budget owner and a dated migration plan; record the accepted risk against the device group in the change record until then.
- Monitor the remaining units as high-risk assets: NetFlow baseline, syslog to the SIEM, alerts on new outbound flows, admin logins and configuration changes.

**Legacy / unpatchable gear.** This item is the legacy case. End-of-support Gen 6 gear will not receive a fix for anything found from now on: isolate it behind a supported inspecting firewall, deny inbound from untrusted zones, apply strict ACLs with no direct management access and full session logging, set a replacement date and budget owner, and document the accepted risk until the hardware is gone.

**Detection.** Administrative logins from unexpected sources, configuration changes outside the change window, new local admin accounts, unexpected outbound flows from the firewall itself, and SSL VPN authentication attempts from unusual geographies or ASNs.

**AI angle.** None stated.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/11/19/sonicwall-gen-6-firewalls-reach-end-of-support/)**

---

## Check Point

*1 item(s) — 1 watch*

### Other

#### ⚪ WATCH — The Kill Switch is the Last Resort. Agentic Steering Is the First Move.

*[Check Point Blog](https://blog.checkpoint.com/ai-security/ai-agent-kill-switch-agentic-steering/) · 2026-11-19* `AI` `AI-defense`

> Check Point describes agentic steering and an automatic kill switch in Check Point AI Security for constraining AI agents that go off-task or get hijacked.

**Affected:** No device vulnerability. Vendor capability description covering AI agent deployments.  

**What happened.** Check Point published a post describing how Check Point AI Security handles AI agents: it provides an automatic way to stop an agent, which the vendor keeps as a last resort, and what it calls agentic steering as the first response. The post separates two cases - an agent hijacked by an attacker, which should be stopped, and the more common case of an agent taking a risky shortcut to finish its job, which the vendor argues needs a different answer because switching an agent off stops the risk and the work together.

**Why it matters.** Not a patching item, but it is the control question network teams will be asked as agents start touching infrastructure: what stops an agent mid-action, and what constrains it short of stopping. The distinction the vendor draws - hijacked versus merely reckless - matters operationally, because a network change made by an agent taking a shortcut does the same damage as one made by an attacker.

**Recommended actions**

- Before granting any agent access to network configuration, define the stop control and who can invoke it, and test it in a change window.
- Scope agent credentials to read-only for network devices and require human approval for any configuration write.
- Log every agent-initiated action against the device it touched, so an unexpected change can be attributed and reversed.
- If evaluating Check Point AI Security, confirm with the vendor which of these controls are available in your deployment model before relying on them in a design.

**Legacy / unpatchable gear.** Do not place agents or AI assistants in the management path of end-of-life devices: credentials there cannot be rotated safely at agent speed, and the devices will not receive a fix if the integration is abused. Keep them behind a supported firewall with manual-only administration.

**Detection.** Configuration changes on network devices attributable to an agent service account, especially outside approved windows; agent actions with no matching change record; API calls from agent credentials to write endpoints.

**AI angle.** This is the defensive side of AI on the network: a vendor-described control plane for AI agents, with a kill switch as the last resort and agentic steering as the first move. Recorded in vendor AI defences.

📄 **[Read the full report at Check Point Blog →](https://blog.checkpoint.com/ai-security/ai-agent-kill-switch-agentic-steering/)**

---

## 🛡️ Vendor AI defences introduced

AI capabilities the vendors shipped this cycle - the answer side of the AI story.

### Check Point

**Check Point AI Security (agentic steering and automatic agent kill switch)** — Check Point describes AI Security as providing an automatic way to stop an AI agent, held as a last resort, with what it calls agentic steering as the first-line response. The stated rationale is that switching an agent off stops the risk and the work at the same time, so steering an agent away from risky behaviour precedes stopping it.

- *Defends against:* AI agents hijacked by an attacker, and the more common case the vendor describes of an agent taking a risky shortcut to complete its task - both of which can result in unintended privileged action.
- *Availability:* described-only - the post describes the capability; the source states no release date, version or availability tier.
- *Worth evaluating if:* Network and security teams about to let agents or AI assistants touch firewall, routing or VPN configuration, who need a defined stop control and a permission boundary short of stopping, and existing Check Point customers mapping agent governance onto a platform they already run.
- 📄 [Read the announcement at Check Point Blog →](https://blog.checkpoint.com/ai-security/ai-agent-kill-switch-agentic-steering/)

---

## 🧠 AI & network devices — Agent control arrives as a network-vendor feature, while an LLM proxy lands in KEV

Two items this cycle are genuinely about AI rather than about AI marketing. Check Point published a description of how Check Point AI Security handles AI agents: an automatic kill switch to stop a hijacked agent, and what it calls agentic steering as the first-line response for the more common case of an agent taking a risky shortcut to finish its job. The vendor's framing is that switching an agent off stops the risk and the work together, so steering precedes stopping.

Separately, CISA added BerriAI LiteLLM to the Known Exploited Vulnerabilities catalog alongside three other actively exploited flaws, with the reported exploitation tied to ransomware, cryptomining and LLMjacking. LiteLLM is an LLM proxy, which in most deployments is a piece of infrastructure sitting on an internal network segment with outbound internet access and provider API keys on it - an exposure network teams own even when the AI stack is not theirs.

For network teams the practical reading is unchanged: anything that lets a model or an agent act on infrastructure is a privileged administrative path, and anything that proxies model traffic is an internet-facing service with credentials on it. Both belong in the asset inventory and in the patch SLA for edge systems.

**Vendors with an AI angle in this edition:** Check Point, Cisco, F5, Multiple, Palo Alto Networks, SonicWall

**Preventing it on current systems**

- Treat any AI/LLM integration on network devices (assistants, AIOps, MCP or agent connectors) as a privileged admin path: scoped read-only credentials, no unattended config write, full audit trail.
- Inventory LLM proxies and gateways on your network as internet-facing services: patch them on the edge-device SLA, hold the provider API keys in a secret store, and alert on their outbound destinations.
- Require a stop control and a scoped permission boundary before any agent is allowed to touch firewall, routing or VPN configuration - and test that the stop control actually works.
- Test AI-facing management surfaces for prompt injection from attacker-controlled data (device logs, hostnames, ticket text) before letting an agent act on them.
- Assume exploit development and mass scanning are faster than your historical patch SLA - shorten the window for internet-facing network devices to days, not months.
- Harden against AI-enabled social engineering of the network team: callback verification for any out-of-band request to change firewall or VPN configuration.

**Preventing it on legacy / end-of-life systems**

- Do not connect AI assistants, AIOps collectors or agent tooling to end-of-life gear - credentials for unsupported devices cannot be rotated safely at agent speed, and the devices will not get a fix if the integration is abused.
- Keep legacy devices behind a supported inspecting firewall with strict ACLs and management access only from a dedicated jump host with full session logging.
- Baseline NetFlow for legacy segments and alert on new outbound flows - detection is the only remaining control once patching is unavailable.

---

<!--index
{
 "url": "/2026/11/19/network-security-digest/",
 "date": "2026-11-19",
 "entries": [
  {
   "title": "Everything Is a Web Application, Including Your Firewall",
   "link": "https://thehackernews.com/2026/11/everything-is-web-application-including.html",
   "source": "The Hacker News",
   "relevance": "critical",
   "device_types": [
    "firewall",
    "management-platform"
   ],
   "vendors": [
    "Palo Alto Networks"
   ],
   "cves": [
    "CVE-2026-109618",
    "CVE-2026-109619"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Palo Alto Networks confirmed attackers exploiting two unauthenticated PHP code-execution flaws against internet-exposed PAN-OS management interfaces.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "F5 Patches Critical NGINX2Root Flaws as PoC Exploits Go Public",
   "link": "https://www.securityweek.com/f5-patches-critical-nginx2root-flaws-as-poc-exploits-go-public/",
   "source": "SecurityWeek",
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
   "ai_angle": "None stated.",
   "summary": "F5 patched five NGINX2Root flaws that let remote unauthenticated attackers run code, with proof-of-concept exploits now public.",
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
   "ai_angle": "None stated.",
   "summary": "Cisco is tracking CVE-2026-114938 in OpenJDK (Java4Shell), an RCE in Java 17.0.6 and later on x86-64 that affects an as-yet unlisted set of Cisco products.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "SonicWall urges customers to replace Gen 6 firewalls now",
   "link": "https://www.helpnetsecurity.com/2026/11/19/sonicwall-gen-6-firewalls-reach-end-of-support/",
   "source": "Help Net Security",
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
   "ai_angle": "None stated.",
   "summary": "SonicWall Gen 6 and Gen 6.5 firewalls on SonicOS 6.x are now out of support and will get no further patches; SonicWall says replace them and pull them off the edge.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "F5 Warns 6 Critical BIG-IP Flaws Can Crash Traffic in Specific Setups, May Allow RCE",
   "link": "https://thehackernews.com/2026/11/f5-warns-6-critical-big-ip-flaws-can.html",
   "source": "The Hacker News",
   "relevance": "high",
   "device_types": [
    "load-balancer"
   ],
   "vendors": [
    "F5"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Six critical F5 BIG-IP flaws let an unauthenticated attacker crash the Traffic Management Microkernel and drop traffic on virtual servers with specific iRule or APM config.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Hackers Exploit Cisco Expressway Vulnerability Soon After PoC Release",
   "link": "https://www.securityweek.com/hackers-exploit-cisco-expressway-vulnerability-soon-after-poc-release/",
   "source": "SecurityWeek",
   "relevance": "high",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Cisco Expressway devices were exploited shortly after a proof of concept went public, with attackers hitting a clustered deployment and staging lateral movement.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Java4Shell Exploitation Attempts Begin as Vendors Rush Out Advisories",
   "link": "https://www.securityweek.com/java4shell-exploitation-attempts-begin-as-vendors-rush-out-advisories/",
   "source": "SecurityWeek",
   "relevance": "high",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Multiple"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Exploit attempts against the OpenJDK Java4Shell flaw have started, with Cisco among the vendors publishing impact advisories.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "CISA Warns of 4 Exploited Flaws Linked to Ransomware, Cryptomining, and LLMjacking Attacks",
   "link": "https://thehackernews.com/2026/11/cisa-warns-of-4-exploited-flaws-linked.html",
   "source": "The Hacker News",
   "relevance": "watch",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Multiple"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "LLMjacking exploitation of BerriAI LiteLLM is the AI element: an LLM proxy abused for its provider credentials and inference capacity. See the AI section.",
   "summary": "CISA added four actively exploited flaws to KEV, including Synology DiskStation Manager and the LiteLLM proxy, tied to ransomware, cryptomining and LLMjacking.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "ClamAV Vulnerabilities Affecting Cisco Products: November 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-clamav-multi-e93M3vdA",
   "source": "Cisco PSIRT",
   "relevance": "watch",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Cisco shipped updates for ClamAV flaws that let a remote attacker bypass scan limits and stall scanning, rated High only on Windows platforms.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "The Kill Switch is the Last Resort. Agentic Steering Is the First Move.",
   "link": "https://blog.checkpoint.com/ai-security/ai-agent-kill-switch-agentic-steering/",
   "source": "Check Point Blog",
   "relevance": "watch",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "This is the defensive side of AI on the network: a vendor-described control plane for AI agents, with a kill switch as the last resort and agentic steering as the first move. Recorded in vendor AI defences.",
   "summary": "Check Point describes agentic steering and an automatic kill switch in Check Point AI Security for constraining AI agents that go off-task or get hijacked.",
   "ai_defense": true,
   "defense": {
    "solution": "Check Point AI Security (agentic steering and automatic agent kill switch)",
    "what_it_does": "Check Point describes AI Security as providing an automatic way to stop an AI agent, held as a last resort, with what it calls agentic steering as the first-line response. The stated rationale is that switching an agent off stops the risk and the work at the same time, so steering an agent away from risky behaviour precedes stopping it.",
    "defends_against": "AI agents hijacked by an attacker, and the more common case the vendor describes of an agent taking a risky shortcut to complete its task - both of which can result in unintended privileged action.",
    "availability": "described-only - the post describes the capability; the source states no release date, version or availability tier."
   }
  }
 ]
}
-->

## How this was produced

- Feeds polled: 23 ok, 0 failed
- Raw items: 739 → in window: 59 → network-device relevant: 14 → published: 10
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
