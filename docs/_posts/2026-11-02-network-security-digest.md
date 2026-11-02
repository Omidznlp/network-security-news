---
layout: post
title: "Network Device Security Digest - 2026-11-02"
date: 2026-11-02 15:21:01 +0000
edition: daily
critical_count: 2
item_count: 4
kev: "CVE-2026-111357, CVE-2026-118527"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-11-02

*daily edition · window 24h · 23 feeds · 4 items · 2 critical · generated 2026-11-02 15:21 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — Two more Ivanti Connect Secure zero-days chained in active attacks, both now in CISA KEV

Ivanti has released security updates for two further Connect Secure (ICS) remote access gateway vulnerabilities, CVE-2026-111357 and CVE-2026-118527, that have been chained together in zero-day attacks since at least 1 October. CISA added both to the Known Exploited Vulnerabilities catalog on 30 October, with remediation due 2 November for CVE-2026-111357 and 13 November for CVE-2026-118527 under BOD 26-04.

Per CISA's entries, CVE-2026-111357 is a missing-authentication-for-critical-function flaw that lets a local authenticated attacker escalate privileges, and CVE-2026-118527 is a predictable identifier generation flaw that lets a remote authenticated attacker write attacker-controlled file content to a known location on disk. Both are documented as chainable with CVE-2026-104935. Individually each needs some level of authentication; combined with the earlier flaw in the chain they are being used as a working intrusion path into the appliance.

This matters because Connect Secure sits at the perimeter and terminates remote access for the whole workforce. An attacker who reaches code execution on the gateway holds every session, every cached credential and a trusted position inside the network - and has held it, in some environments, since early October. Patching closes the hole but does not evict anyone who already got in.

The sources do not state fixed build numbers, so confirm your target version against Ivanti's advisory before scheduling. Treat any internet-facing ICS appliance as potentially compromised until forensic triage says otherwise; CISA's KEV action explicitly points federal stakeholders at its Forensics Triage Requirements, which is sound practice for everyone else too.

**Do this first**

- Inventory every Ivanti Connect Secure appliance, note its running build, and patch to the fixed release named in Ivanti's advisory - check the vendor advisory for the fixed release, as the sources do not state build numbers.
- Run Ivanti's Integrity Checker Tool before and after the upgrade and preserve the output as evidence.
- Treat the KEV deadlines as your SLA: CVE-2026-111357 was due 2 November 2026, CVE-2026-118527 is due 13 November 2026.
- After upgrading, terminate all active VPN sessions and revoke session tokens - assume stolen session cookies survive the patch.
- Rotate everything stored on or reachable from the appliance: local admin credentials, service accounts, LDAP/AD bind accounts, device certificates and API keys.
- Perform forensic triage for the exploitation window back to 1 October 2026: new or modified local accounts, unexpected files written to web-accessible paths, modified web resources, anomalous entries and timestamp gaps in /var logs, unexpected outbound connections from the gateway.
- Restrict the administrative interface to a management VLAN or out-of-band network and enforce phishing-resistant MFA on both admin and user authentication.

---

> **On the CISA KEV catalog in this edition:** CVE-2026-111357, CVE-2026-118527 — treat these as confirmed-exploited and patch on an emergency change.

## Executive summary

- Ivanti Connect Secure: two more zero-days (CVE-2026-111357, CVE-2026-118527) patched after being chained in attacks since at least 1 October; both in CISA KEV, one already past its 2 November due date.
- Patch alone is not containment on ICS - kill sessions, rotate appliance credentials and certificates, and triage back to early October.
- Sophos reports the Aur0ra ransomware group using AI agents, started from VPN logins, to map networks and escalate to domain admin; remote access MFA and machine-pace detection matter more than signatures.
- Synology warns of a C3RB3R ransomware surge against internet-exposed NAS appliances - storage rather than network gear, but it is usually the network team's port forward that exposes it.
- No new vendor AI-defence announcements in this window.
- Three collected items (Windows infostealer, Linux server espionage, a caller-verification phone app) were not about network infrastructure devices and were dropped.

---

## Items by vendor

## Ivanti

*2 item(s) — 2 critical*

### VPN gateways

#### 🔴 CRITICAL — Ivanti Patches Two More Zero-Days Exploited in Connect Secure Attack Chain

*[SecurityWeek](https://www.securityweek.com/ivanti-patches-two-more-zero-days-exploited-in-connect-secure-attack-chain/) · 2026-11-02* `KEV`

> CISA added Ivanti Connect Secure flaws CVE-2026-111357 and CVE-2026-118527 to KEV after both were chained in zero-day attacks on the VPN gateway.

**Affected:** Ivanti Connect Secure (ICS) remote access gateways. Fixed build numbers are not stated in the source - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-111357](https://nvd.nist.gov/vuln/detail/CVE-2026-111357) **[KEV]**, [CVE-2026-118527](https://nvd.nist.gov/vuln/detail/CVE-2026-118527) **[KEV]**  

**What happened.** Ivanti patched two additional Connect Secure vulnerabilities that were exploited as zero-days. CISA added both to the KEV catalog on 2026-10-30: CVE-2026-111357, a missing authentication for critical function issue allowing a local authenticated attacker to escalate privileges, and CVE-2026-118527, a predictable number or identifier generation issue allowing a remote authenticated attacker to upload attacker-controlled file content to a known location on disk. CISA records both as chainable with CVE-2026-104935. KEV due dates are 2026-11-02 for CVE-2026-111357 and 2026-11-13 for CVE-2026-118527.

**Why it matters.** Connect Secure terminates remote access for the entire workforce, so control of the appliance means control of every session and the credentials passing through it. Exploitation is confirmed and chained, which means the authentication prerequisites on each individual CVE are not protecting you. One KEV deadline has already passed.

**Recommended actions**

- Patch every ICS appliance to the fixed release named in Ivanti's advisory; do not assume a build number, confirm it against the advisory.
- Run the Ivanti Integrity Checker Tool and retain the output; a clean ICT result after patching is not proof the appliance was clean before it.
- Terminate all active sessions and revoke session tokens after upgrade - assume session cookies were stolen.
- Rotate local admin credentials, service accounts, LDAP/AD bind accounts, device certificates and any secrets stored on the appliance.
- Remove the administrative interface from internet reachability; restrict it to a management VLAN or out-of-band network with source ACLs.
- Enforce phishing-resistant MFA plus geo/ASN conditional access on the remote access portal.
- Record the patched/mitigated decision and the triage outcome against the device group in the change record.

**Legacy / unpatchable gear.** End-of-life Pulse Connect Secure and out-of-support ICS hardware will not receive these fixes. Take them out of the inbound path now: deny untrusted inbound access at an upstream supported firewall, move users onto a supported gateway, and treat the old unit as compromised rather than merely unpatched. If it must stay online briefly, strict source ACLs, no direct management access, a dedicated jump host and full session logging, with a documented replacement date and budget owner.

**Detection.** Triage back to at least 2026-10-01: unexpected or newly created local accounts, files written to known web-accessible paths on disk, modified or added web resources and login pages, gaps or anomalous timestamps in /var logs, unexpected outbound connections from the appliance, and authentications succeeding without a corresponding MFA event. In the SIEM, alert on any admin login, config change or firmware change on the gateway.

**AI angle.** No AI element reported in this item.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/ivanti-patches-two-more-zero-days-exploited-in-connect-secure-attack-chain/)**

#### 🔴 CRITICAL — Ivanti patches two more Connect Secure zero-days chained in attacks

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/ivanti-patches-two-more-connect-secure-zero-days-chained-in-attacks/) · 2026-11-02* 

> Second confirmation that Ivanti shipped fixes for two Connect Secure gateway flaws chained in zero-day attacks running since at least 1 October 2026.

**Affected:** Ivanti Connect Secure (ICS) remote access gateways. The source does not state fixed builds - check the vendor advisory for the fixed release.  

**What happened.** BleepingComputer reports Ivanti released security updates for two more Connect Secure remote access gateway flaws that have been chained in zero-day attacks since at least 1 October 2026. Same event as item 0; this source supplies the exploitation start date.

**Why it matters.** The 1 October start date sets the forensic window. If your appliance was internet-facing and unpatched through October, patch-only remediation leaves a month of potential attacker activity uninvestigated.

**Recommended actions**

- Use 2026-10-01 as the earliest date for log review and forensic triage on every internet-facing ICS appliance.
- Pull and preserve appliance logs, config exports and ICT output before the upgrade overwrites them.
- Patch to the vendor's fixed build, then re-run the Integrity Checker Tool.
- Diff the exported configuration against your known-good baseline for unexpected admin users, auth servers, roles or modified web resources.
- Rotate credentials and certificates after patching and force re-enrolment of client sessions.

**Legacy / unpatchable gear.** Unsupported gateways get no fix: remove them from the inbound path, front them with a supported inspecting firewall that denies untrusted inbound access, and schedule replacement with a named owner.

**Detection.** Compare authentication volumes and source ASNs against your October baseline; look for sessions that authenticated outside business hours, accounts active after their owner left, and VPN sessions whose internal reach differs from the user's normal pattern.

**AI angle.** No AI element reported in this item.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/ivanti-patches-two-more-connect-secure-zero-days-chained-in-attacks/)**

---

## Unspecified

*1 item(s) — 1 medium*

### VPN gateways

#### 🟡 MEDIUM — Ransomware gang talked AI agents into doing much of its dirty work

*[Help Net Security](https://www.helpnetsecurity.com/2026/11/02/aur0ra-ransomware-ai-agents/) · 2026-11-02* `AI`

> Sophos found the Aur0ra ransomware group giving AI agents VPN logins and proxy access, letting them map networks, harvest credentials and reach domain admin.

**Affected:** No specific product vulnerability. The exposure is credential-based remote access on any VPN gateway, plus flat internal networks reachable from a VPN session.  

**What happened.** Sophos reports that the operators of the Aur0ra ransomware, tracked as GOLD OGRE, used AI agents for much of their intrusion work. Artifacts recovered from the Cursor AI coding assistant in late August show the attackers supplying the agents with VPN logins, proxy access, targets and objectives; the agents then mapped the network, collected credentials and escalated toward domain administrator access.

**Why it matters.** The initial access was a valid VPN login, and what followed happened at machine pace. Detection tuned to human-speed reconnaissance, and VPN gateways protected by passwords alone, are both outmatched by this workflow.

**Recommended actions**

- Audit every account that can authenticate to a remote access gateway and enforce phishing-resistant MFA on all of them, including contractor and service accounts.
- Apply geo/ASN and device-posture conditional access on VPN portals; alert on first-seen ASN per user.
- Build detection for post-login enumeration bursts: one VPN session performing broad port scanning, LDAP/SMB enumeration across many hosts within minutes, or touching an abnormal number of VLANs.
- Segment the VPN pool away from domain controllers and from network-device management planes; require a logged jump host for administrative access.
- Rotate any VPN credential set that is shared, static or lacks MFA, and disable concurrent sessions per account where the gateway supports it.

**Legacy / unpatchable gear.** Legacy concentrators that cannot enforce phishing-resistant MFA should be fronted by an identity-aware proxy or retired, because a stolen password is the whole attack path here. Until then, restrict to known source ranges, deny any route to management networks, baseline NetFlow and alert on new outbound flows.

**Detection.** Alert on VPN authentication success without an MFA assertion; on a single VPN source IP contacting more than a handful of internal hosts per minute; on new admin group membership within hours of a VPN login; and on outbound connections to AI API endpoints from servers that have no business reaching them.

**AI angle.** This is the AI item of the cycle: AI agents were driven as the intrusion operator, given VPN credentials and objectives, compressing network mapping and privilege escalation into machine-paced activity. It is an attacker-side capability with no defensive product attached.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/11/02/aur0ra-ransomware-ai-agents/)**

---

## Synology

*1 item(s) — 1 watch*

### Other

#### ⚪ WATCH — Synology warns of C3RB3R ransomware attacks targeting NAS devices

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/synology-warns-customers-of-ongoing-c3rb3r-ransomware-attacks/) · 2026-11-02* 

> Synology told customers to update NAS appliances immediately as C3RB3R ransomware attacks against the devices surged.

**Affected:** Synology network-attached storage appliances. The source names no CVE and no fixed firmware version - check the vendor advisory for the fixed release.  

**What happened.** Synology warned customers to update their NAS devices immediately because of a surge in C3RB3R ransomware attacks against them.

**Why it matters.** Storage rather than network infrastructure, so it is background for this audience - but these appliances are usually reachable because someone added a port forward or UPnP mapping on the edge router, which makes the exposure a network-team problem even when the device is not.

**Recommended actions**

- Query the edge firewall and router config for port forwards, NAT rules and UPnP mappings that expose NAS management or file services to the internet, and remove them.
- Disable UPnP on edge routers so appliances cannot open their own inbound holes.
- Put NAS appliances in a dedicated VLAN with no inbound access from untrusted zones and no outbound internet access beyond what they need for updates.
- Apply the firmware update Synology points to in its advisory; do not assume a version number.
- Verify that backups of the NAS are on separate, offline or immutable storage that a compromise of the NAS cannot reach.

**Legacy / unpatchable gear.** Out-of-support NAS models receive no firmware fix: take them off any internet-reachable path entirely, restrict access to a single internal subnet via firewall ACL, and treat them as high-risk assets with NetFlow monitoring until replaced.

**Detection.** Alert on inbound connections from the internet to NAS ports at the firewall, on new NAT or port-forward rules appearing in edge device configs, and on unusual outbound traffic volumes from the storage VLAN.

**AI angle.** No AI element reported in this item.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/synology-warns-customers-of-ongoing-c3rb3r-ransomware-attacks/)**

---

## 🧠 AI & network devices — Aur0ra ransomware crew handed AI agents VPN logins and let them map the network

Sophos reports that the group behind the Aur0ra ransomware, which it tracks as GOLD OGRE, has been using AI agents to do much of its lateral movement - handing the agents VPN logins, proxy access, targets and objectives, after which the agents mapped the network, collected credentials and worked up to domain administrator. Sophos recovered the artifacts from the Cursor AI coding assistant in late August.

For a network team the significant detail is the entry point: a valid VPN login was enough to start the chain, and what followed was machine-paced reconnaissance rather than a human clicking around. Credential-based remote access and the time between first login and privilege escalation are the two things that got compressed. Detection logic that assumes a human-speed operator - and patch windows that assume slow exploitation - are both calibrated wrong.

This is an attacker-side use of AI. The report describes no defensive product, so there is nothing to evaluate here, only a tempo change to account for.

**Vendors with an AI angle in this edition:** Ivanti, Synology, Unspecified

**Preventing it on current systems**

- Enforce phishing-resistant MFA on every remote access gateway account, including contractor, service and break-glass accounts - a stolen password alone should not start a session.
- Add geo/ASN and device-posture conditional access on VPN portals, and alert on a successful authentication from a new ASN for an existing user.
- Alert on machine-paced behaviour after VPN login: broad internal port scanning, SMB/LDAP enumeration across many hosts in minutes, or a single VPN session touching an unusual number of subnets.
- Segment so that a VPN session cannot reach domain controllers or network-device management planes directly - force traffic through a jump host with full session logging.
- Shorten the patch window for internet-facing network devices to days, not months; assume exploit development and mass scanning now outpace your historical SLA.
- Treat any AI/LLM or agent integration on network gear (assistants, AIOps, MCP connectors) as a privileged admin path: scoped read-only credentials, no unattended config write, full audit trail.

**Preventing it on legacy / end-of-life systems**

- Legacy VPN concentrators that cannot do phishing-resistant MFA should be fronted by a supported identity-aware proxy or retired - password-only remote access is the exact door this chain walked through.
- Where an old gateway must stay, restrict it to known source ranges, deny it any route to management networks, and put a NetFlow baseline on it with alerting on new outbound flows.
- Set a replacement date and a budget owner, and record the accepted risk until the device is gone.

---

<!--index
{
 "url": "/2026/11/02/network-security-digest/",
 "date": "2026-11-02",
 "entries": [
  {
   "title": "Ivanti Patches Two More Zero-Days Exploited in Connect Secure Attack Chain",
   "link": "https://www.securityweek.com/ivanti-patches-two-more-zero-days-exploited-in-connect-secure-attack-chain/",
   "source": "SecurityWeek",
   "relevance": "critical",
   "device_types": [
    "vpn-gateway"
   ],
   "vendors": [
    "Ivanti"
   ],
   "cves": [
    "CVE-2026-111357",
    "CVE-2026-118527"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "No AI element reported in this item.",
   "summary": "CISA added Ivanti Connect Secure flaws CVE-2026-111357 and CVE-2026-118527 to KEV after both were chained in zero-day attacks on the VPN gateway.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Ivanti patches two more Connect Secure zero-days chained in attacks",
   "link": "https://www.bleepingcomputer.com/news/security/ivanti-patches-two-more-connect-secure-zero-days-chained-in-attacks/",
   "source": "BleepingComputer",
   "relevance": "critical",
   "device_types": [
    "vpn-gateway"
   ],
   "vendors": [
    "Ivanti"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "No AI element reported in this item.",
   "summary": "Second confirmation that Ivanti shipped fixes for two Connect Secure gateway flaws chained in zero-day attacks running since at least 1 October 2026.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Ransomware gang talked AI agents into doing much of its dirty work",
   "link": "https://www.helpnetsecurity.com/2026/11/02/aur0ra-ransomware-ai-agents/",
   "source": "Help Net Security",
   "relevance": "medium",
   "device_types": [
    "vpn-gateway"
   ],
   "vendors": [
    "Unspecified"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "This is the AI item of the cycle: AI agents were driven as the intrusion operator, given VPN credentials and objectives, compressing network mapping and privilege escalation into machine-paced activity. It is an attacker-side capability with no defensive product attached.",
   "summary": "Sophos found the Aur0ra ransomware group giving AI agents VPN logins and proxy access, letting them map networks, harvest credentials and reach domain admin.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Synology warns of C3RB3R ransomware attacks targeting NAS devices",
   "link": "https://www.bleepingcomputer.com/news/security/synology-warns-customers-of-ongoing-c3rb3r-ransomware-attacks/",
   "source": "BleepingComputer",
   "relevance": "watch",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Synology"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "No AI element reported in this item.",
   "summary": "Synology told customers to update NAS appliances immediately as C3RB3R ransomware attacks against the devices surged.",
   "ai_defense": false,
   "defense": null
  }
 ]
}
-->

## How this was produced

- Feeds polled: 23 ok, 0 failed
- Raw items: 709 → in window: 42 → network-device relevant: 8 → published: 4
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
