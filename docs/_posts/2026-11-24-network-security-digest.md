---
layout: post
title: "Network Device Security Digest - 2026-11-24"
date: 2026-11-24 17:24:31 +0000
edition: daily
critical_count: 1
item_count: 9
kev: "none"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-11-24

*daily edition · window 24h · 23 feeds · 9 items · 1 critical · generated 2026-11-24 17:24 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — Check Point Security Management servers compromised in 22 organizations

Threat actors have chained multiple vulnerabilities to compromise roughly two dozen Check Point Security Management servers belonging to 22 organizations, in a campaign reported as ongoing. The attacks were launched from 99 IP addresses and hit organizations across multiple industries and countries, and the reporting describes the campaign as AI-assisted. Neither report names the CVEs involved or a fixed build, so treat the specific vulnerability chain as unknown for now and check the Check Point advisory for the fixed release before planning an upgrade.

This is a management-plane compromise, which is the worst case for a firewall estate. A Security Management server holds the policy for every gateway it manages, the SIC certificate authority that the gateways trust, administrator credentials and API tokens, and the log pipeline. An attacker with code on that host does not need to attack the gateways individually: policy pushes, object changes and new administrator accounts all become legitimate, signed operations. Compromise of 22 separate organizations from a shared infrastructure set also indicates mass scanning for exposed management servers rather than targeted work against one victim.

The immediate question for each reader is not whether a patch exists but whether the management server was ever reachable from an untrusted network. Management servers are frequently given a public address for remote SmartConsole access or for Infinity/cloud integration, and that exposure is what turns a vulnerability chain into 22 victims. Treat an internet-reachable management server as in scope for compromise assessment now, not after the advisory lands.

**Do this first**

- Identify every Check Point Security Management server, Multi-Domain Server and log server in the estate and determine which of them accept connections from outside the management network - check the firewall rule base and any NAT or port forward to TCP/19009, 18190, 18191, 18264 and 443.
- Remove management-server exposure from untrusted networks immediately: restrict SmartConsole, Web SmartConsole and API access to a dedicated management VLAN or a jump host reached over VPN with MFA.
- Check the Check Point advisory for the fixed release and schedule the upgrade - the public reporting names no CVE and no fixed build, so do not assume your current version is unaffected.
- Hunt on the management server: new or modified administrator accounts, unexpected API keys, changes in $FWDIR/conf and $FWDIR/log, unscheduled policy installations, cpview/audit log gaps, and outbound connections from the management server to addresses that are not Check Point update or cloud endpoints.
- Review the audit trail of policy installations and object changes for the last 60 days against your change records; an unexplained policy push is the signal that matters most.
- If compromise cannot be ruled out, rotate everything the server holds - administrator passwords, API keys, OPSEC and SIC certificates (reset SIC with each gateway), LDAP/RADIUS bind accounts and any credentials stored in scripts - and assume existing sessions are stolen.
- Block the reported attack infrastructure where you can enumerate it, but do not treat IP blocking as the control; 99 source addresses means the next wave will use different ones.

---

## Executive summary

- Lead item: multiple vulnerabilities were chained to compromise around two dozen Check Point Security Management servers across 22 organizations, from 99 source IP addresses, in a campaign reported as ongoing and AI-assisted. No CVE or fixed build has been named publicly - restrict management access and hunt now.
- Cisco published three advisories for third-party code in its products - OpenJDK CVE-2026-114938 (Java4Shell, RCE, SIR High), PHP CVE-2026-126714 (phpAfterShell, critical RCE) and five nginx RCE flaws. All three say the affected-product list is still being compiled; none yet states fixed Cisco releases.
- Broadcom shipped patches for 23 VMware vulnerabilities, including CVE-2026-123903, a CVSS 9.8 authentication bypass in the vCenter directory service, with Avi Load Balancer also among the patched products - relevant to anyone whose load-balancing tier sits on VMware.
- China-aligned Webworm is deploying a modified SoftEther VPN Bridge disguised as network-monitoring software for covert remote access; this is a rogue-tunnel detection problem, not a device patch.
- Cisco and OpenAI released Shirudo, an open-source Linux runtime in which AI agents write eBPF controls to block exploitation of unpatched flaws without restarting services. Explicitly not production-ready; watch it, do not deploy it.

---

## Items by vendor

## Check Point

*2 item(s) — 1 critical, 1 high*

### Management platforms

#### 🔴 CRITICAL — Hackers compromise Check Point Servers in 22 organizations

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/hackers-compromise-check-point-servers-in-22-organizations/) · 2026-11-24* 

> Attackers chained multiple vulnerabilities to compromise around two dozen Check Point Security Management servers across 22 organisations in an ongoing campaign.

**Affected:** Check Point Security Management servers. The reporting states multiple vulnerabilities were exploited but names no CVE and no fixed build - check the vendor advisory for the fixed release and for the affected versions.  

**What happened.** Threat actors exploited multiple vulnerabilities to compromise two dozen Check Point Security management servers in a campaign described as ongoing and targeting 22 organisations.

**Why it matters.** The management server is the one host in a Check Point estate that can change every gateway's policy. It holds the SIC certificate authority the gateways trust, administrator credentials, API tokens and the log pipeline, so code execution there converts into signed, legitimate-looking policy pushes, new administrator accounts and log tampering across the whole firewall estate. Twenty-two organisations compromised in one campaign means exposed management servers are being found at scale, so the question is exposure and compromise assessment, not whether your version is on a list.

**Recommended actions**

- Inventory every Security Management server, Multi-Domain Server and log server, and establish which accepted connections from outside the management network.
- Remove management-server reachability from untrusted networks now: restrict SmartConsole, Web SmartConsole and API access to a management VLAN or a jump host reached by VPN with MFA, and delete any NAT or port forward that publishes it.
- Check the Check Point advisory for the fixed release and schedule the upgrade; with no public CVE, do not assume your build is unaffected.
- Hunt for compromise on the management server: unexpected administrator accounts and API keys, modified files under $FWDIR, policy installations and object changes with no change record, audit log gaps, and outbound connections to anything other than Check Point update and cloud endpoints.
- If compromise cannot be excluded, rotate administrator passwords, API keys, OPSEC certificates and LDAP/RADIUS bind accounts, reset SIC with every gateway to reissue trust, and invalidate existing administrator sessions.
- Export the current policy and object database and diff it against your last known-good export, looking for permissive rules, new network objects and modified VPN communities.
- Enforce MFA on all management administrator accounts and remove shared local admin accounts.

**Legacy / unpatchable gear.** A management server on an unsupported version will not receive a fix and cannot be made safe by configuration alone. Take it off any untrusted-reachable path immediately, restrict administrative access to a dedicated jump host with full session recording, place strict ACLs in front of it, treat it as a high-risk asset with NetFlow alerting on any new outbound flow, and prioritise an upgrade to a supported release over every other item in this digest.

**Detection.** Alert on administrator logins to the management server from outside the jump-host path, on policy installations and object changes outside change windows, on new administrator or API accounts, on SIC resets you did not perform, and on any outbound connection from the management server to a destination that is not a known Check Point service. Gaps in the audit log are themselves a finding.

**AI angle.** Companion reporting on this campaign describes it as AI-assisted; see item 12.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/hackers-compromise-check-point-servers-in-22-organizations/)**

#### 🟠 HIGH — Check Point Security Management Servers Hacked in AI-Assisted Campaign

*[SecurityWeek](https://www.securityweek.com/check-point-security-management-servers-hacked-in-ai-assisted-campaign/) · 2026-11-24* 

> The Check Point management server intrusions were AI-assisted and launched from 99 IP addresses against organisations in multiple industries and countries.

**Affected:** Check Point Security Management servers, as in item 11. No CVE or fixed build is named in the reporting - check the vendor advisory for the fixed release.  

**What happened.** SecurityWeek reports the campaign against Check Point Security Management servers as AI-assisted, launched from 99 IP addresses and targeting organisations across multiple industries and countries.

**Why it matters.** The AI-assisted element and the 99-address source set change your planning assumptions rather than your patch list. Breadth across industries and countries is now cheap, so being small or obscure is not cover; and a campaign that moves this fast closes the gap between disclosure and exploitation to less than a normal monthly patch cycle. Blocking the reported addresses buys little, because the infrastructure is the disposable part.

**Recommended actions**

- Treat the remediation in item 11 as the action list; this item adds urgency and scope, not separate steps.
- Shorten the patch SLA for management platforms and internet-facing devices to days and get sign-off for that now, before the advisory lands.
- Do not rely on blocking the 99 reported source addresses as a control; implement default-deny source restrictions on the management interface so any source set is irrelevant.
- Assume mass scanning for exposed management interfaces: enumerate your own external attack surface for management ports and fix what you find, rather than waiting for a vulnerability report to tell you what is exposed.
- Brief the network team that out-of-band requests to change firewall or management configuration need callback verification, since AI-enabled social engineering accompanies campaigns of this profile.

**Legacy / unpatchable gear.** Unpatchable management platforms are the targets a fast, broad campaign finds first. Remove them from internet reachability entirely, front them with a supported inspecting firewall, allow administrative access only from a dedicated jump host with session logging, monitor them as high-risk assets with alerting on any new outbound flow, and set a hard replacement or upgrade date with a budget owner.

**Detection.** Alert on authentication attempts to management interfaces from any source outside the permitted management ranges, especially spread across many distinct addresses in a short window. Correlate scanning against management ports at the perimeter with authentication logs on the management server.

**AI angle.** The campaign is reported as AI-assisted, with 99 source addresses hitting organisations across multiple industries and countries - the operational reading is that reconnaissance for exposed management interfaces and chaining of known flaws now outpace a monthly patch cycle.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/check-point-security-management-servers-hacked-in-ai-assisted-campaign/)**

---

## Cisco

*5 item(s) — 3 high, 1 medium, 1 watch*

### Load balancers

#### 🟠 HIGH — Vulnerabilities in nginx Affecting Cisco Products: November 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nginx-nov-2026-e8jska2s) · 2026-11-23* `vendor-advisory`

> Five nginx remote code execution flaws, fixed upstream in nginx 1.31.9 and 1.30.7, are being assessed for Cisco products that embed nginx.

**Affected:** nginx releases 1.9.5 through 1.31.8, fixed upstream in 1.31.9 (mainline) and 1.30.7 (stable). CVE-2026-113425 (ngx_mail_proxy_module), CVE-2026-121810 (ngx_http_v3_module), CVE-2026-123007 and CVE-2026-123633 (ngx_http_v2_module), CVE-2026-123634 (resolver). Cisco has not yet published affected products or fixed Cisco releases - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-113425](https://nvd.nist.gov/vuln/detail/CVE-2026-113425), [CVE-2026-121810](https://nvd.nist.gov/vuln/detail/CVE-2026-121810), [CVE-2026-123007](https://nvd.nist.gov/vuln/detail/CVE-2026-123007), [CVE-2026-123633](https://nvd.nist.gov/vuln/detail/CVE-2026-123633), [CVE-2026-123634](https://nvd.nist.gov/vuln/detail/CVE-2026-123634)  

**What happened.** nginx published fixes on November 18, 2026 for multiple critical vulnerabilities that can lead to remote code execution, spanning the HTTP/2 and HTTP/3 modules, the mail proxy module and the resolver. Cisco opened an advisory for its own products and will update it as information becomes available.

**Why it matters.** The affected components are exactly the ones that face untrusted traffic on a reverse proxy or load-balancing tier: HTTP/2 and HTTP/3 request handling, DNS resolution of upstreams, and mail proxying. Any Cisco product that terminates client HTTP/2 or HTTP/3 on nginx is reachable by anyone who can open a connection to it, with no credentials needed. The same CVEs also apply to any nginx or NGINX-derived proxy you run yourself, which you can patch today without waiting on Cisco.

**Recommended actions**

- Patch the nginx instances you control to 1.31.9 (mainline) or 1.30.7 (stable) as stated in the advisory, prioritising internet-facing reverse proxies and load balancers.
- Inventory Cisco products that embed nginx and record versions; watch the Cisco advisory for the affected-product list and its fixed releases.
- Where HTTP/3 or QUIC is enabled but unused on an internet-facing proxy, disable it to remove the ngx_http_v3_module attack surface until patched.
- Disable the mail proxy module on any instance that does not proxy mail, and restrict the resolver to trusted internal DNS servers over a controlled path.
- Apply the fix to all units in a device group and verify configuration sync after upgrade; remove any administrative or status interface of the proxy tier from public exposure.
- Rotate TLS private keys, admin credentials and API tokens on any proxy that was internet-facing and cannot be proven unexploited after patching.

**Legacy / unpatchable gear.** Embedded nginx in an appliance you cannot update - an end-of-life load balancer or proxy - cannot be patched independently of the vendor image. Put it behind a supported inspecting proxy or firewall, terminate untrusted TLS and HTTP/2 upstream of it, deny direct inbound reach from untrusted zones, and plan replacement with a budget owner.

**Detection.** Monitor proxy logs and process state for worker crashes and restarts, which are the usual first sign of memory-corruption exploitation attempts against HTTP/2 and HTTP/3 handling. Alert on malformed HTTP/2 frame or HTTP/3 error counters rising, on resolver responses from unexpected servers, and on any outbound connection initiated by the proxy's worker user.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nginx-nov-2026-e8jska2s)**

### Other

#### 🟠 HIGH — Vulnerability in Oracle OpenJDK (Java4Shell) Affecting Cisco Products: November 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nov2026-java-rce-eCa6ss9z) · 2026-11-23* `vendor-advisory`

> Cisco is assessing product exposure to CVE-2026-114938, an OpenJDK remote code execution flaw in Java 17.0.6 and later on x86-64 triggered by Base64 processing.

**Affected:** Cisco products that bundle OpenJDK. The underlying flaw affects OpenJDK Java 17.0.6 and later on x86-64 platforms when processing Base64 encoded content. Cisco's advisory does not yet enumerate affected products or fixed releases - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-114938](https://nvd.nist.gov/vuln/detail/CVE-2026-114938)  

**What happened.** Oracle announced fixes on November 17, 2026 for CVE-2026-114938, an OpenJDK vulnerability that can lead to remote code execution, referred to as Java4Shell. Cisco published an advisory with a Security Impact Rating of High, stating it will be updated as more information becomes available; Talos has issued a threat advisory on the same CVE.

**Why it matters.** Java runs inside Cisco management, controller and policy software rather than in the data plane of switches and routers, so a third-party RCE of this kind usually lands on several management-tier products at once. The exposure is public before the product list is, which means the window where you cannot answer 'are we affected' is the window attackers get for free.

**Recommended actions**

- Inventory which Cisco products you run that embed a Java runtime - management, controller, policy and analytics platforms are the likely candidates - and record their versions ready for the moment the affected-product list is published.
- Subscribe to updates on the Cisco advisory and re-check it daily this week; check the vendor advisory for the fixed release rather than inferring a Cisco version from the OpenJDK fix.
- Reduce exposure now: management, HTTPS and API interfaces of Cisco management platforms off the internet, management-plane ACLs restricting them to a management VLAN or out-of-band network, MFA on all administrative access.
- Where a product exposes a Base64-accepting upload, API or import function to untrusted users, restrict it to authenticated internal sources until the product's status is known.
- Ship management-platform logs to the SIEM and alert on administrator logins, configuration changes and service restarts for the duration.
- Record the decision (patched / mitigated / awaiting product list) against each device group in the change record.

**Legacy / unpatchable gear.** Cisco products that are past end of software maintenance will not receive a fix for this CVE even once the product list is published. Isolate them behind a supported inspecting firewall, deny inbound access from untrusted zones, allow administrative access only from a dedicated jump host with full session logging, and set a replacement date with a named budget owner.

**Detection.** Alert on unexpected child processes and outbound connections from Java service accounts on Cisco management hosts, and on administrative logins or configuration changes that do not match a change record. Review web and API access logs for unusual Base64-heavy request bodies to import, upload or deserialization endpoints.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nov2026-java-rce-eCa6ss9z)**

#### 🟠 HIGH — Vulnerability in PHP Affecting Cisco Products: November 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-phpaftershell-nov-2026-kT9pL4Xe) · 2026-11-23* `vendor-advisory`

> A critical PHP remote code execution flaw, CVE-2026-126714 (phpAfterShell), is under assessment for Cisco products; no affected products or fixed releases are listed yet.

**Affected:** Cisco products that bundle PHP. Cisco rates the advisory Critical but has not yet published the affected-product list or fixed releases - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-126714](https://nvd.nist.gov/vuln/detail/CVE-2026-126714)  

**What happened.** On November 19, 2026 the PHP Foundation released security fixes for a critical vulnerability that can lead to remote code execution, tracked as CVE-2026-126714 and referred to as phpAfterShell (details in GHSA-fm2m-fjf5-v3cp). Cisco opened an advisory it will update as information becomes available.

**Why it matters.** PHP in network products almost always means a web administration interface. If an affected product turns out to be one whose GUI is reachable from a user or internet-facing segment, a critical pre-authentication-class RCE in that stack is directly exploitable against the management plane, and the product list is not out yet.

**Recommended actions**

- List the Cisco products in your estate whose administrative or captive-portal interface is PHP-based and note versions now, so the affected-product list can be actioned in hours rather than days.
- Monitor the Cisco advisory for updates and the fixed releases; do not derive a Cisco fixed version from the upstream PHP release numbers.
- Restrict every web administration interface to the management VLAN or out-of-band network and remove it from any internet-facing or user-facing interface.
- Terminate web administration exposure to guest and captive-portal segments specifically - those are untrusted networks that often reach the appliance GUI by design.
- Enforce MFA on administrative accounts and remove shared local admin accounts on affected management tiers.
- Alert on HTTP 500s, unusual POST bodies and new files appearing under web roots on affected appliances once the product list is known.

**Legacy / unpatchable gear.** Appliances past end of support will not get a PHP fix. Deny inbound access to their web interface from every untrusted zone, reach them only through a dedicated jump host with session recording, place strict ACLs in front of them, and document the accepted risk with a replacement date.

**Detection.** Watch appliance web server logs for requests to unusual paths, long or encoded POST bodies, and responses that deviate from the normal status-code mix. On the appliance, alert on new or modified files under the web root and on web-server processes spawning shells.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-phpaftershell-nov-2026-kT9pL4Xe)**

#### 🟡 MEDIUM — ClamAV Vulnerabilities Affecting Cisco Products: November 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-clamav-multi-e93M3vdA) · 2026-11-24* `vendor-advisory`

> ClamAV flaws let a remote attacker bypass configured file and scan-time limits and stall scanning on Cisco platforms; Cisco has shipped updates and there is no workaround.

**Affected:** Cisco platforms that embed ClamAV. Cisco Secure Endpoint Connector for Windows is rated High because the scanning process runs in a privileged security context; the Security Impact Rating is Medium on other platforms including Linux. Cisco has released software updates - check the vendor advisory for the fixed release for each product you run.  

**What happened.** Multiple ClamAV vulnerabilities allow a remote attacker to bypass configured file and scan time limits and cause a denial of service, interrupting scanning operations. Cisco has released software updates for affected platforms and states there are no workarounds.

**Why it matters.** This is availability of inspection, not code execution on the network path. Where ClamAV backs scanning on a Cisco security appliance, an attacker who can stall the scanner gets a window in which content passes with inspection degraded - so the practical risk is a fail-open gap in your content security, and the fix is the only remedy since no workaround exists.

**Recommended actions**

- Identify which Cisco products in your estate use ClamAV-based scanning and check each one's status and fixed release in the advisory; the highest-rated platform here is an endpoint connector, so scope the work accordingly.
- Apply the released software updates in your change window; there is no workaround, so no mitigation substitutes for the upgrade.
- Check the fail-open versus fail-closed behaviour of any appliance whose scanning engine this affects, and decide deliberately which you want while unpatched.
- Alert on scanning-service restarts, scan timeouts and sustained queue growth on affected appliances - the symptom of this class of flaw is interrupted scanning, not an alert.
- Record the patch decision per device group in the change record.

**Legacy / unpatchable gear.** An appliance out of support cannot take the ClamAV update. Keep untrusted content from reaching it directly, place a supported inspecting layer upstream so the loss of one scanning engine is not the loss of all inspection, and set a replacement date for the device.

**Detection.** Alert on repeated scanner process restarts, scan-time-limit and file-size-limit events, growing scan queues, and gaps in scan verdict logging on ClamAV-backed Cisco platforms.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-clamav-multi-e93M3vdA)**

#### ⚪ WATCH — New Shirudo tool uses AI to shield unpatched Linux systems from exploits

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/new-shirudo-tool-uses-ai-to-shield-unpatched-linux-systems-from-exploits/) · 2026-11-24* `AI` `AI-defense`

> Cisco and OpenAI released Shirudo, an open-source Linux runtime that lets AI agents write eBPF controls to block exploitation of unpatched flaws without restarting services.

**Affected:** No device is affected - this is a tool release. Shirudo is an open-source Linux runtime; the reporting describes it as not yet production-ready.  

**What happened.** OpenAI and Cisco launched Shirudo, an open-source Linux security runtime that deploys temporary runtime protections to block exploitation of a vulnerability while administrators prepare and roll out patches, without restarting services. It works by letting AI agents write dynamic eBPF controls. The reporting states it is not yet production-ready.

**Why it matters.** The capability targets the real gap for network teams - the days or weeks between a disclosure and a maintenance window on a device you cannot reboot during business hours - but it runs on Linux hosts, not on appliance firmware, so it applies to your management servers, controllers, collectors and jump hosts rather than to a FortiGate or a Catalyst. Two caveats are load-bearing: it is pre-production, and an AI agent writing kernel-level enforcement rules is a privileged administrative path that needs the same governance as any other.

**Recommended actions**

- Track the project rather than deploying it; the reporting explicitly says it is not production-ready, so keep it out of change-controlled infrastructure for now.
- If you evaluate it, do so in a lab on a non-production Linux management host, with a documented rollback path and a record of every control the agent writes.
- Scope any agent credentials to the minimum, require human approval before an AI-authored control is enforced, and log agent actions to the SIEM as administrative events.
- Use the evaluation to answer a more general governance question first: which AI agents in your environment are allowed to change enforcement on infrastructure hosts, under what approval, and with what audit trail.
- Do not treat runtime shielding as a substitute for the vendor patch - it is cover for the window, and the upgrade still has to happen.

**Legacy / unpatchable gear.** Unpatchable appliances are the gear this idea sounds most attractive for, and the one place it does not apply: it needs a Linux host you control with eBPF, not vendor firmware. For legacy devices the compensating controls remain network-layer - isolation behind a supported inspecting firewall, no direct management access, strict ACLs, a dedicated jump host and flow-based monitoring - plus a replacement date.

**Detection.** Not applicable. If evaluated, treat eBPF program loads and agent-initiated control changes on the host as auditable administrative events and alert on any that occur outside an approved evaluation window.

**AI angle.** AI agents author the runtime eBPF controls, making this a defensive AI capability from a network vendor - and a worked example of an AI agent holding a privileged administrative path on infrastructure hosts.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/new-shirudo-tool-uses-ai-to-shield-unpatched-linux-systems-from-exploits/)**

---

## Broadcom

*1 item(s) — 1 high*

### Load balancers

#### 🟠 HIGH — VMware vCenter Hit by Another Critical 9.8-Rated Auth Bypass Flaw After Zero-Day Exploitation

*[The Hacker News](https://thehackernews.com/2026/11/vmware-vcenter-hit-by-another-critical.html) · 2026-11-24* 

> Broadcom patched 23 VMware flaws, including CVE-2026-123903, a CVSS 9.8 authentication bypass in the vCenter directory service, with Avi Load Balancer also among the patched products.

**Affected:** VMware ESX, vCenter, Workstation, Fusion and Avi Load Balancer. The critical issue, CVE-2026-123903 (CVSS 9.8), is in the VMware Directory Service (vmdir) and allows authentication bypass. Patches have shipped; check the vendor advisory for the fixed release for each product.  
**CVEs:** [CVE-2026-123903](https://nvd.nist.gov/vuln/detail/CVE-2026-123903)  

**What happened.** Broadcom released patches for 23 vulnerabilities across the VMware line. The critical one, CVE-2026-123903, resides in vmdir and permits a malicious actor to bypass authentication. The reporting notes it follows earlier zero-day exploitation against vCenter.

**Why it matters.** Two things matter here for a network team. Avi Load Balancer is part of the data path for applications and is in the patch set, so it belongs in your load-balancer patch cycle rather than someone else's. And a 9.8 authentication bypass in vCenter is a hypervisor-management compromise, which in most estates means control over the virtual firewalls, virtual load balancers and management servers that run on it - including, frequently, the firewall management server itself. A vCenter takeover reaches your network control plane regardless of how well the gateways are patched.

**Recommended actions**

- Apply the Broadcom patches to vCenter first, then Avi Load Balancer, then hosts; verify configuration sync across Avi Controller clusters after upgrade.
- Confirm vCenter and the Avi Controller have no exposure to untrusted or internet-facing networks; restrict both to the management VLAN and reach them via a jump host with MFA.
- Map which network functions run as virtual machines on the affected clusters - virtual firewalls, VPN gateways, load balancers, firewall management servers - and treat a vCenter compromise as a compromise of those too in your incident plan.
- Rotate vCenter SSO administrator credentials, service accounts and API tokens, plus Avi Controller admin credentials and certificates, after patching.
- Review Avi virtual service and policy configuration for unauthorised changes, and vCenter for unexpected SSO users, permissions or role grants.
- Enforce MFA on vCenter SSO and keep the identity source for hypervisor management separate from general user directory access where possible.

**Legacy / unpatchable gear.** Unsupported vCenter or Avi versions past end of general support will not receive these fixes. Isolate the management network entirely from user segments, permit administrative access only from a dedicated hardened jump host, log all sessions, and move any network function that cannot tolerate a hypervisor-management compromise onto supported infrastructure or physical appliances. Record the accepted risk with a replacement date.

**Detection.** Alert on vCenter SSO authentication events that lack a corresponding source in your jump-host logs, on new SSO users or permission grants, and on LDAP or vmdir service anomalies. On Avi Controller, alert on administrative logins, virtual service configuration changes and certificate changes outside the change window.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/11/vmware-vcenter-hit-by-another-critical.html)**

---

## Unspecified

*1 item(s) — 1 medium*

### VPN gateways

#### 🟡 MEDIUM — Webworm Suspected of Modifying SoftEther VPN to Evade Detection and Hide Remote Access

*[The Hacker News](https://thehackernews.com/2026/11/webworm-suspected-of-modifying.html) · 2026-11-23* 

> China-aligned Webworm is running a modified SoftEther VPN Bridge disguised as network-monitoring software to keep covert remote access into compromised networks.

**Affected:** Not a product vulnerability. The open-source SoftEther VPN Bridge tool, modified by the actor and masquerading as network-monitoring software on compromised hosts inside victim networks.  

**What happened.** The China-aligned actor Webworm is suspected of using a custom build of SoftEther VPN Bridge with recognisable detection fingerprints removed, implementing techniques to hide from security tools, and disguised as network-monitoring software to establish covert remote access.

**Why it matters.** This is a rogue-tunnel problem on your side of the perimeter, and it is a network control failure rather than a patching one. A SoftEther bridge dialling out from an internal host creates an attacker-controlled layer-2 or layer-3 path into the network that bypasses the VPN gateway, its MFA and its logging entirely. Because the binary is reshaped to defeat signatures and named to look like monitoring software, detection has to come from egress control and flow analysis, not from the endpoint's verdict.

**Recommended actions**

- Deny outbound traffic by default from server and infrastructure segments, and allow egress only to named destinations through an inspecting proxy - a tunnel tool cannot phone home through a default-deny egress policy.
- Block the protocols SoftEther can use for transport where you do not need them: outbound TCP/443 from hosts with no business need, UDP-based tunnel transports, and ICMP or DNS tunnelling - and alert when a host that has never made such connections starts.
- Baseline NetFlow for internal hosts and alert on long-lived, high-upload-ratio outbound sessions to a single external address, which is what a persistent bridge looks like regardless of binary naming.
- Audit for unauthorised VPN and remote-access software on servers and management hosts, including services whose display name claims to be monitoring software; compare the installed service inventory against your approved list rather than trusting names.
- Verify no internal host has been granted a firewall exception or NAT entry that lets it build outbound tunnels, and review recent rule-base additions for one.
- Enforce phishing-resistant MFA and conditional access on the legitimate remote-access path so the sanctioned gateway stays the cheapest route in and rogue tunnels stand out.

**Legacy / unpatchable gear.** Legacy and end-of-life devices often cannot export flow data or run any agent, so they are the ideal host for a covert bridge and the hardest place to spot one. Put them in their own segment behind a supported inspecting firewall, apply a default-deny egress policy at that firewall rather than on the device, collect flow data at the segment boundary instead of on the host, and alert on any new outbound flow at all.

**Detection.** Alert on new long-lived outbound sessions from infrastructure segments, on persistent connections with sustained outbound data volumes to a single external IP, and on TLS sessions to destinations with no corresponding DNS lookup or SNI you recognise. Review service inventories for remote-access or tunnelling software presenting itself as a monitoring agent, and compare running services against a known-good baseline.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/11/webworm-suspected-of-modifying.html)**

---

## 🛡️ Vendor AI defences introduced

AI capabilities the vendors shipped this cycle - the answer side of the AI story.

### Cisco

**Shirudo** — An open-source Linux security runtime, released by Cisco with OpenAI, that lets AI agents write dynamic eBPF controls to deploy temporary runtime protections against exploitation of a vulnerability, without restarting services, while administrators prepare and roll out the patch.

- *Defends against:* Exploitation during the gap between disclosure and patch deployment on Linux hosts - the window that AI-accelerated exploit development and mass scanning have made shorter than a normal maintenance cycle.
- *Availability:* Announced - released as open source, and described in the reporting as not yet production-ready.
- *Worth evaluating if:* You run Linux-based network management servers, controllers, log collectors or jump hosts that cannot be patched or restarted quickly, and you have a lab to assess it in. Lab only for now given the pre-production status, and your AI governance owner should be in the evaluation because an agent writing kernel-level enforcement rules is a privileged administrative path. It does not apply to vendor appliance firmware.
- 📄 [Read the announcement at BleepingComputer →](https://www.bleepingcomputer.com/news/security/new-shirudo-tool-uses-ai-to-shield-unpatched-linux-systems-from-exploits/)

---

## 🧠 AI & network devices — AI on both sides of the management plane: an AI-assisted intrusion campaign, and AI agents writing runtime shields

Two of this cycle's items are genuinely about AI and network infrastructure, and they point in opposite directions. On the attack side, the campaign against Check Point Security Management servers is reported as AI-assisted, run from 99 source addresses against 22 organizations in multiple industries and countries. The reporting does not detail which parts were machine-driven, so the operationally useful reading is narrow and unglamorous: reconnaissance for exposed management interfaces and the chaining of known vulnerabilities now move faster than a monthly patch cycle, and breadth is no longer evidence of a large crew.

On the defensive side, Cisco and OpenAI released Shirudo, an open-source Linux runtime that lets AI agents write dynamic eBPF controls to block exploitation of a vulnerability while a patch is tested and rolled out, without restarting services. The reporting is explicit that it is not yet production-ready. It is worth watching rather than deploying, and it is also a template for a risk you should already be governing: an AI agent authoring enforcement rules on infrastructure hosts is a privileged admin path and needs the same controls as any other one.

No prompt-injection or LLM-integration flaw in network device management was reported in this window.

**Vendors with an AI angle in this edition:** Check Point, Cisco

**Preventing it on current systems**

- Shorten the patch window for internet-facing network devices and management platforms to days: exploit development and mass scanning now outrun a monthly cycle, as a single campaign reaching 22 organizations shows.
- Treat any AI or LLM integration on network gear - assistants, AIOps, MCP or agent connectors, or agents that write enforcement rules - as a privileged admin path: scoped read-only credentials by default, no unattended config write, full audit trail of every action.
- Test AI-facing management surfaces for prompt injection from attacker-controlled data such as device logs, hostnames, certificate subjects and ticket text before letting an agent act on them.
- Evaluate AI-written runtime mitigations such as Shirudo in a lab only while they are pre-production, and require a rollback path and change-record entry before any such control reaches an infrastructure host.
- Use AI-assisted detection on your own side: baseline NetFlow and syslog from management servers and alert on deviation - a new outbound flow from a policy server - rather than relying on signature updates alone.
- Harden against AI-enabled social engineering of the network team: callback verification on any out-of-band request to change firewall, VPN or management-server configuration.

**Preventing it on legacy / end-of-life systems**

- Unsupported management platforms and gateways will not get a fix for the next AI-accelerated vulnerability chain: put them behind a supported inspecting firewall and deny all inbound access from untrusted zones.
- Give legacy gear no direct administrative reachability at all - dedicated jump host, strict ACLs, full session logging - since fast automated scanning finds exposed legacy management ports first.
- Monitor each unpatchable device as a high-risk asset with a NetFlow baseline and alerting on any new outbound flow, which is the detection that survives when patching is not an option.
- Set a replacement date and a budget owner for every end-of-life device carrying management or VPN function, and document the accepted risk until then.

---

<!--index
{
 "url": "/2026/11/24/network-security-digest/",
 "date": "2026-11-24",
 "entries": [
  {
   "title": "Hackers compromise Check Point Servers in 22 organizations",
   "link": "https://www.bleepingcomputer.com/news/security/hackers-compromise-check-point-servers-in-22-organizations/",
   "source": "BleepingComputer",
   "relevance": "critical",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "Companion reporting on this campaign describes it as AI-assisted; see item 12.",
   "summary": "Attackers chained multiple vulnerabilities to compromise around two dozen Check Point Security Management servers across 22 organisations in an ongoing campaign.",
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
   "ai": false,
   "ai_angle": "",
   "summary": "Cisco is assessing product exposure to CVE-2026-114938, an OpenJDK remote code execution flaw in Java 17.0.6 and later on x86-64 triggered by Base64 processing.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Vulnerability in PHP Affecting Cisco Products: November 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-phpaftershell-nov-2026-kT9pL4Xe",
   "source": "Cisco PSIRT",
   "relevance": "high",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [
    "CVE-2026-126714"
   ],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "A critical PHP remote code execution flaw, CVE-2026-126714 (phpAfterShell), is under assessment for Cisco products; no affected products or fixed releases are listed yet.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Vulnerabilities in nginx Affecting Cisco Products: November 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nginx-nov-2026-e8jska2s",
   "source": "Cisco PSIRT",
   "relevance": "high",
   "device_types": [
    "load-balancer",
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [
    "CVE-2026-113425",
    "CVE-2026-121810",
    "CVE-2026-123007",
    "CVE-2026-123633",
    "CVE-2026-123634"
   ],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Five nginx remote code execution flaws, fixed upstream in nginx 1.31.9 and 1.30.7, are being assessed for Cisco products that embed nginx.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "VMware vCenter Hit by Another Critical 9.8-Rated Auth Bypass Flaw After Zero-Day Exploitation",
   "link": "https://thehackernews.com/2026/11/vmware-vcenter-hit-by-another-critical.html",
   "source": "The Hacker News",
   "relevance": "high",
   "device_types": [
    "load-balancer",
    "management-platform"
   ],
   "vendors": [
    "Broadcom"
   ],
   "cves": [
    "CVE-2026-123903"
   ],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Broadcom patched 23 VMware flaws, including CVE-2026-123903, a CVSS 9.8 authentication bypass in the vCenter directory service, with Avi Load Balancer also among the patched products.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Check Point Security Management Servers Hacked in AI-Assisted Campaign",
   "link": "https://www.securityweek.com/check-point-security-management-servers-hacked-in-ai-assisted-campaign/",
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
   "ai_angle": "The campaign is reported as AI-assisted, with 99 source addresses hitting organisations across multiple industries and countries - the operational reading is that reconnaissance for exposed management interfaces and chaining of known flaws now outpace a monthly patch cycle.",
   "summary": "The Check Point management server intrusions were AI-assisted and launched from 99 IP addresses against organisations in multiple industries and countries.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "ClamAV Vulnerabilities Affecting Cisco Products: November 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-clamav-multi-e93M3vdA",
   "source": "Cisco PSIRT",
   "relevance": "medium",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "ClamAV flaws let a remote attacker bypass configured file and scan-time limits and stall scanning on Cisco platforms; Cisco has shipped updates and there is no workaround.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Webworm Suspected of Modifying SoftEther VPN to Evade Detection and Hide Remote Access",
   "link": "https://thehackernews.com/2026/11/webworm-suspected-of-modifying.html",
   "source": "The Hacker News",
   "relevance": "medium",
   "device_types": [
    "vpn-gateway"
   ],
   "vendors": [
    "Unspecified"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "China-aligned Webworm is running a modified SoftEther VPN Bridge disguised as network-monitoring software to keep covert remote access into compromised networks.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "New Shirudo tool uses AI to shield unpatched Linux systems from exploits",
   "link": "https://www.bleepingcomputer.com/news/security/new-shirudo-tool-uses-ai-to-shield-unpatched-linux-systems-from-exploits/",
   "source": "BleepingComputer",
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
   "ai_angle": "AI agents author the runtime eBPF controls, making this a defensive AI capability from a network vendor - and a worked example of an AI agent holding a privileged administrative path on infrastructure hosts.",
   "summary": "Cisco and OpenAI released Shirudo, an open-source Linux runtime that lets AI agents write eBPF controls to block exploitation of unpatched flaws without restarting services.",
   "ai_defense": true,
   "defense": {
    "solution": "Shirudo",
    "what_it_does": "An open-source Linux security runtime, released by Cisco with OpenAI, that lets AI agents write dynamic eBPF controls to deploy temporary runtime protections against exploitation of a vulnerability, without restarting services, while administrators prepare and roll out the patch.",
    "defends_against": "Exploitation during the gap between disclosure and patch deployment on Linux hosts - the window that AI-accelerated exploit development and mass scanning have made shorter than a normal maintenance cycle.",
    "availability": "Announced - released as open source, and described in the reporting as not yet production-ready."
   }
  }
 ]
}
-->

## How this was produced

- Feeds polled: 23 ok, 0 failed
- Raw items: 739 → in window: 63 → network-device relevant: 18 → published: 9
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
