---
layout: post
title: "Network Device Security Digest - 2026-10-21"
date: 2026-10-21 11:33:53 +0000
edition: daily
critical_count: 3
item_count: 5
kev: "CVE-2026-0257"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-10-21

*daily edition · window 24h · 22 feeds · 5 items · 3 critical · generated 2026-10-21 11:33 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — Ransomware crews are still breaking into PAN-OS GlobalProtect through CVE-2026-0257

Palo Alto Networks reports continuing exploitation of CVE-2026-0257 (CVSS 7.8), an authentication bypass in the GlobalProtect portal and gateway of PAN-OS, and in Prisma Access. The flaw comes from the firewall accepting cookies without detailed validation and integrity checking, which lets a remote, unauthenticated attacker bypass security restrictions and establish an unauthorised VPN connection. Palo Alto Networks patched it in May 2026; CISA added it to the KEV catalog on 2026-05-29 with a 2026-06-01 remediation due date and records it as used in ransomware operations.

Unit 42's threat brief, updated 20 October, names Spikey Scorpius (Qilin) and Tarnished Scorpius (Inc) among the operators. Reporting puts exploitation as far back as June - after the patch and after the KEV deadline - against organisations in healthcare, education, government and energy. Because the bypass produces a VPN session without a valid authentication exchange, enforcing MFA on the portal does not substitute for the upgrade; one of this cycle's headlines describes the activity precisely as MFA bypass.

An unpatched GlobalProtect portal is a remote-access entry point with routed reach into the internal network, which is why ransomware affiliates keep returning to it. Treat any firewall that was internet-facing and unpatched between May and now as a suspected initial-access point, not merely as a patching task: sessions established before the upgrade, and any credentials or certificates held on the appliance, survive it.

**Do this first**

- Inventory every GlobalProtect portal and gateway, plus Prisma Access tenants, and compare the running PAN-OS build against the vendor's fixed-release table - check the vendor advisory for the fixed release rather than assuming a build number.
- Upgrade inside the change window, HA pair secondary first; where a window is not available yet, apply the mitigations Palo Alto Networks documents in its advisory.
- After upgrade, terminate all active GlobalProtect sessions and revoke session cookies and tokens - the vulnerability abuses cookie handling, so stolen cookies must be assumed.
- Rotate local admin credentials, API keys, device certificates and VPN pre-shared keys on every affected firewall; a pre-patch compromise survives the upgrade.
- Hunt back to June: VPN logins with no matching MFA event, sessions from unexpected ASNs or geographies, new local accounts, unexpected outbound connections from the firewall, and configuration changes outside change records.
- Export and review the configuration for unexpected admin users, static routes, scripts or modified login pages.

---

> **On the CISA KEV catalog in this edition:** CVE-2026-0257 — treat these as confirmed-exploited and patch on an emergency change.

## Executive summary

- CVE-2026-0257, the PAN-OS GlobalProtect authentication bypass patched in May 2026 and listed in CISA KEV since 2026-05-29, is still being exploited by Qilin and Inc ransomware operators; exploitation is reported as far back as June.
- A public proof-of-concept now exists for CVE-2026-106173 (CVSS 9.4), an unauthenticated path-traversal code execution flaw in Fortinet FortiSIEM - a platform that typically holds privileged credentials for the network estate.
- Cisco patched a blind SQL injection in the Unified Intelligence Center web management interface; exploitation requires valid credentials and there are no workarounds.
- Autonomous AI agents were used to attack more than 1,200 websites with minimal operator involvement - no network-device exploitation reported, but a reminder that exposed management surfaces are enumerated continuously.
- No vendor AI-defence capability was announced or described in this cycle's items.

---

## Items by vendor

## Palo Alto Networks

*3 item(s) — 2 critical, 1 high*

### Firewalls

#### 🔴 CRITICAL — Ransomware gangs have been exploiting Palo Alto GlobalProtect VPN flaw since June

*[Help Net Security](https://www.helpnetsecurity.com/2026/10/21/ransomware-groups-exploit-palo-alto-globalprotect-cve-2026-0257/) · 2026-10-21* `KEV`

> Ransomware groups have exploited CVE-2026-0257, an authentication bypass in PAN-OS GlobalProtect, since June against healthcare, education, government and energy targets.

**Affected:** GlobalProtect portal and gateway on Palo Alto Networks PAN-OS firewalls (the source text is truncated where it lists form factors). Patched in May 2026 - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-0257](https://nvd.nist.gov/vuln/detail/CVE-2026-0257) **[KEV]**  

**What happened.** Several ransomware groups are exploiting CVE-2026-0257, an authentication bypass in PAN-OS GlobalProtect. The firewall relies on cookies without performing detailed validation and integrity checking, allowing a remote, unauthenticated attacker to bypass security restrictions and establish an unauthorised VPN connection. CISA lists the CVE in KEV (added 2026-05-29, due 2026-06-01, ransomware use Known). Victims span healthcare, education, government and energy.

**Why it matters.** A GlobalProtect portal is internet-facing by design and terminates into the internal network, so a working bypass is initial access for a ransomware affiliate. Exploitation continuing months after the May patch means unpatched firewalls are being found and used, not merely scanned.

**Recommended actions**

- Compare the running PAN-OS build on every GlobalProtect portal and gateway against the vendor's fixed-release table; check the vendor advisory for the fixed release.
- Schedule the upgrade in the change window, HA pair secondary first; apply the vendor's documented mitigations in the meantime.
- Terminate all active VPN sessions and revoke session cookies and tokens after upgrade - the flaw abuses cookie validation, so assume stolen cookies.
- Rotate local admin credentials, API keys, certificates and VPN pre-shared keys on the appliance after patching.
- Enforce phishing-resistant MFA and geo/ASN conditional access on the portal, while treating MFA as a control that this bypass sidesteps rather than a substitute for the patch.
- Remove administrative access (HTTPS/SSH/API) from internet-facing interfaces and restrict it to a management VLAN or out-of-band network.

**Legacy / unpatchable gear.** Firewalls past end of support will not receive this fix: take the GlobalProtect service off the internet, terminate remote access on a supported gateway instead, place the legacy unit behind a supported inspecting firewall with strict ACLs, and set a replacement date with a budget owner.

**Detection.** Search firewall and SIEM data back to June for GlobalProtect logins with no corresponding MFA or authentication event, sessions from unexpected ASNs or countries, repeated portal requests carrying reused or malformed cookies, new or modified local admin accounts, configuration commits outside change windows, and outbound connections initiated by the firewall itself.

**AI angle.** None reported in this item.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/10/21/ransomware-groups-exploit-palo-alto-globalprotect-cve-2026-0257/)**

#### 🔴 CRITICAL — Qilin and INC Ransomware Groups Exploit Palo Alto Networks Flaw to Bypass MFA

*[The Hacker News](https://thehackernews.com/2026/10/qilin-and-inc-ransomware-groups-exploit.html) · 2026-10-20* `KEV`

> Palo Alto Networks says Qilin and INC ransomware operators are still exploiting CVE-2026-0257 in PAN-OS and Prisma Access to stand up VPN sessions that bypass MFA.

**Affected:** PAN-OS and Prisma Access. Patched by Palo Alto Networks in May 2026 - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-0257](https://nvd.nist.gov/vuln/detail/CVE-2026-0257) **[KEV]**  

**What happened.** Palo Alto Networks disclosed continuing exploitation of CVE-2026-0257 (CVSS 7.8) by ransomware actors against PAN-OS and Prisma Access. The authentication bypass lets attackers set up unauthorised VPN connections, which the reporting characterises as bypassing MFA.

**Why it matters.** Prisma Access brings the same flaw into a cloud-delivered remote-access path, so the exposure is not limited to on-premises firewall hardware. Because authentication is bypassed rather than defeated, MFA posture and password hygiene do not reduce the risk - only the fix and session invalidation do.

**Recommended actions**

- Confirm the patch state of both on-premises PAN-OS firewalls and Prisma Access tenants; check the vendor advisory for the fixed release and the tenant-side remediation steps.
- Revoke active sessions and rotate service-account credentials and secrets stored on or used by the remote-access service after remediation.
- Audit remote-access authorisation: which internal networks a GlobalProtect or Prisma Access session can reach, and tighten it to named destinations.
- Record, per device group, whether the item is patched, mitigated or accepted risk in the change record.

**Legacy / unpatchable gear.** Where an unsupported firewall still terminates VPN users, move those users onto a supported gateway and leave the legacy device with no inbound exposure; isolate it behind a supported firewall and log all management sessions through a jump host.

**Detection.** Alert on VPN session establishment without a preceding successful MFA assertion, on concurrent sessions for one account from different geographies, and on first-seen source ASNs on the portal. For Prisma Access, review tenant authentication logs for the same pattern.

**AI angle.** None reported in this item.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/10/qilin-and-inc-ransomware-groups-exploit.html)**

#### 🟠 HIGH — Threat Brief: Active Exploitation of PAN-OS CVE-2026-0257 (Updated October 20)

*[Unit 42](https://unit42.paloaltonetworks.com/active-exploitation-of-pan-os-cve-2026-0257/) · 2026-10-20* `KEV`

> Unit 42's threat brief on PAN-OS CVE-2026-0257, updated 20 October, attributes active exploitation to the Qilin and Inc ransomware operations.

**Affected:** PAN-OS, as covered by the vendor threat brief. No fixed release is restated in this entry - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-0257](https://nvd.nist.gov/vuln/detail/CVE-2026-0257) **[KEV]**  

**What happened.** Unit 42 updated its threat brief on 20 October, reporting that ransomware operators including Spikey Scorpius (aka Qilin) and Tarnished Scorpius (aka Inc) are exploiting PAN-OS vulnerability CVE-2026-0257.

**Why it matters.** This is the vendor's own tracking of the campaign and the most likely place indicators and mitigation detail are kept current, so it is the reference to attach to the change record and the hunt.

**Recommended actions**

- Use the vendor threat brief as the authoritative source for indicators and mitigation steps, and re-read it before closing the incident - it is being updated.
- Map the named operators' post-access tradecraft to existing detections for lateral movement from the firewall segment.
- Feed any indicators published in the brief into the SIEM and EDR as a retrospective search, not just a forward-looking block.

**Legacy / unpatchable gear.** The same isolation guidance applies to unsupported firewalls: no inbound exposure, strict management ACLs, dedicated jump host, full session logging, and a documented replacement date.

**Detection.** Run the vendor-published indicators retrospectively across firewall logs, VPN authentication records and NetFlow for the period since June, rather than only from today forward.

**AI angle.** None reported in this item.

📄 **[Read the full report at Unit 42 →](https://unit42.paloaltonetworks.com/active-exploitation-of-pan-os-cve-2026-0257/)**

---

## Fortinet

*1 item(s) — 1 critical*

### Management platforms

#### 🔴 CRITICAL — Critical FortiSIEM Flaw Gets Public Exploit PoC for Unauthenticated Remote Code Execution

*[The Hacker News](https://thehackernews.com/2026/10/critical-fortisiem-flaw-gets-public.html) · 2026-10-20* 

> A public proof-of-concept exploit now exists for CVE-2026-106173, an unauthenticated path-traversal code execution flaw rated CVSS 9.4 in Fortinet FortiSIEM.

**Affected:** Fortinet FortiSIEM. The source describes the flaw as recently patched but states no version ranges or fixed build - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-106173](https://nvd.nist.gov/vuln/detail/CVE-2026-106173)  

**What happened.** Researchers released a proof-of-concept exploit for CVE-2026-106173 (CVSS 9.4), a critical path traversal vulnerability in Fortinet FortiSIEM that allows an unauthenticated attacker to execute unauthorised code or commands via specially crafted input.

**Why it matters.** Unauthenticated code execution plus a public PoC collapses the time between disclosure and opportunistic exploitation. FortiSIEM is a collection and analysis platform: it holds log data from the whole estate and commonly stores credentials used to poll network devices, so compromising it yields both visibility and a credential set - and the attacker lands inside the system you would use to detect them.

**Recommended actions**

- Identify FortiSIEM supervisors, workers and collectors from the asset inventory and apply the Fortinet fixed release; check the vendor advisory for the fixed release and the upgrade order between node roles.
- Remove the FortiSIEM web interface and collector ports from any internet-facing path immediately; restrict access to a management VLAN reached through a jump host.
- Rotate FortiSIEM admin credentials, API tokens and - importantly - the device credentials FortiSIEM stores for polling firewalls, routers and switches.
- Review the appliance for unexpected files, scheduled tasks, processes and new local accounts before declaring it clean.
- Ship FortiSIEM's own audit logs to a second destination so that a compromise of the platform cannot quietly erase its own trail.

**Legacy / unpatchable gear.** A FortiSIEM deployment on a release line that is out of support cannot be fixed: cut its network exposure to the collector sources only, deny all other inbound access, and treat the upgrade or replacement as funded work with a date rather than a backlog item.

**Detection.** Search FortiSIEM web access logs for path-traversal patterns (encoded and plain ../ sequences) and for requests to unexpected endpoints from outside the management network; alert on new processes or outbound connections from the FortiSIEM host, on new administrative accounts, and on gaps in log ingestion that could mask tampering.

**AI angle.** None reported in this item.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/10/critical-fortisiem-flaw-gets-public.html)**

---

## Cisco

*1 item(s) — 1 watch*

### Other

#### ⚪ WATCH — Cisco Unified Intelligence Center SQL Injection Vulnerability

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-cuic-sql-inject-2qbfWSm5) · 2026-10-20* `vendor-advisory`

> Cisco released updates for a blind SQL injection in the Unified Intelligence Center web management interface that lets an authenticated local user read the internal database.

**Affected:** Cisco Unified Intelligence Center, web-based management interface. The feed entry gives no CVE ID and no version numbers; software updates are available and there are no workarounds - check the Cisco advisory for the fixed release.  

**What happened.** A vulnerability in the web-based management interface of Cisco Unified Intelligence Center could allow an authenticated, local attacker to perform a blind SQL injection attack through a crafted request, due to insufficient validation of user-supplied input. A successful exploit reads the contents of the device's internal database. Valid user credentials on the device are required. Cisco has released software updates; no workarounds address the vulnerability.

**Why it matters.** This is not core network infrastructure, so it is a watch item rather than an emergency: exploitation needs valid credentials and yields database read access, not code execution. It matters where the same team owns the reporting platform and the network, because no workaround exists - the only remedy is the upgrade.

**Recommended actions**

- Check the running Unified Intelligence Center version against the Cisco advisory's fixed-release table and schedule the update; there is no workaround to fall back on.
- Review who holds accounts on the device and remove stale or shared logins - the attack requires valid credentials.
- Keep the management interface off user and internet-facing networks, restricted to the management VLAN.
- Enable and forward audit logging for administrative logins and report execution to the SIEM.

**Legacy / unpatchable gear.** If the deployment is on a release train Cisco no longer updates, the fix is not available: restrict access to a named administrator group from a jump host only, deny all other reachability, and schedule the platform upgrade.

**Detection.** Alert on authenticated sessions issuing malformed or unusually long parameters to the management interface, on report or query parameters containing SQL syntax, and on administrative logins from outside the management network.

**AI angle.** None reported in this item.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-cuic-sql-inject-2qbfWSm5)**

---

## 🧠 AI & network devices — Autonomous AI agents ran a 1,200-site intrusion campaign with almost no operator help

A Chinese-speaking criminal operator pointed autonomous AI agents at company websites and let them break in with almost no human assistance; a researcher observed the campaign because the operators left their control panels exposed on the internet. The agents attempted more than 1,200 sites and reached assets belonging to more than 100 companies, including Deere.

The reported targets were websites, not network appliances, so there is no network-device exploitation to act on here. The operational read-across is about speed and breadth: an agent-driven loop enumerates and attacks every exposed surface it can reach, at a rate that does not respect a monthly patch cycle. Internet-facing management interfaces, VPN portals and device web UIs sit in exactly that surface, and this cycle's PAN-OS story shows how long an exposed, unpatched gateway stays profitable.

**Vendors with an AI angle in this edition:** Cisco, Fortinet, Palo Alto Networks

**Preventing it on current systems**

- Shorten the patch window for internet-facing network devices to days, not months - assume scanning and exploit attempts are automated and continuous.
- Remove device management interfaces (HTTPS/SSH/API) from the internet entirely; an interface that is not reachable is not enumerable by an agent.
- Treat any AI/LLM integration on network gear (assistants, AIOps, MCP or agent connectors) as a privileged admin path: scoped read-only credentials, no unattended config write, full audit trail.
- Test AI-facing management surfaces for prompt injection from attacker-controlled data such as device logs, hostnames and ticket text before letting an agent act on them.
- Baseline NetFlow and syslog and alert on deviation, rather than relying on signature updates alone to catch high-volume automated probing.

**Preventing it on legacy / end-of-life systems**

- Legacy or end-of-life gear will not receive a fix: isolate it behind a supported inspecting firewall and deny inbound access from untrusted zones.
- Put compensating controls in place - strict management ACLs, no direct management access, a dedicated jump host, full session logging.
- Monitor unpatchable devices as high-risk assets: NetFlow baseline, alert on any new outbound flow or new administrative login.
- Set a replacement date and a budget owner, and document the accepted risk until the device is gone.

---

<!--index
{
 "url": "/2026/10/21/network-security-digest/",
 "date": "2026-10-21",
 "entries": [
  {
   "title": "Ransomware gangs have been exploiting Palo Alto GlobalProtect VPN flaw since June",
   "link": "https://www.helpnetsecurity.com/2026/10/21/ransomware-groups-exploit-palo-alto-globalprotect-cve-2026-0257/",
   "source": "Help Net Security",
   "relevance": "critical",
   "device_types": [
    "firewall",
    "vpn-gateway"
   ],
   "vendors": [
    "Palo Alto Networks"
   ],
   "cves": [
    "CVE-2026-0257"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None reported in this item.",
   "summary": "Ransomware groups have exploited CVE-2026-0257, an authentication bypass in PAN-OS GlobalProtect, since June against healthcare, education, government and energy targets.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Qilin and INC Ransomware Groups Exploit Palo Alto Networks Flaw to Bypass MFA",
   "link": "https://thehackernews.com/2026/10/qilin-and-inc-ransomware-groups-exploit.html",
   "source": "The Hacker News",
   "relevance": "critical",
   "device_types": [
    "firewall",
    "vpn-gateway",
    "sdwan"
   ],
   "vendors": [
    "Palo Alto Networks"
   ],
   "cves": [
    "CVE-2026-0257"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None reported in this item.",
   "summary": "Palo Alto Networks says Qilin and INC ransomware operators are still exploiting CVE-2026-0257 in PAN-OS and Prisma Access to stand up VPN sessions that bypass MFA.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Critical FortiSIEM Flaw Gets Public Exploit PoC for Unauthenticated Remote Code Execution",
   "link": "https://thehackernews.com/2026/10/critical-fortisiem-flaw-gets-public.html",
   "source": "The Hacker News",
   "relevance": "critical",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Fortinet"
   ],
   "cves": [
    "CVE-2026-106173"
   ],
   "kev": false,
   "ai": true,
   "ai_angle": "None reported in this item.",
   "summary": "A public proof-of-concept exploit now exists for CVE-2026-106173, an unauthenticated path-traversal code execution flaw rated CVSS 9.4 in Fortinet FortiSIEM.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Threat Brief: Active Exploitation of PAN-OS CVE-2026-0257 (Updated October 20)",
   "link": "https://unit42.paloaltonetworks.com/active-exploitation-of-pan-os-cve-2026-0257/",
   "source": "Unit 42",
   "relevance": "high",
   "device_types": [
    "firewall",
    "vpn-gateway"
   ],
   "vendors": [
    "Palo Alto Networks"
   ],
   "cves": [
    "CVE-2026-0257"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None reported in this item.",
   "summary": "Unit 42's threat brief on PAN-OS CVE-2026-0257, updated 20 October, attributes active exploitation to the Qilin and Inc ransomware operations.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco Unified Intelligence Center SQL Injection Vulnerability",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-cuic-sql-inject-2qbfWSm5",
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
   "ai_angle": "None reported in this item.",
   "summary": "Cisco released updates for a blind SQL injection in the Unified Intelligence Center web management interface that lets an authenticated local user read the internal database.",
   "ai_defense": false,
   "defense": null
  }
 ]
}
-->

## How this was produced

- Feeds polled: 22 ok, 1 failed
- Raw items: 689 → in window: 64 → network-device relevant: 11 → published: 5
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
