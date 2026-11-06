---
layout: post
title: "Network Device Security Digest - 2026-11-06"
date: 2026-11-06 14:05:44 +0000
edition: daily
critical_count: 2
item_count: 10
kev: "CVE-2023-20198, CVE-2026-111497"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-11-06

*daily edition · window 24h · 23 feeds · 10 items · 2 critical · generated 2026-11-06 14:05 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — Barracuda Email Security Gateway: pre-auth chain to remote root, with the full chain now public

watchTowr Labs published a two-flaw chain against Barracuda's Email Security Gateway (ESG) that ends in command execution as root with no login at all. The reporting names CVE-2026-116606 as an authorization bypass an unauthenticated attacker can reach directly, chained with CVE-2026-111560. Barracuda released patches on the Monday before disclosure; watchTowr published the technical write-up on the Thursday. Neither source states a fixed build number, so confirm the fixed release against Barracuda's own advisory before you schedule anything.

This matters more than a typical appliance bug because of where an ESG sits. A mail security gateway is reachable from the internet by design - it has to accept SMTP from anywhere - so the usual first move of pulling the exposed service behind an ACL is not available the way it is for a management interface. Barracuda ESG also has history: this is the same product line that was mass-exploited in 2023, and attackers who have burned effort on a product once tend to return to it.

Nothing in this cycle's sources reports exploitation of these two CVEs, and neither is on the CISA KEV catalog today. Treat that as a short head start rather than reassurance: the chain is documented publicly, pre-authentication, and gives root. Patch on an emergency change, then work on the assumption that anything reachable before the patch may already have been reached.

**Do this first**

- Identify every Barracuda ESG appliance (physical, virtual and hosted) and confirm in the appliance UI which build is actually installed - do not assume an update channel has applied the fix.
- Apply Barracuda's patched release; check the vendor advisory for the fixed release, as the public reporting does not state a build number.
- Rotate everything the appliance holds after patching: admin credentials, API tokens, LDAP/AD bind accounts, SMTP relay credentials and TLS certificates - a pre-patch compromise survives the upgrade.
- Restrict the ESG administrative interface to a management VLAN or jump host and remove it from internet exposure entirely; SMTP exposure is unavoidable, the admin plane is not.
- Hunt for compromise: unexpected local accounts, new or modified scripts and web resources, outbound connections from the appliance to unfamiliar hosts, and configuration restores you did not perform.
- If the appliance cannot be patched in this window, isolate it behind an inspecting firewall, deny all inbound access except SMTP from your mail path, and log every session to it.

---

> **On the CISA KEV catalog in this edition:** CVE-2023-20198, CVE-2026-111497 — treat these as confirmed-exploited and patch on an emergency change.

## Executive summary

- **Barracuda ESG** - a public chain of CVE-2026-116606 and CVE-2026-111560 gives unauthenticated remote root on the Email Security Gateway; patches are out, exploitation is not reported yet. Patch on an emergency change.
- **Cisco** - the October Node.js/Brotli bundle advisory includes CVE-2026-111497, which is on the CISA KEV catalog with a BOD 26-04 due date of 2026-11-10. Four days out, and the affected-product list is in the advisory, not in the feed.
- **D-Link** - multiple zero-days in DGS-3130 series managed switches have mitigations but no patch. Treat these as permanently exposed until D-Link says otherwise.
- **Cisco IOS XE** - SANS honeypots are logging exploit attempts consistent with CVE-2023-20198 (KEV, 2023). Three years on, the Web UI is still being swept. Check for privilege-15 accounts you did not create.
- **Cisco / certificate trust** - emergency distrust of eMudhra root and intermediate CAs after eight intermediate private keys proved brute-forceable in seconds. Audit anything on your network that trusts them.
- **OpenSSH** - CVE-2026-123158 is being exploited within hours of disclosure: a password-auth bypass on musl-based Linux with PAM disabled, fixed in 10.6p2. Ask your appliance vendors whether their firmware is in scope.
- **AI** - Check Point Research documents ARTEX, an autonomous AI pentest platform in criminal use with 1,608 live public instances. Plan patch windows for internet-facing devices in days.

---

## Items by vendor

## Cisco

*4 item(s) — 1 critical, 2 high, 1 watch*

### Firewalls

#### ⚪ WATCH — Firewalling the AI Era: Why the Network Still Decides

*[Cisco Security Blog](https://blogs.cisco.com/security/firewalling-the-ai-era-why-the-network-still-decides/) · 2026-11-05* `AI` `AI-defense`

> Cisco argues the firewall is the control point for containing autonomous AI agent traffic, enforcing policy at network boundaries.

**Affected:** No vulnerability and no specific product version. A Cisco Security Blog position piece on firewalls and AI agent traffic.  

**What happened.** Cisco published a blog post arguing that as AI agents act across systems, firewalls reveal what those agents reach, enforce policy at boundaries, and help contain the risks of autonomous action. No product, version or availability date is named in the source.

**Why it matters.** There is nothing to patch here, but the framing is worth a read if agents are starting to appear in your environment: egress policy and boundary visibility are the controls you already own, and they work on agent traffic without needing anything AI-specific. Treat it as a vendor's position rather than a capability you can procure from this post.

**Recommended actions**

- Identify where AI agents or assistants already reach across your network, and confirm that traffic crosses a firewall where you can see it.
- Write explicit egress policy for agent and automation traffic instead of letting it inherit broad server-zone rules.
- Log and review what those flows actually reach, and alert when an automation path touches a destination outside its stated scope.
- Keep any AI integration that can write configuration on a scoped, audited credential, and keep the approval step human.

**Legacy / unpatchable gear.** Legacy firewalls that cannot do application or identity-aware egress policy will not give you useful visibility into agent traffic. Do not try to retrofit it; place the agent-reachable segments behind a device that can, and keep the legacy firewall doing simple reachability denial only.

**Detection.** Baseline the destinations reached by automation and agent credentials, and alert on any new destination or protocol from those sources.

**AI angle.** Cisco's position is that the network boundary, not the model, is where autonomous agent behaviour becomes visible and containable - firewalls show what agents reach and enforce policy on it. A described approach, with no product named in the source.

📄 **[Read the full report at Cisco Security Blog →](https://blogs.cisco.com/security/firewalling-the-ai-era-why-the-network-still-decides/)**

### Routers

#### 🟠 HIGH — Vulnerability Scans for Older Cisco IOS Vulnerability / Opt-Out Headers, (Thu, Nov 5th)

*[SANS Internet Storm Center](https://isc.sans.edu/diary/rss/33492) · 2026-11-05* `KEV`

> SANS honeypots are logging exploit attempts against the Cisco IOS XE Web UI consistent with CVE-2023-20198, a KEV flaw from 2023.

**Affected:** Cisco IOS XE devices with the HTTP/HTTPS Web UI enabled. CVE-2023-20198, on the CISA KEV catalog since 2023-10-16 with a due date of 2023-10-20.  
**CVEs:** [CVE-2023-20198](https://nvd.nist.gov/vuln/detail/CVE-2023-20198) **[KEV]**  

**What happened.** SANS Internet Storm Center reported honeypot traffic that it assesses as scanning for CVE-2023-20198, the IOS XE Web UI privilege escalation flaw that allows a remote unauthenticated attacker to create a privilege-level-15 account.

**Why it matters.** Nothing here is new, which is the point: three years after the KEV listing, opportunistic scanning for this flaw is still profitable enough for someone to keep running it. That only works if unpatched IOS XE Web UIs are still reachable from the internet. A successful hit yields a level-15 account - full control of a router or switch, not a foothold on one.

**Recommended actions**

- Confirm whether the Web UI is needed at all; if not, disable the HTTP and HTTPS server features on IOS XE devices rather than relying on patching alone.
- Where the Web UI is required, restrict it with management-plane ACLs to your management subnet and move it onto an out-of-band management network.
- Verify the running version against Cisco's fixed-release table for CVE-2023-20198 and upgrade anything still exposed - check the vendor advisory for the fixed release.
- Audit the local user database on every IOS XE device for privilege-level-15 accounts nobody created, and review the configuration for unexpected tunnels, static routes or added scripts.
- Rotate local credentials, enable AAA via TACACS+/RADIUS with per-admin accounts, and ship device logs to the SIEM with alerting on user-account and configuration changes.

**Legacy / unpatchable gear.** IOS XE devices past end of support will not receive the fixed release. Disable the Web UI outright, place the device behind a supported inspecting firewall with no inbound access from untrusted zones, manage it only from a jump host, and schedule replacement. Treat any such device still reachable from the internet as an incident waiting to be filed.

**Detection.** Alert on creation of privilege-15 local accounts, on HTTP/HTTPS requests to the Web UI from outside the management network, and on configuration writes from unfamiliar source addresses. SANS reports this scanning is active now, so absence of alerts should be verified as absence of exposure, not assumed.

📄 **[Read the full report at SANS Internet Storm Center →](https://isc.sans.edu/diary/rss/33492)**

### Other

#### 🔴 CRITICAL — Vulnerabilities in Node.js and Brotli Affecting Cisco Products: October 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-oct2026-node-js-Jj3SQ19u) · 2026-11-05* `KEV` `vendor-advisory`

> Cisco's October bundle advisory covers four High-severity Node.js and Brotli flaws, including KEV-listed CVE-2026-111497, in products that embed Node.js 22.x, 24.x or 26.x.

**Affected:** Cisco products embedding Node.js release lines 22.x, 24.x and 26.x. CVE-2026-111497 (Brotli decoder out-of-bounds write, potential RCE), CVE-2026-105482 (DNS resolution truncation via embedded NUL), CVE-2026-105483 and CVE-2026-106069 (TLS hostname validation bypasses). Cisco rates all four High SIR. The affected-product table is in the advisory and the advisory states it will be updated - check it for the fixed releases.  
**CVEs:** [CVE-2026-105482](https://nvd.nist.gov/vuln/detail/CVE-2026-105482), [CVE-2026-105483](https://nvd.nist.gov/vuln/detail/CVE-2026-105483), [CVE-2026-106069](https://nvd.nist.gov/vuln/detail/CVE-2026-106069), [CVE-2026-111497](https://nvd.nist.gov/vuln/detail/CVE-2026-111497) **[KEV]**  

**What happened.** Node.js published fixes on 2026-10-21 for flaws in its Brotli decoder, DNS resolution and TLS certificate validation. Cisco's bundle advisory maps them onto its own products. CVE-2026-111497 is listed on the CISA KEV catalog, added 2026-10-27 under BOD 26-04 with a due date of 2026-11-10.

**Why it matters.** The KEV listing makes CVE-2026-111497 a confirmed-exploited flaw with a federal remediation deadline four days out, and the Brotli decoder path is reached by anything that decompresses attacker-supplied content. The two TLS hostname validation bypasses are quieter but worse in a different way: a device that accepts a certificate for the wrong name will build a trusted tunnel to the wrong endpoint, which is exactly the failure mode your management and telemetry channels depend on not having.

**Recommended actions**

- Pull the affected-product list from the Cisco advisory and match it against your asset inventory by model and running version - the feed entry does not name products.
- Apply the Cisco fixed releases where published; check the vendor advisory for the fixed release and re-check it, as Cisco states the advisory will be updated.
- Prioritise CVE-2026-111497: it is KEV-listed with a 2026-11-10 due date, so handle it as an emergency change rather than a scheduled one.
- Where no fixed release exists yet, reduce exposure: management and API interfaces off the internet, management-plane ACLs, MFA on all admin access.
- Treat the TLS validation bypasses as a trust problem, not just a patch: verify that device-to-manager and device-to-cloud channels pin or validate names as you expect once patched.

**Legacy / unpatchable gear.** Products past end of software maintenance will not get a fixed Node.js build. Isolate them behind a supported inspecting firewall, deny inbound access from untrusted zones, and restrict management strictly to a jump host with full session logging. Document the accepted risk and a replacement date; do not let a KEV-listed flaw sit on an unsupported device indefinitely.

**Detection.** Alert on unexpected restarts or crashes of web and API services on affected Cisco products, and on admin logins, configuration changes and firmware changes outside change windows. For the TLS flaws, log and review certificate validation failures and any newly trusted peer names.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-oct2026-node-js-Jj3SQ19u)**

#### 🟠 HIGH — Cisco Informational for Emergency Distrust of eMudhra Root and Intermediate Certificate Authorities

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-emudhra-info-ScS22A1r) · 2026-11-05* `vendor-advisory`

> Cisco issued an emergency-distrust notice for eMudhra root and intermediate CAs after researchers showed eight intermediate private keys can be brute-forced in seconds.

**Affected:** Cisco products that trust eMudhra root or intermediate certificate authorities. The advisory lists affected products and states it will be updated; products not listed are, per Cisco, not affected. Cisco also points to its Talos blog post and to separate Splunk guidance.  

**What happened.** A compromise involving eMudhra-managed certificate authorities was uncovered between 17 and 20 October 2026. A researcher demonstrated that the private keys of eight eMudhra intermediate certificates were fundamentally broken by an error in the April 2025 Key Generation Ceremony, allowing anyone to brute-force them within seconds. Cisco published an informational advisory on emergency distrust.

**Why it matters.** Anyone who can derive those private keys can mint certificates that your devices will accept as valid. On network infrastructure that means TLS interception and impersonation wherever those roots sit in a trust store - management channels, portal certificates, EAP/802.1X server validation, API callbacks. This is not patched by a software upgrade; it is fixed by removing trust, which means finding every trust store you own.

**Recommended actions**

- Read Cisco's advisory product list and identify which of your Cisco products are named; re-check it, since Cisco states the advisory will be updated.
- Inventory certificates issued under eMudhra roots or intermediates across your network devices - management interfaces, VPN portals, RADIUS/EAP server certificates and any device-to-manager trust - and replace them with certificates from a trusted CA.
- Remove or distrust the affected eMudhra roots and intermediates in every trust store you control: device trust stores, management platforms, jump hosts and administrative browsers.
- Where a device's trust store cannot be edited, restrict what that device will talk to with ACLs rather than relying on certificate validation, and raise the gap with the vendor.
- Record which devices were checked and cleared - the risk here is an unexamined trust store, so the audit trail is the control.

**Legacy / unpatchable gear.** Old network gear is where immutable or unmanageable trust stores live: devices whose CA bundle is baked into firmware that is no longer updated will keep trusting these CAs permanently. Assume TLS from such a device cannot be trusted to reach the right endpoint. Constrain it with strict egress ACLs to named destinations, terminate its management traffic on a jump host, log the sessions in full, and move replacement up the schedule.

**Detection.** Review TLS inspection and proxy logs for certificates chaining to eMudhra roots or intermediates, especially on management and administrative paths. Alert on any newly observed certificate issuer on device management channels.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-emudhra-info-ScS22A1r)**

---

## Barracuda

*2 item(s) — 1 critical, 1 high*

### Other

#### 🔴 CRITICAL — Barracuda Networks is your new best friend (Barracuda Email Security Gateway Pre-Auth Remote Root RCE CVE-2026-116606 / CVE-2026-111560)

*[watchTowr Labs](https://labs.watchtowr.com/barracuda-networks-is-your-new-best-friend-barracuda-email-security-gateway-pre-auth-remote-root-rce-cve-2026-116606-cve-2026-111560/) · 2026-11-05* 

> watchTowr published a chain of two Barracuda Email Security Gateway flaws that gives an unauthenticated attacker remote root on the appliance.

**Affected:** Barracuda Email Security Gateway (ESG). CVE-2026-116606 and CVE-2026-111560. Patches are released; neither source states a fixed build number - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-111560](https://nvd.nist.gov/vuln/detail/CVE-2026-111560), [CVE-2026-116606](https://nvd.nist.gov/vuln/detail/CVE-2026-116606)  

**What happened.** watchTowr Labs published a technical write-up chaining two ESG vulnerabilities, CVE-2026-116606 and CVE-2026-111560, into pre-authentication remote code execution as root. The write-up is public and detailed.

**Why it matters.** A mail security gateway accepts traffic from the internet as its job, so you cannot ACL away the exposure the way you can for a management interface. Pre-auth root means no credential, no session and no user interaction is needed, and the full chain is now in public hands. This product line was mass-exploited in 2023, so attacker familiarity with it is already paid for.

**Recommended actions**

- Inventory all ESG appliances - physical, virtual and hosted - and read the installed build from the appliance UI rather than trusting an update channel.
- Apply the patched release Barracuda has published; check the vendor advisory for the fixed release and record the build you land on.
- Rotate admin credentials, API tokens, LDAP/AD bind accounts, SMTP relay credentials and appliance TLS certificates after patching.
- Remove the administrative interface from internet exposure; restrict it to a management VLAN or a dedicated jump host, with MFA on admin access.
- Review the appliance configuration for changes you did not make, including restored backups, new local accounts and modified web resources.
- Ship appliance logs to the SIEM and record the decision (patched / mitigated / accepted risk) against the device group in the change record.

**Legacy / unpatchable gear.** An out-of-support ESG model will not receive this fix. Put it behind a supported inspecting firewall, permit only the SMTP flows your mail path needs, deny all other inbound traffic, and set a replacement date with a named budget owner. Monitor it as a high-risk asset with a NetFlow baseline and an alert on any new outbound flow - an appliance reaching out to somewhere new is the signal you will get.

**Detection.** Alert on any process on the appliance initiating outbound connections to hosts outside your mail path, on administrative sessions outside change windows, and on configuration-restore events. Review web server logs for unauthenticated requests to administrative endpoints preceding a configuration change.

📄 **[Read the full report at watchTowr Labs →](https://labs.watchtowr.com/barracuda-networks-is-your-new-best-friend-barracuda-email-security-gateway-pre-auth-remote-root-rce-cve-2026-116606-cve-2026-111560/)**

#### 🟠 HIGH — Experts Demonstrate Barracuda Email Security Gateway Chain Enabling Root Access Without Login

*[The Hacker News](https://thehackernews.com/2026/11/experts-demonstrate-barracuda-email.html) · 2026-11-06* 

> Reporting on the Barracuda ESG chain confirms CVE-2026-116606 is an authorization bypass usable with no login, with patches released on the Monday before disclosure.

**Affected:** Barracuda Email Security Gateway, the email-filtering appliance. CVE-2026-116606 (authorization bypass). Patches released; no fixed build number is stated - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-116606](https://nvd.nist.gov/vuln/detail/CVE-2026-116606)  

**What happened.** News coverage of the watchTowr research states the two flaws can be chained to run commands on the appliance as root with no login required, identifies CVE-2026-116606 as an authorization bypass reachable by an unauthenticated attacker, and notes Barracuda released patches on Monday with the research published Thursday.

**Why it matters.** This is the same issue as the lead item, and it matters here because it fixes the timeline: the patch preceded the public write-up by days, not weeks. If your appliances were not updated in that gap, you are now patching after the exploit path is public rather than before.

**Recommended actions**

- Confirm the patch Barracuda released is actually installed on every ESG appliance, and note the date it landed relative to the public disclosure.
- If the appliance was unpatched after the disclosure date, treat it as potentially compromised: rotate credentials, tokens and certificates, and hunt before you close the ticket.
- Keep the administrative interface off the internet and restricted to a management network with MFA.
- Subscribe the appliance's maintenance notifications to a monitored mailbox so vendor patch releases do not sit unnoticed for days.

**Legacy / unpatchable gear.** Unsupported ESG hardware or software versions will not receive this patch. Place the appliance behind a supported inspecting firewall, allow only required SMTP flows inbound, block all other inbound access, and plan migration to a supported platform with a documented accepted risk in the meantime.

**Detection.** Search appliance and perimeter logs for unauthenticated requests to administrative paths, for configuration restore operations, and for any new outbound connection from the appliance after the disclosure date.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/11/experts-demonstrate-barracuda-email.html)**

---

## D-Link

*1 item(s) — 1 high*

### Switches

#### 🟠 HIGH — D-Link warns of multiple zero-days in DGS-3130 Series switches

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/d-link-warns-of-multiple-zero-days-in-dgs-3130-series-switches/) · 2026-11-06* 

> D-Link published mitigation measures, not patches, for multiple zero-day flaws in DGS-3130 series managed stackable switches.

**Affected:** D-Link DGS-3130 series managed Gigabit stackable switches. No CVE IDs and no fixed firmware release are stated in the source; D-Link has shared mitigation measures only.  

**What happened.** D-Link warned of multiple unpatched security flaws in the DGS-3130 series and published mitigation guidance rather than a fix.

**Why it matters.** These are aggregation-layer managed switches, so a compromise puts an attacker in the middle of VLAN enforcement and the management plane rather than at the edge of it. With no patch available, the only controls you have are exposure reduction and monitoring - and 'unpatched' on a D-Link managed switch has a habit of meaning 'unpatched for a long time'.

**Recommended actions**

- Apply D-Link's published mitigation measures exactly as written, then verify them on each unit rather than assuming the template took.
- Restrict the management plane now: management ACLs permitting only your management subnet, out-of-band management where you have it, and no management reachability from user or guest VLANs.
- Disable unused services and legacy management protocols (Telnet, TFTP, plain HTTP) and harden SNMP to v3 with auth and privacy only - no public/private community strings.
- Replace default and reused credentials, move to per-admin accounts via TACACS+/RADIUS, and centralise logging.
- Check for persistence while you wait for firmware: unexpected admin accounts, modified configuration, unexpected static routes, and VLAN or port changes you did not make.
- Track D-Link's advisory for a fixed firmware release and apply it when published - check the vendor advisory for the fixed release.

**Legacy / unpatchable gear.** If your DGS-3130 units are out of support, assume no fix is coming at all. Isolate them behind a supported inspecting firewall, deny any inbound access from untrusted zones, and reach management only through a dedicated jump host with full session logging. Put a replacement date and a budget owner against them, monitor them as high-risk assets with a NetFlow baseline, and alert on new outbound flows from the switch management interface.

**Detection.** Alert on any management-plane connection to these switches sourced from outside the management subnet, on administrative logins outside change windows, and on configuration or firmware changes. Baseline the switch's own outbound traffic - a managed switch initiating connections is rarely legitimate.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/d-link-warns-of-multiple-zero-days-in-dgs-3130-series-switches/)**

---

## Unspecified

*3 item(s) — 3 medium*

### Other

#### 🟡 MEDIUM — OpenSSH flaw exploited in the wild just hours after disclosure (CVE-2026-123158)

*[Help Net Security](https://www.helpnetsecurity.com/2026/11/06/openssh-sshole-cve-2026-123158-exploitation/) · 2026-11-06* 

> A password-authentication bypass in OpenSSH, fixed in 10.6p2, is being exploited in the wild against musl-based Linux systems running sshd with PAM disabled.

**Affected:** OpenSSH on musl-based Linux systems with PAM disabled - Alpine Linux is the common example. CVE-2026-123158, nicknamed 'sshole', fixed in OpenSSH 10.6p2 released 2026-11-05. No network-device firmware is named in the source.  
**CVEs:** [CVE-2026-123158](https://nvd.nist.gov/vuln/detail/CVE-2026-123158)  

**What happened.** Threat intelligence firm Previdian reported exploitation of CVE-2026-123158 within hours of its disclosure. Per the reporting, an attacker who can reach the SSH port can log into passwordless accounts without knowing a password, in a single connection with no retries.

**Why it matters.** No vendor has stated that network-device firmware is affected, so this is not a confirmed appliance issue - but sshd is the management plane for most of your estate, Alpine and musl are common in appliance firmware and in the containers that run network tooling, and exploitation started the same day. The question worth an hour of your time is whether anything in your management path is in scope, and the answer has to come from your vendors.

**Recommended actions**

- Upgrade OpenSSH to 10.6p2 on any musl-based host in your management path - jump hosts, bastions, automation runners, network-tooling containers built on Alpine.
- Ask your appliance vendors directly whether their firmware ships an affected OpenSSH build on musl with PAM disabled, and record the answer per product.
- Audit for accounts with no password set on any host exposing sshd, and lock or remove them.
- Confirm management SSH is reachable only from the management network or a bastion, not from user VLANs or the internet, and enforce key-based authentication with password and keyboard-interactive authentication explicitly disabled.
- Review SSH authentication logs for successful logins to service or system accounts, and for logins from addresses outside the management network, since 2026-11-05.

**Legacy / unpatchable gear.** Unsupported appliances and hosts will not receive OpenSSH 10.6p2. For those, the control is reachability: deny access to port 22 from everything except a single jump host, enforce it on the upstream firewall rather than on the device, and log every session. Where the device offers a serial or out-of-band console, prefer it and close the network management path entirely.

**Detection.** Alert on successful SSH authentications for accounts that should never log in interactively, on single-attempt successes against accounts with no password history, and on any sshd connection from outside the management network.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/11/06/openssh-sshole-cve-2026-123158-exploitation/)**

#### 🟡 MEDIUM — A Critical SSH Password Bypass Hits musl-Based Linux

*[Check Point Blog](https://blog.checkpoint.com/exposure-management/a-critical-ssh-password-bypass-hits-musl-based-linux/) · 2026-11-05* 

> Check Point's analysis of the OpenSSH password bypass adds that the attack needs a second active account with a real password, and that preconditions are observable before login.

**Affected:** sshd on musl-based Linux with PAM disabled, Alpine Linux being the common example. Fixed in OpenSSH 10.6p2, released 2026-11-05.  

**What happened.** Check Point's write-up of the critical OpenSSH password-authentication bypass adds two findings from its own testing: the attack requires a second account that is active and has a real password, because a stored hash alone is not sufficient; and two of the preconditions are visible remotely before login, one being the OpenSSH version in the banner.

**Why it matters.** Both details change triage. The precondition about a second active account narrows which of your hosts are genuinely exploitable, so this is useful for prioritising a fleet rather than patching it alphabetically. The remotely observable preconditions cut the other way: attackers can identify exploitable targets by scanning, without touching authentication at all, which is consistent with exploitation starting within hours.

**Recommended actions**

- Use the stated preconditions to triage: hosts running affected OpenSSH on musl with PAM disabled and at least one active password-holding account go first.
- Assume your exposed SSH banners have already been inventoried by someone else; prioritise by internet reachability, not by internal importance.
- Suppress or restrict SSH version disclosure where your platform supports it, as a scanning-friction measure only - it is not a fix.
- Patch to OpenSSH 10.6p2 on affected hosts in the management path and verify the running version afterwards rather than trusting the package manager's report.

**Legacy / unpatchable gear.** Where an unsupported host or appliance cannot be upgraded, remove its SSH port from any untrusted reachability at the firewall, restrict it to one jump host, and log sessions in full. Treat the banner as public information and the host as permanently exploitable until replaced.

**Detection.** Scan your own ranges for the affected OpenSSH versions in SSH banners to find what an attacker would find, and alert on successful authentications to accounts that have no interactive login history.

📄 **[Read the full report at Check Point Blog →](https://blog.checkpoint.com/exposure-management/a-critical-ssh-password-bypass-hits-musl-based-linux/)**

#### 🟡 MEDIUM — Inside ARTEX: How Autonomous AI Agents Are Being Used in Real-World Cyberattacks

*[Check Point Blog](https://blog.checkpoint.com/research/inside-artex-how-autonomous-ai-agents-are-being-used-in-real-world-cyberattacks/) · 2026-11-05* `AI` `AI-defense`

> Check Point Research found 1,608 publicly accessible instances of ARTEX, an AI penetration-testing platform criminals use to run autonomous exploitation at scale.

**Affected:** No specific device or vendor. The research concerns attacker tooling that targets internet-reachable systems generally, which includes internet-facing network devices.  

**What happened.** Check Point Research analysed ARTEX, an AI-powered penetration-testing platform it reports is increasingly adopted by cybercriminals, with visibility from inside operator consoles. Per CPR, attackers set an initial objective and leave autonomous agents to find vulnerabilities, exploit systems and pursue the goal at scale with very little human involvement. CPR identified 2,557 systems hosting ARTEX, including 1,608 live, publicly accessible instances at the time of the research.

**Why it matters.** This is a threat report, not a device advisory, and it changes one planning assumption rather than requiring a change window: the time between a public write-up and attempts against your edge. If finding and exploiting are objective-driven and automated, a patch SLA calibrated on human attacker effort is calibrated on the wrong thing. Read it next to this cycle's Barracuda chain, which is exactly the kind of public write-up such a platform consumes.

**Recommended actions**

- Re-baseline the patch SLA for internet-facing network devices to days, and define who can authorise an emergency change outside business hours.
- Reduce the attack surface the automation enumerates: management interfaces off the internet, unused services disabled, and an accurate inventory of what you actually expose.
- Baseline NetFlow and syslog on edge devices and alert on deviation, rather than depending on signature updates to recognise novel tooling.
- Run your own external discovery against your ranges on a schedule, so your inventory of exposed devices is not less current than an attacker's.

**Legacy / unpatchable gear.** Unpatchable gear is the obvious target for indiscriminate automated scanning, since it stays vulnerable indefinitely. Take it off internet reachability entirely, place it behind a supported inspecting firewall, deny inbound access from untrusted zones, and monitor it as a high-risk asset with alerting on any new outbound flow. Obscurity is not a control against a platform that enumerates continuously.

**Detection.** Alert on bursts of varied exploit attempts against the same edge device from a single source or ASN, and on sequential probing across your public ranges. Review perimeter logs for exploitation attempts following within hours of any public vulnerability write-up affecting your gear.

**AI angle.** This is the AI story of the cycle: autonomous agents, operating from an objective rather than a script, doing vulnerability discovery and exploitation with minimal human involvement - and already deployed at scale, with 1,608 live public instances counted.

📄 **[Read the full report at Check Point Blog →](https://blog.checkpoint.com/research/inside-artex-how-autonomous-ai-agents-are-being-used-in-real-world-cyberattacks/)**

---

## 🛡️ Vendor AI defences introduced

AI capabilities the vendors shipped this cycle - the answer side of the AI story.

### Cisco

**Firewall policy enforcement at network boundaries for AI agent traffic (no product named in the source)** — Cisco describes using firewalls as the point where AI agent activity becomes visible: revealing what agents reach, enforcing policy at network boundaries, and helping contain the risks of autonomous action. The post names no product, version or availability date.

- *Defends against:* Unbounded or unmonitored action by autonomous AI agents operating across internal systems.
- *Availability:* described-only - a vendor position piece, not a product announcement
- *Worth evaluating if:* Network and security teams whose organisations are deploying AI agents that reach across internal systems, and who need to decide where to put the control point before the agents arrive. Read it as framing for egress policy work you can do on existing firewalls, not as a procurement item.
- 📄 [Read the announcement at Cisco Security Blog →](https://blogs.cisco.com/security/firewalling-the-ai-era-why-the-network-still-decides/)

---

## 🧠 AI & network devices — Autonomous agents are shrinking the gap between disclosure and exploitation

Two items this cycle put the AI question in concrete terms for network teams. Check Point Research analysed ARTEX, an AI-powered penetration-testing platform it says is being adopted by criminals: operators set an initial objective and leave autonomous agents to find vulnerabilities, exploit systems and pursue that goal at scale with very little human involvement. CPR counted 2,557 systems that hosted ARTEX, of which 1,608 were live and publicly accessible at the time of the research.

The practical consequence for anyone running internet-facing network devices is timing, not novelty. If objective-driven agents are doing the finding and the exploiting, the interval between a public write-up - such as this cycle's Barracuda chain - and opportunistic attempts against your edge is measured in hours, not in the weeks a quarterly patch cycle assumes.

Cisco's security blog argues the other half of this from the defender's side: as AI agents act across systems, the firewall is where you can see what they actually reach and enforce policy at the boundary. That is a position, not a product announcement, and it is worth taking only as far as it goes - the containment it describes depends on having boundaries and egress policy in place before an agent, yours or someone else's, starts acting.

**Vendors with an AI angle in this edition:** Cisco, Unspecified

**Preventing it on current systems**

- Shorten the patch window for internet-facing network devices to days, not months - assume exploit development and mass scanning now outpace your historical SLA.
- Treat any AI/LLM integration on network gear (assistants, AIOps, MCP or agent connectors) as a privileged admin path: scoped read-only credentials, no unattended config write, full audit trail.
- Test AI-facing management surfaces for prompt injection from attacker-controlled data - device logs, hostnames, ticket text - before letting an agent act on them.
- Enforce egress policy and segmentation at the firewall so that an agent, or an attacker driving one, cannot reach beyond the boundary it is supposed to work inside.
- Baseline NetFlow and syslog and alert on deviation rather than relying on signature updates alone; automated attack tooling does not wait for your next feed update.
- Require callback verification for any out-of-band request to change firewall or VPN configuration - AI-assisted social engineering of the network team is cheaper than exploiting your edge.

**Preventing it on legacy / end-of-life systems**

- End-of-life gear will not get a fix for anything an agent finds: isolate it behind a supported inspecting firewall and deny inbound access from untrusted zones.
- Assume unpatchable devices are discoverable - automated scanning is cheap and indiscriminate. Remove them from internet reachability rather than relying on obscurity.
- Put compensating controls around them: strict ACLs, no direct management access, dedicated jump host, full session logging.
- Monitor legacy devices as high-risk assets with a NetFlow baseline and alerts on any new outbound flow, and set a replacement date with a budget owner.

---

<!--index
{
 "url": "/2026/11/06/network-security-digest/",
 "date": "2026-11-06",
 "entries": [
  {
   "title": "Barracuda Networks is your new best friend (Barracuda Email Security Gateway Pre-Auth Remote Root RCE CVE-2026-116606 / CVE-2026-111560)",
   "link": "https://labs.watchtowr.com/barracuda-networks-is-your-new-best-friend-barracuda-email-security-gateway-pre-auth-remote-root-rce-cve-2026-116606-cve-2026-111560/",
   "source": "watchTowr Labs",
   "relevance": "critical",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Barracuda"
   ],
   "cves": [
    "CVE-2026-111560",
    "CVE-2026-116606"
   ],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "watchTowr published a chain of two Barracuda Email Security Gateway flaws that gives an unauthenticated attacker remote root on the appliance.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Vulnerabilities in Node.js and Brotli Affecting Cisco Products: October 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-oct2026-node-js-Jj3SQ19u",
   "source": "Cisco PSIRT",
   "relevance": "critical",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [
    "CVE-2026-105482",
    "CVE-2026-105483",
    "CVE-2026-106069",
    "CVE-2026-111497"
   ],
   "kev": true,
   "ai": false,
   "ai_angle": "",
   "summary": "Cisco's October bundle advisory covers four High-severity Node.js and Brotli flaws, including KEV-listed CVE-2026-111497, in products that embed Node.js 22.x, 24.x or 26.x.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "D-Link warns of multiple zero-days in DGS-3130 Series switches",
   "link": "https://www.bleepingcomputer.com/news/security/d-link-warns-of-multiple-zero-days-in-dgs-3130-series-switches/",
   "source": "BleepingComputer",
   "relevance": "high",
   "device_types": [
    "switch"
   ],
   "vendors": [
    "D-Link"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "D-Link published mitigation measures, not patches, for multiple zero-day flaws in DGS-3130 series managed stackable switches.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Experts Demonstrate Barracuda Email Security Gateway Chain Enabling Root Access Without Login",
   "link": "https://thehackernews.com/2026/11/experts-demonstrate-barracuda-email.html",
   "source": "The Hacker News",
   "relevance": "high",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Barracuda"
   ],
   "cves": [
    "CVE-2026-116606"
   ],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Reporting on the Barracuda ESG chain confirms CVE-2026-116606 is an authorization bypass usable with no login, with patches released on the Monday before disclosure.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Vulnerability Scans for Older Cisco IOS Vulnerability / Opt-Out Headers, (Thu, Nov 5th)",
   "link": "https://isc.sans.edu/diary/rss/33492",
   "source": "SANS Internet Storm Center",
   "relevance": "high",
   "device_types": [
    "router",
    "switch"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [
    "CVE-2023-20198"
   ],
   "kev": true,
   "ai": false,
   "ai_angle": "",
   "summary": "SANS honeypots are logging exploit attempts against the Cisco IOS XE Web UI consistent with CVE-2023-20198, a KEV flaw from 2023.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco Informational for Emergency Distrust of eMudhra Root and Intermediate Certificate Authorities",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-emudhra-info-ScS22A1r",
   "source": "Cisco PSIRT",
   "relevance": "high",
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
   "summary": "Cisco issued an emergency-distrust notice for eMudhra root and intermediate CAs after researchers showed eight intermediate private keys can be brute-forced in seconds.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "OpenSSH flaw exploited in the wild just hours after disclosure (CVE-2026-123158)",
   "link": "https://www.helpnetsecurity.com/2026/11/06/openssh-sshole-cve-2026-123158-exploitation/",
   "source": "Help Net Security",
   "relevance": "medium",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Unspecified"
   ],
   "cves": [
    "CVE-2026-123158"
   ],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "A password-authentication bypass in OpenSSH, fixed in 10.6p2, is being exploited in the wild against musl-based Linux systems running sshd with PAM disabled.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "A Critical SSH Password Bypass Hits musl-Based Linux",
   "link": "https://blog.checkpoint.com/exposure-management/a-critical-ssh-password-bypass-hits-musl-based-linux/",
   "source": "Check Point Blog",
   "relevance": "medium",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Unspecified"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Check Point's analysis of the OpenSSH password bypass adds that the attack needs a second active account with a real password, and that preconditions are observable before login.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Inside ARTEX: How Autonomous AI Agents Are Being Used in Real-World Cyberattacks",
   "link": "https://blog.checkpoint.com/research/inside-artex-how-autonomous-ai-agents-are-being-used-in-real-world-cyberattacks/",
   "source": "Check Point Blog",
   "relevance": "medium",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Unspecified"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "This is the AI story of the cycle: autonomous agents, operating from an objective rather than a script, doing vulnerability discovery and exploitation with minimal human involvement - and already deployed at scale, with 1,608 live public instances counted.",
   "summary": "Check Point Research found 1,608 publicly accessible instances of ARTEX, an AI penetration-testing platform criminals use to run autonomous exploitation at scale.",
   "ai_defense": true,
   "defense": null
  },
  {
   "title": "Firewalling the AI Era: Why the Network Still Decides",
   "link": "https://blogs.cisco.com/security/firewalling-the-ai-era-why-the-network-still-decides/",
   "source": "Cisco Security Blog",
   "relevance": "watch",
   "device_types": [
    "firewall"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "Cisco's position is that the network boundary, not the model, is where autonomous agent behaviour becomes visible and containable - firewalls show what agents reach and enforce policy on it. A described approach, with no product named in the source.",
   "summary": "Cisco argues the firewall is the control point for containing autonomous AI agent traffic, enforcing policy at network boundaries.",
   "ai_defense": true,
   "defense": {
    "solution": "Firewall policy enforcement at network boundaries for AI agent traffic (no product named in the source)",
    "what_it_does": "Cisco describes using firewalls as the point where AI agent activity becomes visible: revealing what agents reach, enforcing policy at network boundaries, and helping contain the risks of autonomous action. The post names no product, version or availability date.",
    "defends_against": "Unbounded or unmonitored action by autonomous AI agents operating across internal systems.",
    "availability": "described-only - a vendor position piece, not a product announcement"
   }
  }
 ]
}
-->

## How this was produced

- Feeds polled: 23 ok, 0 failed
- Raw items: 719 → in window: 57 → network-device relevant: 12 → published: 10
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
