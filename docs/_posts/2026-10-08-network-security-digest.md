---
layout: post
title: "Network Device Security Digest - 2026-10-08"
date: 2026-10-08 13:15:55 +0000
edition: daily
critical_count: 6
item_count: 19
kev: "none"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-10-08

*daily edition · window 24h · 22 feeds · 19 items · 6 critical · generated 2026-10-08 13:15 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — FBI warns FortiBleed attacks are ongoing: attackers are locking admins out of FortiGate firewalls and SSL VPN gateways

The FBI has issued a warning that the attack campaign reported as "FortiBleed" is still active against internet-exposed Fortinet FortiGate firewalls and SSL VPN gateways. Unlike most edge-device campaigns, the reported goal is not quiet persistence: the attackers create new administrative accounts and then delete existing accounts and passwords, locking the legitimate administrators out of their own perimeter device. Two independent reports in this cycle (BleepingComputer citing the FBI, and SecurityWeek) describe the same behaviour.

This matters more than a typical advisory because it inverts the usual incident sequence. On a device you can no longer authenticate to, you cannot read the configuration, pull logs, check for added admin users, or push a fixed image - the normal first five steps of edge-device incident response are unavailable. Recovery becomes console or out-of-band access, or a rebuild from a known-good configuration backup, under time pressure, on the box that terminates your remote access.

Neither source names a CVE or a fixed build, so do not assume you know which flaw is being used or that any particular version is safe. Treat exposure as the variable you control: the campaign is described as targeting exposed FortiGate firewalls and SSL VPN gateways, so anything with an administrative or SSL-VPN interface reachable from the internet is in scope until you have confirmed the running build against Fortinet's advisory.

Act on the premise that a successful intrusion may already have happened and simply has not reached the lockout stage on your device yet. Confirm you still have administrative access and a current configuration backup before anything else, because that is the asset the attackers destroy.

**Do this first**

- Confirm right now that you can authenticate to every internet-facing FortiGate and that an out-of-band path (console server, serial, dedicated management link) works - verify this before you start changing anything.
- Export and store a current configuration backup off-device for every FortiGate and SSL VPN gateway, and verify the export is readable.
- Remove the administrative interface (HTTPS/SSH/API) from all internet-facing interfaces; restrict administration to a dedicated management VLAN or out-of-band network.
- Audit the local administrator list, API users and admin profiles for accounts you did not create, and compare against your last known-good configuration backup.
- Check the running version against Fortinet's advisory and schedule the upgrade to the vendor's fixed build - check the vendor advisory for the fixed release, as no fixed version is stated in these reports.
- Terminate all active SSL VPN sessions, revoke session tokens, and rotate local admin credentials, API keys, certificates and VPN pre-shared keys after upgrading - a pre-patch compromise survives the upgrade.
- Enforce phishing-resistant MFA on administrative and VPN accounts, and add geo/ASN conditional access on the remote-access portal.
- Alert in the SIEM on FortiGate admin account creation, admin account deletion and password changes, and treat any such event on a perimeter device as an incident until proven otherwise.

---

## Executive summary

- The FBI says FortiBleed attacks against exposed Fortinet FortiGate firewalls and SSL VPN gateways are ongoing, with attackers creating admin accounts and deleting existing ones to lock legitimate administrators out. This is the one to act on today.
- Cisco published a large 7 October 2026 bundle dominated by NX-OS: unauthenticated remote code execution as root on Nexus 3000 and 9000 switches via MPLS OAM (CVE-2026-76465, CVSS 9.8), further RCE via NGOAM, RCE via NX-API, and a critical NX-OS hardening release (CVE-2026-76453/76455/76456/76457/76458/76459, CVSS 9.8). None have workarounds - the fix is the upgrade.
- Cisco APIC, the ACI fabric controller, takes three issues including API command injection to root and file access that exposes key material usable to elevate to root on managed switches. A fabric controller compromise is a whole-fabric compromise.
- SonicWall shipped hotfixes for four SMA1000 flaws, the worst a pre-authentication SSRF rated CVSS 10.0; SonicWall states it has no evidence any of the four is being exploited.
- TP-Link is facing US state lawsuits over ISP router flaws, with SEC Consult now publishing technical detail on the vulnerabilities named in the complaint - relevant to anyone with ISP-supplied TP-Link CPE in branch or home-office estates.
- Cisco's advance notification ties this disclosure format - findings grouped by CWE with one CVE per grouping - to its stated response to AI-accelerated vulnerability discovery, so triage must now key on model and version, not CVE count.

---

## Items by vendor

## Cisco

*14 item(s) — 4 critical, 5 high, 5 medium*

### Switches

#### 🔴 CRITICAL — Cisco Nexus 3000 and 9000 Series Switches MPLS OAM Remote Code Execution Vulnerability

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-moam-rce-uBTzYV7) · 2026-10-07* `vendor-advisory`

> An unauthenticated attacker can get root on Cisco Nexus 3000 and 9000 switches by sending one crafted MPLS echo-request; CVE-2026-76465, CVSS 9.8, no workaround.

**Affected:** Cisco Nexus 3000 Series Switches and Cisco Nexus 9000 Series Switches running Cisco NX-OS Software with the MPLS Operation, Administration and Maintenance (OAM) feature. CVE-2026-76465, rated Critical with a CVSS base score of 9.8 per Cisco's advance notification. Check the vendor advisory's Vulnerable Products section for the affected and fixed releases.  

**What happened.** Improper validation when an affected device processes an MPLS echo-request packet. An attacker sends a crafted MPLS echo-request to an IP address on the device and can execute arbitrary code with root privileges, or crash the process and force a device reload.

**Why it matters.** This is pre-authentication root on data-centre access and spine switches, reachable by a single packet to any IP address the device answers on. Root on a Nexus switch means traffic interception, VLAN and routing manipulation, and a persistence point that survives most host-level rebuilds. Cisco states there are no workarounds, so the only remediation is the software update.

**Recommended actions**

- Inventory Nexus 3000 and 9000 switches and check whether MPLS OAM is configured; the advisory's Vulnerable Products section defines the exact feature condition.
- Upgrade NX-OS to the fixed release from the advisory - validate the ISSU/hitless path for spine and distribution switches first, and verify image integrity against Cisco's published hash before loading.
- Until the upgrade lands, apply management and control-plane ACLs so MPLS OAM packets are only accepted from trusted peer addresses, and police control-plane traffic.
- Restrict every switch IP address that need not be reachable from user, guest or partner segments; the attack targets any IP on the device, not just the management address.
- Record the decision (patched / mitigated / accepted risk) against the switch group in the change record.
- Harden SNMP to v3 with auth and privacy only and remove public/private community strings while you are in the change window.

**Legacy / unpatchable gear.** Nexus platforms past end of software maintenance will not receive this fix. Deny MPLS and control-plane traffic to them from any untrusted zone with upstream ACLs on a supported device, place them behind a supported inspecting firewall, restrict management to a dedicated jump host with full session logging, and set a replacement date with a named budget owner.

**Detection.** Alert on unexpected MPLS echo-request traffic arriving at switch IP addresses from non-peer sources (NetFlow, or control-plane counters). Treat unexplained process crashes and device reloads on Nexus 3000/9000 as possible exploitation attempts and preserve the crash logs. Ship switch syslog to the SIEM and alert on configuration changes, image changes and administrative logins.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-moam-rce-uBTzYV7)**

#### 🔴 CRITICAL — Cisco Nexus 3000 and 9000 Series Switches NGOAM Remote Code Execution Vulnerabilities

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-ngoam-rce-LWKQ4BU) · 2026-10-07* `vendor-advisory`

> Multiple NGOAM flaws in Cisco NX-OS let an unauthenticated attacker run code as root on Nexus 3000 and 9000 switches with crafted packets; no workaround exists.

**Affected:** Cisco Nexus 3000 Series and Nexus 9000 Series Switches running Cisco NX-OS Software with Next Generation OAM (NGOAM) enabled. Cisco notes feature requirements vary per vulnerability - check the vendor advisory's Vulnerable Products section for the affected and fixed releases.  

**What happened.** Multiple vulnerabilities caused by improper input validation of IP traffic in the NGOAM feature. An attacker sends crafted packets to an IP interface on the device and can execute arbitrary code with root privileges, or crash processes and force a reload.

**Why it matters.** A second unauthenticated root path into the same switch families as the MPLS OAM flaw, triggered by traffic to an IP interface. Fixing one does not fix the other, and the feature conditions differ per CVE, so a switch that looks out of scope for one advisory may be in scope for this one. Cisco states there are no workarounds.

**Recommended actions**

- Check whether NGOAM is enabled on each Nexus 3000/9000; the per-vulnerability feature conditions are in the advisory's Vulnerable Products section and are not uniform.
- Upgrade NX-OS to the fixed release, bundling this with the MPLS OAM and NX-API upgrades into a single change window rather than three.
- Disable NGOAM where it is not operationally required, and verify after the change that no fabric verification tooling depended on it.
- Apply control-plane policing and management ACLs so the switch only accepts OAM traffic from trusted fabric peers.
- Verify image integrity against Cisco's hash before loading, and upgrade the standby/secondary of any pair first.

**Legacy / unpatchable gear.** Unsupported Nexus hardware will not get the fixed image. Block OAM-class traffic to it at a supported upstream device, segment it away from user and guest networks entirely, permit management only from a dedicated jump host, and document the accepted risk with a replacement date.

**Detection.** Baseline which hosts legitimately source OAM traffic inside the fabric and alert on NGOAM-class packets from anything else. Correlate process crashes or unexpected reloads across multiple switches in the same window - a repeated crash pattern is more likely exploitation than hardware fault.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-ngoam-rce-LWKQ4BU)**

#### 🔴 CRITICAL — Cisco NX-OS Software NX-API Remote Code Execution Vulnerability

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-napi-rce-r2shwu2j) · 2026-10-07* `vendor-advisory`

> A crafted HTTP request to the NX-API on Cisco NX-OS gives an unauthenticated attacker arbitrary code execution as root, or crashes the device into a reload.

**Affected:** Cisco NX-OS Software with the NX-API feature enabled. Check the vendor advisory for the affected platforms and the fixed release.  

**What happened.** Insufficient input validation of data sent to the NX-API. An unauthenticated remote attacker sends a crafted HTTP request to the NX-API and can execute arbitrary code with root privileges, or cause process crashes leading to a device reload and denial of service.

**Why it matters.** NX-API is the programmatic management surface, so it is exactly the interface automation platforms, CI pipelines and monitoring systems reach - and therefore the one most likely to have been opened up beyond a strict management VLAN. Pre-authentication root through it needs nothing but HTTP reachability. Cisco states there are no workarounds.

**Recommended actions**

- Determine where NX-API (the 'feature nxapi' HTTP/HTTPS server) is enabled, and disable it on every switch that no automation actually drives.
- Upgrade NX-OS to the fixed release from the advisory.
- Restrict NX-API to the management VLAN or out-of-band network with a management ACL naming only the automation hosts and jump hosts that need it; it must not be reachable from user, guest, server or internet-facing segments.
- Confirm no load balancer, reverse proxy or NAT rule publishes NX-API outside the management network.
- After upgrading, rotate the credentials and API tokens your automation uses against NX-API.
- Enable control-plane policing for the HTTP/HTTPS management service to limit crash-and-reload abuse.

**Legacy / unpatchable gear.** On NX-OS versions that will not receive a fix, disable NX-API outright and drive the device by SSH from a jump host instead - this one is genuinely mitigable by turning the feature off. If NX-API cannot be disabled because automation depends on it, restrict it with an upstream ACL to a single automation host and log every request.

**Detection.** Alert on any NX-API HTTP request from a source outside your automation allowlist. Review the NX-API access log for malformed or oversized request bodies and for requests that precede a process crash. Alert in the SIEM on NX-OS process crashes and unscheduled reloads.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-napi-rce-r2shwu2j)**

#### 🔴 CRITICAL — Cisco NX-OS Software Security Hardening Release: October 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-nxosw1-cWzSbtR) · 2026-10-07* `vendor-advisory`

> Cisco's NX-OS hardening release fixes internally found vulnerabilities rated Critical at CVSS 9.8 (CVE-2026-76453/76455/76456/76457/76458/76459), with no workarounds available.

**Affected:** Cisco NX-OS Software. Per Cisco's advance notification the release covers CVE-2026-76453, CVE-2026-76455, CVE-2026-76456, CVE-2026-76457, CVE-2026-76458 and CVE-2026-76459, rated Critical with a CVSS base score of 9.8. Check the vendor advisory for the affected platforms and the fixed releases.  

**What happened.** A Cisco NX-OS internal security review produced hardening releases addressing multiple internally discovered vulnerabilities, grouped by Common Weakness Enumeration class with a single CVE ID assigned per grouping. Cisco states they were found during internal testing and are not known to be actively exploited. Software updates are available; there are no workarounds.

**Why it matters.** 'Hardening release' language understates this one: the advance notification scores it Critical at CVSS 9.8, the same as the Nexus MPLS OAM remote code execution. Internally discovered and not known exploited means you have time the FortiBleed victims did not, but 9.8 on the NOS running your data-centre fabric belongs in the next change window, not the next quarter. Because the CVEs are grouped by weakness class, scanning for individual CVE IDs will not tell you whether a given switch is affected - version comparison will.

**Recommended actions**

- Treat this as the anchor of a single NX-OS change window covering the MPLS OAM, NGOAM, NX-API, control-plane DoS and Python sandbox advisories together.
- Match your switches to the advisory by platform and running version rather than by CVE ID, since each CVE covers a class of defects.
- Upgrade NX-OS to the fixed release, validating the ISSU/hitless path for core and distribution switches first and standby units before active.
- Verify image integrity against Cisco's published hash before loading.
- Restrict the management plane with management ACLs and control-plane policing, and move switch administration to out-of-band.
- Record a patched / mitigated / accepted-risk decision per switch group in the change record.

**Legacy / unpatchable gear.** Nexus platforms excluded from the hardening release have no remediation path. Segment them so they carry no untrusted traffic, deny inbound access from untrusted zones with ACLs on a supported upstream device, permit management only from a dedicated jump host with session logging, monitor them as high-risk assets with a NetFlow baseline, and set a replacement date and budget owner.

**Detection.** There is no published exploitation signature, so detect on the fundamentals: alert on NX-OS administrative logins, configuration changes, image changes, unexpected process crashes and unscheduled reloads. Use an inventory query of running versions as the primary control, and alert when a switch reappears on a pre-fix image after a replacement or RMA.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-nxosw1-cWzSbtR)**

#### 🟠 HIGH — Cisco Nexus 9000 Series Fabric Switches in ACI Mode Endpoint Group Contract Bypass Vulnerability

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-aci-epgcbp-SfDU7NLf) · 2026-10-07* `vendor-advisory`

> Crafted DHCP-port UDP traffic bypasses endpoint group contracts on Cisco Nexus 9000 fabric switches in ACI mode, defeating the segmentation policy the fabric enforces.

**Affected:** Cisco Nexus 9000 Series Fabric Switches in ACI Mode, in the endpoint group (EPG) contract functionality. Check the vendor advisory for the affected and fixed releases.  

**What happened.** Improper control within EPG contracts. An unauthenticated remote attacker sends IPv4 or IPv6 packets using the UDP source and destination ports assigned to DHCP traffic through an affected device, and the configured EPG contracts are bypassed.

**Why it matters.** EPG contracts are the segmentation policy in an ACI fabric - the control that stops a compromised workload in one tenant or zone from reaching another. A bypass means your documented east-west segmentation is not actually being enforced for traffic shaped to look like DHCP, and nothing in your policy configuration will show it. This is a compliance and blast-radius problem rather than a device-takeover one, which is why it reads lower than the RCE advisories but still needs scheduling.

**Recommended actions**

- Upgrade the affected fabric switches to the fixed release; Cisco states there are no workarounds.
- Where DHCP relay is not needed between specific EPGs, filter the DHCP UDP ports explicitly at a firewall or external device rather than relying on contracts alone.
- Re-test your segmentation assumptions: run reachability checks between EPGs that should be isolated, using DHCP-port UDP traffic specifically, and record the result.
- Do not treat EPG contracts as the only boundary between differently-trusted tenants; place a inspecting firewall between zones whose separation is a compliance requirement.
- Record the finding against any audit control that cites EPG contracts as the enforcing mechanism.

**Legacy / unpatchable gear.** For fabric switches at end of support that will not receive the fixed release, enforce the boundary off-box: route inter-zone traffic through a supported firewall and deny DHCP-port UDP between zones that must stay isolated, rather than trusting in-fabric contracts.

**Detection.** Look for IPv4/IPv6 UDP flows using DHCP source and destination ports between endpoint groups that your contracts should forbid - NetFlow between EPG subnets is the quickest way to spot it. Any DHCP-port traffic that is not reaching an actual DHCP server or relay is worth investigating.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-aci-epgcbp-SfDU7NLf)**

#### 🟠 HIGH — Cisco Advance Notification for Publication of October 7, 2026, Security Advisories

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-notice-fBn58ELx) · 2026-10-07* `vendor-advisory`

> Cisco's advance notification for its 7 October 2026 bundle lists the NX-OS hardening release and the Nexus MPLS OAM RCE as Critical at CVSS 9.8, with CVE IDs attached.

**Affected:** The full set of advisories Cisco PSIRT published on 7 October 2026. Named in the notification: Cisco NX-OS Software Security Hardening Release October 2026 (CVE-2026-76453, CVE-2026-76455, CVE-2026-76456, CVE-2026-76457, CVE-2026-76458, CVE-2026-76459) rated Critical, CVSS 9.8; and Cisco Nexus 3000 and 9000 Series Switches MPLS OAM Remote Code Execution Vulnerability (CVE-2026-76465) rated Critical, CVSS 9.8. Read the notification itself for the complete table.  
**CVEs:** [CVE-2026-76453](https://nvd.nist.gov/vuln/detail/CVE-2026-76453), [CVE-2026-76455](https://nvd.nist.gov/vuln/detail/CVE-2026-76455), [CVE-2026-76456](https://nvd.nist.gov/vuln/detail/CVE-2026-76456), [CVE-2026-76457](https://nvd.nist.gov/vuln/detail/CVE-2026-76457), [CVE-2026-76458](https://nvd.nist.gov/vuln/detail/CVE-2026-76458), [CVE-2026-76459](https://nvd.nist.gov/vuln/detail/CVE-2026-76459), [CVE-2026-76465](https://nvd.nist.gov/vuln/detail/CVE-2026-76465)  

**What happened.** Cisco PSIRT published its advance notification listing the advisories released on 7 October 2026, with CVE IDs, security impact ratings and CVSS base scores, and recommends customers upgrade to the fixed software indicated in each advisory. The notification also points to Cisco's stated changes in PSIRT vulnerability disclosure, titled 'Strengthening the Foundation: A Predictable, Customer-Focused Response to AI-Accelerated Vulnerability Discovery'.

**Why it matters.** This is the index for the whole bundle and the only place in this cycle's sources where the CVE IDs and CVSS scores appear together, so it is what you plan the change window from. It also signals the new disclosure shape: large batched releases with findings grouped by weakness class, which means the number of CVEs no longer correlates with the number of devices you have to touch.

**Recommended actions**

- Read the notification's tables and build one change plan for the bundle rather than reacting to individual advisories as they are noticed.
- Prioritise the two Critical CVSS 9.8 entries - the NX-OS hardening release and the Nexus MPLS OAM RCE - ahead of the rest.
- Map each listed advisory to your asset inventory by platform and running version; do not filter by CVE ID alone, because grouped CVEs cover several distinct defects.
- Subscribe the network team to Cisco PSIRT advance notifications so the next bundle is known before the advisories land.
- Record a patched / mitigated / accepted-risk decision per device group for every advisory in the bundle.

**Legacy / unpatchable gear.** Use the notification to identify which of your platforms are no longer listed as receiving fixed software at all - that absence is the signal to start a replacement plan. In the meantime isolate those devices behind a supported inspecting firewall with strict management-plane ACLs and document the accepted risk.

**Detection.** Not applicable - this is a disclosure index, not a vulnerability. Use it to drive an inventory query: every Nexus, APIC, Meraki and License On-Prem instance with its exact running version, compared against the fixed releases in the linked advisories.

**AI angle.** Cisco explicitly frames its revised disclosure process as a response to AI-accelerated vulnerability discovery, and the CWE-grouped CVE assignment in this bundle is the practical consequence.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-notice-fBn58ELx)**

#### 🟡 MEDIUM — Cisco NX-OS Software Control Plane Denial of Service Vulnerability

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nxos-nscpdos-SnderkC7) · 2026-10-07* `vendor-advisory`

> Improper rate limiting in Cisco NX-OS lets an unauthenticated attacker destabilise routing and control-plane protocols with a flood of UDP or TCP connections to a data-plane interface.

**Affected:** Cisco NX-OS Software. Check the vendor advisory for the affected platforms and the fixed release.  

**What happened.** Rate limiting was improperly applied to some protocols. An unauthenticated remote attacker sends a high rate of UDP or TCP connections to a data-plane interface, exhausting system resources and causing packet loss and temporary disruption to routing and control-plane protocols. Cisco states the condition clears without manual intervention shortly after the traffic stops.

**Why it matters.** A denial of service rather than a compromise, and self-clearing, which is why it sits below the RCE advisories. It still matters because the trigger is ordinary-looking connection volume to a data-plane interface - something a compromised server inside your own network can produce - and the effect lands on routing adjacencies, so a single switch under load can ripple into reconvergence across the fabric.

**Recommended actions**

- Fold the upgrade to the fixed NX-OS release into the same change window as the other NX-OS advisories in this bundle.
- Configure and tune control-plane policing on affected platforms so control-plane protocol traffic is protected from data-plane connection floods.
- Apply ingress rate limiting or storm control on access ports facing user and server segments.
- Monitor routing adjacency flaps and control-plane CPU as first-class alerts, not just interface counters.
- Verify that routing protocol timers are not so aggressive that brief packet loss forces unnecessary reconvergence.

**Legacy / unpatchable gear.** Where no fixed release exists, control-plane policing and ingress rate limits on the devices in front of it are the mitigation. Keep unsupported switches out of the path of untrusted traffic entirely, and baseline their control-plane CPU so a flood is visible as a deviation.

**Detection.** Alert on NX-OS control-plane CPU spikes, control-plane policing drop counters, and routing protocol adjacency flaps correlated with high new-connection rates to data-plane interfaces. A burst of UDP or TCP connections to switch interface addresses from a single internal host is the signature to look for in NetFlow.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nxos-nscpdos-SnderkC7)**

#### 🟡 MEDIUM — Cisco NX-OS Software Python Sandbox Escape Vulnerability

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nxos-mppe-dhKZAFgb) · 2026-10-07* `vendor-advisory`

> A low-privileged authenticated user with Python execution rights on Cisco NX-OS can escape the Python sandbox and run commands on the switch's underlying operating system.

**Affected:** Cisco NX-OS Software Python interpreter. The attacker must be authenticated locally and hold Python execution privileges. Check the vendor advisory for the affected platforms and the fixed release.  

**What happened.** Insufficient validation of user-supplied input in the NX-OS Python interpreter. An authenticated local attacker with low privileges manipulates specific interpreter functions to escape the Python sandbox and execute arbitrary commands on the underlying operating system with the privileges of the authenticated user.

**Why it matters.** This is a privilege-boundary failure rather than a remote entry point, which is why it ranks below the unauthenticated RCEs - but the boundary it breaks is one people rely on. On-box Python is commonly granted to operators and automation accounts that are deliberately not given shell access, on the assumption the sandbox contains them. Where that assumption underpins your separation of duties or an audit control, it does not hold.

**Recommended actions**

- Fold the upgrade into the same NX-OS change window as the other advisories in this bundle.
- Audit which roles and accounts currently hold Python execution privileges on NX-OS and remove it from any that do not demonstrably need it.
- Stop treating the Python sandbox as a security boundary in role design - assume Python execution privilege is equivalent to shell access on the device until patched.
- Review on-box Python scripts (including EEM-invoked scripts) for ones nobody owns or recognises.
- Re-check any audit control or separation-of-duties claim that cites the Python sandbox as the enforcing mechanism.

**Legacy / unpatchable gear.** On NX-OS versions with no fix, revoke Python execution privileges entirely from non-administrative roles - that removes the precondition. Where automation depends on on-box Python, run it under a dedicated named account from an allowlisted jump host with full command logging, and treat that account as privileged.

**Detection.** Alert on NX-OS Python interpreter invocation by accounts that do not normally use it, and on shell commands or process launches on the underlying operating system attributed to non-administrative users. Log and review on-box script execution, and alert on new or modified scripts in the device's script storage.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nxos-mppe-dhKZAFgb)**

### Management platforms

#### 🟠 HIGH — Cisco Application Policy Infrastructure Controller Unauthorized File Access Vulnerability

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-apic-info-priv-enAdB5vD) · 2026-10-07* `vendor-advisory`

> A Cisco APIC export-policy flaw lets an authenticated admin read files from the controller's filesystem, including key material usable to gain root on APIC and on managed switches.

**Affected:** Cisco Application Policy Infrastructure Controller (APIC), in the export policies functionality. Exploitation requires valid administrative credentials. Check the vendor advisory for the affected and fixed releases.  

**What happened.** Insufficient access control to filesystem resources. An authenticated remote attacker with valid administrative credentials submits crafted values in specific UI fields and can read sensitive files from the underlying filesystem, including key material that can be used to elevate privileges to root on the APIC itself and on the switches it manages.

**Why it matters.** APIC is the ACI fabric controller, so this is a management-platform issue with fabric-wide reach: an administrator account that should have been bounded by APIC's own role model turns into root on every managed switch. It converts a lower-privilege or stolen admin credential, or an insider with legitimate access, into full fabric compromise. The admin-credential precondition is the only thing keeping this below the unauthenticated RCEs - and credential theft is routine.

**Recommended actions**

- Upgrade APIC to the fixed release; Cisco states there are no workarounds.
- Restrict APIC's management interface to the out-of-band management network and a dedicated jump host; it should not be reachable from user or server segments, let alone the internet.
- Enforce MFA on every APIC administrative account and remove shared local admin accounts in favour of named per-admin accounts via TACACS+/RADIUS.
- Audit who currently holds APIC administrative roles and reduce that set - the vulnerability's blast radius is defined by your admin count.
- After upgrading, rotate APIC certificates, admin credentials, API tokens and any key material stored on the controller, and re-issue fabric certificates - exposed key material stays valid until rotated.
- Review the APIC configuration and managed-switch configuration for unexpected changes, new accounts or modified export policies.

**Legacy / unpatchable gear.** An APIC version with no fixed release available cannot be safely administered over a shared network: move it to strictly out-of-band management, permit access only from one hardened jump host with full session recording, cut the administrative account list to the minimum, and plan the controller upgrade or replacement as a funded project.

**Detection.** Alert on APIC export-policy creation and modification events, and on administrative logins from sources outside the jump-host range. Review APIC audit logs for crafted or unusual values in export-policy UI fields, and for file-access activity by admin accounts that do not normally run exports.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-apic-info-priv-enAdB5vD)**

#### 🟠 HIGH — Cisco Application Policy Infrastructure Controller API Command Injection Vulnerability

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-apic-cmdinj-L6VR4E7) · 2026-10-07* `vendor-advisory`

> Crafted input to the Cisco APIC management API lets an authenticated administrator execute arbitrary commands as root on the ACI fabric controller's operating system.

**Affected:** Cisco Application Policy Infrastructure Controller (APIC) web-based management API. Exploitation requires valid administrative credentials. Check the vendor advisory for the affected and fixed releases.  

**What happened.** Insufficient input validation of user-controlled command arguments in the APIC management API. An attacker who authenticates to the API and sends crafted input can execute arbitrary commands on the underlying operating system with root privileges.

**Why it matters.** This breaks the boundary between APIC's application-level role model and the controller's operating system: any account that can reach the API with administrative credentials becomes root on the fabric controller. Combined with the export-policy file-access flaw in the same bundle, an attacker with one APIC admin credential has two independent routes to root on the controller and, from there, on the managed switches.

**Recommended actions**

- Upgrade APIC to the fixed release; Cisco states there are no workarounds.
- Restrict API access to the out-of-band management network and the specific automation hosts that use it - APIC's API should not be reachable from user or server segments.
- Enforce MFA on interactive APIC administrative accounts and use scoped service accounts, not shared admin credentials, for automation against the API.
- Reduce the number of accounts holding APIC administrative privilege; that count is the attack surface for this flaw.
- Rotate APIC admin credentials, API tokens and certificates after upgrading.
- Review the controller and managed-switch configurations for unexpected changes, and check the APIC filesystem for unexpected files or accounts if you have any reason to suspect pre-patch access.

**Legacy / unpatchable gear.** Where the deployed APIC version has no fix, treat every administrative API credential as equivalent to root on the fabric: issue them only to named individuals and scoped automation, allow API access only from an allowlisted management range, log every API call, and fund the controller upgrade.

**Detection.** Alert on APIC API authentication from sources outside the automation and jump-host allowlist. Review API audit logs for command arguments containing shell metacharacters or unexpected path components, and alert on any shell process or outbound connection originating from the APIC operating system.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-apic-cmdinj-L6VR4E7)**

#### 🟠 HIGH — Cisco License (Smart Software Manager) On-Prem Vulnerabilities

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-ssm-access-nttb2dhE) · 2026-10-07* `vendor-advisory`

> Cisco rates multiple Smart Software Manager On-Prem flaws Critical: remote attackers can gain unauthorised access, read sensitive data, cause denial of service or elevate privileges.

**Affected:** Cisco License On-Prem, formerly Cisco Smart Software Manager On-Prem (SSM On-Prem), in the web-based management interface and API endpoints. Security Impact Rating: Critical. Check the vendor advisory for the affected and fixed releases.  

**What happened.** Multiple vulnerabilities in the web management interface and API endpoints could allow a remote attacker to gain unauthorised access, access sensitive information, cause a denial of service, or elevate privileges. Software updates are available; there are no workarounds.

**Why it matters.** Critical-rated flaws in a server that most organisations deployed once and then forgot. License On-Prem is trusted by the network devices that register against it and typically holds credentials and tokens for them, so unauthorised access plus privilege escalation on this host gives an attacker a trusted foothold inside the management network. The advisory does not state that the interface is commonly internet-facing, but licensing servers are frequently reachable from far more of the internal network than they need to be.

**Recommended actions**

- Upgrade License On-Prem to the fixed release, together with the separate hardening release for the same product in this bundle.
- Restrict the web management interface and API to the management network and the specific devices that must register; verify nothing publishes it to the internet.
- Enforce MFA on administrative accounts and replace shared local admin accounts with named accounts.
- Rotate administrative credentials, API tokens and device registration tokens held on the server after upgrading.
- Review the server's accounts and configuration for unexpected administrative users or changes.
- Add the licensing server to the asset inventory and the regular patch rotation if it is not already there.

**Legacy / unpatchable gear.** If the deployed version has no fixed release, isolate the server onto a management-only segment reachable solely by registered devices and a jump host, front it with a supported inspecting firewall, log all access, and plan migration to a supported release with a budget owner and date.

**Detection.** Alert on authentication to the License On-Prem web interface and API from outside the management and registered-device ranges, on administrative account creation and privilege changes, and on unexpected outbound connections from the server. Ship its application and OS logs to the SIEM.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-ssm-access-nttb2dhE)**

#### 🟡 MEDIUM — Cisco License (Smart Software Manager) On-Prem Security Hardening Release: October 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-ssm-Ph77wdhf) · 2026-10-07* `vendor-advisory`

> Cisco released a hardening release for Cisco License On-Prem (formerly Smart Software Manager On-Prem) covering multiple internally found vulnerabilities, with no workarounds.

**Affected:** Cisco License On-Prem, formerly Cisco Smart Software Manager On-Prem (SSM On-Prem). Check the vendor advisory for the affected and fixed releases.  

**What happened.** A Cisco internal security review of License On-Prem produced hardening releases addressing multiple internally discovered vulnerabilities. Cisco states they were found during internal testing and are not known to be actively exploited, with issues grouped by CWE class and a single CVE ID per grouping. Software updates are available; there are no workarounds.

**Why it matters.** License On-Prem is an infrastructure management server that holds credentials for, and is trusted by, the network devices it licenses. It is rarely in anyone's patch rotation and is often left reachable from more of the network than it needs to be, which is what makes a licensing server worth attention disproportionate to its apparent function. See also the separate critical-rated License On-Prem advisory in this bundle.

**Recommended actions**

- Upgrade License On-Prem to the fixed release, and treat the separate critical-rated SSM On-Prem advisory in this bundle as the same change.
- Restrict network access to the License On-Prem server to the management network and the specific devices that must register against it.
- Enforce MFA on its administrative accounts and remove shared local admin accounts.
- Rotate the administrative credentials and any device registration tokens held on the server after upgrading.
- Confirm the server is in your asset inventory and patch rotation - licensing infrastructure is routinely missed.

**Legacy / unpatchable gear.** If the deployed version cannot be upgraded, move the server onto an isolated management segment reachable only by the devices that register against it, front it with a supported firewall, allow administration only from a jump host with session logging, and plan the migration to a supported release.

**Detection.** Ship License On-Prem application and OS logs to the SIEM and alert on administrative logins, account creation and configuration changes. Alert on connections to the server from any source outside the registered-device and management ranges.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-ssm-Ph77wdhf)**

#### 🟡 MEDIUM — Cisco Application Policy Infrastructure Controller Security Hardening Release: October 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-apic-UOXWtfh) · 2026-10-07* `vendor-advisory`

> Cisco shipped an APIC security hardening release for multiple vulnerabilities found in internal review of the ACI fabric controller; updates only, no workarounds.

**Affected:** Cisco Application Policy Infrastructure Controller (APIC). Check the vendor advisory for the affected and fixed releases.  

**What happened.** A Cisco internal security review of APIC produced hardening releases addressing multiple internally discovered vulnerabilities. Cisco states they were found during internal testing and are not known to be actively exploited, with findings grouped by CWE class and one CVE ID per grouping. Software updates are available; there are no workarounds.

**Why it matters.** This is the third APIC advisory in the same bundle, alongside the export-policy file access and the API command injection. Taken together they argue for upgrading the controller once, promptly, rather than evaluating each advisory separately - and for treating the fabric controller as a crown-jewel asset whose management exposure needs the same scrutiny as a perimeter device.

**Recommended actions**

- Upgrade APIC once to the fixed release that covers all three APIC advisories in this bundle rather than scheduling three changes.
- Confirm APIC management is reachable only from the out-of-band management network and a dedicated jump host.
- Enforce MFA on all APIC administrative accounts and cut the administrator list to the minimum.
- Take and verify a configuration backup and snapshot before the controller upgrade, and validate fabric policy after it.
- Review the APIC audit log for unexpected administrative activity since the last known-good review.

**Legacy / unpatchable gear.** An APIC release with no available fix should be administered strictly out of band, from one hardened jump host, with full session recording and a minimal administrator set, while the controller upgrade is funded and scheduled as a project.

**Detection.** Alert on APIC administrative logins outside the jump-host range, on role and account changes, and on fabric policy modifications that do not correspond to a change record.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-apic-UOXWtfh)**

### Other

#### 🟡 MEDIUM — Cisco Meraki Security Hardening Release: October 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-meraki-os-drbEX9GH) · 2026-10-08* `vendor-advisory`

> Cisco shipped a Meraki security hardening release for multiple vulnerabilities found in internal review, grouped by weakness class with one CVE per grouping and no workarounds.

**Affected:** Cisco Meraki, per the Meraki Security Hardening Release October 2026 advisory. The advisory does not enumerate affected product lines in the collected text - check the vendor advisory for the affected products and the fixed releases.  

**What happened.** A Cisco internal security review produced software hardening releases addressing multiple internally discovered vulnerabilities in Meraki. Cisco states these were found during internal testing and are not known to be actively exploited, and that issues have been grouped by Common Weakness Enumeration class with a single CVE ID per CWE grouping. Software updates are available; there are no workarounds.

**Why it matters.** Meraki is cloud-managed, so for most of the estate the upgrade is a firmware schedule decision in the dashboard rather than a per-device change window - which makes this cheap to act on. The catch is the disclosure format: one CVE covers a class of defects, so you cannot judge your exposure from the CVE alone and must read the advisory's product and version tables. Internally discovered and not known exploited is a reason to schedule rather than scramble.

**Recommended actions**

- Read the advisory's affected-product and fixed-release tables and compare against the firmware your Meraki networks are pinned to.
- Schedule the firmware upgrade through the Meraki dashboard's maintenance window per network, starting with internet-facing MX appliances.
- Confirm no Meraki network is held back on an old firmware branch for a configuration reason that nobody has revisited.
- Enforce MFA on Meraki dashboard administrator accounts and review the organisation's administrator list and API keys.
- Review the dashboard API keys in use and rotate any that are shared between people or systems.

**Legacy / unpatchable gear.** Meraki hardware past end of support will not receive the hardening firmware and, because the platform is cloud-managed, cannot be meaningfully isolated from its management plane. Identify end-of-support models in the dashboard inventory, place them behind a supported inspecting firewall where they terminate any untrusted traffic, and set a hardware replacement date with a budget owner.

**Detection.** Use the Meraki dashboard change log to alert on administrator additions, API key creation and configuration changes. Export the firmware version per network and treat any network still on a pre-fix branch after your scheduled window as an exception requiring sign-off.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-meraki-os-drbEX9GH)**

---

## Fortinet

*2 item(s) — 1 critical, 1 high*

### Firewalls

#### 🔴 CRITICAL — FBI: Ongoing FortiBleed attacks lock out FortiGate VPN admins

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/fbi-ongoing-fortibleed-attacks-lock-out-fortigate-vpn-admins/) · 2026-10-07* 

> The FBI warns FortiBleed attacks are still running against exposed Fortinet FortiGate firewalls and SSL VPN gateways, and are locking legitimate administrators out of the devices.

**Affected:** Internet-exposed Fortinet FortiGate firewalls and SSL VPN gateways. The report names no CVE and no fixed build - check the vendor advisory for the fixed release and confirm your running version against it.  

**What happened.** The FBI issued a warning that FortiBleed attacks are ongoing, targeting exposed FortiGate firewalls and SSL VPN gateways and locking out legitimate administrators. Reporting in this cycle describes attackers creating new accounts and deleting existing accounts and passwords to prevent legitimate access.

**Why it matters.** Confirmed, ongoing, government-flagged exploitation of internet-facing perimeter devices - the highest-urgency category this digest tracks. The destructive twist is what sets it apart: once administrators are locked out, the standard response playbook is unavailable. You cannot read the configuration, collect logs, enumerate added admin accounts or push a fixed image on a device you cannot authenticate to, and the device in question is the one terminating your remote access. Recovery falls back to console or out-of-band access and a rebuild from a known-good backup.

**Recommended actions**

- Verify today that you can still authenticate to every internet-facing FortiGate, and that console or out-of-band access works - do this before any other change.
- Export a current configuration backup off-device for each FortiGate and SSL VPN gateway and confirm the file is readable; this is the asset the attackers destroy.
- Remove management access (HTTPS/SSH/API) from every internet-facing interface and restrict administration to a dedicated management VLAN or out-of-band network.
- Audit local administrators, API users and admin profiles against your last known-good backup and investigate any account you did not create.
- Check the running version against Fortinet's advisory and upgrade to the vendor's fixed build - check the vendor advisory for the fixed release, as none is stated in these reports; patch the HA secondary first.
- Terminate all active SSL VPN sessions and revoke session tokens after upgrading; assume stolen session cookies.
- Rotate local admin credentials, API keys, certificates and VPN pre-shared keys, plus any service-account credentials stored on the appliance - a pre-patch compromise survives the upgrade.
- Enforce phishing-resistant MFA on administrative and VPN accounts and apply geo/ASN conditional access to the remote-access portal.
- Run Fortinet's integrity checking tooling where one is offered for your platform, and review the configuration for unexpected admin users, scripts, static routes or modified login pages.

**Legacy / unpatchable gear.** FortiGate models past end of support will not receive a fixed build and must not terminate internet-facing SSL VPN. Move remote access to a supported gateway now - this is an active campaign, not a theoretical risk. Until then, deny inbound access from untrusted zones at an upstream supported firewall, remove all internet-reachable management and VPN listeners, keep an offline configuration backup and a tested console recovery procedure, and set a replacement date with a named budget owner.

**Detection.** Alert with highest priority on FortiGate administrator account creation, administrator account deletion and administrator password changes - the reported lockout behaviour is exactly these events. Alert on successful admin logins from unexpected source addresses or ASNs, on SSL VPN authentication successes following bursts of failures, and on configuration changes with no matching change record. Because the device logs are what you lose first, confirm FortiGate syslog is shipping to the SIEM in real time rather than only being held locally.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/fbi-ongoing-fortibleed-attacks-lock-out-fortigate-vpn-admins/)**

#### 🟠 HIGH — FortiBleed Attackers Locking Victims Out of Fortinet Devices

*[SecurityWeek](https://www.securityweek.com/fortibleed-attackers-locking-victims-out-of-fortinet-devices/) · 2026-10-08* 

> FortiBleed attackers are creating new accounts and deleting existing accounts and passwords on Fortinet devices to keep legitimate administrators out.

**Affected:** Fortinet devices targeted in the FortiBleed campaign - see the FBI-warning item in this edition for the full action list. No CVE or fixed build is named in this report; check the vendor advisory for the fixed release.  

**What happened.** SecurityWeek reports that attackers in the FortiBleed campaign are creating new accounts and deleting existing accounts and passwords on victim Fortinet devices in order to prevent legitimate access.

**Why it matters.** Independent confirmation of the specific destructive behaviour in this edition's top story, and it pins down exactly what to detect: account creation, account deletion and password changes on the perimeter device. Those are ordinary administrative events that most SIEM configurations log but few alert on, which is how a lockout arrives as a surprise rather than as a page.

**Recommended actions**

- Follow the action list in this edition's FBI FortiBleed item - this report describes the same campaign.
- Create or raise SIEM alerts specifically for Fortinet administrator account creation, account deletion and password change events, at paging severity on internet-facing devices.
- Verify an off-device configuration backup exists and is readable for every Fortinet device, and test that you can restore from it.
- Confirm console or out-of-band access works on each device before you need it.
- Reconcile the current administrator list on every Fortinet device against your known-good baseline today.

**Legacy / unpatchable gear.** Unsupported Fortinet devices in an actively exploited campaign need compensating controls immediately: no internet-reachable management or VPN listeners, inbound access denied at a supported upstream firewall, an offline configuration backup, a tested console recovery procedure, and a replacement date with a named budget owner.

**Detection.** Alert at paging severity on Fortinet admin account creation, admin account deletion and admin password change. Reconcile the administrator list against a stored baseline on a schedule and alert on any difference. Because device-local logs can be destroyed along with the accounts, confirm real-time syslog forwarding to the SIEM is actually working on every Fortinet device rather than assumed.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/fortibleed-attackers-locking-victims-out-of-fortinet-devices/)**

---

## SonicWall

*2 item(s) — 1 critical, 1 medium*

### VPN gateways

#### 🔴 CRITICAL — SonicWall Patches CVSS 10.0 Pre-Authentication SSRF Flaw in SMA1000 Appliances

*[The Hacker News](https://thehackernews.com/2026/10/sonicwall-patches-cvss-100-pre.html) · 2026-10-07* 

> SonicWall hotfixed four SMA1000 flaws, the worst a pre-authentication SSRF rated CVSS 10.0 that lets an attacker without a login reach internal functions through the remote-access gateway.

**Affected:** SonicWall SMA1000 appliances - the gateways providing remote workers access to internal networks and applications. Four flaws are addressed by hotfixes; the most serious is a pre-authentication SSRF that SonicWall rates 10.0 on the CVSS scale. SonicWall says it has no evidence that any of the four is being exploited. Check the vendor advisory for the fixed hotfix build.  

**What happened.** SonicWall released hotfixes for four vulnerabilities in SMA1000 appliances. The most serious allows an attacker with no login to send requests through the appliance and reach internal functions, rated CVSS 10.0. SonicWall states it has no evidence of exploitation of any of the four.

**Why it matters.** A CVSS 10.0 pre-authentication flaw on an internet-facing remote-access gateway is the exact profile that gets added to CISA's KEV catalogue weeks after the patch ships - SMA appliances have a history of it. No evidence of exploitation today is a scheduling advantage, not a reprieve, and the window closes as soon as someone diffs the hotfix. SSRF through the gateway is particularly unpleasant on this class of device because the appliance is deliberately positioned to reach internal services that nothing on the internet should reach.

**Recommended actions**

- Apply the SonicWall hotfix to every SMA1000 appliance now, not at the next quarterly window - check the vendor advisory for the hotfix build; patch the secondary of any HA pair first.
- Until patched, restrict which internal services the SMA1000 can reach: the appliance should have an egress ACL to the specific application endpoints it proxies, not open internal access.
- Remove the appliance's administrative interface from internet exposure entirely; administer it from the management network only.
- After patching, terminate all active sessions and revoke session tokens; assume stolen session cookies.
- Rotate appliance certificates, local admin credentials, API tokens and any service-account credentials stored on the appliance.
- Enforce phishing-resistant MFA and geo/ASN conditional access on the remote-access portal.
- Hunt for prior compromise: unexpected outbound connections from the appliance, new local accounts, modified web resources and unusual timestamps in the appliance logs.

**Legacy / unpatchable gear.** SMA appliances outside the hotfix's supported versions have no fix path and should not be terminating internet-facing remote access. Migrate remote access to a supported gateway on a funded timeline; in the interim, restrict the portal to an allowlisted source-address or ASN set, apply a strict egress ACL from the appliance to internal services, enforce MFA, and log all sessions in full.

**Detection.** Alert on requests from the SMA1000 to internal services that are not on its proxy allowlist - that is the SSRF signature and the egress ACL gives you both the control and the log source. Review appliance logs for unauthenticated requests to internal paths, for new local accounts, and for modified web resources. Alert on admin logins from unexpected sources and on any outbound connection from the appliance to an unfamiliar destination.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/10/sonicwall-patches-cvss-100-pre.html)**

#### 🟡 MEDIUM — SonicWall and Splunk Patch Critical Vulnerabilities

*[SecurityWeek](https://www.securityweek.com/sonicwall-and-splunk-patch-critical-vulnerabilities/) · 2026-10-08* 

> SecurityWeek's round-up of the SonicWall and Splunk patches describes flaws allowing authentication bypass, arbitrary code execution and privilege escalation.

**Affected:** SonicWall (the SMA1000 hotfixes covered in detail in the dedicated item) and Splunk, which is not a network device. Check the vendor advisories for the affected and fixed releases.  

**What happened.** SecurityWeek reports that SonicWall and Splunk have patched critical and high-severity vulnerabilities which could allow attackers to bypass authentication, execute arbitrary code and elevate privileges.

**Why it matters.** Secondary coverage of the SonicWall SMA1000 hotfixes already covered in this edition, so the network-device action is the same and lives there. The Splunk half is out of scope for network gear, but worth forwarding: Splunk is where many teams' network device logs land, and losing the log platform costs you the detection capability the rest of this digest's advice depends on.

**Recommended actions**

- Treat the SonicWall action as the dedicated SMA1000 item in this edition - apply the hotfix and rotate credentials and session tokens.
- Forward the Splunk advisory to the SIEM or logging platform owner; it is out of scope for the network team but it holds your network device logs.
- Confirm after any Splunk maintenance that network device syslog ingestion resumed - a silent ingestion gap is a detection gap.

**Legacy / unpatchable gear.** See the dedicated SonicWall SMA1000 item for unsupported-appliance guidance. For an unsupported logging platform, keep a secondary copy of network device syslog outside it so a compromised or unpatched log platform does not cost you the evidence.

**Detection.** Alert on gaps in network device log ingestion, not just on device events - if a network device stops sending logs to the SIEM, that absence should page someone.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/sonicwall-and-splunk-patch-critical-vulnerabilities/)**

---

## TP-Link

*1 item(s) — 1 medium*

### Routers

#### 🟡 MEDIUM — TP-Link Faces State Lawsuits and New Scrutiny Over ISP Router Flaws

*[SecurityWeek](https://www.securityweek.com/tp-link-faces-state-lawsuits-and-new-scrutiny-over-isp-router-flaws/) · 2026-10-08* 

> SEC Consult published technical detail on TP-Link ISP router vulnerabilities named in a complaint filed by several US states, which TP-Link now faces in court.

**Affected:** TP-Link ISP-supplied routers, per the vulnerabilities described in the US state complaint and the SEC Consult technical write-up. The collected report does not state specific models, firmware versions or fixed releases - check the vendor advisory and the SEC Consult publication for the affected models and fixed firmware.  

**What happened.** Several US states filed a complaint against TP-Link referencing vulnerabilities in its ISP routers, and SEC Consult has now published technical details of the flaws mentioned in that complaint. The report describes legal action and public disclosure of technical detail; it does not state exploitation in the wild.

**Why it matters.** Published technical detail shortens the distance between disclosure and working exploit code for devices that sit at the edge of branch offices and home-working connections, are often ISP-managed rather than enterprise-managed, and are rarely in anyone's patch inventory. The litigation itself also matters for procurement: it is a supply-risk signal for estates standardising on ISP-supplied TP-Link CPE.

**Recommended actions**

- Inventory TP-Link CPE in branch, retail and home-worker locations, including ISP-managed units your network team does not administer - you cannot patch what is not on the list.
- Ask the ISP in writing which models are affected, what firmware is deployed, and when the fixed firmware will be pushed; ISP-managed CPE is patched on the ISP's schedule, not yours.
- Upgrade firmware to the current fixed release where you control the device, and verify image integrity against the vendor hash before loading.
- Disable WAN-side administration, UPnP, remote management and unused services (Telnet, TFTP, HTTP) on every unit you control.
- Replace default and reused credentials, and move to per-admin accounts with centralised logging where the platform supports it.
- Treat ISP-managed CPE as untrusted: terminate your own IPsec or ZTNA tunnel on a device you control behind it, so the CPE carries only encrypted transit.
- Check for persistence on reachable units: unexpected static routes, changed DNS servers, port forwards, tunnels and modified firmware.

**Legacy / unpatchable gear.** ISP-supplied and consumer TP-Link routers frequently reach end of firmware support while still in service, and some of the models in scope may never receive a fix. Treat them as untrusted transit: place a supported firewall behind the CPE, terminate all corporate traffic in an encrypted tunnel on that firewall, deny management access from the WAN and from the corporate side, and plan replacement with enterprise-managed CPE where the location matters.

**Detection.** Baseline outbound flows from branch locations with NetFlow and alert on new destinations, particularly DNS traffic going anywhere other than your resolvers - DNS redirection is a common CPE persistence mechanism. Periodically verify the CPE's firmware version, DNS configuration and port-forward list against an expected baseline, and alert on changes nobody requested.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/tp-link-faces-state-lawsuits-and-new-scrutiny-over-isp-router-flaws/)**

---

## 🛡️ Vendor AI defences introduced

AI capabilities the vendors shipped this cycle - the answer side of the AI story.

### Cisco

**Strengthening the Foundation: A Predictable, Customer-Focused Response to AI-Accelerated Vulnerability Discovery** — Cisco's documented change to how PSIRT discloses vulnerabilities, referenced from the advance notification for the 7 October 2026 advisories. In this bundle it takes the form of proactive internal security reviews published as 'security hardening releases', with internally discovered findings grouped by Common Weakness Enumeration class and a single CVE ID assigned per CWE grouping to streamline customer patching.

- *Defends against:* AI-accelerated vulnerability discovery - the shortening interval between a flaw existing in shipped network operating systems and someone else finding and weaponising it.
- *Availability:* described-only - referenced from Cisco's advance notification and visible in the four hardening releases published in this bundle; no product, launch or availability date is stated.
- *Worth evaluating if:* Teams that triage Cisco advisories by CVE count or scan-report CVE matching need to change method: grouped CVEs mean the CVE ID no longer identifies a single defect, so exposure has to be judged by platform and running version against the advisory's fixed-release table. Also relevant to anyone sizing change windows for Cisco gear, since batched bundle dates now drive the upgrade calendar.
- 📄 [Read the announcement at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-notice-fBn58ELx)

---

## 🧠 AI & network devices — Cisco reshapes its disclosure process around AI-accelerated vulnerability discovery - and the advisories now read differently

Cisco's advance notification for the 7 October 2026 bundle points readers to a document it titles "Strengthening the Foundation: A Predictable, Customer-Focused Response to AI-Accelerated Vulnerability Discovery" for the changes in how PSIRT now discloses. The effect is visible in this cycle's output: four of the advisories are "security hardening releases" from internal security review, and they state explicitly that findings are grouped by Common Weakness Enumeration class with a single CVE ID assigned per CWE grouping rather than per bug.

For a network engineer that changes the mechanics of triage. A single critical CVE now stands for a class of internally found defects, so the CVE ID alone no longer tells you which code path on your Nexus or Meraki device is affected - you have to read the advisory's vulnerable-products table rather than match CVEs against an inventory report. Batches are also larger, which means upgrade planning for a data-centre fabric is driven by the bundle date, not by individual advisories trickling in.

Separately in this cycle, Help Net Security reports that Anthropic's Claude Haiku 5.5 is measurably better at finding vulnerabilities and writing exploits than its predecessor, measured with its cybersecurity safeguards switched off. That item is about a model rather than network gear, but it is the other half of the same picture: both vendor-side discovery and attacker-side exploit development are getting faster at once, and the patch window for an internet-facing network device is set by whichever side moves quicker.

**Vendors with an AI angle in this edition:** Cisco

**Preventing it on current systems**

- Shorten the patch SLA for internet-facing network devices to days, not months - assume exploit development and mass scanning now outpace your historical change cadence.
- Stop triaging vendor bundles by CVE count: for grouped-by-CWE hardening releases, read the vulnerable-products and fixed-release tables and match on model plus running version instead.
- Treat any AI/LLM integration on network devices (assistants, AIOps, MCP or agent connectors) as a privileged admin path: scoped read-only credentials, no unattended configuration write, full audit trail.
- Test AI-facing management surfaces for prompt injection from attacker-controlled data - device logs, hostnames, interface descriptions, ticket text - before letting an agent act on them.
- Use AI-assisted detection on your own side: baseline NetFlow and syslog and alert on deviation, rather than relying on signature updates alone.
- Require callback verification for any out-of-band request to change a firewall or VPN configuration, to blunt AI-enabled social engineering of the network team.

**Preventing it on legacy / end-of-life systems**

- Gear that no longer receives hardening releases will never get these grouped fixes at all: isolate it behind a supported inspecting firewall and deny inbound access from untrusted zones.
- Apply compensating controls where no fix will arrive - strict management-plane ACLs, no direct administrative access, a dedicated jump host, full session logging.
- Monitor end-of-life devices as high-risk assets with a NetFlow baseline and alerting on new outbound flows, since faster exploit development hits unpatchable devices first.
- Set a replacement date and a budget owner for each unsupported device, and document the accepted risk until then.

---

<!--index
{
 "url": "/2026/10/08/network-security-digest/",
 "date": "2026-10-08",
 "entries": [
  {
   "title": "Cisco Nexus 3000 and 9000 Series Switches MPLS OAM Remote Code Execution Vulnerability",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-moam-rce-uBTzYV7",
   "source": "Cisco PSIRT",
   "relevance": "critical",
   "device_types": [
    "switch"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "An unauthenticated attacker can get root on Cisco Nexus 3000 and 9000 switches by sending one crafted MPLS echo-request; CVE-2026-76465, CVSS 9.8, no workaround.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco Nexus 3000 and 9000 Series Switches NGOAM Remote Code Execution Vulnerabilities",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-ngoam-rce-LWKQ4BU",
   "source": "Cisco PSIRT",
   "relevance": "critical",
   "device_types": [
    "switch"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Multiple NGOAM flaws in Cisco NX-OS let an unauthenticated attacker run code as root on Nexus 3000 and 9000 switches with crafted packets; no workaround exists.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco NX-OS Software NX-API Remote Code Execution Vulnerability",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-napi-rce-r2shwu2j",
   "source": "Cisco PSIRT",
   "relevance": "critical",
   "device_types": [
    "switch",
    "management-platform"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "A crafted HTTP request to the NX-API on Cisco NX-OS gives an unauthenticated attacker arbitrary code execution as root, or crashes the device into a reload.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco NX-OS Software Security Hardening Release: October 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-nxosw1-cWzSbtR",
   "source": "Cisco PSIRT",
   "relevance": "critical",
   "device_types": [
    "switch"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Cisco's NX-OS hardening release fixes internally found vulnerabilities rated Critical at CVSS 9.8 (CVE-2026-76453/76455/76456/76457/76458/76459), with no workarounds available.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "FBI: Ongoing FortiBleed attacks lock out FortiGate VPN admins",
   "link": "https://www.bleepingcomputer.com/news/security/fbi-ongoing-fortibleed-attacks-lock-out-fortigate-vpn-admins/",
   "source": "BleepingComputer",
   "relevance": "critical",
   "device_types": [
    "firewall",
    "vpn-gateway"
   ],
   "vendors": [
    "Fortinet"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "The FBI warns FortiBleed attacks are still running against exposed Fortinet FortiGate firewalls and SSL VPN gateways, and are locking legitimate administrators out of the devices.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "SonicWall Patches CVSS 10.0 Pre-Authentication SSRF Flaw in SMA1000 Appliances",
   "link": "https://thehackernews.com/2026/10/sonicwall-patches-cvss-100-pre.html",
   "source": "The Hacker News",
   "relevance": "critical",
   "device_types": [
    "vpn-gateway"
   ],
   "vendors": [
    "SonicWall"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "SonicWall hotfixed four SMA1000 flaws, the worst a pre-authentication SSRF rated CVSS 10.0 that lets an attacker without a login reach internal functions through the remote-access gateway.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco Nexus 9000 Series Fabric Switches in ACI Mode Endpoint Group Contract Bypass Vulnerability",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-aci-epgcbp-SfDU7NLf",
   "source": "Cisco PSIRT",
   "relevance": "high",
   "device_types": [
    "switch"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Crafted DHCP-port UDP traffic bypasses endpoint group contracts on Cisco Nexus 9000 fabric switches in ACI mode, defeating the segmentation policy the fabric enforces.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco Advance Notification for Publication of October 7, 2026, Security Advisories",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-notice-fBn58ELx",
   "source": "Cisco PSIRT",
   "relevance": "high",
   "device_types": [
    "switch",
    "management-platform"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [
    "CVE-2026-76453",
    "CVE-2026-76455",
    "CVE-2026-76456",
    "CVE-2026-76457",
    "CVE-2026-76458",
    "CVE-2026-76459",
    "CVE-2026-76465"
   ],
   "kev": false,
   "ai": true,
   "ai_angle": "Cisco explicitly frames its revised disclosure process as a response to AI-accelerated vulnerability discovery, and the CWE-grouped CVE assignment in this bundle is the practical consequence.",
   "summary": "Cisco's advance notification for its 7 October 2026 bundle lists the NX-OS hardening release and the Nexus MPLS OAM RCE as Critical at CVSS 9.8, with CVE IDs attached.",
   "ai_defense": false,
   "defense": {
    "solution": "Strengthening the Foundation: A Predictable, Customer-Focused Response to AI-Accelerated Vulnerability Discovery",
    "what_it_does": "Cisco's documented change to how PSIRT discloses vulnerabilities, referenced from the advance notification for the 7 October 2026 advisories. In this bundle it takes the form of proactive internal security reviews published as 'security hardening releases', with internally discovered findings grouped by Common Weakness Enumeration class and a single CVE ID assigned per CWE grouping to streamline customer patching.",
    "defends_against": "AI-accelerated vulnerability discovery - the shortening interval between a flaw existing in shipped network operating systems and someone else finding and weaponising it.",
    "availability": "described-only - referenced from Cisco's advance notification and visible in the four hardening releases published in this bundle; no product, launch or availability date is stated."
   }
  },
  {
   "title": "Cisco Application Policy Infrastructure Controller Unauthorized File Access Vulnerability",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-apic-info-priv-enAdB5vD",
   "source": "Cisco PSIRT",
   "relevance": "high",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "A Cisco APIC export-policy flaw lets an authenticated admin read files from the controller's filesystem, including key material usable to gain root on APIC and on managed switches.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco Application Policy Infrastructure Controller API Command Injection Vulnerability",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-apic-cmdinj-L6VR4E7",
   "source": "Cisco PSIRT",
   "relevance": "high",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Crafted input to the Cisco APIC management API lets an authenticated administrator execute arbitrary commands as root on the ACI fabric controller's operating system.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco License (Smart Software Manager) On-Prem Vulnerabilities",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-ssm-access-nttb2dhE",
   "source": "Cisco PSIRT",
   "relevance": "high",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Cisco rates multiple Smart Software Manager On-Prem flaws Critical: remote attackers can gain unauthorised access, read sensitive data, cause denial of service or elevate privileges.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "FortiBleed Attackers Locking Victims Out of Fortinet Devices",
   "link": "https://www.securityweek.com/fortibleed-attackers-locking-victims-out-of-fortinet-devices/",
   "source": "SecurityWeek",
   "relevance": "high",
   "device_types": [
    "firewall",
    "vpn-gateway"
   ],
   "vendors": [
    "Fortinet"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "FortiBleed attackers are creating new accounts and deleting existing accounts and passwords on Fortinet devices to keep legitimate administrators out.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco NX-OS Software Control Plane Denial of Service Vulnerability",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nxos-nscpdos-SnderkC7",
   "source": "Cisco PSIRT",
   "relevance": "medium",
   "device_types": [
    "switch"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Improper rate limiting in Cisco NX-OS lets an unauthenticated attacker destabilise routing and control-plane protocols with a flood of UDP or TCP connections to a data-plane interface.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco Meraki Security Hardening Release: October 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-meraki-os-drbEX9GH",
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
   "summary": "Cisco shipped a Meraki security hardening release for multiple vulnerabilities found in internal review, grouped by weakness class with one CVE per grouping and no workarounds.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco License (Smart Software Manager) On-Prem Security Hardening Release: October 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-ssm-Ph77wdhf",
   "source": "Cisco PSIRT",
   "relevance": "medium",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Cisco released a hardening release for Cisco License On-Prem (formerly Smart Software Manager On-Prem) covering multiple internally found vulnerabilities, with no workarounds.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco Application Policy Infrastructure Controller Security Hardening Release: October 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-hardening-apic-UOXWtfh",
   "source": "Cisco PSIRT",
   "relevance": "medium",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Cisco shipped an APIC security hardening release for multiple vulnerabilities found in internal review of the ACI fabric controller; updates only, no workarounds.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco NX-OS Software Python Sandbox Escape Vulnerability",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nxos-mppe-dhKZAFgb",
   "source": "Cisco PSIRT",
   "relevance": "medium",
   "device_types": [
    "switch"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "A low-privileged authenticated user with Python execution rights on Cisco NX-OS can escape the Python sandbox and run commands on the switch's underlying operating system.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "TP-Link Faces State Lawsuits and New Scrutiny Over ISP Router Flaws",
   "link": "https://www.securityweek.com/tp-link-faces-state-lawsuits-and-new-scrutiny-over-isp-router-flaws/",
   "source": "SecurityWeek",
   "relevance": "medium",
   "device_types": [
    "router"
   ],
   "vendors": [
    "TP-Link"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "SEC Consult published technical detail on TP-Link ISP router vulnerabilities named in a complaint filed by several US states, which TP-Link now faces in court.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "SonicWall and Splunk Patch Critical Vulnerabilities",
   "link": "https://www.securityweek.com/sonicwall-and-splunk-patch-critical-vulnerabilities/",
   "source": "SecurityWeek",
   "relevance": "medium",
   "device_types": [
    "vpn-gateway"
   ],
   "vendors": [
    "SonicWall"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "SecurityWeek's round-up of the SonicWall and Splunk patches describes flaws allowing authentication bypass, arbitrary code execution and privilege escalation.",
   "ai_defense": false,
   "defense": null
  }
 ]
}
-->

## How this was produced

- Feeds polled: 22 ok, 1 failed
- Raw items: 709 → in window: 73 → network-device relevant: 24 → published: 19
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
