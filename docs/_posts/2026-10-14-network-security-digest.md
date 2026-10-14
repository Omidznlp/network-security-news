---
layout: post
title: "Network Device Security Digest - 2026-10-14"
date: 2026-10-14 11:36:27 +0000
edition: daily
critical_count: 5
item_count: 9
kev: "CVE-2026-104935"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-10-14

*daily edition · window 24h · 22 feeds · 9 items · 5 critical · generated 2026-10-14 11:36 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — Ivanti Connect Secure: three chained zero-days, root on fully patched appliances, CVE-2026-104935 in KEV with a 16 October deadline

Ivanti published an advisory on 13 October 2026 for CVE-2026-104935, an authentication bypass by spoofing (CWE-290) in Ivanti Connect Secure carrying a CVSSv3.0 score of 8.6 and allowing a remote, unauthenticated attacker to execute arbitrary commands. Ivanti states that exploitation in the wild occurred before disclosure, affecting a limited number of customers - this was a zero-day. CISA confirmed exploitation and added the CVE to the Known Exploited Vulnerabilities catalog the same day, with a remediation due date of 16 October 2026 and ransomware use listed as unknown.

Volexity, which reported the flaws, says the activity it found in the first week of October 2026 was an exploit chain of three vulnerabilities used by an unknown actor tracked as UTA0576, and that the compromised appliance was fully patched at the time, running version 22.8R2.4. That detail is the one that should drive your response: being current on patches before 13 October did not keep an appliance out of scope. The actor collected credentials, exfiltrated configuration data and performed network reconnaissance, which is the standard pre-lateral-movement pattern against a remote access gateway.

Ivanti has released fixes for two of the three flaws and told customers on affected versions to act immediately; per Volexity a fix for the third is still pending. Treat that as an incomplete remediation: patching closes the disclosed path but does not by itself establish that your appliance was not already used. Because the appliance terminates VPN sessions and holds credentials, certificates and the full configuration, a successful exploit converts directly into authenticated access to the internal network, and an upgrade alone does not evict an attacker who already took those secrets.

The source material does not state a fixed build number - check the Ivanti advisory for the fixed release before scheduling the change, and do not infer a version from the 22.8R2.4 figure, which is the version that was compromised.

**Do this first**

- Inventory every Ivanti Connect Secure appliance with its exact running version today; check the vendor advisory for the fixed release and apply it in an emergency window, not the next routine cycle.
- Run the Ivanti Integrity Checker Tool and preserve its output before and after the upgrade; a clean post-upgrade result does not clear a pre-upgrade compromise.
- Assume credential and configuration theft: terminate all active VPN sessions, revoke session tokens, and rotate local admin credentials, service accounts, certificates and any secrets stored on or reachable from the appliance.
- Treat the third, still-unfixed flaw as live exposure - restrict the administrative interface to a management network or jump host, and apply geo/ASN conditional access on the user-facing portal.
- Hunt now rather than after the change window: review /var logs for unusual timestamps, look for new or modified local accounts, altered web resources, and unexpected outbound connections from the appliance.
- Record the CISA KEV due date of 16 October 2026 against the device group in the change record, with the decision taken (patched, mitigated, or accepted risk).

---

> **On the CISA KEV catalog in this edition:** CVE-2026-104935 — treat these as confirmed-exploited and patch on an emergency change.

## Executive summary

- An unknown actor, UTA0576, chained three Ivanti Connect Secure zero-days to unauthenticated root command execution on an appliance that was fully patched at the time (22.8R2.4); CVE-2026-104935 is in CISA KEV with a 16 October 2026 due date.
- Two of the three Ivanti flaws now have fixes and a third is still pending per Volexity, so patching is necessary but not sufficient - rotate credentials, certificates and session tokens and hunt for prior access.
- Citrix, Fortinet and Ivanti collectively patched 29 vulnerabilities, including critical authentication bypass, code execution and authentication-information-leak flaws; pull the per-vendor advisories and match them to your running versions.
- Cisco disclosed a UEFI Shell Secure Boot bypass on UCS servers and UCS-based appliances, exploitable by an admin-role account or anyone with physical access - relevant if your management or security appliances run on UCS.
- Check Point Research found two high-risk flaws in the LiteLLM LLM gateway, including read-only-to-admin takeover: AI infrastructure is inside the perimeter and often outside the appliance patch cycle.
- Four items in this window concerned an arrest at OpenAI over a hidden recording device and are not network infrastructure news; they are dropped.

---

## Items by vendor

## Ivanti

*5 item(s) — 5 critical*

### VPN gateways

#### 🔴 CRITICAL — Ivanti Connect Secure zero-day exploited in the wild (CVE-2026-104935)

*[Rapid7 Blog](https://www.rapid7.com/blog/post/etr-ivanti-connect-secure-zero-day-exploited-in-the-wild-cve-2026-104935) · 2026-10-13* `KEV`

> Ivanti disclosed CVE-2026-104935, an authentication bypass in Connect Secure already exploited as a zero-day, and CISA added it to KEV the same day with a 16 October due date.

**Affected:** Ivanti Connect Secure. The source does not list affected version ranges or a fixed build - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-104935](https://nvd.nist.gov/vuln/detail/CVE-2026-104935) **[KEV]**  

**What happened.** Ivanti published an advisory on 13 October 2026 for CVE-2026-104935, a high-severity authentication bypass by spoofing (CWE-290) with a CVSSv3.0 score of 8.6, allowing a remote unauthenticated attacker to execute arbitrary commands. Ivanti reports exploitation in the wild prior to disclosure affecting a limited number of customers. CISA confirmed exploitation and added the CVE to the KEV catalog on the day of disclosure, due 16 October 2026, ransomware use unknown. Volexity published a detailed account of threat actor activity detected in early October 2026.

**Why it matters.** Unauthenticated command execution on the appliance that terminates your remote access sessions is a direct route to internal network access, and confirmed pre-disclosure exploitation means the window to act closed before the advisory appeared. The KEV listing makes it mandatory remediation for US federal agencies and a defensible deadline for everyone else.

**Recommended actions**

- Confirm the running version on every Connect Secure appliance against the vendor's fixed-release table; check the vendor advisory for the fixed release and schedule an emergency upgrade.
- Run the Ivanti Integrity Checker Tool and keep the output as evidence; run it again after the upgrade.
- Terminate all active sessions and revoke session tokens after upgrade - assume stolen session cookies.
- Rotate certificates, local admin and service-account credentials, and any secrets stored on the appliance.
- Enforce phishing-resistant MFA and geo/ASN conditional access on the remote access portal.
- Remove administrative access from untrusted interfaces; restrict the management plane to a dedicated management network or jump host.

**Legacy / unpatchable gear.** Connect Secure appliances past end of support will not receive this fix. Take the portal off the internet, isolate the appliance behind a supported inspecting firewall, deny inbound from untrusted zones, and migrate users to a supported remote access path on a dated plan with a budget owner. Monitor it as a high-risk asset (NetFlow baseline, alert on any new outbound flow) until it is gone.

**Detection.** Alert on successful administrative authentication from new source ASNs or geographies; diff the appliance configuration against a known-good export and flag new admin users, altered web resources or new static routes. In /var logs look for entries with inconsistent timestamps and for process activity running as root outside upgrade windows. On the network side, alert on any outbound connection initiated by the appliance to a destination outside your sanctioned update and SSO endpoints.

**AI angle.** None stated in the source.

📄 **[Read the full report at Rapid7 Blog →](https://www.rapid7.com/blog/post/etr-ivanti-connect-secure-zero-day-exploited-in-the-wild-cve-2026-104935)**

#### 🔴 CRITICAL — Ivanti Zero-Day Chain Gave Attackers Root Access to Fully Patched VPN

*[SecurityWeek](https://www.securityweek.com/ivanti-zero-day-chain-gave-attackers-root-access-to-fully-patched-vpn/) · 2026-10-13* `KEV`

> A zero-day chain gave attackers unauthenticated remote code execution with elevated privileges on Ivanti Connect Secure appliances that were fully patched at the time.

**Affected:** Ivanti Connect Secure, including appliances that were fully patched when compromised. No fixed release is stated in this item - check the vendor advisory.  
**CVEs:** [CVE-2026-104935](https://nvd.nist.gov/vuln/detail/CVE-2026-104935) **[KEV]**  

**What happened.** CVE-2026-104935 affects Ivanti Connect Secure and can be exploited for unauthenticated remote code execution with elevated privileges, used as part of a zero-day chain that gave attackers root access to fully patched VPN appliances.

**Why it matters.** 'Fully patched and still compromised' removes patch currency as a sufficient control for this device class. Root on the appliance means the configuration, stored credentials and certificates are all attacker-readable, so the incident response scope is every secret that appliance ever held.

**Recommended actions**

- Patch to the vendor's fixed build and run the vendor Integrity Checker Tool.
- Treat any appliance that was internet-facing in early October 2026 as in scope for investigation regardless of its patch level at the time.
- Rotate every credential, certificate and pre-shared key the appliance held; a pre-patch compromise survives the upgrade.
- Restrict the management interface to an out-of-band or dedicated management VLAN and enforce MFA on all administrative accounts.
- Preserve appliance logs and a configuration export off-box before upgrading, so forensic comparison is still possible afterwards.

**Legacy / unpatchable gear.** Unsupported gateways get no fix for any link in this chain: isolate behind a supported firewall, deny inbound access from untrusted zones, and put strict ACLs and a dedicated jump host in front of management. Set a replacement date and document the accepted risk until then.

**Detection.** Hunt for root-level process execution and file writes on the appliance outside maintenance windows, unexpected outbound sessions from the appliance, and modified web resources under the portal paths. Correlate VPN authentication logs with internal authentication for accounts that appear to log in from the appliance's own address.

**AI angle.** None stated in the source.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/ivanti-zero-day-chain-gave-attackers-root-access-to-fully-patched-vpn/)**

#### 🔴 CRITICAL — Ivanti warns of Connect Secure zero-day exploited in attacks

*[BleepingComputer](https://www.bleepingcomputer.com/news/security/ivanti-warns-of-connect-secure-zero-day-exploited-in-attacks/) · 2026-10-13* 

> Ivanti urged Connect Secure customers to patch immediately after confirming active zero-day exploitation of the remote access gateway.

**Affected:** Ivanti Connect Secure remote access gateway. Fixed build not stated here - check the vendor advisory for the fixed release.  

**What happened.** Ivanti urged customers on Tuesday 13 October 2026 to immediately patch an actively exploited zero-day vulnerability in the Ivanti Connect Secure remote access gateway.

**Why it matters.** A vendor telling customers to act immediately, with exploitation already confirmed, is the trigger for an emergency change rather than a scheduled one. This item corroborates the Rapid7 and Volexity reporting from the vendor's own position.

**Recommended actions**

- Raise an emergency change for the Connect Secure estate today; upgrade the HA secondary first, then fail over and complete the pair.
- Check the vendor advisory for the fixed release before building the change - do not assume a version from press coverage.
- Revoke active sessions and rotate appliance-held secrets after the upgrade.
- If a maintenance window cannot be obtained within hours, reduce exposure instead: restrict portal access by source, and remove administrative access from the internet entirely.
- Confirm after the upgrade that the running version matches the vendor's fixed build on every unit, including cold-standby appliances.

**Legacy / unpatchable gear.** If the appliance cannot be patched because it is out of support, take it off the internet now and route remote access through a supported gateway; compensating controls are strict ACLs, no direct management access, a jump host and full session logging.

**Detection.** Alert on VPN authentications that precede the upgrade and originate from source addresses not seen in your 90-day baseline, and on any configuration change event on the appliance not tied to a change record.

**AI angle.** None stated in the source.

📄 **[Read the full report at BleepingComputer →](https://www.bleepingcomputer.com/news/security/ivanti-warns-of-connect-secure-zero-day-exploited-in-attacks/)**

#### 🔴 CRITICAL — Attackers chained three Ivanti zero-days to gain root on fully patched VPN appliances

*[Help Net Security](https://www.helpnetsecurity.com/2026/10/14/ivanti-zero-days-gain-root-patched-vpn-appliances/) · 2026-10-14* 

> Volexity attributes the Ivanti Connect Secure intrusions to UTA0576, which reached root on a fully patched 22.8R2.4 appliance and exfiltrated credentials and configuration data.

**Affected:** Ivanti Connect Secure appliances; Volexity's compromised case was fully patched at the time, running version 22.8R2.4. That is the compromised version, not a fixed one - check the vendor advisory for the fixed release.  

**What happened.** Volexity reports that in the first week of October 2026 it discovered an unknown threat actor, tracked as UTA0576, abusing a zero-day exploit chain of three vulnerabilities to compromise a fully patched Ivanti Connect Secure VPN appliance running 22.8R2.4. The actor breached organisations, collected credentials, exfiltrated configuration data and performed network reconnaissance. Ivanti says exploitation affected a limited number of customers.

**Why it matters.** This is the attribution and tradecraft detail behind the headline: credential collection plus configuration exfiltration plus reconnaissance is pre-positioning for lateral movement, not smash-and-grab. Every credential in that configuration - LDAP bind accounts, RADIUS secrets, certificates - should be treated as known to the actor on any appliance in scope.

**Recommended actions**

- Scope the investigation by exposure window, not by patch level: any internet-facing Connect Secure appliance during the first week of October 2026 onward is in scope.
- Rotate everything the configuration contained: LDAP/AD bind accounts, RADIUS and TACACS+ shared secrets, certificates, API keys and VPN pre-shared keys.
- Hunt for the follow-on activity the report describes - internal reconnaissance from the appliance's address, new authentications for recovered service accounts, and unexpected outbound connections.
- Check the vendor advisory for the fixed release and upgrade; then run the Integrity Checker Tool and keep the result.
- If credential theft is confirmed, force password resets for accounts that authenticated through the gateway and revoke their session tokens.

**Legacy / unpatchable gear.** An out-of-support appliance cannot be remediated against a three-flaw chain where one flaw has no fix. Pull it off the internet, keep it behind a supported inspecting firewall with inbound denied from untrusted zones, and treat its stored secrets as already exposed - rotate them regardless of whether you find evidence.

**Detection.** Query for SMB, LDAP, SNMP or SSH connections sourced from the VPN appliance's management address to internal hosts - the appliance should rarely initiate those. Alert on service accounts recovered from appliance configuration authenticating from new sources, and on large outbound transfers from the appliance. Review /var logs for timestamp anomalies and for configuration export events.

**AI angle.** None stated in the source.

📄 **[Read the full report at Help Net Security →](https://www.helpnetsecurity.com/2026/10/14/ivanti-zero-days-gain-root-patched-vpn-appliances/)**

#### 🔴 CRITICAL — Attacker Chains Three Ivanti Connect Secure Zero-Days for Root Access, Only Two Patched So Far

*[The Hacker News](https://thehackernews.com/2026/10/attacker-chains-three-ivanti-connect.html) · 2026-10-13* 

> Two of the three Ivanti Connect Secure zero-days used in the attack chain now have fixes; per Volexity a fix for the third is still pending.

**Affected:** Ivanti Connect Secure. Fixes exist for two of the three chained flaws; the third is unpatched as reported. Check the vendor advisory for the fixed release and for the status of the remaining flaw.  

**What happened.** An attacker chained three zero-day flaws in Ivanti Connect Secure to run commands as root without logging in, as reported by Volexity on 13 October 2026. Ivanti has released fixes for two of the flaws and told customers on affected versions to act immediately. A fix for the third is still pending according to Volexity.

**Why it matters.** A partially fixed chain means the upgrade is mandatory but not terminal - you will need to come back for the third fix, and in the meantime exposure reduction is doing the real work. Plan for a second change window and keep the device group flagged rather than closing the ticket after the first patch.

**Recommended actions**

- Apply the available fixes now, and leave the Connect Secure device group open in the tracker pending the third fix.
- Subscribe to the Ivanti advisory for updates and set a review date so the remaining flaw is not forgotten after the first upgrade.
- Compensate while the third flaw is unpatched: restrict portal access by source geography/ASN where your user base allows, keep administration off the internet, and enforce phishing-resistant MFA.
- Increase log retention and monitoring on these appliances for the interim period, and ship appliance logs to the SIEM if they are not already there.
- Re-run the Integrity Checker Tool on a schedule during the interim rather than once at upgrade.

**Legacy / unpatchable gear.** For gear that cannot be patched at all, the partial-fix situation is academic: isolate it behind a supported inspecting firewall, deny inbound from untrusted zones, and move remote access to a supported platform on a dated plan with a named owner.

**Detection.** Alert on any successful root-level command execution or shell activity on the appliance, on new local accounts, and on changes to portal web resources. During the interim period, treat any unexplained appliance restart or configuration change as an incident until proven otherwise.

**AI angle.** None stated in the source.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/10/attacker-chains-three-ivanti-connect.html)**

---

## Multiple

*1 item(s) — 1 high*

### Firewalls

#### 🟠 HIGH — Citrix, Fortinet, Ivanti Patch 29 Vulnerabilities Across Their Products

*[SecurityWeek](https://www.securityweek.com/citrix-fortinet-ivanti-patch-29-vulnerabilities-across-their-products/) · 2026-10-14* 

> Citrix, Fortinet and Ivanti shipped fixes for 29 vulnerabilities, including critical authentication bypass, code execution and authentication-information-leak flaws.

**Affected:** Products across Citrix, Fortinet and Ivanti. This item does not enumerate the individual products, CVEs or fixed versions - pull each vendor's advisories and match them against your running versions.  

**What happened.** Citrix, Fortinet and Ivanti patched 29 vulnerabilities across their products. The critical-severity flaws could lead to authentication bypass, code execution and leaks of sensitive authentication information.

**Why it matters.** Three of the most commonly internet-facing appliance vendors patched on the same cycle, during a week when one of them already has a KEV-listed zero-day under active exploitation. Authentication bypass and authentication-information leaks on perimeter appliances are the two classes that have historically turned into mass exploitation within days of a patch being reverse-engineered.

**Recommended actions**

- Pull the current advisory set from each vendor's PSIRT and build a list filtered to the models and versions in your asset inventory - do not work from the aggregate count.
- Prioritise anything internet-facing with authentication bypass or authentication-information disclosure, then schedule the rest into the normal change cycle.
- Apply the vendor fixed release, or the documented workaround where a maintenance window is not yet available.
- Where an authentication-information leak is in scope, rotate the exposed credential class (admin accounts, API tokens, certificates) after patching rather than assuming the patch is enough.
- Record the decision (patched / mitigated / accepted risk) against each device group in the change record.

**Legacy / unpatchable gear.** Models that have dropped out of support will not appear in these fixed-release tables at all. Confirm support status per model before planning, isolate anything unsupported behind a supported inspecting firewall, and set a replacement date and budget owner.

**Detection.** After patching, alert on administrative logins, configuration changes and firmware changes shipped from these appliances to the SIEM; for authentication-bypass classes, specifically alert on successful admin sessions with no preceding authentication event.

**AI angle.** None stated in the source.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/citrix-fortinet-ivanti-patch-29-vulnerabilities-across-their-products/)**

---

## Cisco

*2 item(s) — 1 medium, 1 watch*

### Other

#### 🟡 MEDIUM — Cisco UCS and UCS-Based Appliances UEFI Shell Secure Boot Bypass Vulnerability

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-ucs-uefi-sb-bypass-eb6xC5GW) · 2026-10-13* `vendor-advisory`

> A UEFI Shell flaw lets an admin-role account, or anyone with physical access, bypass Secure Boot validation on Cisco UCS servers and UCS-based appliances and run unsigned code.

**Affected:** Cisco UCS Servers and UCS-based appliances. The advisory text in this item does not state fixed release versions - check the Cisco advisory for the fixed release.  

**What happened.** A vulnerability in the UEFI Shell implementation of Cisco UCS Servers and UCS-based appliances could allow an authenticated attacker holding a user- or admin-role account, or an unauthenticated attacker with physical access, to bypass UEFI Secure Boot validation and execute unauthorized software. The cause is the availability of memory write commands in the UEFI Shell while Secure Boot is enabled. An attacker selects the UEFI Shell boot option at boot time and uses shell commands to modify UEFI memory variables, manipulating the preboot environment and overwriting Secure Boot-related memory values.

**Why it matters.** This matters to a network team to the extent that your management and security appliances run on UCS hardware - a Secure Boot bypass at the preboot layer defeats the integrity guarantee that firmware-level persistence is supposed to be caught by. Exploitation needs valid credentials or physical presence, which keeps it out of the mass-exploitation class, but it raises the stakes on who holds CIMC accounts and on physical access to colocation racks.

**Recommended actions**

- Identify which UCS servers and UCS-based appliances carry management or security roles, and check the Cisco advisory for the fixed firmware release for each.
- Reduce the account population that can reach the UCS management plane; remove shared local accounts and enforce per-admin accounts with TACACS+/RADIUS and MFA.
- Restrict CIMC/UCS management reachability to an out-of-band management network with management ACLs; it should not be reachable from user VLANs.
- Set a boot-time password and restrict interactive boot-option selection where the platform supports it, so UEFI Shell cannot be chosen by someone at the console.
- Review physical access controls and logging for racks holding these devices, including vendor and smart-hands access.

**Legacy / unpatchable gear.** Older UCS generations that no longer receive firmware updates cannot be fixed here. Treat them as physically sensitive assets: locked racks with access logging, no management plane reachable outside the out-of-band network, and a documented replacement date.

**Detection.** Alert on unexpected host reboots and on UEFI/BIOS setting changes reported through CIMC logs; forward CIMC and KVM session logs to the SIEM and alert on console sessions outside change windows. Track boot-time measurements where you collect them, and investigate any change in the preboot measurement that is not tied to an approved firmware update.

**AI angle.** None stated in the source.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-ucs-uefi-sb-bypass-eb6xC5GW)**

#### ⚪ WATCH — Always-On Assurance: How Cisco is Reimagining the Secure Development Lifecycle

*[Cisco Security Blog](https://blogs.cisco.com/security/always-on-assurance/) · 2026-10-13* `AI` `AI-defense`

> Cisco describes how its Secure Development Lifecycle is evolving to support AI-driven software development across its products.

**Affected:** No specific vulnerability or device. Cisco's internal development process, the Cisco Secure Development Lifecycle (CSDL), as described by Cisco.  

**What happened.** Cisco published a blog post describing how its Secure Development Lifecycle (CSDL) is evolving to support AI-driven development, with the stated aim of secure and resilient software delivered at speed.

**Why it matters.** No action for the network team today, but it is a data point for vendor assurance questions: as vendors accelerate development with AI assistance, the controls around that process become part of your supply-chain risk picture. Useful as a reference when procurement or audit asks how a vendor assures the code running on your gear.

**Recommended actions**

- File this with vendor assurance material; cite it when your procurement or audit process asks Cisco how AI-assisted development is governed.
- Add a question to your vendor security questionnaires about AI use in development and the assurance applied to AI-generated code.
- Keep patch SLAs for network devices tied to exploitation risk rather than vendor process claims - a described lifecycle is not a control you operate.

**Legacy / unpatchable gear.** Process improvements at the vendor do not reach end-of-life gear, which stops receiving the output of any development lifecycle. Keep unsupported devices isolated behind a supported inspecting firewall with a documented replacement date and accepted risk.

**Detection.** Not applicable - no vulnerability or indicator is described.

**AI angle.** Vendor-side: Cisco is adapting its secure development process to AI-assisted software production, which is the assurance half of the AI question rather than an attack or an integration on the device itself.

📄 **[Read the full report at Cisco Security Blog →](https://blogs.cisco.com/security/always-on-assurance/)**

---

## Check Point

*1 item(s) — 1 watch*

### Other

#### ⚪ WATCH — Check Point Research Discovers High Risk Vulnerabilities in LiteLLM, One of the Most Widely Used LLM Gateways

*[Check Point Blog](https://blog.checkpoint.com/security/check-point-research-discovers-high-risk-vulnerabilities-in-litellm-one-of-the-most-widely-used-llm-gateways/) · 2026-10-13* `AI` `AI-defense`

> Check Point Research found two high-risk flaws in the LiteLLM open-source LLM gateway, one letting read-only users reach admin functions and take over the gateway.

**Affected:** LiteLLM, the open-source LLM gateway, tracked as CVE-2026-35029 and CVE-2026-59823. The item does not state fixed versions - check the LiteLLM project advisory for the fixed release.  
**CVEs:** [CVE-2026-35029](https://nvd.nist.gov/vuln/detail/CVE-2026-35029), [CVE-2026-59823](https://nvd.nist.gov/vuln/detail/CVE-2026-59823)  

**What happened.** Check Point Research disclosed two high-risk vulnerabilities in LiteLLM, which organisations use to connect applications to AI providers such as OpenAI, Google and Anthropic. CVE-2026-35029 allowed any user, including those with read-only access, to reach administrative functions and take over the gateway. CVE-2026-59823 allowed attackers to redirect traffic. Check Point cites 97 million downloads per month and over 59,000 GitHub stars, with deployments inside large enterprises, startups and platforms used to build AI applications.

**Why it matters.** This is not a network appliance, so it is tracked as a watch item rather than a device action - but it is increasingly deployed like one: in the traffic path, holding organisation-wide provider API credentials, and often outside the appliance patch and monitoring regime. Gateway takeover plus traffic redirection is the combination that enables silent interception of prompts and responses, which for many organisations now carries the same sensitivity as the data crossing the perimeter.

**Recommended actions**

- Find out whether LiteLLM is deployed in your environment, where it sits in the network, and what it can reach - application teams often stand it up without network review.
- Check the project advisory for the fixed release and get it patched under a named owner; add the component to the asset inventory with a patch SLA.
- Restrict access to the gateway's administrative endpoints at the network layer rather than relying on its internal role model, which CVE-2026-35029 defeats.
- Rotate the AI provider API keys the gateway holds after patching.
- Constrain and monitor egress: allow the gateway to reach only sanctioned provider endpoints, and alert on any other outbound destination as possible traffic redirection.

**Legacy / unpatchable gear.** Where an AI gateway runs on infrastructure nobody will patch - an unowned container host, an abandoned pilot - isolate it behind an inspecting firewall, deny inbound from untrusted zones, revoke its provider credentials, and decommission it rather than carrying it. If it must stay, apply strict ACLs, no direct management access, a dedicated jump host and full session logging, with a documented replacement date.

**Detection.** Alert on administrative API calls to the gateway from source addresses outside the admin network, on role or key changes within the gateway, and on outbound connections from the gateway host to destinations other than the approved provider endpoints. Baseline request volume per API key and alert on deviation.

**AI angle.** Direct: this is a vulnerability in core AI plumbing - the gateway brokering enterprise access to LLM providers - and it shows AI components entering the network perimeter faster than they enter the inventory.

📄 **[Read the full report at Check Point Blog →](https://blog.checkpoint.com/security/check-point-research-discovers-high-risk-vulnerabilities-in-litellm-one-of-the-most-widely-used-llm-gateways/)**

---

## 🛡️ Vendor AI defences introduced

AI capabilities the vendors shipped this cycle - the answer side of the AI story.

### Cisco

**Cisco Secure Development Lifecycle (CSDL)** — Cisco describes CSDL evolving to support AI-driven development, with the stated goal of secure and resilient software at speed. The source is a vendor description of its approach, not a product launch.

- *Defends against:* Security defects introduced when software - including the software running on network devices - is produced with AI assistance and at higher velocity.
- *Availability:* described-only
- *Worth evaluating if:* You run vendor assurance or procurement review and need a documented position on how a network vendor governs AI-assisted development of the code on your appliances.
- 📄 [Read the announcement at Cisco Security Blog →](https://blogs.cisco.com/security/always-on-assurance/)

---

## 🧠 AI & network devices — LLM gateways are becoming edge infrastructure, and they are being attacked like it

Check Point Research disclosed two high-risk vulnerabilities in LiteLLM, the open-source LLM gateway organisations use to connect applications to AI providers: CVE-2026-35029, which allowed any user including a read-only account to reach administrative functions and take over the gateway, and CVE-2026-59823, which allowed attackers to redirect traffic. Check Point puts LiteLLM at 97 million downloads per month and over 59,000 GitHub stars, running inside large enterprises and inside platforms used to build AI applications.

This matters to a network team because an LLM gateway sits in the traffic path, holds provider API credentials for the whole organisation, and is frequently stood up by application teams outside the normal appliance lifecycle - unpatched, unmonitored, and sometimes reachable from more of the network than anyone intended. A privilege-escalation-to-takeover bug in that position is the same shape of problem as a management interface left exposed on a gateway appliance, and it deserves the same controls. Separately, Cisco describes its Secure Development Lifecycle adapting to AI-driven development, which is the vendor-side half of the same question: how assurance holds up when software is produced faster.

Neither item is an AI-assisted attack on network gear. The honest read is that AI infrastructure is arriving inside the network perimeter faster than it is being inventoried.

**Vendors with an AI angle in this edition:** Check Point, Cisco, Ivanti, Multiple

**Preventing it on current systems**

- Add LLM gateways and AI proxies to the network asset inventory with a named owner and a patch SLA; if nobody can tell you the running version, that is the finding.
- Treat any AI/LLM integration reachable from network devices (assistants, AIOps, MCP or agent connectors) as a privileged admin path: scoped read-only credentials, no unattended config write, full audit trail.
- Do not rely on a read-only role in an AI gateway as a security boundary - CVE-2026-35029 is exactly that assumption failing; segment the gateway and restrict who can reach its admin endpoints at the network layer.
- Rotate AI provider API keys held by a gateway after patching it, and alert on egress from the gateway to destinations other than the sanctioned provider endpoints.
- Test AI-facing management surfaces for prompt injection from attacker-controlled data (device logs, hostnames, ticket text) before letting an agent act on them.

**Preventing it on legacy / end-of-life systems**

- Where an AI gateway runs on unsupported or unmanaged infrastructure that cannot be patched, put it behind an inspecting firewall and deny inbound access from untrusted zones.
- Apply compensating controls to any AI component you cannot upgrade: strict ACLs, no direct management access, dedicated jump host, full session logging.
- Baseline NetFlow for these hosts and alert on new outbound flows; set a replacement or migration date with a budget owner and document the accepted risk until then.

---

<!--index
{
 "url": "/2026/10/14/network-security-digest/",
 "date": "2026-10-14",
 "entries": [
  {
   "title": "Ivanti Connect Secure zero-day exploited in the wild (CVE-2026-104935)",
   "link": "https://www.rapid7.com/blog/post/etr-ivanti-connect-secure-zero-day-exploited-in-the-wild-cve-2026-104935",
   "source": "Rapid7 Blog",
   "relevance": "critical",
   "device_types": [
    "vpn-gateway"
   ],
   "vendors": [
    "Ivanti"
   ],
   "cves": [
    "CVE-2026-104935"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None stated in the source.",
   "summary": "Ivanti disclosed CVE-2026-104935, an authentication bypass in Connect Secure already exploited as a zero-day, and CISA added it to KEV the same day with a 16 October due date.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Ivanti Zero-Day Chain Gave Attackers Root Access to Fully Patched VPN",
   "link": "https://www.securityweek.com/ivanti-zero-day-chain-gave-attackers-root-access-to-fully-patched-vpn/",
   "source": "SecurityWeek",
   "relevance": "critical",
   "device_types": [
    "vpn-gateway"
   ],
   "vendors": [
    "Ivanti"
   ],
   "cves": [
    "CVE-2026-104935"
   ],
   "kev": true,
   "ai": true,
   "ai_angle": "None stated in the source.",
   "summary": "A zero-day chain gave attackers unauthenticated remote code execution with elevated privileges on Ivanti Connect Secure appliances that were fully patched at the time.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Ivanti warns of Connect Secure zero-day exploited in attacks",
   "link": "https://www.bleepingcomputer.com/news/security/ivanti-warns-of-connect-secure-zero-day-exploited-in-attacks/",
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
   "ai_angle": "None stated in the source.",
   "summary": "Ivanti urged Connect Secure customers to patch immediately after confirming active zero-day exploitation of the remote access gateway.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Attackers chained three Ivanti zero-days to gain root on fully patched VPN appliances",
   "link": "https://www.helpnetsecurity.com/2026/10/14/ivanti-zero-days-gain-root-patched-vpn-appliances/",
   "source": "Help Net Security",
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
   "ai_angle": "None stated in the source.",
   "summary": "Volexity attributes the Ivanti Connect Secure intrusions to UTA0576, which reached root on a fully patched 22.8R2.4 appliance and exfiltrated credentials and configuration data.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Attacker Chains Three Ivanti Connect Secure Zero-Days for Root Access, Only Two Patched So Far",
   "link": "https://thehackernews.com/2026/10/attacker-chains-three-ivanti-connect.html",
   "source": "The Hacker News",
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
   "ai_angle": "None stated in the source.",
   "summary": "Two of the three Ivanti Connect Secure zero-days used in the attack chain now have fixes; per Volexity a fix for the third is still pending.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Citrix, Fortinet, Ivanti Patch 29 Vulnerabilities Across Their Products",
   "link": "https://www.securityweek.com/citrix-fortinet-ivanti-patch-29-vulnerabilities-across-their-products/",
   "source": "SecurityWeek",
   "relevance": "high",
   "device_types": [
    "firewall",
    "vpn-gateway",
    "load-balancer"
   ],
   "vendors": [
    "Multiple"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "None stated in the source.",
   "summary": "Citrix, Fortinet and Ivanti shipped fixes for 29 vulnerabilities, including critical authentication bypass, code execution and authentication-information-leak flaws.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Cisco UCS and UCS-Based Appliances UEFI Shell Secure Boot Bypass Vulnerability",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-ucs-uefi-sb-bypass-eb6xC5GW",
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
   "ai": true,
   "ai_angle": "None stated in the source.",
   "summary": "A UEFI Shell flaw lets an admin-role account, or anyone with physical access, bypass Secure Boot validation on Cisco UCS servers and UCS-based appliances and run unsigned code.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Check Point Research Discovers High Risk Vulnerabilities in LiteLLM, One of the Most Widely Used LLM Gateways",
   "link": "https://blog.checkpoint.com/security/check-point-research-discovers-high-risk-vulnerabilities-in-litellm-one-of-the-most-widely-used-llm-gateways/",
   "source": "Check Point Blog",
   "relevance": "watch",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [
    "CVE-2026-35029",
    "CVE-2026-59823"
   ],
   "kev": false,
   "ai": true,
   "ai_angle": "Direct: this is a vulnerability in core AI plumbing - the gateway brokering enterprise access to LLM providers - and it shows AI components entering the network perimeter faster than they enter the inventory.",
   "summary": "Check Point Research found two high-risk flaws in the LiteLLM open-source LLM gateway, one letting read-only users reach admin functions and take over the gateway.",
   "ai_defense": true,
   "defense": null
  },
  {
   "title": "Always-On Assurance: How Cisco is Reimagining the Secure Development Lifecycle",
   "link": "https://blogs.cisco.com/security/always-on-assurance/",
   "source": "Cisco Security Blog",
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
   "ai_angle": "Vendor-side: Cisco is adapting its secure development process to AI-assisted software production, which is the assurance half of the AI question rather than an attack or an integration on the device itself.",
   "summary": "Cisco describes how its Secure Development Lifecycle is evolving to support AI-driven software development across its products.",
   "ai_defense": true,
   "defense": {
    "solution": "Cisco Secure Development Lifecycle (CSDL)",
    "what_it_does": "Cisco describes CSDL evolving to support AI-driven development, with the stated goal of secure and resilient software at speed. The source is a vendor description of its approach, not a product launch.",
    "defends_against": "Security defects introduced when software - including the software running on network devices - is produced with AI assistance and at higher velocity.",
    "availability": "described-only"
   }
  }
 ]
}
-->

## How this was produced

- Feeds polled: 22 ok, 1 failed
- Raw items: 709 → in window: 68 → network-device relevant: 14 → published: 9
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
