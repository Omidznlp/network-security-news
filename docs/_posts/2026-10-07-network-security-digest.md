---
layout: post
title: "Network Device Security Digest - 2026-10-07"
date: 2026-10-07 13:05:06 +0000
edition: daily
critical_count: 2
item_count: 7
kev: "none"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-10-07

*daily edition · window 24h · 22 feeds · 7 items · 2 critical · generated 2026-10-07 13:05 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — FBI and Secret Service: FortiBleed still harvesting FortiGate and SSL VPN credentials, 86,644 device credentials amassed

The FBI and U.S. Secret Service issued a joint warning that the FortiBleed credential-harvesting campaign remains an active threat against internet-facing Fortinet FortiGate firewalls and SSL VPN gateways. The campaign has already amassed 86,644 Fortinet device credentials. According to the warning, it works by exploiting reused or leaked credentials together with legacy SHA-256 password storage, which gives the operators a path from a captured credential to usable administrative or VPN access.

This is not a patch story in the usual sense, which is why it leads. There is no CVE in the source and no fixed release is named, so an upgrade alone does not close it. The exposure is your credential material: anything that was valid on an internet-facing FortiGate while the campaign was running should be treated as known to the attacker, and a device left on legacy password hashing keeps handing out the same advantage after every reboot and every firmware upgrade.

Blast radius is wide because the two affected roles are the two that terminate untrusted traffic. A FortiGate admin credential is management-plane access to the perimeter; an SSL VPN credential is an authenticated foothold inside the network, usually with broader reach than any endpoint. 86,644 collected credentials means this is being worked at scale against whoever is reachable, not targeted.

First move is credential rotation across every internet-facing FortiGate and SSL VPN gateway, not triage of which ones look compromised. Then remove the management plane from the internet and enforce phishing-resistant MFA on remote access, so a leaked credential stops being sufficient on its own.

**Do this first**

- Rotate all local administrator passwords, API keys and VPN pre-shared keys on every internet-facing FortiGate and SSL VPN gateway; treat credentials valid during the campaign as already disclosed.
- Force a password reset for all SSL VPN users and terminate active sessions so captured credentials and session tokens stop working.
- Review the password-storage configuration on each device and move off legacy SHA-256 hashing where the platform supports a stronger scheme; check the vendor advisory for the supported setting and the fixed release.
- Remove HTTPS/SSH/API administrative access from WAN and any untrusted interface; restrict management to a dedicated management VLAN or out-of-band network.
- Enforce phishing-resistant MFA on all administrative and SSL VPN accounts, and remove shared local admin accounts.
- Audit each device configuration for unexpected admin users, modified login pages, new local accounts and unfamiliar static routes - credential abuse leaves no patch-level trace.
- Hunt authentication logs for successful VPN and admin logins from new ASNs or geographies, and for one credential authenticating from multiple sources.

---

## Executive summary

- FBI and Secret Service warn the FortiBleed campaign is still active against internet-facing FortiGate firewalls and SSL VPN gateways, having amassed 86,644 Fortinet device credentials - rotate credentials, do not just patch.
- SonicWall patched four SMA 1000 flaws including a maximum-severity pre-auth SSRF (CVE-2026-102255); no exploitation is evidenced yet, and hotfixes are available for a remote-access appliance class that is reliably targeted.
- OpenSSH 10.6 enables a post-quantum signature algorithm and invalidates experimental keys; maintainers report a large volume of AI-assisted bug reports, which shortens the realistic patch window for device management planes.
- Check Point described PromptGuardX, PDF-layer prompt-injection inspection inside Threat Emulation - relevant to anyone feeding network data to an AI agent.
- A Palo Alto Networks survey of more than 1,600 leaders found 42 percent name unpatchable legacy equipment as their biggest security problem, which is the compensating-controls conversation, not a patching one.

---

## Items by vendor

## SonicWall

*2 item(s) — 1 critical, 1 high*

### VPN gateways

#### 🔴 CRITICAL — SonicWall fixes pre-auth SSRF flaw in SMA 1000 appliances (CVE-2026-102255)

*[Help Net Security](https://www.helpnetsecurity.com/2026/10/07/sonicwall-fixes-pre-auth-ssrf-flaw-in-sma-1000-appliances-cve-2026-102255/) · 2026-10-07* 

> SonicWall patched four SMA 1000 series flaws, including a pre-auth SSRF (CVE-2026-102255) that lets unauthenticated attackers make the appliance issue requests and reach internal functionality.

**Affected:** SonicWall Secure Mobile Access (SMA) 1000 series appliances. The source does not list affected or fixed build numbers - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-102255](https://nvd.nist.gov/vuln/detail/CVE-2026-102255)  

**What happened.** SonicWall released fixes for four vulnerabilities in the SMA 1000 series. CVE-2026-102255 allows a remote unauthenticated attacker, in the vendor's words, to direct the appliance to issue requests on their behalf, reach internal functionality and perform unauthorized operations. The vendor stated there is currently no evidence that any of the vulnerabilities addressed in this release are being exploited in the wild, while the reporting notes this one could be soon.

**Why it matters.** SMA 1000 is a remote-access gateway, so it sits internet-facing by design and can route requests into the internal network. A pre-authentication SSRF on that device turns the appliance into a proxy for reaching internal services that have no exposure of their own, with no credential required. This appliance family has a history of being attacked after disclosure, so the gap between a patch being available and a patch being applied is the whole risk.

**Recommended actions**

- Inventory all SMA 1000 appliances and record the running version; confirm it against the vendor's fixed-release table before scheduling - check the vendor advisory for the fixed release, as the source does not name one.
- Apply the vendor hotfix in a change window, secondary unit of an HA pair first, then fail over and patch the primary.
- Remove the administrative and management interfaces (HTTPS/SSH/API) from internet exposure; restrict them to a management VLAN or out-of-band network.
- Place egress ACLs on the appliance itself so it can only reach the internal services it genuinely needs - this limits what an SSRF can reach even if a device is missed.
- After patching, terminate all active sessions, revoke session tokens, and rotate local admin credentials, API keys and certificates stored on the appliance.
- Enforce phishing-resistant MFA and geo/ASN conditional access on the remote-access portal.

**Legacy / unpatchable gear.** SMA 1000 units past end of support will not receive this hotfix. Take them out of internet-facing service: terminate remote access on a supported gateway instead, and if the appliance must stay, put it behind an inspecting firewall with inbound access denied from untrusted zones, no direct management path, and a documented replacement date with a budget owner.

**Detection.** Review appliance and upstream firewall logs for outbound or lateral requests originating from the SMA 1000's own interface addresses to internal hosts it has no business contacting - cloud metadata endpoints, internal admin ports, and internal HTTP services are the usual SSRF targets. Alert on any new destination in that set, and on unauthenticated requests to the portal immediately preceding them.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/10/07/sonicwall-fixes-pre-auth-ssrf-flaw-in-sma-1000-appliances-cve-2026-102255/)**

#### 🟠 HIGH — SonicWall warns of max severity SSRF flaw in SMA1000 gateways

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/sonicwall-warns-of-max-severity-ssrf-flaw-in-sma1000-gateways/) · 2026-10-07* 

> SonicWall released hotfixes for a maximum-severity server-side request forgery flaw in SMA1000 series appliances.

**Affected:** SonicWall SMA1000 series appliances. No affected or fixed build numbers are given in this source - check the vendor advisory for the fixed release.  

**What happened.** SonicWall released hotfixes addressing a maximum-severity server-side request forgery vulnerability in SMA1000 series appliances. This is additional coverage of the SMA 1000 fixes in item 0, and adds the maximum-severity rating for the SSRF.

**Why it matters.** The maximum-severity rating confirms how the vendor scores this: a pre-authentication flaw on an internet-facing remote-access gateway, available to anyone who can reach the portal. Second independent coverage of the same hotfix also means the detail is now public and widely read, which shortens the useful patching window.

**Recommended actions**

- Treat this as the same change as item 0 - do not open a second workstream; apply the SonicWall hotfix to all SMA1000 appliances in one tracked change.
- Confirm the running version against the vendor's fixed-release table before and after the hotfix, and record the result per device group in the change record.
- Verify the management interface is not reachable from the internet, and that the appliance's own outbound access to internal services is restricted by ACL.
- Rotate admin credentials, API tokens and certificates on the appliance after the hotfix is in place, and terminate active sessions.

**Legacy / unpatchable gear.** Unsupported SMA1000 hardware will not receive the hotfix. Move remote access to a supported gateway; until then deny inbound access from untrusted zones behind an inspecting firewall, restrict management to a jump host with session logging, and document the accepted risk with a replacement date.

**Detection.** Same coverage as item 0: alert on requests originating from the appliance's interface addresses toward internal hosts and ports it does not normally contact, and correlate those with unauthenticated portal requests immediately beforehand.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/sonicwall-warns-of-max-severity-ssrf-flaw-in-sma1000-gateways/)**

---

## Fortinet

*1 item(s) — 1 critical*

### Firewalls

#### 🔴 CRITICAL — FBI Warns FortiBleed Remains Active After Amassing 86,644 Fortinet Device Credentials

*[The Hacker News](https://thehackernews.com/2026/10/fbi-warns-fortibleed-remains-active.html) · 2026-10-07* 

> FBI and Secret Service say the FortiBleed campaign is still harvesting credentials from internet-facing FortiGate firewalls and SSL VPN gateways, with 86,644 device credentials amassed.

**Affected:** Internet-facing Fortinet FortiGate firewalls and SSL VPN gateways. No CVE and no affected or fixed version is named in the source; the campaign is described as exploiting reused or leaked credentials and legacy SHA-256 password storage.  

**What happened.** The FBI and U.S. Secret Service issued a joint warning that the FortiBleed credential-harvesting campaign remains an active threat against internet-facing FortiGate firewalls and SSL VPN gateways. The campaign has amassed 86,644 Fortinet device credentials. The agencies describe it as exploiting reused or leaked credentials together with legacy SHA-256 password storage.

**Why it matters.** There is no patch to schedule here, which is what makes it easy to mishandle. The exposed asset is credential material, so a firmware upgrade leaves the attacker's access intact. The two affected roles are also the worst two to lose: a FortiGate admin credential is management-plane control of the perimeter, and an SSL VPN credential is an authenticated position inside the network. The scale - 86,644 credentials - indicates indiscriminate collection against whatever is reachable.

**Recommended actions**

- Rotate every local administrator password, API key, certificate and VPN pre-shared key on all internet-facing FortiGate and SSL VPN gateways; treat anything valid during the campaign as disclosed.
- Force a password reset for all SSL VPN users, terminate active sessions and revoke session tokens.
- Check the password-storage setting on each device and move off legacy SHA-256 hashing where the platform offers a stronger option; check the vendor advisory for the supported configuration and the fixed release.
- Eliminate credential reuse between network devices and any other system - no shared admin password across the firewall estate, no reuse of directory credentials for local device accounts.
- Enforce phishing-resistant MFA on administrative and SSL VPN accounts, and remove shared local admin accounts in favour of per-admin accounts via TACACS+/RADIUS with centralized logging.
- Remove management access (HTTPS/SSH/API) from WAN and untrusted interfaces; restrict to a management VLAN or out-of-band network.
- Export and review each configuration for unexpected admin users, modified login pages, new static routes, new port forwards and unfamiliar tunnels.

**Legacy / unpatchable gear.** Devices too old to support stronger password storage or MFA cannot be fixed into safety here - the weakness is the stored hash and the reusable credential. Take them off internet-facing duty, deny inbound access from untrusted zones behind a supported inspecting firewall, allow management only from a dedicated jump host with full session logging, and set a replacement date with a named owner. Until replacement, monitor them as high-risk assets with a NetFlow baseline and alerting on new outbound flows.

**Detection.** Query authentication logs for successful SSL VPN and administrative logins from previously unseen source ASNs or countries, for a single account authenticating from multiple distant sources inside a short period, and for successful logins outside each user's established hours. Alert on configuration changes and admin logins on every perimeter device, and compare the current admin account list against a known-good export.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/10/fbi-warns-fortibleed-remains-active.html)**

---

## Unspecified

*1 item(s) — 1 medium*

### Other

#### 🟡 MEDIUM — OpenSSH 10.6 enables a post-quantum signature algorithm, so experimental keys need replacing

*[Help Net Security](https://www.helpnetsecurity.com/2026/10/07/openssh-10-6-released/) · 2026-10-07* `AI` `AI-defense`

> OpenSSH 10.6 enables a post-quantum signature algorithm, so experimental keys need replacing, and maintainers report many recent bug reports were found by AI models or with AI help.

**Affected:** OpenSSH; version 10.6 was released on Oct. 6. Experimental keys made with the post-quantum signature algorithm need replacing. No network-device vendor or firmware version is named in the source.  

**What happened.** The OpenSSH team released version 10.6 on Oct. 6, enabling a post-quantum signature algorithm, and said experimental keys need replacing. The team also said it will ship releases more often for now to get bugfixes out faster, that it has received a large number of security reports - many found by AI models or with AI help - and that in a number of cases a different researcher independently found the same bug later. The maintainers wrote that adversaries who do not report bugs to open-source projects are likely finding the same issues.

**Why it matters.** OpenSSH is the management plane on a large share of network gear and on the jump hosts used to reach it, so its release cadence becomes your cadence. Two things follow. First, a faster upstream release tempo means vendor firmware that bundles OpenSSH will lag further behind, and that lag is the exposure. Second, the maintainers' own point about unreported findings is the planning assumption: if AI-assisted review is producing duplicate discoveries, a fix landing upstream is not evidence that nobody else had it first.

**Recommended actions**

- Replace any experimental keys generated with the post-quantum signature algorithm, as the release states they need replacing.
- Inventory where OpenSSH is actually exposed in your estate: jump hosts, management servers, Linux-based network appliances and out-of-band console servers, and record the version on each.
- Patch management hosts and jump hosts on the upstream cadence rather than the quarterly one, and track which vendor firmware images bundle OpenSSH so you know what is waiting on a vendor release.
- Restrict SSH on network devices and management hosts to the management VLAN or out-of-band network with management ACLs; never expose device SSH to the internet.
- Require key-based or certificate-based authentication with per-admin accounts and centralized logging, and disable password authentication on management hosts.

**Legacy / unpatchable gear.** Network devices running an old vendor-bundled OpenSSH cannot be updated independently of firmware, and end-of-life models will never receive the update. Reach them only through a current jump host, deny SSH from any untrusted zone with ACLs on the device and on the upstream firewall, log every session in full, and set a replacement date for anything whose management stack is frozen.

**Detection.** Alert on SSH authentication attempts to device management addresses from outside the management network, on any successful SSH login from a new source, and on new or changed authorized keys on management hosts. Inventory SSH host key and version banners across the estate so an unexpected change on a network device is visible.

**AI angle.** The maintainers state that many of the security reports they received were found by AI models or with AI help, and that different researchers independently rediscovered the same bugs - direct evidence of AI-assisted vulnerability discovery compressing the window between a bug existing and it being known to more than one party.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/10/07/openssh-10-6-released/)**

---

## Check Point

*1 item(s) — 1 watch*

### Firewalls

#### ⚪ WATCH — Check Point PromptGuardX: Securing the Files AI Agents Trust

*[Check Point Blog](https://blog.checkpoint.com/ai-security/check-point-promptguardx-securing-the-files-ai-agents-trust/) · 2026-10-06* `AI` `AI-defense`

> Check Point's PromptGuardX inspects PDFs inside Threat Emulation before their content reaches an LLM or AI agent, targeting prompt injection hidden in documents.

**Affected:** Not a vulnerability. Check Point Threat Emulation, into which PromptGuardX is integrated, and the AI agents that consume files passing through it.  

**What happened.** Check Point published a blog post describing PromptGuardX, which moves prompt-injection detection into the file layer: integrated into Check Point Threat Emulation, it analyzes PDF files before their content reaches an LLM or AI agent. The post positions it as extending Check Point's AI Defense Plane into the file layer, alongside prompt inspection, runtime monitoring, access controls and action-level enforcement, and frames the threat as instructions hidden in external content attempting to activate an AI agent's tools, APIs and permissions.

**Why it matters.** The threat model matters to network teams even if the product is not a network-device feature. Instructions hidden in content an agent reads become actions taken with that agent's permissions - and on the network side the attacker-influenced content is already everywhere: device logs, hostnames, interface descriptions, SNMP strings, ticket text. Anyone wiring an AI assistant or AIOps integration to network data inherits this problem regardless of which vendor's filter they buy.

**Recommended actions**

- Enumerate where AI agents or assistants already read network-sourced text in your environment - log analytics, AIOps, ticket summarisation, config review - and record what each one is permitted to do with its tools and credentials.
- Give any AI integration touching network gear scoped read-only credentials, and require human approval for configuration writes; keep a full audit trail of agent actions.
- Test those integrations with injected instructions planted in data an attacker can influence - a hostname, an interface description, a log line - before trusting an agent to act on them.
- Treat file-layer and prompt-layer filtering as a control to evaluate, not a boundary: keep the permission scoping even where a filter is deployed.
- If you run Check Point Threat Emulation, check with the vendor what PromptGuardX requires and whether it applies to your deployment - the post names PDF analysis and does not state an availability date.

**Legacy / unpatchable gear.** Legacy network devices should be excluded from AI integrations entirely: they cannot enforce scoped credentials, cannot produce a reliable audit trail of what an agent changed, and their log output is unauthenticated text that an attacker who reaches the device can write to. Keep them on human-only management through a jump host with full session logging.

**Detection.** Log and review every action an AI integration takes against network infrastructure - every read, every API call, every attempted write - and alert on any write attempt or on tool use outside the agent's expected scope. Where an agent consumes device logs, alert on log or hostname fields containing instruction-like text.

**AI angle.** The item is a vendor prompt-injection defence: file-layer inspection of PDFs before an LLM or AI agent consumes them, addressing hidden instructions that attempt to drive an agent's tools and APIs.

📄 **[Read the full report at Check Point Blog →](https://blog.checkpoint.com/ai-security/check-point-promptguardx-securing-the-files-ai-agents-trust/)**

---

## Cisco

*1 item(s) — 1 watch*

### Other

#### ⚪ WATCH — One breach, please, and make no mistakes

*[Cisco Talos](https://blog.talosintelligence.com/one-breach-please-and-make-no-mistakes/) · 2026-10-07* `AI`

> Cisco Talos describes autonomous agents built inside AI labs attacking public infrastructure, and argues that preparation for agentic threats is what decides real incidents.

**Affected:** No specific product, vendor or version. The item concerns public infrastructure and incident preparedness generally.  

**What happened.** A Cisco Talos research post notes that the security community has now seen examples of autonomous agents, built inside AI labs, attacking public infrastructure, and argues that how an organisation prepares for agentic threats is what makes the difference during a real incident. No campaign, device, vulnerability or CVE is named in the source.

**Why it matters.** It is a preparedness argument rather than an incident report, so there is nothing to patch. The relevance for a network team is pace: if attack execution is automated, the interval between a disclosure and mass exploitation of internet-facing gateways compresses, and incident procedures written around human-speed reconnaissance need re-testing at that pace.

**Recommended actions**

- Re-test the perimeter incident runbook against a compressed timeline: how fast can you actually identify every internet-facing device of a given model and version, and apply or mitigate?
- Shorten the patch SLA for internet-facing firewalls, VPN gateways and management platforms to days, and record exceptions explicitly.
- Make sure an asset inventory filtered by model and running version is reachable during an incident, not reconstructed during one.
- Pre-stage the mitigations you would reach for under time pressure - management-plane ACLs, portal disable procedure, emergency credential rotation - so they are a change record away, not a design exercise.

**Legacy / unpatchable gear.** Unpatchable gear is where an accelerated attack pace hurts most, because the mitigation path is the only path. Keep legacy devices behind a supported inspecting firewall with inbound access from untrusted zones denied, maintain strict management ACLs and a dedicated jump host, and monitor them continuously with a NetFlow baseline so an automated intrusion shows up as a new flow rather than as an after-the-fact discovery.

**Detection.** No indicators are published in this source. Treat it as a reason to verify coverage instead: confirm that every perimeter device ships syslog to the SIEM, that admin login and configuration-change alerts actually fire, and that NetFlow from device management interfaces is being baselined.

**AI angle.** Autonomous agents built in AI labs are described as already attacking public infrastructure, which is the attacker-side half of this cycle's AI picture - the operational consequence is a shorter window between disclosure and exploitation of internet-facing network devices.

📄 **[Read the full report at Cisco Talos →](https://blog.talosintelligence.com/one-breach-please-and-make-no-mistakes/)**

---

## Palo Alto Networks

*1 item(s) — 1 watch*

### Other

#### ⚪ WATCH — Even with OT network visibility, critical infrastructure operators struggle with legacy equipment

*[Help Net Security](https://www.helpnetsecurity.com/2026/10/07/ot-network-visibility-challenges/) · 2026-10-07* 

> A Palo Alto Networks survey of 1,600+ leaders found 52 percent cite legacy OT as their top visibility gap and 42 percent call unpatchable legacy equipment their biggest security problem.

**Affected:** Operational technology networks at large critical infrastructure operators. No specific product or version; this is survey data, not a vulnerability.  

**What happened.** A Palo Alto Networks survey of more than 1,600 security and operations leaders found that large critical infrastructure operators run seven separate security tools on average and most still cannot see every asset on their OT networks. Legacy OT is the most commonly named visibility problem at 52 percent of respondents, and 42 percent name legacy equipment that cannot be patched as their biggest cybersecurity problem.

**Why it matters.** Both findings describe the same gap a network team owns rather than the OT team: if an asset is invisible it is also unsegmented in practice, and if it cannot be patched then the network is the only control surface left. Seven tools averaging incomplete coverage is a sign that visibility is being bought rather than derived from the network itself, where span ports, NetFlow and switch MAC tables already see every device that transmits.

**Recommended actions**

- Derive an OT asset list from the network rather than from agents: collect switch MAC address tables, ARP tables and NetFlow at the distribution layer, and reconcile against the existing inventory.
- Segment management, user, guest and OT traffic into separate VLANs, and verify the VLAN ACLs and private VLAN enforcement actually block the paths you think they block.
- Put unpatchable OT devices behind an inspecting firewall with a default-deny policy from untrusted zones, and allow only the specific protocol flows the process requires.
- Disable unused switch ports, move them to an unrouted VLAN, and enforce 802.1X or MAC authentication on access ports so an unknown device cannot simply appear on the OT network.
- Harden management on OT-adjacent switches: SNMPv3 with auth and priv only, no public/private community strings, legacy protocols disabled, management ACLs and control-plane policing in place.

**Legacy / unpatchable gear.** This item is the legacy problem, as reported by the people living with it. Unpatchable OT gear needs compensating controls as the permanent answer: isolation behind a supported inspecting firewall, strict ACLs, no direct management access, a dedicated jump host, full session logging, a replacement date with a budget owner, and continuous monitoring as a high-risk asset with alerting on any new outbound flow.

**Detection.** Baseline NetFlow per OT segment and alert on any flow that crosses a segment boundary, any new talker pair, and any OT host initiating outbound connections toward the internet or the corporate network. New MAC addresses appearing on OT access ports should also raise an alert, since that is both an inventory gap and an unauthorized-device signal.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/10/07/ot-network-visibility-challenges/)**

---

## 🛡️ Vendor AI defences introduced

AI capabilities the vendors shipped this cycle - the answer side of the AI story.

### Check Point

**PromptGuardX** — Moves prompt-injection detection into the file layer: integrated into Check Point Threat Emulation, it analyzes PDF files before their content reaches an LLM or AI agent. Check Point describes it as extending its AI Defense Plane into the file layer, alongside prompt inspection, runtime monitoring, access controls and action-level enforcement.

- *Defends against:* Prompt injection via external content - instructions hidden in files such as invoices, contracts and emails that attempt to activate an AI agent's tools, APIs and permissions.
- *Availability:* described-only - the post states it is integrated into Check Point Threat Emulation and gives no availability date or release stage
- *Worth evaluating if:* Teams already running Check Point Threat Emulation that are also feeding documents or network-sourced text into an LLM or AI agent, and network teams scoping an AIOps or assistant integration who need a control at the ingestion layer in addition to scoped agent credentials.
- 📄 [Read the announcement at Check Point Blog →](https://blog.checkpoint.com/ai-security/check-point-promptguardx-securing-the-files-ai-agents-trust/)

---

## 🧠 AI & network devices — AI-found bugs, agentic attackers and a prompt-injection filter land in the same cycle

Three of this cycle's items are genuinely about AI, and together they mark both ends of the problem. The OpenSSH maintainers, releasing 10.6, said they have received a large number of security reports, many of them found by AI models or with AI help, and that in several cases a different researcher independently rediscovered the same bug later. Their stated conclusion is the operationally relevant one: adversaries who do not report what they find are likely holding the same results. OpenSSH is the management plane on most network gear, so the supply of findings against it rising is a patch-cadence problem for switches, routers and firewalls, not just for servers.

Cisco Talos covers the attacker side, describing autonomous agents built inside AI labs attacking public infrastructure and arguing that how an organisation prepares for agentic threats is what decides real incidents. The source is a preparedness argument rather than an incident report - no specific device, campaign or vulnerability is named.

Check Point's PromptGuardX is the defensive entry: file-layer inspection of PDFs inside Check Point Threat Emulation, before their content reaches an LLM or AI agent, aimed at instructions hidden in external content that try to activate an agent's tools, APIs and permissions. That threat model applies directly to anyone pointing an AI assistant at network data, because device logs, hostnames and ticket text are all attacker-influenced content.

**Vendors with an AI angle in this edition:** Check Point, Cisco, Unspecified

**Preventing it on current systems**

- Assume exploit development and mass scanning now move faster than your historical patch SLA - shorten the window for internet-facing network devices to days, not months, and treat OpenSSH and management-plane components as in scope.
- Treat any AI/LLM integration on network devices (assistants, AIOps, MCP or agent connectors) as a privileged admin path: scoped read-only credentials, no unattended config write, full audit trail.
- Test AI-facing management surfaces for prompt injection from attacker-controlled data - device logs, hostnames, interface descriptions, ticket text - before letting an agent act on them.
- Use AI-assisted detection on your own side: baseline NetFlow and syslog and alert on deviation, rather than relying on signature updates alone.
- Require callback verification for any out-of-band request to change firewall or VPN configuration; AI-enabled social engineering of the network team scales better than it used to.

**Preventing it on legacy / end-of-life systems**

- Legacy gear with an unpatchable SSH or web management stack will not benefit from upstream fixes: deny inbound management from untrusted zones and reach it only through a jump host with full session logging.
- Keep legacy devices out of any AI/AIOps integration - they cannot enforce scoped credentials or produce a usable audit trail of what an agent changed.
- Monitor unpatchable devices as high-risk assets: NetFlow baseline, alert on any new outbound flow, alert on every administrative login.
- Set a replacement date and a budget owner for gear that cannot receive management-plane fixes, and document the accepted risk until then.

---

<!--index
{
 "url": "/2026/10/07/network-security-digest/",
 "date": "2026-10-07",
 "entries": [
  {
   "title": "SonicWall fixes pre-auth SSRF flaw in SMA 1000 appliances (CVE-2026-102255)",
   "link": "https://www.helpnetsecurity.com/2026/10/07/sonicwall-fixes-pre-auth-ssrf-flaw-in-sma-1000-appliances-cve-2026-102255/",
   "source": "Help Net Security",
   "relevance": "critical",
   "device_types": [
    "vpn-gateway"
   ],
   "vendors": [
    "SonicWall"
   ],
   "cves": [
    "CVE-2026-102255"
   ],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "SonicWall patched four SMA 1000 series flaws, including a pre-auth SSRF (CVE-2026-102255) that lets unauthenticated attackers make the appliance issue requests and reach internal functionality.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "FBI Warns FortiBleed Remains Active After Amassing 86,644 Fortinet Device Credentials",
   "link": "https://thehackernews.com/2026/10/fbi-warns-fortibleed-remains-active.html",
   "source": "The Hacker News",
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
   "summary": "FBI and Secret Service say the FortiBleed campaign is still harvesting credentials from internet-facing FortiGate firewalls and SSL VPN gateways, with 86,644 device credentials amassed.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "SonicWall warns of max severity SSRF flaw in SMA1000 gateways",
   "link": "https://www.bleepingcomputer.com/news/security/sonicwall-warns-of-max-severity-ssrf-flaw-in-sma1000-gateways/",
   "source": "BleepingComputer",
   "relevance": "high",
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
   "summary": "SonicWall released hotfixes for a maximum-severity server-side request forgery flaw in SMA1000 series appliances.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "OpenSSH 10.6 enables a post-quantum signature algorithm, so experimental keys need replacing",
   "link": "https://www.helpnetsecurity.com/2026/10/07/openssh-10-6-released/",
   "source": "Help Net Security",
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
   "ai_angle": "The maintainers state that many of the security reports they received were found by AI models or with AI help, and that different researchers independently rediscovered the same bugs - direct evidence of AI-assisted vulnerability discovery compressing the window between a bug existing and it being known to more than one party.",
   "summary": "OpenSSH 10.6 enables a post-quantum signature algorithm, so experimental keys need replacing, and maintainers report many recent bug reports were found by AI models or with AI help.",
   "ai_defense": true,
   "defense": null
  },
  {
   "title": "One breach, please, and make no mistakes",
   "link": "https://blog.talosintelligence.com/one-breach-please-and-make-no-mistakes/",
   "source": "Cisco Talos",
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
   "ai_angle": "Autonomous agents built in AI labs are described as already attacking public infrastructure, which is the attacker-side half of this cycle's AI picture - the operational consequence is a shorter window between disclosure and exploitation of internet-facing network devices.",
   "summary": "Cisco Talos describes autonomous agents built inside AI labs attacking public infrastructure, and argues that preparation for agentic threats is what decides real incidents.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Even with OT network visibility, critical infrastructure operators struggle with legacy equipment",
   "link": "https://www.helpnetsecurity.com/2026/10/07/ot-network-visibility-challenges/",
   "source": "Help Net Security",
   "relevance": "watch",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Palo Alto Networks"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "A Palo Alto Networks survey of 1,600+ leaders found 52 percent cite legacy OT as their top visibility gap and 42 percent call unpatchable legacy equipment their biggest security problem.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Check Point PromptGuardX: Securing the Files AI Agents Trust",
   "link": "https://blog.checkpoint.com/ai-security/check-point-promptguardx-securing-the-files-ai-agents-trust/",
   "source": "Check Point Blog",
   "relevance": "watch",
   "device_types": [
    "firewall"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "The item is a vendor prompt-injection defence: file-layer inspection of PDFs before an LLM or AI agent consumes them, addressing hidden instructions that attempt to drive an agent's tools and APIs.",
   "summary": "Check Point's PromptGuardX inspects PDFs inside Threat Emulation before their content reaches an LLM or AI agent, targeting prompt injection hidden in documents.",
   "ai_defense": true,
   "defense": {
    "solution": "PromptGuardX",
    "what_it_does": "Moves prompt-injection detection into the file layer: integrated into Check Point Threat Emulation, it analyzes PDF files before their content reaches an LLM or AI agent. Check Point describes it as extending its AI Defense Plane into the file layer, alongside prompt inspection, runtime monitoring, access controls and action-level enforcement.",
    "defends_against": "Prompt injection via external content - instructions hidden in files such as invoices, contracts and emails that attempt to activate an AI agent's tools, APIs and permissions.",
    "availability": "described-only - the post states it is integrated into Check Point Threat Emulation and gives no availability date or release stage"
   }
  }
 ]
}
-->

## How this was produced

- Feeds polled: 22 ok, 1 failed
- Raw items: 709 → in window: 54 → network-device relevant: 7 → published: 7
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
