---
layout: post
title: "Network Device Security Digest - 2026-11-23"
date: 2026-11-23 17:51:11 +0000
edition: daily
critical_count: 4
item_count: 9
kev: "CVE-2026-123890, CVE-2026-76466"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-11-23

*daily edition · window 24h · 23 feeds · 9 items · 4 critical · generated 2026-11-23 17:51 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — Cisco Expressway database-service flaw (CVE-2026-76466) exploited for root access, now KEV-listed

Cisco Expressway Series devices - Expressway-C and Expressway-E - carry a hard-coded credential / persistent cookie flaw in the database service feature tracked as CVE-2026-76466. An unauthenticated remote attacker with network access to the device and possession of persistent cookies can connect to the database service, invoke specific remote procedure call (RPC) commands, and execute arbitrary operating system commands as root. Cisco's advisory also notes insufficient validation of client certificates when TLS Verify Mode is set to Enabled for clustering, meaning client certificates are not validated.

This is no longer theoretical. Russian security company F6 reported attackers exploiting the flaw to breach organizations, and CISA added CVE-2026-76466 to the Known Exploited Vulnerabilities catalog on 2026-11-20 with a remediation due date of 2026-11-23. CISA has separately warned federal agencies of active in-the-wild exploitation. The same KEV batch added CVE-2026-123890, a VMware vCenter authentication bypass reported as exploited as a zero-day for months by a suspected China-nexus espionage group; that one is not a network device, but it is likely to compete for the same change window.

Expressway-E normally sits in the DMZ facing the internet to provide traversal and remote access for collaboration traffic, so the reachable attack surface is the part of the estate that is hardest to hide. Root on the underlying operating system means the device is fully attacker-controlled: configuration, certificates, cluster trust and any credentials stored on it. Because exploitation depends on possession of persistent cookies and on unvalidated clustering client certificates, an upgrade alone does not undo a pre-patch compromise - session material and device credentials have to be treated as burned.

Cisco has published a fixed release path (reporting describes the flaw as recently patched), but no fixed build number is stated in the collected sources: check the vendor advisory (cisco-sa-vcs-static-cred-hr7EWZ8t) for the fixed release that applies to your running train, and upgrade before today's KEV due date or record the deviation.

**Do this first**

- Inventory every Expressway-C and Expressway-E node, including clustered peers and lab or decommission-pending units, and record the running version against the vendor's fixed-release table; check the vendor advisory for the fixed release.
- Upgrade to the Cisco fixed release now - the KEV remediation due date is 2026-11-23. Upgrade cluster members in a controlled order and verify cluster health and config sync afterwards.
- Until the upgrade lands, restrict the database service and clustering ports so they are reachable only from the cluster peers' addresses, and remove any internet or untrusted-zone path to them with a firewall ACL in front of the DMZ node.
- Remove administrative access (HTTPS/SSH/API) from internet-facing interfaces; restrict management to a dedicated management VLAN or out-of-band network and enforce MFA on all admin accounts.
- After patching, rotate everything the device held: local admin credentials, API keys, device and clustering certificates, and any service-account or integration credentials configured on Expressway. Invalidate existing sessions and persistent cookies.
- Hunt for pre-patch compromise: unexpected root-level process or command execution, new or modified local accounts, unexpected outbound connections from the DMZ node, modified web resources, and RPC or database-service connections from addresses that are not cluster peers.
- Confirm TLS Verify Mode and clustering certificate validation settings are as the vendor advisory requires after the upgrade, and ship Expressway logs to the SIEM with alerts on admin login, config change and software change.

---

> **On the CISA KEV catalog in this edition:** CVE-2026-123890, CVE-2026-76466 — treat these as confirmed-exploited and patch on an emergency change.

## Executive summary

- Cisco Expressway CVE-2026-76466 is on CISA's KEV list (added 2026-11-20, due 2026-11-23) and confirmed exploited in the wild: unauthenticated root command execution on Expressway-C and Expressway-E via the database service. Patch today and rotate device credentials and certificates.
- GreyNoise found an active intrusion campaign against Check Point Security Management Servers, with confirmed activity between 17 October and 2 November 2026 affecting at least 22 organizations globally. A compromised management server reaches every gateway it manages - audit it before you audit the gateways.
- GL.iNet's GoodCloud remote management service was compromised and used to install malware on connected routers and KVM devices; users report backdoors and reverse shells. Device count is still unknown and the vendor is still investigating.
- Autonomous AI agents on the open-source Cairn framework went from zero-day discovery to root in 11 minutes in real intrusions. Nothing network-specific was targeted, but it sets the patch-window expectation for internet-facing edge devices.
- The same KEV batch added CVE-2026-123890, a VMware vCenter authentication bypass exploited as a zero-day for months. Not a network device, but it will compete for the same maintenance window this week.

---

## Items by vendor

## Cisco

*4 item(s) — 2 critical, 1 high, 1 medium*

### VPN gateways

#### 🔴 CRITICAL — Cisco Expressway vulnerability exploited to breach internal networks (CVE-2026-76466)

*[Help Net Security](https://www.helpnetsecurity.com/2026/11/23/cisco-expressway-vulnerability-exploited-cve-2026-76466/) · 2026-11-23* `KEV`

> CISA added CVE-2026-76466 to its KEV catalog after attackers used the Cisco Expressway database-service flaw to run commands as root and breach internal networks.

**Affected:** Cisco Expressway Series (Expressway-C and Expressway-E). No affected or fixed version numbers are stated in the source - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-123890](https://nvd.nist.gov/vuln/detail/CVE-2026-123890) **[KEV]**, [CVE-2026-76466](https://nvd.nist.gov/vuln/detail/CVE-2026-76466) **[KEV]**  

**What happened.** F6 reported attackers exploiting CVE-2026-76466 in Cisco Expressway gateways to breach organizations. CISA added the flaw to the Known Exploited Vulnerabilities catalog on 2026-11-20 with a due date of 2026-11-23. The flaw lets an unauthenticated remote attacker with network access and persistent cookies execute commands as root on a clustered device via the database service. The same KEV batch added CVE-2026-123890, a VMware vCenter authentication bypass that the source says has been exploited as a zero-day for months by a suspected China-nexus espionage group.

**Why it matters.** Expressway-E is normally deployed internet-facing in the DMZ, so the vulnerable surface is reachable without a foothold. Root on the operating system puts configuration, certificates, cluster trust and stored credentials in attacker hands, and the source says the access was used to reach internal networks - this is a perimeter-to-inside pivot, not a device-local bug.

**Recommended actions**

- Identify all Expressway-C and Expressway-E nodes from asset inventory, filtered by model and running version, and compare against the vendor's fixed-release table; check the vendor advisory for the fixed release.
- Apply the Cisco fixed release inside the change window today - the KEV remediation date is 2026-11-23 - or record the deviation with a compensating control.
- Restrict the database service and clustering ports to cluster peer addresses only with a management-plane ACL, and remove any untrusted-zone path to them.
- Rotate local admin credentials, API keys, clustering and device certificates, and integration service accounts after patching; invalidate existing sessions and persistent cookies.
- Export and review the configuration for unexpected admin users, modified login pages, scripts or static routes.
- Track CVE-2026-123890 (VMware vCenter) separately with the virtualisation team - same KEV batch, same due date, different owner.

**Legacy / unpatchable gear.** An Expressway node on an unsupported train will not receive this fix. Deny inbound access from untrusted zones, place it behind a supported inspecting firewall, allow the database and clustering services only from cluster peers, manage it from a dedicated jump host with full session logging, and set a replacement date with a named budget owner.

**Detection.** Alert on connections to the Expressway database service and clustering ports from any address that is not a cluster peer, on RPC invocations against the database service, and on root-level command execution or shell spawns on the appliance. Review web and admin logs for session reuse and for requests bearing persistent cookies from unexpected source ASNs. No vendor IOC list is published in the source.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/11/23/cisco-expressway-vulnerability-exploited-cve-2026-76466/)**

#### 🔴 CRITICAL — Cisco Expressway Series Remote Code Execution Vulnerability

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-vcs-static-cred-hr7EWZ8t) · 2026-11-23* `vendor-advisory`

> Cisco's advisory details unauthenticated root command execution in the Expressway database service, reachable with persistent cookies against clustered Expressway-C and Expressway-E.

**Affected:** Cisco Expressway Series - Expressway Core (Expressway-C) and Expressway Edge. The advisory excerpt states no fixed build; check the vendor advisory (cisco-sa-vcs-static-cred-hr7EWZ8t) for the fixed release.  

**What happened.** A vulnerability in the database service feature of Cisco Expressway Series allows an unauthenticated remote attacker with network access to execute arbitrary operating system commands as root. The cause is the use of persistent cookies, compounded by insufficient validation of client certificates when TLS Verify Mode is set to Enabled for clustering - client certificates are not validated. An attacker holding the cookies exploits it by connecting to the database service and invoking specific RPC commands.

**Why it matters.** This is the vendor's own account of the flaw CISA lists as exploited, and it names the preconditions that matter operationally: the clustering path and the database service, not the user-facing collaboration ports. Teams that only firewall the obvious services can still be reachable on the cluster path, and the broken client-certificate validation means clustering trust is not the barrier it appears to be.

**Recommended actions**

- Read the advisory and map your running train to the fixed release before scheduling; check the vendor advisory for the fixed release rather than assuming the latest build.
- Upgrade all cluster members and verify cluster state and configuration sync after the upgrade.
- Audit which interfaces expose the database service and clustering ports; bind them to a dedicated cluster or management network and ACL them to peer addresses only.
- Verify TLS Verify Mode and client-certificate validation behaviour after upgrading, per the advisory.
- Rotate clustering certificates and device credentials once patched; a stolen persistent cookie or unvalidated client certificate survives the upgrade otherwise.

**Legacy / unpatchable gear.** If the deployed Expressway version is past support and no fixed release applies, treat the cluster path as untrusted: isolate cluster members on a dedicated segment with no untrusted-zone reachability, deny inbound to the database service from everything except peers, and plan replacement.

**Detection.** Log and alert on RPC commands against the database service, on database-service connections whose client certificate is absent or unexpected, and on root command execution on the appliance. Compare current cluster peer lists against the expected inventory.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-vcs-static-cred-hr7EWZ8t)**

#### 🟠 HIGH — CISA flags critical Cisco RCE flaw as actively exploited in attacks

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/cisa-flags-critical-cisco-expressway-rce-flaw-as-actively-exploited/) · 2026-11-23* 

> CISA warned federal agencies that the critical Cisco Expressway remote code execution flaw is being exploited in the wild.

**Affected:** Cisco Expressway. The source describes the flaw as recently patched but states no version numbers - check the vendor advisory for the fixed release.  

**What happened.** CISA notified federal agencies that attackers are now exploiting a recently patched critical Cisco Expressway vulnerability in the wild. This is the government-directive side of the same CVE-2026-76466 activity reported by F6 and carried in the KEV catalog.

**Why it matters.** A KEV listing with a federal due date is the strongest available signal that exploitation is real and ongoing rather than researcher proof-of-concept. For non-federal operators it is still the cleanest justification for an out-of-cycle change window.

**Recommended actions**

- Treat the KEV due date of 2026-11-23 as the deadline for your own estate, not only for federal systems.
- Confirm the running Expressway version against the vendor's fixed-release table and schedule the upgrade immediately.
- Where the upgrade cannot land today, restrict reachability of the affected service to cluster peers and remove internet exposure of the management plane as the interim control.
- Record the decision (patched / mitigated / accepted risk) against the Expressway device group in the change record.

**Legacy / unpatchable gear.** Unsupported Expressway hardware or software trains get no fix from a KEV listing: isolate behind a supported firewall, strip all direct management access, and document the accepted risk with a replacement date.

**Detection.** Monitor for administrative logins and configuration changes on Expressway that do not match a change record, and for new outbound connections originating from the DMZ node.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/cisa-flags-critical-cisco-expressway-rce-flaw-as-actively-exploited/)**

#### 🟡 MEDIUM — SAP NetWeaver, Cisco Expressway Flaws Exploited in Attacks

*[SecurityWeek](https://www.securityweek.com/sap-netweaver-cisco-expressway-flaws-exploited-in-attacks/) · 2026-11-23* 

> A roundup confirms in-the-wild exploitation of the Cisco Expressway root command execution flaw, alongside an unrelated SAP NetWeaver bug.

**Affected:** Cisco Expressway. The SAP NetWeaver component in the same report is not network infrastructure and is out of scope here.  

**What happened.** SecurityWeek reports that flaws in SAP NetWeaver and Cisco Expressway are being exploited in attacks, and that threat actors could use them to leak or modify server information and to execute commands as root.

**Why it matters.** Third independent confirmation of the Expressway exploitation, which matters only for corroboration - the action is the same as for the KEV listing. The SAP NetWeaver half belongs to the application team.

**Recommended actions**

- No separate action beyond the Expressway upgrade and credential rotation already tracked; use this as corroboration when requesting the change window.
- Hand the SAP NetWeaver item to the application owners rather than the network team.

**Legacy / unpatchable gear.** As for the primary Expressway advisory: unsupported nodes get no fix and must be isolated behind a supported firewall with management access removed and a replacement date recorded.

**Detection.** No new indicators beyond those for the Expressway database-service flaw.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/sap-netweaver-cisco-expressway-flaws-exploited-in-attacks/)**

---

## GL.iNet

*2 item(s) — 1 critical, 1 high*

### Routers

#### 🔴 CRITICAL — GL.iNet cloud compromise hits routers and KVMs, older backdoors also found

*[Help Net Security](https://www.helpnetsecurity.com/2026/11/23/gl-inet-cloud-compromise-malware-backdoors/) · 2026-11-23* 

> Attackers compromised GL.iNet's GoodCloud remote management service and used it to install malware on connected routers and KVM devices.

**Affected:** GL.iNet routers and KVM (keyboard, video, mouse) devices connected to the GoodCloud cloud remote management service. The number of affected devices is not yet known and the vendor is still investigating. No fixed firmware version is stated - check the vendor advisory for the fixed release.  

**What happened.** Attackers compromised GoodCloud, GL.iNet's cloud remote management service, and used it to install malware on routers and KVM devices connected to it. The Chinese company, best known for its small travel routers, is still investigating. The incident prompted further research by RemoteToHome Consulting (RTH), a GL.iNet Services Partner, and by users, some of whom reported finding older backdoors as well.

**Why it matters.** The trust direction is inverted here: the malware arrived through the vendor's own management channel, so device-side hygiene and a WAN-side firewall would not have stopped it. These devices are common in home-office, travel and branch-of-one deployments, which means they frequently sit on a tunnel back to a corporate network while belonging to no asset inventory. The reports of older backdoors mean an unknown dwell time, so patching alone does not establish a clean state.

**Recommended actions**

- Locate every GL.iNet router or KVM in use, including user-purchased travel routers used for remote work, and determine whether each is enrolled in GoodCloud.
- Disconnect affected devices from the cloud management service and disable cloud/remote management until the vendor confirms the service is clean.
- Treat an enrolled device as compromised rather than suspect: factory reset, reflash current firmware verified against the vendor hash, then reconfigure from scratch instead of restoring a saved backup.
- Rotate every credential the device held or could observe: device admin password, Wi-Fi PSK, VPN and tunnel keys, and any corporate credential used from behind it.
- Terminate and re-key any site-to-site or client VPN tunnel that terminated on an affected device, and revoke its certificates.
- Disable WAN-side administration, UPnP and unused services (Telnet, TFTP, HTTP) on the rebuilt device; check for persistence in static routes, DNS server settings, port forwards and tunnels.
- Put these devices behind a segmented path with no route to management or server networks, and require remote workers to use the corporate VPN client rather than a router-terminated tunnel where possible.

**Legacy / unpatchable gear.** Older GL.iNet models that no longer receive firmware - and the ones where users found pre-existing backdoors - should be retired rather than cleaned, since a cloud-delivered implant on an unsupported device cannot be reliably removed or detected. Until replacement, isolate the device behind a supported inspecting firewall, deny it any route to management or server networks, and monitor it as a high-risk asset with a NetFlow baseline and alerting on new outbound flows.

**Detection.** Hunt NetFlow and firewall logs for outbound connections from GL.iNet device addresses to unfamiliar destinations, particularly long-lived sessions and reverse-shell patterns on non-standard ports. Check for unexpected processes, cron entries and listening sockets on the device, and for DNS or port-forward changes that no admin made. No vendor IOC list appears in the source.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/11/23/gl-inet-cloud-compromise-malware-backdoors/)**

#### 🟠 HIGH — Hackers exploit GL.iNet cloud flaw to infect routers, KVM devices

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/hackers-exploit-glinet-cloud-flaw-to-infect-routers-kvm-devices/) · 2026-11-22* 

> GL.iNet confirmed attackers abused a flaw in its cloud management service to install custom malware, with users finding backdoors and reverse shells on routers.

**Affected:** GL.iNet routers and KVM devices managed through the vendor's cloud remote management services. No version detail is given; check the vendor advisory for the fixed release.  

**What happened.** GL.iNet confirmed that hackers exploited a vulnerability in its cloud remote management services to install custom malware on customers' routers and KVM devices. Some affected users report backdoors and reverse shells granting remote access to infected devices.

**Why it matters.** Vendor confirmation moves this from user reports to a known incident, and the specific finding of reverse shells means interactive attacker access to devices sitting between a remote worker and the corporate network.

**Recommended actions**

- Follow the vendor's incident guidance as it is published, and do not re-enrol devices in cloud management until the service is confirmed clean.
- Reflash and reconfigure from scratch any device that was cloud-managed; a reverse shell implies the running configuration cannot be trusted.
- Replace default and reused credentials with per-admin accounts, and rotate any shared PSK that a compromised device knew.
- Block outbound traffic from these device segments to anything except the destinations they actually need, so a reverse shell has nowhere to call.

**Legacy / unpatchable gear.** Where the device is end-of-life and no fix will come, replace it; in the interim deny inbound access from untrusted zones, remove direct management access, and log all sessions through a dedicated jump host.

**Detection.** Alert on outbound sessions from router management IPs, on connections to high non-standard ports, and on any device beaconing at a regular interval - the reverse-shell pattern users reported.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/hackers-exploit-glinet-cloud-flaw-to-infect-routers-kvm-devices/)**

---

## Check Point

*1 item(s) — 1 critical*

### Management platforms

#### 🔴 CRITICAL — 99 IPs but a C2 ain’t one: An Active Intrusion Campaign Against Check Point Security Management Servers

*[GreyNoise Blog](https://www.greynoise.io/blog/check-point-security-management-servers-intrusion-campaign) · 2026-11-23* 

> GreyNoise found an intrusion campaign against Check Point Security Management Servers affecting at least 22 organizations globally.

**Affected:** Check Point Security Management Servers. The source names no CVE, no affected version and no fixed release - check the vendor advisory for the fixed release and for any vendor guidance on this campaign.  

**What happened.** GreyNoise reports discovering an intrusion campaign targeting Check Point Security Management Servers, with confirmed attack activity between 17 October and 2 November 2026, impacting at least 22 distinct organizations globally. The report describes command-and-control infrastructure across a set of IP addresses; the initial access vector is not stated in the collected summary.

**Why it matters.** A Security Management Server is the policy source of truth for every gateway it manages. An intruder there does not need a gateway vulnerability - they can push policy, read the rule base and object database, extract VPN configuration, and see the whole topology. Twenty-two confirmed organizations and a three-week confirmed window mean this was running quietly; the activity predates this digest window, so the relevant question is not whether to patch but whether you were already hit.

**Recommended actions**

- Read the GreyNoise report and block or hunt its published C2 addresses in firewall and proxy logs covering at least 17 October to 2 November 2026 - retrospectively, not just going forward.
- Treat this as an incident review on the management server, not a patch task: check for unexpected administrator accounts, unexplained policy installs, new or modified scripts, and SmartConsole or API logins outside change windows.
- Remove the management server's exposure entirely - management and API access reachable only from a dedicated management network or jump host, never from the internet or a user VLAN.
- Enforce MFA on all management server administrator accounts and remove shared local admin accounts.
- Rotate management server admin credentials, API keys and certificates, and review the rule base and object database against a known-good export for unauthorized changes.
- Verify the running version against the vendor's fixed-release table and apply any available vendor update or hardening guidance for the management server.
- Confirm which gateways trust this management server, and treat their policy as suspect until the rule base has been diffed against a trusted baseline.

**Legacy / unpatchable gear.** An unsupported management server cannot be left in place for this: a platform that pushes policy to production gateways and no longer receives fixes is the single highest-value unpatchable asset in the estate. Isolate it to an out-of-band management segment with no inbound access from any user or untrusted zone, allow administration only from a logged jump host, monitor it as a high-risk asset, and give the migration to a supported version a date and a budget owner.

**Detection.** Search firewall, proxy and NetFlow logs for the C2 addresses named in the GreyNoise report across the 17 October - 2 November 2026 window and after. Alert on management server administrative logins from unexpected sources, on policy installs with no matching change record, on API token creation, and on any outbound connection initiated by the management server itself.

📄 **[Read the full report at GreyNoise Blog →](https://www.greynoise.io/blog/check-point-security-management-servers-intrusion-campaign)**

---

## Unspecified

*2 item(s) — 2 watch*

### Other

#### ⚪ WATCH — Open-source AI pentesting framework Cairn keeps turning up in real-world attacks

*[Help Net Security](https://www.helpnetsecurity.com/2026/11/23/cairn-ai-agents-autonomous-attacks/) · 2026-11-23* `AI`

> Autonomous agents on the open-source Cairn framework found and exploited a zero-day with almost no human direction, per Amazon Threat Intelligence.

**Affected:** No network device is named. The targets were university servers and an open-source medical imaging AI application.  

**What happened.** Amazon Threat Intelligence reports that autonomous AI agents run by a Chinese-nexus threat actor spent nearly a full day failing to break into university servers, then found and exploited a zero-day in an open-source medical imaging AI application and gained a foothold. The agents ran on Cairn, an open-source AI penetration testing framework, with little human direction: the operator supplied a list of target IP addresses and the goal "obtain shell access".

**Why it matters.** Not a network-device story, so it is a watch item rather than an action item. It matters to this audience because of the economics: an unattended agent will spend a day of fruitless probing against an exposed service at negligible cost, which is precisely the profile of an internet-facing VPN or management interface that has been deferred for a quarter.

**Recommended actions**

- Shorten the patch window for internet-facing network devices to days, and keep an explicit list of every internet-reachable port on every edge device.
- Alert on sustained low-yield probing - hours of failed authentication, enumeration or malformed requests against one edge device - rather than only on known-bad signatures.
- Reduce the surface an unattended agent can work on: no management plane on the internet, legacy and unused services disabled on edge devices.
- If any AI assistant, AIOps or agent connector touches your network gear, scope it to read-only credentials with no unattended config write and a full audit trail.

**Legacy / unpatchable gear.** Unpatchable gear is the easiest target for a tireless agent: keep it off any untrusted-reachable path, behind a supported inspecting firewall, with NetFlow baselining and an alert on any new outbound flow.

**Detection.** Baseline per-source request volume on edge devices and alert on long-running enumeration from a single address or ASN, and on a successful authentication that follows a sustained failure burst from the same source.

**AI angle.** AI-assisted attack and autonomous vulnerability discovery: agents on an open-source AI pentesting framework performed the discovery and exploitation themselves with only a target list and a goal.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/11/23/cairn-ai-agents-autonomous-attacks/)**

#### ⚪ WATCH — AI Agents Zip From Flaw Discovery to Root Access in 11 Minutes

*[Dark Reading](https://www.darkreading.com/cyberattacks-data-breaches/ai-agents-flaw-discovery-root-access-11-minutes) · 2026-11-23* `AI`

> A China-nexus actor drove the Cairn pentest framework from flaw discovery to root access in 11 minutes against East Asian university targets.

**Affected:** No network device is named - the exploited application was a medical imaging app at East Asian universities.  

**What happened.** Dark Reading's account of the same Cairn campaign puts a clock on it: a China-nexus actor using the Cairn penetration testing tool against East Asian universities went from discovering a zero-day in a medical imaging app to root access in 11 minutes.

**Why it matters.** The 11-minute figure is the number to quote in a patch-SLA argument. It does not describe a network device, but it bounds how long an exposed and vulnerable edge service stays merely vulnerable rather than compromised.

**Recommended actions**

- Use the measured discovery-to-root interval when setting the emergency change threshold for internet-facing firewalls, VPN gateways and management platforms.
- Ensure detection and response for edge devices does not depend on a human noticing within hours - automate alerting on config change, new admin account and new outbound flow.
- Re-check that no network management platform is reachable from the internet, since these are the highest-value single targets for an automated run.

**Legacy / unpatchable gear.** For gear that cannot be patched, assume the window between exposure and compromise is minutes: the only durable control is denying reachability from untrusted zones, not faster response.

**Detection.** Alert in near real time on first-ever root or enable-level command execution and on new outbound connections from edge devices; retrospective daily review is too slow for this timescale.

**AI angle.** Same Cairn campaign as item 3, reported with the discovery-to-root timing: autonomous agent operation compressed exploitation to 11 minutes.

📄 **[Read the full report at Dark Reading →](https://www.darkreading.com/cyberattacks-data-breaches/ai-agents-flaw-discovery-root-access-11-minutes)**

---

## 🧠 AI & network devices — Autonomous agents on the Cairn framework: discovery to root in 11 minutes

Amazon Threat Intelligence attributes a set of real-world intrusions to autonomous AI agents run by a Chinese-nexus threat actor on Cairn, an open-source AI penetration testing framework. The operator supplied only a list of target IP addresses and a goal - "obtain shell access" - and gave the agents little further direction. The agents spent nearly a full day failing against university servers, then found and exploited a zero-day in an open-source medical imaging AI application and gained a foothold. Dark Reading reports the run from flaw discovery to root access took 11 minutes, in attacks against East Asian universities.

No network device was the target in this campaign, and nothing here says Cairn has been pointed at firewalls or VPN gateways. What it does establish is the cost curve: unattended agents will grind at an exposed service for hours at no analyst cost, and convert a find into root faster than a human team. For network teams the consequence is arithmetic, not novelty - the interval between a flaw existing on your internet-facing edge and someone standing on it is now set by machine patience, not by attacker staffing. That is the same window the Cisco Expressway KEV due date is already compressing.

**Vendors with an AI angle in this edition:** Unspecified

**Preventing it on current systems**

- Assume exploit development and mass scanning are faster than your historical patch SLA - shorten the window for internet-facing network devices to days, not months, and treat KEV due dates as the outer bound rather than the target.
- Cut the number of services an unattended agent can grind against: no management plane on the internet, no legacy or unused daemons on edge devices, and an explicit inventory of every internet-reachable port on every gateway.
- Alert on the behaviour rather than the signature - sustained failed authentication, enumeration and malformed requests against an edge device over hours is the pattern this campaign produced before it succeeded.
- Treat any AI/LLM integration on network devices (assistants, AIOps, MCP or agent connectors) as a privileged admin path: scoped read-only credentials, no unattended config write, full audit trail.
- Test AI-facing management surfaces for prompt injection from attacker-controlled data (device logs, hostnames, ticket text) before letting an agent act on them.
- Baseline NetFlow and syslog and alert on deviation, and add callback verification for any out-of-band request to change a firewall or VPN configuration.

**Preventing it on legacy / end-of-life systems**

- End-of-life gear is exactly what an unattended agent is best at: no fix will arrive, so isolate it behind a supported inspecting firewall and deny inbound access from untrusted zones.
- Put compensating controls in place - strict ACLs, no direct management access, a dedicated jump host and full session logging.
- Monitor legacy devices as high-risk assets: NetFlow baseline, alert on any new outbound flow, and treat a first-ever outbound connection as an incident.
- Set a replacement date and a budget owner, and document the accepted risk until the device is gone.

---

<!--index
{
 "url": "/2026/11/23/network-security-digest/",
 "date": "2026-11-23",
 "entries": [
  {
   "title": "Cisco Expressway vulnerability exploited to breach internal networks (CVE-2026-76466)",
   "link": "https://www.helpnetsecurity.com/2026/11/23/cisco-expressway-vulnerability-exploited-cve-2026-76466/",
   "source": "Help Net Security",
   "relevance": "critical",
   "device_types": [
    "vpn-gateway",
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [
    "CVE-2026-123890",
    "CVE-2026-76466"
   ],
   "kev": true,
   "ai": false,
   "ai_angle": "",
   "summary": "CISA added CVE-2026-76466 to its KEV catalog after attackers used the Cisco Expressway database-service flaw to run commands as root and breach internal networks.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco Expressway Series Remote Code Execution Vulnerability",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-vcs-static-cred-hr7EWZ8t",
   "source": "Cisco PSIRT",
   "relevance": "critical",
   "device_types": [
    "vpn-gateway",
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Cisco's advisory details unauthenticated root command execution in the Expressway database service, reachable with persistent cookies against clustered Expressway-C and Expressway-E.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "GL.iNet cloud compromise hits routers and KVMs, older backdoors also found",
   "link": "https://www.helpnetsecurity.com/2026/11/23/gl-inet-cloud-compromise-malware-backdoors/",
   "source": "Help Net Security",
   "relevance": "critical",
   "device_types": [
    "router",
    "management-platform"
   ],
   "vendors": [
    "GL.iNet"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Attackers compromised GL.iNet's GoodCloud remote management service and used it to install malware on connected routers and KVM devices.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "99 IPs but a C2 ain\u2019t one: An Active Intrusion Campaign Against Check Point Security Management Servers",
   "link": "https://www.greynoise.io/blog/check-point-security-management-servers-intrusion-campaign",
   "source": "GreyNoise Blog",
   "relevance": "critical",
   "device_types": [
    "management-platform"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "GreyNoise found an intrusion campaign against Check Point Security Management Servers affecting at least 22 organizations globally.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "CISA flags critical Cisco RCE flaw as actively exploited in attacks",
   "link": "https://www.bleepingcomputer.com/news/security/cisa-flags-critical-cisco-expressway-rce-flaw-as-actively-exploited/",
   "source": "BleepingComputer",
   "relevance": "high",
   "device_types": [
    "vpn-gateway",
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "CISA warned federal agencies that the critical Cisco Expressway remote code execution flaw is being exploited in the wild.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Hackers exploit GL.iNet cloud flaw to infect routers, KVM devices",
   "link": "https://www.bleepingcomputer.com/news/security/hackers-exploit-glinet-cloud-flaw-to-infect-routers-kvm-devices/",
   "source": "BleepingComputer",
   "relevance": "high",
   "device_types": [
    "router",
    "management-platform"
   ],
   "vendors": [
    "GL.iNet"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "GL.iNet confirmed attackers abused a flaw in its cloud management service to install custom malware, with users finding backdoors and reverse shells on routers.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "SAP NetWeaver, Cisco Expressway Flaws Exploited in Attacks",
   "link": "https://www.securityweek.com/sap-netweaver-cisco-expressway-flaws-exploited-in-attacks/",
   "source": "SecurityWeek",
   "relevance": "medium",
   "device_types": [
    "vpn-gateway",
    "other"
   ],
   "vendors": [
    "Cisco"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "A roundup confirms in-the-wild exploitation of the Cisco Expressway root command execution flaw, alongside an unrelated SAP NetWeaver bug.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Open-source AI pentesting framework Cairn keeps turning up in real-world attacks",
   "link": "https://www.helpnetsecurity.com/2026/11/23/cairn-ai-agents-autonomous-attacks/",
   "source": "Help Net Security",
   "relevance": "watch",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Unspecified"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "AI-assisted attack and autonomous vulnerability discovery: agents on an open-source AI pentesting framework performed the discovery and exploitation themselves with only a target list and a goal.",
   "summary": "Autonomous agents on the open-source Cairn framework found and exploited a zero-day with almost no human direction, per Amazon Threat Intelligence.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "AI Agents Zip From Flaw Discovery to Root Access in 11 Minutes",
   "link": "https://www.darkreading.com/cyberattacks-data-breaches/ai-agents-flaw-discovery-root-access-11-minutes",
   "source": "Dark Reading",
   "relevance": "watch",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Unspecified"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "Same Cairn campaign as item 3, reported with the discovery-to-root timing: autonomous agent operation compressed exploitation to 11 minutes.",
   "summary": "A China-nexus actor drove the Cairn pentest framework from flaw discovery to root access in 11 minutes against East Asian university targets.",
   "ai_defense": false,
   "defense": null
  }
 ]
}
-->

## How this was produced

- Feeds polled: 23 ok, 0 failed
- Raw items: 739 → in window: 42 → network-device relevant: 12 → published: 9
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
