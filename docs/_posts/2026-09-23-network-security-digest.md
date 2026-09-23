---
layout: post
title: "Network Device Security Digest - 2026-09-23"
date: 2026-09-23 11:15:54 +0000
edition: daily
critical_count: 6
item_count: 14
kev: "CVE-2026-85102, CVE-2026-93616, CVE-2026-93952, CVE-2026-94127"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-09-23

*daily edition · window 24h · 22 feeds · 14 items · 6 critical · generated 2026-09-23 11:15 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — Check Point Security Management Server zero-day exploited since July; gateway pre-auth RCE under attack alongside it

Check Point has released emergency Jumbo hotfixes for CVE-2026-93616, a pre-authentication path traversal in Security Management Server that lets an unauthenticated attacker reach the server's web service and upload and execute arbitrary scripts. Check Point Research says the flaw was exploited in a small number of targeted attacks as far back as July 23, 2026, and published the fix on September 22. CISA added it to the KEV catalog on September 22 with a remediation due date of September 25. Per the KEV entry, the same path traversal covers Multi-Domain Security Management Server, Log Server, Multi-Domain Log Server and SmartEvent.

In the same advisory Check Point confirms active exploitation of CVE-2026-85102, an improper certificate validation flaw in Security Gateway and Spark firewalls running Site-to-Site or Remote Access VPN that allows unauthenticated remote code execution on the gateway. A fix for that one has been available since September 9; Check Point says probing began within days of the patch release, so anyone still on the pre-September-9 build has been exposed for two weeks. It too is in KEV with a September 25 due date.

This leads the cycle because the target is the management plane, not a single appliance. A compromised Security Management Server controls firewall policy for every gateway it manages: an attacker running scripts there can push rules, read the whole estate's configuration, and harvest credentials and certificates for devices that were never themselves vulnerable. Treat management-server compromise as estate-wide compromise until logs prove otherwise.

Do the management server first, then the gateways. The July 23 exploitation date matters for scoping: patching closes the hole but does not evict anyone who got in during the preceding two months, so the credential rotation and configuration review are not optional follow-ups.

**Do this first**

- Apply the Check Point Jumbo hotfix for CVE-2026-93616 to Security Management Server, Multi-Domain Security Management Server, Log Server, Multi-Domain Log Server and SmartEvent now - the KEV due date is 2026-09-25.
- Confirm every Security Gateway and Spark firewall carries the CVE-2026-85102 fix released on September 9; if the build predates it, treat the device as having been exposed to active exploitation.
- Remove the management server's web service from any internet-reachable path and restrict it to a dedicated management VLAN or out-of-band network with an explicit source ACL.
- Assume pre-patch compromise back to 2026-07-23: rotate management and gateway admin credentials, API keys, certificates and VPN pre-shared keys after the upgrade.
- Export and diff the policy database and gateway configurations against a known-good backup, looking for unexpected admin users, rules, scripts or modified login pages.
- Review management-server web service logs and SmartEvent for script uploads and unauthenticated requests dating back to July 2026, and enforce MFA on all administrative access.

---

> **On the CISA KEV catalog in this edition:** CVE-2026-85102, CVE-2026-93616, CVE-2026-93952, CVE-2026-94127 — treat these as confirmed-exploited and patch on an emergency change.

## Executive summary

- Check Point patched CVE-2026-93616, a pre-auth script upload and execution flaw in Security Management Server exploited in targeted attacks since 2026-07-23; KEV due date 2026-09-25.
- Check Point also confirms active exploitation of CVE-2026-85102, unauthenticated RCE on Security Gateway and Spark firewalls running Site-to-Site or Remote Access VPN; fix available since 2026-09-09.
- F5 disclosed CVE-2026-94127 on 2026-09-22, a CVSS 9.8 heap overflow giving unauthenticated RCE on BIG-IP APM - only where a virtual server has both an APM access policy and an OAuth profile. Engineering hotfixes released; exploited as a zero-day, now in KEV.
- Arista reported active exploitation of CVE-2026-93952 in on-premises VeloCloud Orchestrator (CVSS 10.0), affecting only orchestrators that authenticate Edges with certificates. In KEV, due 2026-09-25.
- D-Link disclosed a maximum-severity flaw in legacy DIR-822A Wi-Fi routers, CVE-2026-86296, with public PoC code and no patch - replacement or isolation is the only answer.
- A Chinese-speaking threat actor exploited Zyxel GS1900 smart managed switches, reaching 996 devices and more than 18,500 records in backend databases.
- Three of the four exploited flaws this cycle hit management and orchestration platforms rather than data-plane devices; prioritise the controllers before the edge.

---

## Items by vendor

## Check Point

*4 item(s) — 2 critical, 2 high*

### Management platforms

#### 🔴 CRITICAL — Check Point Warns of Management Server Zero-Day Exploited in Targeted Attacks

*[The Hacker News](https://thehackernews.com/2026/09/check-point-warns-of-management-server.html) · 2026-09-22* `KEV`

> Check Point's Security Management Server carried a zero-day, CVE-2026-93616, letting unauthenticated attackers run scripts on the server that controls firewall policy.

**Affected:** Check Point Security Management Server; per the KEV entry also Multi-Domain Security Management Server, Log Server, Multi-Domain Log Server and SmartEvent. Fix released 2026-09-22.  
**CVEs:** [CVE-2026-93616](https://nvd.nist.gov/vuln/detail/CVE-2026-93616) **[KEV]**  

**What happened.** Check Point says attackers exploited a previously unknown flaw in its Security Management Server in a handful of targeted attacks on 2026-07-23. CVE-2026-93616 allows an attacker who can reach the server's web service to run scripts on it without logging in. The fix shipped on 2026-09-22.

**Why it matters.** This is the box that holds policy for the whole firewall estate. Script execution there means the attacker can read every rule and object, push changes to managed gateways, and take credentials and certificates for devices that had no vulnerability of their own. The July exploitation date means a two-month dwell window for anyone targeted.

**Recommended actions**

- Install the Check Point hotfix on the management server, multi-domain servers, log servers and SmartEvent - the KEV due date is 2026-09-25.
- Restrict the management web service to a dedicated management VLAN or out-of-band network; it should not be reachable from user or internet zones.
- Rotate management admin credentials, API keys and the certificates used to authenticate managed gateways after patching.
- Diff the policy database and object store against a known-good backup from before 2026-07-23, looking for added admin users, rules or scripts.
- Enforce MFA on all administrative access to the management server and remove shared local admin accounts.

**Legacy / unpatchable gear.** A management server on an unsupported release that cannot take the hotfix must be removed from all untrusted reachability at once - jump-host access only, full session logging - and the upgrade treated as an emergency project, not a scheduled one.

**Detection.** Search management web service logs from July 2026 onward for unauthenticated requests and file uploads, and alert on script execution, new administrator objects and policy installs outside change windows.

**AI angle.** None stated.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/09/check-point-warns-of-management-server.html)**

#### 🔴 CRITICAL — Security Advisory – Action Required – Active Exploitation of CVE-2026-85102 and a Management Pre-Authentication Vulnerability CVE-2026-93616

*[Check Point Blog](https://blog.checkpoint.com/security/security-advisory-action-required-active-exploitation-of-cve-2026-85102-and-a-management-pre-authentication-vulnerability-cve-2026-93616/) · 2026-09-22* `KEV` `AI` `AI-defense`

> Check Point's own advisory confirms active exploitation of two flaws: CVE-2026-85102 on Security Gateway and the new pre-auth CVE-2026-93616 in Security Management.

**Affected:** Check Point Security Gateway and Security Management. Fix for CVE-2026-85102 available since 2026-09-09; fix for CVE-2026-93616 available now as part of the Jumbo hotfix stream.  
**CVEs:** [CVE-2026-85102](https://nvd.nist.gov/vuln/detail/CVE-2026-85102) **[KEV]**, [CVE-2026-93616](https://nvd.nist.gov/vuln/detail/CVE-2026-93616) **[KEV]**  

**What happened.** Check Point Research identified active exploitation of two vulnerabilities. CVE-2026-85102 has had a fix since 2026-09-09 and customers who applied it are already protected. CVE-2026-93616 is a newly discovered pre-authentication zero-day in Security Management, with a fix available now in the Jumbo hotfix release.

**Why it matters.** This is the vendor's own confirmation, which is what makes the two CVEs actionable rather than reported. The pairing is what matters: one flaw on the gateways, one on the server that manages them, both under exploitation at the same time.

**Recommended actions**

- Install the current Jumbo hotfix on Security Management to close CVE-2026-93616; per the KEV entry the remediation date is 2026-09-25.
- Verify every Security Gateway carries the CVE-2026-85102 fix available since 2026-09-09 - the vendor states applied customers are already protected.
- Restrict the management web service and gateway administrative access to a management VLAN or out-of-band network; enforce MFA on all admin and VPN accounts.
- Rotate admin credentials, API keys, certificates and VPN pre-shared keys after the hotfix; certificate validation was the failure mode in CVE-2026-85102, so treat VPN trust material as suspect.
- Export and review gateway and management configuration for unexpected admin users, scripts, static routes or modified login pages.

**Legacy / unpatchable gear.** Gateways and management servers on releases no longer receiving Jumbo hotfixes cannot be fixed here. Terminate Site-to-Site and Remote Access VPN on a supported device, deny inbound access to the unsupported unit from untrusted zones, manage it only through a logged jump host, and set a replacement date with a budget owner.

**Detection.** Search management web service logs back to July 2026 for unauthenticated requests and uploads; on gateways, alert on VPN peer certificate validation anomalies, new tunnels, and administrative logins from unexpected sources.

**AI angle.** The advisory frames these hotfixes as part of Check Point's Frontier AI Readiness Program, but the vulnerabilities themselves have no stated AI involvement.

📄 **[Read the full report at Check Point Blog →](https://blog.checkpoint.com/security/security-advisory-action-required-active-exploitation-of-cve-2026-85102-and-a-management-pre-authentication-vulnerability-cve-2026-93616/)**

#### 🟠 HIGH — Check Point Patches Exploited Management Server Zero-Day

*[SecurityWeek](https://www.securityweek.com/check-point-patches-exploited-management-server-zero-day/) · 2026-09-23* 

> Check Point patched an exploited Management Server zero-day that let unauthenticated attackers upload and execute arbitrary scripts.

**Affected:** Check Point Security Management Server. This report names no CVE or version; see the Check Point advisory for CVE-2026-93616.  

**What happened.** SecurityWeek reports Check Point patched a critical-severity Management Server flaw, already exploited, that allows unauthenticated attackers to upload and execute arbitrary scripts.

**Why it matters.** Corroborates the management-plane compromise path. Upload-plus-execute without authentication is as direct as it gets on a server that holds the firewall policy for the estate.

**Recommended actions**

- Patch per the CVE-2026-93616 workstream; check the vendor advisory for the fixed release.
- Restrict the management web service to an out-of-band or management-only network.
- Rotate management credentials and gateway-authentication certificates after patching.
- Review the server filesystem and web service logs for uploaded scripts.

**Legacy / unpatchable gear.** Unsupported management server releases that cannot take the hotfix should be isolated to jump-host access only with full session logging, pending an emergency upgrade.

**Detection.** Alert on file writes into web-accessible directories on the management server and on any script execution by the web service account.

**AI angle.** None stated.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/check-point-patches-exploited-management-server-zero-day/)**

#### 🟠 HIGH — Check Point warns of Management Server zero-day exploited in attacks

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/check-point-patches-management-server-zero-day-exploited-in-attacks/) · 2026-09-22* 

> Check Point issued emergency hotfixes for a critical Security Management Server flaw that could let attackers run arbitrary scripts.

**Affected:** Check Point Security Management Server. This report names no CVE or version; see the Check Point advisory for CVE-2026-93616.  

**What happened.** BleepingComputer reports Check Point released emergency hotfixes for a critical Security Management Server vulnerability allowing attackers to run arbitrary scripts.

**Why it matters.** Emergency hotfix release outside the normal cadence is the vendor's own signal of urgency; treat it as an out-of-cycle change, not a queued one.

**Recommended actions**

- Install the emergency hotfix on management servers now; check the vendor advisory for the fixed release.
- Keep the management web service off any internet-facing interface.
- Rotate admin credentials and certificates post-upgrade.
- Compare the current policy package against a pre-incident backup.

**Legacy / unpatchable gear.** If the management server release is out of support and the hotfix does not apply, isolate it behind strict ACLs with no direct management access and prioritise the upgrade.

**Detection.** Monitor for arbitrary script execution by the management web service and for administrator objects created outside change windows.

**AI angle.** None stated.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/check-point-patches-management-server-zero-day-exploited-in-attacks/)**

---

## F5

*4 item(s) — 2 critical, 2 high*

### Load balancers

#### 🔴 CRITICAL — F5 Patches Critical BIG-IP APM Zero-Day Exploited for Unauthenticated RCE on OAuth Servers

*[The Hacker News](https://thehackernews.com/2026/09/f5-patches-critical-big-ip-apm-zero-day.html) · 2026-09-23* `KEV`

> F5 says attackers are exploiting CVE-2026-94127 in BIG-IP APM to run code without logging in, on systems where APM acts as an OAuth authorization server.

**Affected:** F5 BIG-IP Access Policy Manager where APM serves as an OAuth authorization server issuing access tokens to applications. Disclosed 2026-09-22.  
**CVEs:** [CVE-2026-94127](https://nvd.nist.gov/vuln/detail/CVE-2026-94127) **[KEV]**  

**What happened.** F5 disclosed CVE-2026-94127 in an advisory on 2026-09-22 and released engineering hotfixes. The flaw allows an attacker to execute code on a BIG-IP system without authenticating, and F5 states it is being exploited.

**Why it matters.** BIG-IP APM is the front door for application access. Unauthenticated RCE there gives the attacker the TLS termination point, the token issuer and a position inside the DMZ in one step - and session tokens issued by a compromised authorization server cannot be trusted afterwards.

**Recommended actions**

- Apply F5's engineering hotfix to all units in the device group and verify config sync after upgrade; check the vendor advisory for the fixed release that applies to your branch.
- Inventory virtual servers for the exposed combination - an APM access policy together with an OAuth profile - and prioritise those; other configurations are not affected.
- Remove the management/TMUI interface from public exposure entirely and restrict it to the management network.
- Rotate device certificates, admin credentials, API tokens and OAuth signing keys after patching, and invalidate outstanding access tokens.
- Review iRules, access policies and virtual server configuration for unauthorized changes.

**Legacy / unpatchable gear.** BIG-IP versions past end of technical support will not get the hotfix. Remove the OAuth profile from the affected virtual server as an interim measure, move OAuth issuance to a supported platform, and isolate the appliance behind an inspecting firewall until it is replaced.

**Detection.** Look for anomalous requests to OAuth endpoints on APM virtual servers, unexpected process restarts or core dumps on the BIG-IP, new local accounts, and outbound connections initiated by the appliance itself.

**AI angle.** None stated.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/09/f5-patches-critical-big-ip-apm-zero-day.html)**

#### 🔴 CRITICAL — CVE-2026-94127: Critical Unauthenticated RCE in F5 BIG-IP APM

*[Rapid7 Blog](https://www.rapid7.com/blog/post/etr-cve-2026-94127-critical-unauthenticated-rce-in-f5-big-ip-apm) · 2026-09-23* `KEV`

> Rapid7 details CVE-2026-94127 as a CVSS 9.8 heap-based buffer overflow in BIG-IP APM, reachable only when a virtual server has both an access policy and an OAuth profile.

**Affected:** F5 BIG-IP Access Policy Manager. Not exposed in a default configuration: exploitation requires a virtual server with both an APM access policy and an OAuth profile configured.  
**CVEs:** [CVE-2026-94127](https://nvd.nist.gov/vuln/detail/CVE-2026-94127) **[KEV]**  

**What happened.** F5 published its advisory for CVE-2026-94127 on 2026-09-22; Rapid7's analysis describes a critical heap-based buffer overflow with CVSS v3.1 9.8, where an unauthenticated attacker with network access to an affected virtual server may achieve RCE by sending specially crafted traffic.

**Why it matters.** The precondition narrows the exposed population, which makes triage tractable - but affected BIG-IP systems typically process traffic at the network edge, so the ones that do match are exactly the internet-facing units.

**Recommended actions**

- Enumerate virtual servers and filter for the pairing of an APM access policy with an OAuth profile; that list is your patch queue.
- Apply the F5 hotfix across the device group and verify config sync; check the vendor advisory for the fixed release for your branch.
- Where a maintenance window is not available immediately, evaluate removing the OAuth profile from the exposed virtual server as a documented interim workaround.
- Rotate admin credentials, API tokens and device certificates after patching.
- Record the decision (patched / mitigated / accepted risk) against each device group in the change record.

**Legacy / unpatchable gear.** For BIG-IP units out of support, the configuration-level mitigation is the only lever available: drop the OAuth profile from affected virtual servers, front the appliance with an inspecting firewall, and schedule replacement.

**Detection.** Monitor for malformed traffic to APM OAuth virtual servers, TMM crashes or core files, and any process spawning from the data-plane path.

**AI angle.** None stated.

📄 **[Read the full report at Rapid7 Blog →](https://www.rapid7.com/blog/post/etr-cve-2026-94127-critical-unauthenticated-rce-in-f5-big-ip-apm)**

#### 🟠 HIGH — Critical F5 BIG-IP Vulnerability Exploited as Zero-Day

*[SecurityWeek](https://www.securityweek.com/critical-f5-big-ip-vulnerability-exploited-as-zero-day/) · 2026-09-23* 

> SecurityWeek confirms the critical F5 BIG-IP flaw was exploited as a zero-day, with unauthenticated attackers reaching remote code execution via malicious traffic.

**Affected:** F5 BIG-IP. This report names no CVE or fixed version; see the F5 advisory of 2026-09-22 for CVE-2026-94127.  

**What happened.** SecurityWeek reports that unauthenticated attackers could send malicious traffic to BIG-IP to achieve remote code execution, and that the flaw was exploited as a zero-day before the fix.

**Why it matters.** Independent confirmation that exploitation preceded disclosure. That removes the option of waiting for the next scheduled change window on internet-facing BIG-IP units.

**Recommended actions**

- Treat this as corroboration of the F5 advisory rather than a separate issue; patch per the CVE-2026-94127 workstream.
- Check the vendor advisory for the fixed release - this report states no version.
- Confirm the BIG-IP management interface is off the public internet.
- Rotate admin credentials and device certificates after the upgrade given pre-disclosure exploitation.

**Legacy / unpatchable gear.** Unsupported BIG-IP units get no hotfix: front them with a supported inspecting device, remove public exposure of the affected virtual servers, and document the accepted risk with a replacement date.

**Detection.** Alert on malformed traffic to BIG-IP virtual servers, appliance process crashes, and outbound connections sourced from the appliance.

**AI angle.** None stated.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/critical-f5-big-ip-vulnerability-exploited-as-zero-day/)**

#### 🟠 HIGH — F5 patches BIG-IP APM zero-day flaw exploited in RCE attacks

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/f5-warns-of-big-ip-apm-remote-code-execution-zero-day-exploited-in-attacks/) · 2026-09-23* 

> F5 shipped security updates for the BIG-IP APM zero-day being used in remote code execution attacks.

**Affected:** F5 BIG-IP APM. This report names no CVE or fixed version; see the F5 advisory of 2026-09-22 for CVE-2026-94127.  

**What happened.** BleepingComputer reports F5 released security updates addressing a critical BIG-IP APM zero-day that is being exploited in remote code execution attacks.

**Why it matters.** Fixes exist, which moves this from mitigation-only to a scheduling problem. The exposed configuration is narrow but sits at the edge.

**Recommended actions**

- Apply the F5 updates to all units in the device group and verify config sync afterwards; check the vendor advisory for the fixed release.
- Prioritise internet-facing APM virtual servers with an OAuth profile configured.
- Rotate admin credentials, API tokens and certificates after the upgrade.
- Review access policies and virtual server configuration for unauthorized changes.

**Legacy / unpatchable gear.** Where the BIG-IP cannot be updated, remove the affected virtual server from public exposure and move the function to a supported platform; keep the unit behind an inspecting firewall until replacement.

**Detection.** Monitor APM authentication logs for sessions with no matching access request, and alert on new local accounts or scheduled tasks on the appliance.

**AI angle.** None stated.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/f5-warns-of-big-ip-apm-remote-code-execution-zero-day-exploited-in-attacks/)**

---

## Arista

*2 item(s) — 1 critical, 1 high*

### SD-WAN / SASE

#### 🔴 CRITICAL — New CVSS 10.0 VeloCloud Orchestrator Flaw Actively Exploited in Certificate-Based Setups

*[The Hacker News](https://thehackernews.com/2026/09/new-cvss-100-velocloud-orchestrator.html) · 2026-09-22* `KEV`

> Arista reports active exploitation of CVE-2026-93952, a CVSS 10.0 input validation flaw in on-premises VeloCloud Orchestrator, in certificate-authenticated deployments.

**Affected:** On-premises Arista VeloCloud Orchestrator (VCO), the server managing Edge devices in a VeloCloud SD-WAN. Per the source, only orchestrators configured to authenticate their Edges with certificates.  
**CVEs:** [CVE-2026-93952](https://nvd.nist.gov/vuln/detail/CVE-2026-93952) **[KEV]**  

**What happened.** Arista said on 2026-09-22 that attackers are exploiting CVE-2026-93952 in on-premises VeloCloud Orchestrator. The flaw may allow a remote attacker with no login access to reach privileged internal functionality and affect the VCO host. CISA added it to KEV on 2026-09-22 with a 2026-09-25 due date.

**Why it matters.** The orchestrator pushes configuration and policy to every SD-WAN Edge in the fabric. An attacker who controls it can reconfigure branch connectivity, redirect or mirror traffic, and reach every site at once - the widest blast radius available in an SD-WAN estate.

**Recommended actions**

- Patch the on-premises orchestrator before touching Edge devices, and validate certificate trust after the upgrade; check the vendor advisory for the fixed release.
- Determine whether your VCO authenticates Edges with certificates - that is the exposed configuration per Arista.
- Restrict orchestrator access to management networks only and enforce MFA on the orchestrator console.
- Audit Edge templates for insecure overrides pushed during the exploitation window, and verify tunnel authentication.
- Rotate orchestrator admin credentials, API tokens and any certificates issued or held by the VCO after patching.

**Legacy / unpatchable gear.** An out-of-support on-premises VCO that cannot take the fix should be taken off any internet-reachable path immediately, reachable only through a jump host with full session logging, while migration to a supported orchestrator version is scheduled.

**Detection.** Alert on unauthenticated access to VCO internal endpoints, on Edge configuration pushes that no change record accounts for, and on new certificates or Edge enrolments in the orchestrator's audit log.

**AI angle.** None stated.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/09/new-cvss-100-velocloud-orchestrator.html)**

#### 🟠 HIGH — Arista Urges Immediate Patching of Exploited VCO Zero-Day

*[SecurityWeek](https://www.securityweek.com/arista-urges-immediate-patching-of-exploited-vco-zero-day/) · 2026-09-23* 

> Arista is urging immediate patching of an exploited VeloCloud Orchestrator zero-day that gives remote attackers privileged internal functionality.

**Affected:** Arista VeloCloud Orchestrator. This report names no CVE or fixed version; see the Arista advisory for CVE-2026-93952.  

**What happened.** SecurityWeek reports Arista is urging immediate patching of a critical-severity VCO flaw, exploited in the wild, that lets remote attackers reach privileged internal functionality.

**Why it matters.** Reinforces the urgency on the SD-WAN control plane. An orchestrator compromise propagates to every Edge it manages.

**Recommended actions**

- Patch the orchestrator ahead of Edge devices; check the vendor advisory for the fixed release.
- Restrict orchestrator access to management networks and enforce MFA on the console.
- Audit Edge templates and configuration pushes against the change record.
- Rotate orchestrator credentials and API tokens after the upgrade.

**Legacy / unpatchable gear.** An unsupported on-premises orchestrator should be removed from internet reachability entirely and accessed only via a logged jump host until it is upgraded or migrated.

**Detection.** Alert on privileged internal API calls without a preceding authentication event and on Edge configuration changes outside change windows.

**AI angle.** None stated.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/arista-urges-immediate-patching-of-exploited-vco-zero-day/)**

---

## Multiple

*1 item(s) — 1 critical*

### Management platforms

#### 🔴 CRITICAL — Attackers hit Check Point Management Servers and Spark firewalls, F5 BIG-IP APM instances

*[Help Net Security](https://www.helpnetsecurity.com/2026/09/23/check-point-f5-big-ip-apm-zero-days-targeted/) · 2026-09-23* `KEV`

> Check Point and F5 are both responding to in-the-wild exploitation: two Check Point CVEs added to KEV, plus an unauthenticated RCE zero-day in F5 BIG-IP APM.

**Affected:** Check Point Security Management Server and Quantum Security Gateway / Spark firewalls (CVE-2026-93616, CVE-2026-85102); F5 BIG-IP APM instances (CVE-2026-94127).  
**CVEs:** [CVE-2026-85102](https://nvd.nist.gov/vuln/detail/CVE-2026-85102) **[KEV]**, [CVE-2026-93616](https://nvd.nist.gov/vuln/detail/CVE-2026-93616) **[KEV]**  

**What happened.** Check Point released emergency fixes for CVE-2026-93616 in Security Management Server, which the company says has been exploited as far back as 2026-07-23, and confirmed that CVE-2026-85102, the pre-authentication RCE in Quantum Security Gateway patched on 2026-09-09, began getting probed a few days after that patch shipped. The same report covers F5 BIG-IP APM instances under attack.

**Why it matters.** Two separate perimeter vendors are under active exploitation in the same window, and in both cases the attacker reaches code execution without authenticating. The Check Point management server sits above the firewall estate, so its compromise is not contained to one appliance.

**Recommended actions**

- Patch the Check Point Security Management Server first, then gateways; then the BIG-IP APM units - management and access control planes before data plane.
- Verify each Check Point gateway is on the 2026-09-09 or later build that fixes CVE-2026-85102; anything older has been exposed to probing for two weeks.
- Remove management access (HTTPS/SSH/API) from internet-facing interfaces on all three product lines; restrict to a management VLAN or out-of-band network.
- Rotate admin credentials, API keys, certificates and VPN pre-shared keys on every affected device after patching - a pre-patch compromise survives the upgrade.
- Export and review configurations for unexpected admin users, scripts, static routes or modified login pages.

**Legacy / unpatchable gear.** Check Point Spark appliances and BIG-IP hardware past end of support will not receive these hotfixes. Deny inbound access from untrusted zones, terminate remote-access VPN on a supported device instead, and set a replacement date with a named budget owner.

**Detection.** Alert on unauthenticated requests and script uploads to the Check Point management web service back to July 2026; on BIG-IP, alert on crashes or restarts of the APM/OAuth processing path and on unexpected outbound connections from the appliance.

**AI angle.** None stated.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/09/23/check-point-f5-big-ip-apm-zero-days-targeted/)**

---

## D-Link

*1 item(s) — 1 high*

### Routers

#### 🟠 HIGH — D-Link warns of max severity zero-day bug in DIR-822A routers

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/d-link-warns-of-max-severity-zero-day-bug-in-dir-822a-routers/) · 2026-09-22* 

> D-Link disclosed a maximum-severity flaw in legacy DIR-822A dual-band Wi-Fi routers, CVE-2026-86296, with public proof-of-concept code and no patch available.

**Affected:** Legacy D-Link DIR-822A dual-band Wi-Fi routers. No patch available per the vendor warning.  
**CVEs:** [CVE-2026-86296](https://nvd.nist.gov/vuln/detail/CVE-2026-86296)  

**What happened.** D-Link warned customers of CVE-2026-86296, a maximum-severity vulnerability in legacy DIR-822A routers. Public proof-of-concept exploit code exists and no fix is being issued.

**Why it matters.** Public PoC plus no patch on a consumer-class router is the standard recipe for botnet recruitment. These devices turn up on branch, remote-worker and shadow-IT links where nobody is tracking firmware, and there is no upgrade path that closes this.

**Recommended actions**

- Find DIR-822A units in the estate - inventory, DHCP fingerprints and remote-worker asset lists - and plan hardware replacement; no fixed release is coming.
- Until replacement, disable WAN-side administration, UPnP, remote management and unused services (Telnet, TFTP, HTTP).
- Isolate any surviving unit behind a supported inspecting firewall and deny inbound access from untrusted zones.
- Replace default and reused credentials on the device and on anything that shares those credentials.
- Check for persistence: unexpected static routes, DNS server changes, port forwards and tunnels.

**Legacy / unpatchable gear.** This is the legacy case outright - the model is end-of-life and will not receive a fix. Set a replacement date and a budget owner, apply strict ACLs and no direct management access in the interim, monitor the device as a high-risk asset with a NetFlow baseline and an alert on new outbound flows, and document the accepted risk until the hardware is gone.

**Detection.** Baseline the router's outbound flows and alert on any new destination, on DNS queries bypassing your resolvers, and on inbound connection attempts to the WAN-side management port.

**AI angle.** None stated.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/d-link-warns-of-max-severity-zero-day-bug-in-dir-822a-routers/)**

---

## Zyxel

*1 item(s) — 1 high*

### Switches

#### 🟠 HIGH — Chinese hackers exploit WordPress, Zyxel flaws to steal govt data

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/chinese-hackers-exploit-multiple-technologies-to-steal-govt-data/) · 2026-09-22* 

> A Chinese-speaking threat actor exploited Zyxel GS1900 smart managed switches, reaching 996 devices and over 18,500 records in backend databases.

**Affected:** ZyXEL GS1900 Smart Managed Switches (alongside WordPress installations). No CVE or fixed release stated in the source.  

**What happened.** BleepingComputer reports a Chinese-speaking threat actor has been exploiting vulnerabilities in ZyXEL GS1900 Smart Managed Switches and WordPress to steal sensitive data from 996 devices and more than 18,500 records held in backend databases.

**Why it matters.** GS1900s are cheap access-layer switches that get installed and forgotten, often with a management interface reachable far more widely than anyone intends. A switch under attacker control gives traffic visibility and a quiet foothold inside the LAN that endpoint tooling will never see.

**Recommended actions**

- Locate GS1900 units in the estate and check the running firmware against Zyxel's current release; check the vendor advisory for the fixed release, as the source names none.
- Restrict the switch management plane with management ACLs and move administration to an out-of-band or management-only VLAN; remove any internet reachability.
- Replace default and reused credentials with per-admin accounts, and harden SNMP to v3 with auth and privacy - no public or private community strings.
- Export and review switch configurations for unexpected accounts, mirrored ports, changed VLAN assignments or altered management ACLs.
- Segment management, user, guest and OT traffic and verify VLAN ACL enforcement so a compromised access switch cannot reach server networks.

**Legacy / unpatchable gear.** GS1900 models past end of support will not receive fixes: put them behind a supported inspecting firewall, deny inbound management from every untrusted zone, use a dedicated jump host with full session logging, and set a replacement date with a budget owner.

**Detection.** Alert on administrative logins to access switches from outside the management VLAN, on configuration changes with no change record, and on new SPAN/mirror sessions; baseline outbound flows from switch management IPs.

**AI angle.** None stated.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/chinese-hackers-exploit-multiple-technologies-to-steal-govt-data/)**

---

## Palo Alto Networks

*1 item(s) — 1 medium*

### Other

#### 🟡 MEDIUM — CVE-2026-0307 GlobalProtect App: Local Privilege Escalation Vulnerabilities (Severity: MEDIUM)

*[Palo Alto Networks PSIRT](https://security.paloaltonetworks.com/CVE-2026-0307) · 2026-09-23* `vendor-advisory`

> Palo Alto Networks published a medium-severity advisory for local privilege escalation vulnerabilities in the GlobalProtect app, CVE-2026-0307.

**Affected:** Palo Alto Networks GlobalProtect app (client software). Severity rated MEDIUM by the vendor; advisory published 2026-09-23.  
**CVEs:** [CVE-2026-0307](https://nvd.nist.gov/vuln/detail/CVE-2026-0307)  

**What happened.** Palo Alto Networks PSIRT issued CVE-2026-0307 covering local privilege escalation vulnerabilities in the GlobalProtect app. The advisory entry carries no further detail in this feed.

**Why it matters.** Local privilege escalation needs an attacker already on the machine, so it is not a perimeter emergency. It matters because the VPN client runs with elevated rights on every remote-access endpoint the network team owns, making it a reliable escalation step after an initial foothold.

**Recommended actions**

- Read the vendor advisory at security.paloaltonetworks.com for the affected and fixed GlobalProtect app versions - this feed entry states none.
- Inventory deployed GlobalProtect app versions through your endpoint management platform and schedule the client update with the normal software cycle.
- Where the app auto-updates from the gateway, stage the fixed client on the portal so remote users pick it up on next connect.
- Review who has local administrator rights on machines running the client; local privilege escalation matters less where the user is already unprivileged and constrained.

**Legacy / unpatchable gear.** Machines pinned to an old GlobalProtect app for OS-compatibility reasons cannot take a new client: restrict those users to a dedicated VPN group with a narrow access policy, and treat the endpoint as untrusted with no local admin rights.

**Detection.** Alert on privilege escalation attempts involving GlobalProtect app processes and services, and on unexpected modification of the client's installation directory.

**AI angle.** None stated.

📄 **[Read the full report at Palo Alto Networks PSIRT →](https://security.paloaltonetworks.com/CVE-2026-0307)**

---

## 🛡️ Vendor AI defences introduced

AI capabilities the vendors shipped this cycle - the answer side of the AI story.

### Check Point

**Frontier AI Readiness Program** — Check Point describes this as the program under which it continues to release Jumbo hotfixes carrying security fixes and hardening improvements for its Firewall and Management products. The advisory gives no further detail on the program's scope.

- *Defends against:* Not stated in the source beyond framing the hardening cadence as AI-era readiness; no specific AI threat is named.
- *Availability:* described-only
- *Worth evaluating if:* Check Point firewall and management operators tracking the vendor's patch cadence - the practical content visible here is the Jumbo hotfix stream itself, so subscribe to it rather than treating the program as a separate product.
- 📄 [Read the announcement at Check Point Blog →](https://blog.checkpoint.com/security/security-advisory-action-required-active-exploitation-of-cve-2026-85102-and-a-management-pre-authentication-vulnerability-cve-2026-93616/)

---

## 🧠 AI & network devices — Malware that asks a model what to do next

BleepingComputer reports a new Windows malware family, ClosedQuorum, that calls Google Gemini, DeepSeek, Qwen and Mistral models to decide autonomously what actions to take during the post-compromise stages of an attack. It is endpoint malware, not a network-device threat, and it is dropped from the digest proper on that basis - but it changes an assumption network teams rely on.

Post-compromise behaviour has historically been scripted, which is why detection can key on fixed command sequences and predictable lateral-movement patterns. Malware that selects its next step from a model at runtime produces different sequences on every host, so signature and sequence-based detection degrades while behavioural baselines - what talks to what, over which port, at what volume - hold up. For the network team the practical consequence is that segmentation and NetFlow baselining carry more of the detection load than they used to, and that internal east-west visibility matters more than another endpoint signature feed.

Nothing in this cycle's items shows AI being used against network devices themselves, and no vendor in these sources claims a detection capability aimed at this family.

**Vendors with an AI angle in this edition:** Arista, Check Point, D-Link, F5, Multiple, Palo Alto Networks, Zyxel

**Preventing it on current systems**

- Baseline NetFlow and syslog per segment and alert on deviation rather than relying on signature updates alone - runtime-decided malware will not repeat a fixed command sequence.
- Enforce east-west segmentation so a compromised endpoint cannot reach management VLANs, controller subnets or device SSH/HTTPS interfaces at all.
- Treat any AI/LLM integration on network gear (assistants, AIOps, MCP or agent connectors) as a privileged admin path: scoped read-only credentials, no unattended config write, full audit trail.
- Shorten the patch window for internet-facing network devices to days, not months; assume exploit development and mass scanning outpace your historical SLA.
- Require callback verification for any out-of-band request to change firewall or VPN configuration - AI-enabled social engineering of the network team is cheap.

**Preventing it on legacy / end-of-life systems**

- Legacy gear cannot run modern telemetry agents: collect its behaviour at the network layer instead, with a NetFlow baseline and an alert on any new outbound flow.
- Isolate unsupported devices behind a supported inspecting firewall and deny inbound access from user and untrusted zones.
- Deny legacy devices direct internet egress; force outbound traffic through a proxy that logs, so autonomous post-compromise activity has to cross a control point.
- Where the device cannot be replaced soon, monitor it as a high-risk asset and document the accepted risk with a replacement date and budget owner.

---

<!--index
{
 "url": "/2026/09/23/network-security-digest/",
 "date": "2026-09-23",
 "entries": [
  {
   "title": "Attackers hit Check Point Management Servers and Spark firewalls, F5 BIG-IP APM instances",
   "link": "https://www.helpnetsecurity.com/2026/09/23/check-point-f5-big-ip-apm-zero-days-targeted/",
   "source": "Help Net Security",
   "relevance": "critical",
   "device_types": [
    "management-platform",
    "firewall",
    "load-balancer"
   ],
   "vendors": [
    "Multiple"
   ],
   "cves": [
    "CVE-2026-85102",
    "CVE-2026-93616"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Check Point and F5 are both responding to in-the-wild exploitation: two Check Point CVEs added to KEV, plus an unauthenticated RCE zero-day in F5 BIG-IP APM.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "F5 Patches Critical BIG-IP APM Zero-Day Exploited for Unauthenticated RCE on OAuth Servers",
   "link": "https://thehackernews.com/2026/09/f5-patches-critical-big-ip-apm-zero-day.html",
   "source": "The Hacker News",
   "relevance": "critical",
   "device_types": [
    "load-balancer"
   ],
   "vendors": [
    "F5"
   ],
   "cves": [
    "CVE-2026-94127"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "F5 says attackers are exploiting CVE-2026-94127 in BIG-IP APM to run code without logging in, on systems where APM acts as an OAuth authorization server.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "New CVSS 10.0 VeloCloud Orchestrator Flaw Actively Exploited in Certificate-Based Setups",
   "link": "https://thehackernews.com/2026/09/new-cvss-100-velocloud-orchestrator.html",
   "source": "The Hacker News",
   "relevance": "critical",
   "device_types": [
    "sdwan",
    "management-platform"
   ],
   "vendors": [
    "Arista"
   ],
   "cves": [
    "CVE-2026-93952"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Arista reports active exploitation of CVE-2026-93952, a CVSS 10.0 input validation flaw in on-premises VeloCloud Orchestrator, in certificate-authenticated deployments.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "CVE-2026-94127: Critical Unauthenticated RCE in F5 BIG-IP APM",
   "link": "https://www.rapid7.com/blog/post/etr-cve-2026-94127-critical-unauthenticated-rce-in-f5-big-ip-apm",
   "source": "Rapid7 Blog",
   "relevance": "critical",
   "device_types": [
    "load-balancer"
   ],
   "vendors": [
    "F5"
   ],
   "cves": [
    "CVE-2026-94127"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Rapid7 details CVE-2026-94127 as a CVSS 9.8 heap-based buffer overflow in BIG-IP APM, reachable only when a virtual server has both an access policy and an OAuth profile.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Check Point Warns of Management Server Zero-Day Exploited in Targeted Attacks",
   "link": "https://thehackernews.com/2026/09/check-point-warns-of-management-server.html",
   "source": "The Hacker News",
   "relevance": "critical",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [
    "CVE-2026-93616"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Check Point's Security Management Server carried a zero-day, CVE-2026-93616, letting unauthenticated attackers run scripts on the server that controls firewall policy.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Security Advisory \u2013 Action Required \u2013 Active Exploitation of CVE-2026-85102 and a Management Pre-Authentication Vulnerability CVE-2026-93616",
   "link": "https://blog.checkpoint.com/security/security-advisory-action-required-active-exploitation-of-cve-2026-85102-and-a-management-pre-authentication-vulnerability-cve-2026-93616/",
   "source": "Check Point Blog",
   "relevance": "critical",
   "device_types": [
    "management-platform",
    "firewall"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [
    "CVE-2026-85102",
    "CVE-2026-93616"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "The advisory frames these hotfixes as part of Check Point's Frontier AI Readiness Program, but the vulnerabilities themselves have no stated AI involvement.",
   "summary": "Check Point's own advisory confirms active exploitation of two flaws: CVE-2026-85102 on Security Gateway and the new pre-auth CVE-2026-93616 in Security Management.",
   "ai_defense": true,
   "defense": {
    "solution": "Frontier AI Readiness Program",
    "what_it_does": "Check Point describes this as the program under which it continues to release Jumbo hotfixes carrying security fixes and hardening improvements for its Firewall and Management products. The advisory gives no further detail on the program's scope.",
    "defends_against": "Not stated in the source beyond framing the hardening cadence as AI-era readiness; no specific AI threat is named.",
    "availability": "described-only"
   }
  },
  {
   "title": "D-Link warns of max severity zero-day bug in DIR-822A routers",
   "link": "https://www.bleepingcomputer.com/news/security/d-link-warns-of-max-severity-zero-day-bug-in-dir-822a-routers/",
   "source": "BleepingComputer",
   "relevance": "high",
   "device_types": [
    "router",
    "wireless"
   ],
   "vendors": [
    "D-Link"
   ],
   "cves": [
    "CVE-2026-86296"
   ],
   "kev": false,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "D-Link disclosed a maximum-severity flaw in legacy DIR-822A dual-band Wi-Fi routers, CVE-2026-86296, with public proof-of-concept code and no patch available.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Critical F5 BIG-IP Vulnerability Exploited as Zero-Day",
   "link": "https://www.securityweek.com/critical-f5-big-ip-vulnerability-exploited-as-zero-day/",
   "source": "SecurityWeek",
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
   "summary": "SecurityWeek confirms the critical F5 BIG-IP flaw was exploited as a zero-day, with unauthenticated attackers reaching remote code execution via malicious traffic.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "F5 patches BIG-IP APM zero-day flaw exploited in RCE attacks",
   "link": "https://www.bleepingcomputer.com/news/security/f5-warns-of-big-ip-apm-remote-code-execution-zero-day-exploited-in-attacks/",
   "source": "BleepingComputer",
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
   "summary": "F5 shipped security updates for the BIG-IP APM zero-day being used in remote code execution attacks.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Check Point Patches Exploited Management Server Zero-Day",
   "link": "https://www.securityweek.com/check-point-patches-exploited-management-server-zero-day/",
   "source": "SecurityWeek",
   "relevance": "high",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Check Point patched an exploited Management Server zero-day that let unauthenticated attackers upload and execute arbitrary scripts.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Arista Urges Immediate Patching of Exploited VCO Zero-Day",
   "link": "https://www.securityweek.com/arista-urges-immediate-patching-of-exploited-vco-zero-day/",
   "source": "SecurityWeek",
   "relevance": "high",
   "device_types": [
    "sdwan",
    "management-platform"
   ],
   "vendors": [
    "Arista"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Arista is urging immediate patching of an exploited VeloCloud Orchestrator zero-day that gives remote attackers privileged internal functionality.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Check Point warns of Management Server zero-day exploited in attacks",
   "link": "https://www.bleepingcomputer.com/news/security/check-point-patches-management-server-zero-day-exploited-in-attacks/",
   "source": "BleepingComputer",
   "relevance": "high",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Check Point issued emergency hotfixes for a critical Security Management Server flaw that could let attackers run arbitrary scripts.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Chinese hackers exploit WordPress, Zyxel flaws to steal govt data",
   "link": "https://www.bleepingcomputer.com/news/security/chinese-hackers-exploit-multiple-technologies-to-steal-govt-data/",
   "source": "BleepingComputer",
   "relevance": "high",
   "device_types": [
    "switch"
   ],
   "vendors": [
    "Zyxel"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "A Chinese-speaking threat actor exploited Zyxel GS1900 smart managed switches, reaching 996 devices and over 18,500 records in backend databases.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "CVE-2026-0307 GlobalProtect App: Local Privilege Escalation Vulnerabilities (Severity: MEDIUM)",
   "link": "https://security.paloaltonetworks.com/CVE-2026-0307",
   "source": "Palo Alto Networks PSIRT",
   "relevance": "medium",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Palo Alto Networks"
   ],
   "cves": [
    "CVE-2026-0307"
   ],
   "kev": false,
   "ai": true,
   "ai_angle": "None stated.",
   "summary": "Palo Alto Networks published a medium-severity advisory for local privilege escalation vulnerabilities in the GlobalProtect app, CVE-2026-0307.",
   "ai_defense": false,
   "defense": null
  }
 ]
}
-->

## How this was produced

- Feeds polled: 22 ok, 1 failed
- Raw items: 709 → in window: 65 → network-device relevant: 18 → published: 14
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
