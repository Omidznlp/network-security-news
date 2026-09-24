---
layout: post
title: "Network Device Security Digest - 2026-09-24"
date: 2026-09-24 11:30:47 +0000
edition: daily
critical_count: 4
item_count: 6
kev: "CVE-2026-85102, CVE-2026-86060, CVE-2026-94127"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-09-24

*daily edition · window 24h · 22 feeds · 6 items · 4 critical · generated 2026-09-24 11:30 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — Check Point Security Gateway VPN pre-auth RCE under active exploitation, KEV deadline 25 September

Check Point has confirmed active exploitation of CVE-2026-85102, a pre-authentication remote code execution flaw in the VPN certificate-handling functionality of Security Gateway. CISA's KEV entry, added 22 September 2026, describes it as an improper certificate validation issue affecting Check Point Security Gateway and Check Point Spark Firewall where Site-to-Site VPN or Remote Access VPN is configured, and states that an unauthenticated remote attacker can execute arbitrary code on the gateway. The remediation due date under BOD 26-04 is 25 September 2026.

This is the perimeter itself. The vulnerable code path is reached by any peer that can complete enough of a VPN negotiation to present a certificate, which on a Remote Access gateway means the open internet. Code execution lands on the device that terminates your tunnels, holds your VPN pre-shared keys and certificates, and sits at the boundary between untrusted and trusted zones. There is no authentication step in front of it and no user interaction to wait for.

Two other unauthenticated RCEs on internet-facing network gear were added to KEV on the same date and share the 25 September deadline: F5 BIG-IP APM (CVE-2026-94127) and, earlier in the month, MikroTik RouterOS (CVE-2026-86060). Treat this as one exposure-reduction exercise across the edge rather than three separate tickets, and start with whichever of these terminates remote access for your users.

Neither the Check Point advisory contents nor the collected reporting states a fixed build in the material available here. Check the vendor advisory for the fixed release and apply it in the change window; do not schedule against a version number from anywhere else. Because exploitation is confirmed and pre-authentication, patching alone is not sufficient - assume a pre-patch compromise survives the upgrade and rotate secrets afterwards.

**Do this first**

- Inventory every Check Point Security Gateway and Spark Firewall with Site-to-Site or Remote Access VPN enabled; these are the configurations CISA names as affected.
- Check the vendor advisory for the fixed release for your train, then upgrade inside the change window, HA pair secondary first.
- Where the upgrade cannot happen before the 25 September KEV due date, apply the vendor's documented mitigation and restrict which peers and source ranges can reach the VPN service.
- Remove management access (HTTPS/SSH/API) from any internet-facing interface; restrict it to a dedicated management VLAN or out-of-band network.
- After patching, rotate local admin credentials, API keys, device certificates and VPN pre-shared keys - a pre-patch compromise survives the upgrade.
- Terminate active VPN sessions and revoke session tokens post-upgrade; assume stolen session material.
- Export and review the gateway configuration for unexpected admin users, scripts, static routes or modified login pages, and hunt for unexpected outbound connections from the gateway.

---

> **On the CISA KEV catalog in this edition:** CVE-2026-85102, CVE-2026-86060, CVE-2026-94127 — treat these as confirmed-exploited and patch on an emergency change.

## Executive summary

- Check Point confirmed active exploitation of CVE-2026-85102, a pre-authentication RCE in Security Gateway VPN certificate handling; it is on CISA KEV with a 25 September 2026 due date.
- F5 BIG-IP APM CVE-2026-94127, an unauthenticated heap overflow reachable via an auth header and listed on KEV, now has a public technical write-up from watchTowr Labs.
- Arista shipped patches for an actively exploited zero-day in VeloCloud Orchestrator On-Prem - an SD-WAN controller, so the blast radius is every edge device it configures.
- CERT Polska's MikroTrick chain (CVE-2026-67279 plus KEV-listed CVE-2026-86060) takes over internet-exposed MikroTik routers with no password or SSH key; attack logs predate disclosure.
- Cisco widened its ASA/FTD SSL VPN denial-of-service advisory: the memory-management flaw affects all ASA and FTD platforms, not only the virtual appliances in version 1.0.
- Three KEV-listed unauthenticated RCEs on internet-facing network devices in one window - run this as a single edge exposure-reduction exercise, and rotate credentials after patching, not just the images.

---

## Items by vendor

## Check Point

*2 item(s) — 1 critical, 1 watch*

### Firewalls

#### 🔴 CRITICAL — Check Point warns of hackers exploiting Security Gateway VPN RCE flaw

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/check-point-warns-of-hackers-exploiting-security-gateway-vpn-rce-flaw/) · 2026-09-23* `KEV`

> Check Point confirms attackers are exploiting CVE-2026-85102, a pre-auth RCE in Security Gateway VPN certificate handling, with a CISA KEV deadline of 25 September.

**Affected:** Check Point Security Gateway and Check Point Spark Firewall with Site-to-Site VPN or Remote Access VPN configured. CISA KEV entry added 2026-09-22, due 2026-09-25. Check the vendor advisory for the fixed release - no fixed build is stated in the available sources.  
**CVEs:** [CVE-2026-85102](https://nvd.nist.gov/vuln/detail/CVE-2026-85102) **[KEV]**  

**What happened.** Check Point confirmed active exploitation of CVE-2026-85102, a pre-authentication remote code execution vulnerability in the VPN certificate-handling functionality of Security Gateway. CISA describes it as improper certificate validation allowing an unauthenticated remote attacker to execute arbitrary code on the gateway, and lists it under BOD 26-04 with a 25 September 2026 remediation date.

**Why it matters.** The vulnerable path is pre-authentication and is reached through the VPN service, which on a remote-access gateway faces the internet. Successful exploitation is code execution on the device that terminates your tunnels and holds your VPN certificates and pre-shared keys, at the boundary between untrusted and trusted networks. Exploitation is confirmed, not theoretical.

**Recommended actions**

- Identify all Security Gateway and Spark Firewall units with Site-to-Site or Remote Access VPN enabled - those are the named affected configurations.
- Check the vendor advisory for the fixed release and schedule the upgrade in the change window, HA pair secondary first.
- If the 25 September KEV date cannot be met, apply the vendor's documented mitigation and restrict source ranges permitted to reach the VPN service.
- Remove management access (HTTPS/SSH/API) from internet-facing interfaces; restrict to a management VLAN or out-of-band network.
- Rotate local admin credentials, API keys, device certificates and VPN pre-shared keys after patching.
- Terminate all active VPN sessions and revoke session tokens after the upgrade.
- Export and review the configuration for unexpected admin users, scripts, static routes or modified login pages.
- Enforce phishing-resistant MFA and geo/ASN conditional access on the remote access portal.

**Legacy / unpatchable gear.** Gateways past end of support will not receive this fix. Take Remote Access VPN off them now and move users to a supported gateway; where a Site-to-Site tunnel must stay up, restrict the peer to a fixed source address, front the device with a supported inspecting firewall, and deny all other inbound access. Set a replacement date with a budget owner and record the accepted risk until then.

**Detection.** Alert on gateway processes crashing or restarting around VPN certificate negotiation, and on any outbound connection initiated by the gateway itself to an address outside your management and update ranges. Review administrative login records and configuration-change history since 22 September 2026, and compare an exported configuration against your last known-good for added admin users, scripts or static routes.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/check-point-warns-of-hackers-exploiting-security-gateway-vpn-rce-flaw/)**

### Management platforms

#### ⚪ WATCH — Workforce AI Security Policy Management Is Now Conversational

*[Check Point Blog](https://blog.checkpoint.com/ai-security/workforce-ai-security-policy-management-is-now-conversational/) · 2026-09-23* `AI` `AI-defense`

> Check Point released Workforce AI MCP, a Model Context Protocol server letting an AI client query and edit employee AI usage policy in natural language.

**Affected:** Check Point Workforce AI Security customers who connect a compatible AI client to the new Workforce AI MCP server. Not a vulnerability.  

**What happened.** Check Point described Workforce AI MCP, its own Model Context Protocol server for Workforce AI Security. A connected AI client can query, analyse and manage employee AI usage policy conversationally - asking which rule applies to a given user and application, or whether a rule is shadowed by a higher-priority one, across users, managed assets, GenAI application usage, DLP data types, and agent and MCP activity - and can create and update rules by request.

**Why it matters.** Relevant to network teams less as a product than as a pattern: security policy management is acquiring an agent-driven write path. An MCP server that can create and update rules is an administrative interface with all the access-control, attribution and audit requirements of one, and it reads data - usernames, application names, activity records - that an attacker can influence. Shadowed-rule analysis is genuinely useful; the rule-writing capability is the part that needs governance before it is enabled.

**Recommended actions**

- If you evaluate this, connect the MCP server with a scoped, read-only service identity first and keep rule creation and update disabled until the audit trail is proven.
- Require a named service account per integration rather than a shared admin credential, so agent-initiated changes are attributable to a person.
- Ship every policy change made through the MCP path to the SIEM and alert on rule creation, deletion and priority changes.
- Test the conversational interface with attacker-influenceable input (crafted usernames, application names, activity records) before granting it write access.
- Keep policy change approval in the existing change process - conversational editing is not a substitute for review.
- Restrict which hosts may reach the MCP endpoint, and treat the AI client host as a privileged management workstation.

**Legacy / unpatchable gear.** No patching dimension here. The legacy concern is different: do not extend agent-driven policy management to unsupported platforms or to devices whose audit logging cannot attribute a change to an identity - on gear with no reliable audit trail, keep configuration changes on the manual, reviewed path.

**Detection.** Log and alert on authentications by the MCP service identity from unexpected hosts, on policy rule creation or modification outside change windows, and on any bulk rule change within a short interval. Reconcile agent-initiated policy changes against change records on a schedule.

**AI angle.** This is an AI/LLM integration on a security management platform: an MCP server that both reads policy context and writes policy rules. It belongs in the prompt-injection threat model for network and security management, not just in the product catalogue.

📄 **[Read the full report at Check Point Blog →](https://blog.checkpoint.com/ai-security/workforce-ai-security-policy-management-is-now-conversational/)**

---

## Arista

*1 item(s) — 1 critical*

### SD-WAN / SASE

#### 🔴 CRITICAL — Arista patches actively exploited VeloCloud Orchestrator zero-day

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/arista-patches-actively-exploited-velocloud-orchestrator-zero-day/) · 2026-09-23* 

> Arista patched an actively exploited zero-day in VeloCloud Orchestrator On-Prem, the controller that pushes configuration to SD-WAN edge devices.

**Affected:** VeloCloud Orchestrator (VCO) On-Prem deployments. No CVE identifier or fixed build is stated in the available reporting - check the vendor advisory for the fixed release and the affected version list.  

**What happened.** Arista Networks released security patches for a zero-day flaw in VeloCloud Orchestrator On-Prem that is reported as actively exploited. The collected reporting does not name a CVE, describe the vulnerability class, or state which VCO versions are fixed.

**Why it matters.** VCO is the orchestrator, not an edge box. It holds the templates, certificates and tunnel policy for every VeloCloud edge device in the fabric, so control of it means control of branch connectivity and the ability to push configuration - including insecure overrides - to sites you would otherwise have to reach individually. A management platform compromise outranks a single gateway compromise, and this one is reported as already being used.

**Recommended actions**

- Confirm whether your VeloCloud deployment is On-Prem; the reporting scopes the flaw to On-Prem VCO.
- Patch the orchestrator before touching edge devices, and validate certificate trust across the fabric after the upgrade.
- Check the vendor advisory for the fixed release and the full affected-version list rather than relying on secondary reporting.
- Restrict orchestrator access to management networks only and enforce MFA on orchestrator administrative accounts.
- Audit configuration templates for insecure overrides pushed to edge devices, and compare current templates against your last known-good.
- Rotate orchestrator admin credentials, API tokens and any service-account secrets after patching, and re-verify tunnel authentication.
- Review the orchestrator's activity log for administrative actions, template edits and device enrolments you cannot attribute to a change record.

**Legacy / unpatchable gear.** An unsupported orchestrator release with no available patch should not be internet-reachable at all: bind it to an out-of-band management network and require a jump host with full session logging for all access. If the platform version is beyond support entirely, treat the whole fabric's trust material as suspect, plan the orchestrator upgrade as the priority item, and until then freeze template changes so an unauthorized push is immediately visible.

**Detection.** Alert on any administrative login to VCO from outside the management network, on template or profile modifications outside change windows, and on new or re-enrolled edge devices. Diff the running template set against a stored baseline on a schedule, and monitor edge devices for tunnel re-establishment or policy changes not initiated by your team.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/arista-patches-actively-exploited-velocloud-orchestrator-zero-day/)**

---

## F5

*1 item(s) — 1 critical*

### Load balancers

#### 🔴 CRITICAL — Is This A Joke? In The Auth Header? (F5 BIG-IP UnAuth Heap-Overflow to RCE CVE-2026-94127)

*[watchTowr Labs](https://labs.watchtowr.com/is-this-a-joke-in-the-auth-header-f5-big-ip-unauth-heap-overflow-to-rce-cve-2026-94127/) · 2026-09-23* `KEV` `AI`

> watchTowr published a technical analysis of CVE-2026-94127, an unauthenticated heap overflow reaching RCE on F5 BIG-IP APM; the CVE is KEV-listed with a 25 September due date.

**Affected:** F5 BIG-IP APM where an access policy and an OAuth profile are configured on a virtual server, per the CISA KEV entry. KEV added 2026-09-22, due 2026-09-25. Check the vendor advisory for the fixed release - the sources here do not state a fixed version.  
**CVEs:** [CVE-2026-94127](https://nvd.nist.gov/vuln/detail/CVE-2026-94127) **[KEV]**  

**What happened.** watchTowr Labs published an analysis of CVE-2026-94127, an unauthenticated heap-based buffer overflow in F5 BIG-IP reached through the auth header and leading to remote code execution. CISA's KEV entry states the overflow occurs when an access policy and an OAuth profile are configured on a virtual server and allows an unauthenticated attacker to achieve remote code execution.

**Why it matters.** BIG-IP APM virtual servers are published to the internet by design - that is their job. The precondition (access policy plus OAuth profile) is a common remote-access configuration, and the trigger is in a header, so exploitation needs nothing but a request. A detailed public write-up shortens the interval before commodity exploitation, and the device sits in the TLS termination path with access to session material for everything behind it.

**Recommended actions**

- Enumerate BIG-IP virtual servers that have both an access policy and an OAuth profile attached - that is the exposed set.
- Check the vendor advisory for the fixed release and apply the hotfix to every unit in the device group; verify config sync after the upgrade.
- Remove the management interface and TMUI from public exposure entirely; restrict to the management network.
- Where the fix cannot be applied before the 25 September KEV date, consider detaching the OAuth profile from exposed virtual servers or fronting them with a WAF policy that rejects malformed auth headers, per the vendor's guidance.
- Rotate device certificates, admin credentials, API tokens and any OAuth client secrets held on the device after patching.
- Review iRules, access policies and virtual server configuration for unauthorized changes.
- Hunt for webshells and modified files on the appliance filesystem and for outbound connections originating from the BIG-IP.

**Legacy / unpatchable gear.** BIG-IP versions out of software support will not get a hotfix. Remove the OAuth-enabled access policy from any internet-facing virtual server on unsupported units and migrate that function to a supported device or to an upstream identity-aware proxy. Until migration, terminate external traffic on a supported inspecting device in front of the appliance, permit no direct inbound access, and monitor the appliance as a high-risk asset.

**Detection.** Alert on oversized or malformed Authorization headers reaching BIG-IP virtual servers, and on TMM process restarts or core dumps. Review access and audit logs for administrative sessions not originating from the management network, and baseline outbound flows from the appliance so any new destination raises an alert.

**AI angle.** watchTowr frames the current volume of vulnerability disclosures as driven by widespread LLM use to find and reproduce bugs - a researcher's aside rather than a measurement, but it argues for treating time-to-exploit on internet-facing appliances as days.

📄 **[Read the full report at watchTowr Labs →](https://labs.watchtowr.com/is-this-a-joke-in-the-auth-header-f5-big-ip-unauth-heap-overflow-to-rce-cve-2026-94127/)**

---

## MikroTik

*1 item(s) — 1 critical*

### Routers

#### 🔴 CRITICAL — MikroTrick Chain Let Attackers Take Over MikroTik Routers Without a Password or SSH Key

*[The Hacker News](https://thehackernews.com/2026/09/mikrotrick-chain-let-attackers-take.html) · 2026-09-23* `KEV`

> Two chained RouterOS SSH flaws, CVE-2026-67279 and CVE-2026-86060, hand attackers full admin control of internet-exposed MikroTik routers with no password or SSH key.

**Affected:** MikroTik RouterOS with SSH reachable from untrusted networks. CVE-2026-67279 (SSH state-machine flaw) chained with CVE-2026-86060 (argument injection in the RouterOS login process). CVE-2026-86060 is on CISA KEV, added 2026-09-10, due 2026-09-13. Check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-67279](https://nvd.nist.gov/vuln/detail/CVE-2026-67279), [CVE-2026-86060](https://nvd.nist.gov/vuln/detail/CVE-2026-86060) **[KEV]**  

**What happened.** CERT Polska documented a chain it calls MikroTrick: an SSH state-machine flaw (CVE-2026-67279) combined with an argument-injection bug in the RouterOS login process (CVE-2026-86060) gives full administrative control of internet-exposed routers without a password, an SSH key, or completed authentication. CISA's KEV entry for CVE-2026-86060 describes improper neutralization of argument delimiters allowing an attacker to change the trusted RouterOS policy mask, leading to privilege escalation. The reporting notes attack logs dating back before disclosure.

**Why it matters.** This is full admin on the router with authentication never completing, and the KEV due date has already passed. MikroTik gear is dense at the edge of ISPs, small sites and home-office deployments, is frequently exposed on purpose for remote administration, and is a long-standing botnet target. Attack logs predating disclosure means the correct assumption for an exposed device is that it was reachable during the exploited period, not that you are ahead of it.

**Recommended actions**

- Find every RouterOS device with SSH reachable from a WAN or untrusted interface - that is the exploitable population.
- Upgrade firmware to the current fixed release; verify the image hash against the vendor's published value before loading.
- Immediately disable WAN-side SSH and all WAN-side administration, along with unused services (Telnet, TFTP, HTTP, UPnP, remote management); administer over VPN or from a management network only.
- Treat any exposed device as compromised until proven otherwise: replace all credentials, rebuild rather than clean where the configuration cannot be fully verified, and re-issue SSH keys.
- Enforce per-admin accounts with centralized authentication and logging; remove shared and default accounts.
- Check for persistence: unexpected static routes, DNS server changes, port forwards, tunnels, scheduler entries, new users and changed group policy masks.
- Verify the RouterOS user group policy masks against a known-good configuration - the KEV entry describes the flaw as changing the trusted policy mask.

**Legacy / unpatchable gear.** RouterOS devices too old for a fixed release, and units where no upgrade path exists, must lose their internet-facing management immediately - deny inbound SSH at the upstream device and administer only from a jump host over an out-of-band path. Place the router behind a supported inspecting firewall, restrict it to the traffic it genuinely needs to pass, set a replacement date with a budget owner, and monitor it with a NetFlow baseline alerting on any new outbound flow.

**Detection.** Review SSH logs for connections that produce administrative activity without a completed authentication event, and for logins from unfamiliar source addresses. Diff the export of each router's configuration against known-good, paying attention to user groups and policy masks, scheduler and script entries, DNS settings, port forwards and tunnels. Alert on new outbound flows from routers, which is how botnet enrolment shows up first.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/09/mikrotrick-chain-let-attackers-take.html)**

---

## Cisco

*1 item(s) — 1 high*

### Firewalls

#### 🟠 HIGH — Cisco Secure Firewall Adaptive Security Appliance and Secure Firewall Threat Defense Software SSL VPN Denial of Service Vulnerability

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-asaftdvirtual-dos-MuenGnYR) · 2026-09-23* `vendor-advisory`

> Cisco widened an ASA/FTD advisory: an unauthenticated memory-exhaustion flaw in the VPN and management web servers halts SSL VPN processing on all platforms, not just virtual.

**Affected:** All Cisco Secure Firewall ASA Software and Cisco Secure Firewall Threat Defense (FTD) Software platforms, per the 16 September 2026 update. Version 1.0 of the advisory covered only ASAv and FTDv. No CVE or fixed release is stated in the collected text - check the vendor advisory for the fixed release.  

**What happened.** Cisco updated its advisory for a denial-of-service vulnerability in the VPN and management web servers of ASA and FTD software. The original version scoped the issue to the virtual appliances (ASAv, FTDv); the update states the vulnerability affects all ASA and FTD platforms. The flaw is a lack of proper memory management for new incoming SSL/TLS connections, allowing an unauthenticated remote attacker to exhaust system memory or buffer blocks, slowing and eventually stopping SSL VPN connection processing.

**Why it matters.** The scope change is the news: teams that read version 1.0 and confirmed they run physical appliances concluded they were unaffected, and that conclusion is now wrong. The impact is availability rather than code execution, but the thing that becomes unavailable is remote access for the whole workforce, triggered by an unauthenticated attacker sending SSL/TLS connections to an interface that has to be open. Recovery generally means a reload of the firewall.

**Recommended actions**

- Re-check this advisory even if you assessed version 1.0 as not applicable - physical ASA and FTD platforms are now in scope.
- Check the vendor advisory for the fixed release for your train and schedule the upgrade, HA pair secondary first so remote access survives the maintenance.
- Remove the management web server from any internet-facing interface; management belongs on a dedicated management VLAN or out-of-band network.
- Where the webvpn service must stay public, apply connection-rate and embryonic-connection limits toward the VPN interface and rate-limit new SSL/TLS connections at an upstream device.
- Monitor memory and buffer block utilisation on ASA/FTD and alert before exhaustion, so you get warning rather than an outage.
- Record the decision (patched / mitigated / accepted risk) against the firewall group in the change record.

**Legacy / unpatchable gear.** ASA hardware past end of software maintenance will not receive a fixed image and cannot be protected from an attacker who can open SSL/TLS connections to it. Move remote-access termination to a supported platform, and in the interim place rate limiting upstream, restrict which source ranges can reach the VPN interface, and keep an out-of-band administrative path so you can recover the device when it stops processing connections. Set a replacement date and document the accepted risk.

**Detection.** Trend memory and buffer block counters on ASA and FTD and alert on sustained decline rather than waiting for exhaustion. Watch for high volumes of new SSL/TLS connections that never complete a session, and for a rising count of failed VPN connection attempts from users while the device remains reachable to ICMP - that pattern is this condition rather than a link failure.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-asaftdvirtual-dos-MuenGnYR)**

---

## 🛡️ Vendor AI defences introduced

AI capabilities the vendors shipped this cycle - the answer side of the AI story.

### Check Point

**Workforce AI MCP** — Check Point's own Model Context Protocol server for Workforce AI Security, letting a connected AI client query, analyse and manage employee AI usage policy in natural language rather than through filters and per-rule checks. It covers users, managed assets, GenAI application usage, DLP data types, and agent and MCP activity in one conversation, and can create and update rules by request.

- *Defends against:* Unsanctioned and unmanaged employee use of GenAI applications and agents, and the policy-management burden of governing it - including shadowed or conflicting rules that leave AI usage effectively ungoverned.
- *Availability:* announced
- *Worth evaluating if:* Security teams already running Check Point Workforce AI Security who are struggling to keep an AI usage policy coherent across many applications and DLP data types. Evaluate the read and analysis capabilities first; treat the rule-writing path as a privileged administrative interface and gate it behind a scoped service identity and full audit logging before enabling it.
- 📄 [Read the announcement at Check Point Blog →](https://blog.checkpoint.com/ai-security/workforce-ai-security-policy-management-is-now-conversational/)

---

## 🧠 AI & network devices — LLM-accelerated exploit reproduction, and MCP arriving as a security-policy management path

Two genuine AI angles in this window, both about network gear rather than about AI in general.

First, watchTowr Labs opens its analysis of the F5 BIG-IP flaw (CVE-2026-94127) by observing that the current flood of vulnerabilities is being driven by widespread use of LLMs to find and reproduce them. That is a researcher's aside, not a measurement, but it matches what defenders should already be planning for: the gap between an advisory landing and a working exploit circulating is compressing, and it compresses hardest for internet-facing appliances where the attack surface is a single parseable request. Three of this window's KEV entries are exactly that shape.

Second, Check Point has released Workforce AI MCP, a Model Context Protocol server that lets a connected AI client query, analyse and modify employee AI usage policy in natural language - including creating and updating rules by request. Whatever its merits as a product, an MCP server that can write policy is an administrative interface. It should be scoped, authenticated and audited like one, and it inherits the standing problem that any attacker-influenced text an agent reads (usernames, application names, log content, ticket text) is a potential instruction to that agent.

**Vendors with an AI angle in this edition:** Check Point, F5

**Preventing it on current systems**

- Shorten the patch SLA for internet-facing network devices to days, not months - assume exploit development and mass scanning now move faster than your historical window.
- Treat any AI/LLM integration on or around network management (assistants, AIOps, MCP or agent connectors) as a privileged admin path: scoped credentials, read-only by default, no unattended config write, full audit trail.
- Require that an MCP or agent connector authenticate as a named service identity, not as a shared admin account, and log every rule change it makes to the SIEM with the initiating human attributed.
- Test AI-facing management surfaces for prompt injection from attacker-controlled data (device logs, hostnames, usernames, application names, ticket text) before letting an agent act on what it reads.
- Baseline NetFlow and syslog and alert on deviation from the baseline, rather than depending on signature updates to catch fast-moving edge-device exploitation.
- Require callback verification for any out-of-band request to change firewall or VPN configuration - AI-enabled social engineering of the network team is cheaper than exploiting the gateway.

**Preventing it on legacy / end-of-life systems**

- Legacy appliances that no longer receive fixes are the ones LLM-accelerated exploitation finds first: isolate them behind a supported inspecting firewall and deny inbound access from untrusted zones.
- Do not connect AI assistants, AIOps collectors or MCP servers to unsupported gear - there is no patch path if the integration itself turns out to be exploitable.
- Where legacy devices must stay reachable, front them with a reverse proxy or jump host that terminates and inspects the session, and log the full session.
- Monitor unsupported devices as high-risk assets: NetFlow baseline, alert on any new outbound flow or new administrative login.

---

<!--index
{
 "url": "/2026/09/24/network-security-digest/",
 "date": "2026-09-24",
 "entries": [
  {
   "title": "Check Point warns of hackers exploiting Security Gateway VPN RCE flaw",
   "link": "https://www.bleepingcomputer.com/news/security/check-point-warns-of-hackers-exploiting-security-gateway-vpn-rce-flaw/",
   "source": "BleepingComputer",
   "relevance": "critical",
   "device_types": [
    "firewall",
    "vpn-gateway"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [
    "CVE-2026-85102"
   ],
   "kev": true,
   "ai": false,
   "ai_angle": "",
   "summary": "Check Point confirms attackers are exploiting CVE-2026-85102, a pre-auth RCE in Security Gateway VPN certificate handling, with a CISA KEV deadline of 25 September.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Is This A Joke? In The Auth Header? (F5 BIG-IP UnAuth Heap-Overflow to RCE CVE-2026-94127)",
   "link": "https://labs.watchtowr.com/is-this-a-joke-in-the-auth-header-f5-big-ip-unauth-heap-overflow-to-rce-cve-2026-94127/",
   "source": "watchTowr Labs",
   "relevance": "critical",
   "device_types": [
    "load-balancer",
    "vpn-gateway"
   ],
   "vendors": [
    "F5"
   ],
   "cves": [
    "CVE-2026-94127"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "watchTowr frames the current volume of vulnerability disclosures as driven by widespread LLM use to find and reproduce bugs - a researcher's aside rather than a measurement, but it argues for treating time-to-exploit on internet-facing appliances as days.",
   "summary": "watchTowr published a technical analysis of CVE-2026-94127, an unauthenticated heap overflow reaching RCE on F5 BIG-IP APM; the CVE is KEV-listed with a 25 September due date.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Arista patches actively exploited VeloCloud Orchestrator zero-day",
   "link": "https://www.bleepingcomputer.com/news/security/arista-patches-actively-exploited-velocloud-orchestrator-zero-day/",
   "source": "BleepingComputer",
   "relevance": "critical",
   "device_types": [
    "sdwan",
    "management-platform"
   ],
   "vendors": [
    "Arista"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Arista patched an actively exploited zero-day in VeloCloud Orchestrator On-Prem, the controller that pushes configuration to SD-WAN edge devices.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "MikroTrick Chain Let Attackers Take Over MikroTik Routers Without a Password or SSH Key",
   "link": "https://thehackernews.com/2026/09/mikrotrick-chain-let-attackers-take.html",
   "source": "The Hacker News",
   "relevance": "critical",
   "device_types": [
    "router"
   ],
   "vendors": [
    "MikroTik"
   ],
   "cves": [
    "CVE-2026-67279",
    "CVE-2026-86060"
   ],
   "kev": true,
   "ai": false,
   "ai_angle": "",
   "summary": "Two chained RouterOS SSH flaws, CVE-2026-67279 and CVE-2026-86060, hand attackers full admin control of internet-exposed MikroTik routers with no password or SSH key.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco Secure Firewall Adaptive Security Appliance and Secure Firewall Threat Defense Software SSL VPN Denial of Service Vulnerability",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-asaftdvirtual-dos-MuenGnYR",
   "source": "Cisco PSIRT",
   "relevance": "high",
   "device_types": [
    "firewall",
    "vpn-gateway"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Cisco widened an ASA/FTD advisory: an unauthenticated memory-exhaustion flaw in the VPN and management web servers halts SSL VPN processing on all platforms, not just virtual.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Workforce AI Security Policy Management Is Now Conversational",
   "link": "https://blog.checkpoint.com/ai-security/workforce-ai-security-policy-management-is-now-conversational/",
   "source": "Check Point Blog",
   "relevance": "watch",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "This is an AI/LLM integration on a security management platform: an MCP server that both reads policy context and writes policy rules. It belongs in the prompt-injection threat model for network and security management, not just in the product catalogue.",
   "summary": "Check Point released Workforce AI MCP, a Model Context Protocol server letting an AI client query and edit employee AI usage policy in natural language.",
   "ai_defense": true,
   "defense": {
    "solution": "Workforce AI MCP",
    "what_it_does": "Check Point's own Model Context Protocol server for Workforce AI Security, letting a connected AI client query, analyse and manage employee AI usage policy in natural language rather than through filters and per-rule checks. It covers users, managed assets, GenAI application usage, DLP data types, and agent and MCP activity in one conversation, and can create and update rules by request.",
    "defends_against": "Unsanctioned and unmanaged employee use of GenAI applications and agents, and the policy-management burden of governing it - including shadowed or conflicting rules that leave AI usage effectively ungoverned.",
    "availability": "announced"
   }
  }
 ]
}
-->

## How this was produced

- Feeds polled: 22 ok, 1 failed
- Raw items: 709 → in window: 60 → network-device relevant: 8 → published: 6
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
