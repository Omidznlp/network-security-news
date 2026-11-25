---
layout: post
title: "Network Device Security Digest - 2026-11-25"
date: 2026-11-25 17:04:11 +0000
edition: daily
critical_count: 3
item_count: 11
kev: "CVE-2026-103406, CVE-2026-111497"
analysis_mode: "claude-code-action"
categories: digest
---

# Network Device Security Digest — 2026-11-25

*daily edition · window 24h · 23 feeds · 11 items · 3 critical · generated 2026-11-25 17:04 UTC*

> Scope: firewalls, VPN gateways, routers, switches, wireless controllers, load balancers, management platforms and SD-WAN edge. Everything else is filtered out.

---

## 📌 Top story — CISA adds HPE Juniper Session Smart Conductor and Router command injection to the KEV catalog - root via a crafted salt minion ID

CISA added CVE-2026-103406 to the Known Exploited Vulnerabilities catalog on 2026-11-25. The flaw is an OS command injection in Hewlett Packard Enterprise (HPE) Juniper Session Smart Conductor and Session Smart Router: a remote attacker can execute arbitrary commands as root by supplying a specially crafted salt minion ID. The federal remediation due date is 2026-11-28, which is three days out - treat that as the signal for how fast this is being exploited.

The blast radius is the reason this leads. Session Smart Conductor is the orchestration and management plane for a Session Smart Router fleet; root on the Conductor is root on every router it manages, plus the configuration templates, credentials and certificates it holds. Session Smart Routers themselves are SD-WAN edge devices, so an attacker who lands on them sits astride branch and WAN traffic. The attack path is the Salt management channel, not the data plane, which makes exposure of the Conductor's management interfaces the deciding factor in whether you are reachable.

Neither the KEV entry nor any other collected source names a fixed release, so do not guess one - check the vendor advisory for the fixed release and the exact affected build range before you open a change. CISA's required action is to apply vendor mitigations in line with BOD 26-04 and its forensics triage requirements, or discontinue use of the product where mitigations are unavailable. Because exploitation is confirmed and grants root, assume pre-patch compromise on any Conductor that was reachable and hunt before you declare the upgrade done.

**Do this first**

- Inventory every Session Smart Conductor and Session Smart Router and record the running version; the Conductor is the priority asset because it controls the routers.
- Check the vendor advisory for the fixed release and the mitigation for CVE-2026-103406, then patch or mitigate the Conductor first and the routers after, inside an emergency change.
- Remove the Conductor's management and Salt channel from any internet-facing or untrusted interface; restrict to a dedicated management VLAN or out-of-band network with an explicit ACL.
- Treat any reachable Conductor as potentially compromised: rotate local admin credentials, API keys, certificates and router onboarding secrets after patching, because root access survives an upgrade otherwise.
- Hunt for persistence on Conductor and routers - unexpected local accounts, new or modified salt states and minion configuration, unexplained root shell history, new outbound connections and changed static routes or tunnels.
- If BOD 26-04 applies to you, record the decision against the 2026-11-28 due date; if no mitigation is available for a device, document the discontinue-use decision instead.

---

> **On the CISA KEV catalog in this edition:** CVE-2026-103406, CVE-2026-111497 — treat these as confirmed-exploited and patch on an emergency change.

## Executive summary

- CISA added CVE-2026-103406 in HPE Juniper Session Smart Conductor and Session Smart Router to the KEV catalog on 2026-11-25 with a 2026-11-28 due date: remote root via a crafted salt minion ID, and the Conductor is the management plane for a router fleet.
- A China-linked group chained PHP2Shell with the Fragnesia flaw in AI-assisted attacks to root Palo Alto Networks firewalls across 25 servers; Palo Alto Networks has shipped patches, so confirm your PAN-OS build against the vendor advisory.
- GreyNoise reports 24 Check Point Security Management servers compromised across 22 organisations using AI-driven offensive tooling - a management-platform compromise, so rotate managed-gateway credentials, not just the server's own.
- Upstream component flaws dominate the vendor advisories: Cisco is assessing Node.js (including KEV-listed Brotli CVE-2026-111497), nginx and PHP (phpAfterShell CVE-2026-126714), and Palo Alto Networks has opened a critical portfolio-status bulletin for the nginx CVEs.
- No collected source states a fixed release for the Juniper KEV entry or for Fragnesia - check the vendor advisory before scheduling any change, and do not act on a version number from a news report.
- Webworm is recompiling SoftEther VPN from source to strip detection artifacts, so inventory-based VPN detection will miss it; hunt on egress behaviour instead.

---

## Items by vendor

## Palo Alto Networks

*4 item(s) — 1 critical, 1 high, 2 watch*

### Firewalls

#### 🔴 CRITICAL — Palo Alto Networks Patches Fragnesia Flaw Used to Root Firewalls

*[SecurityWeek](https://www.securityweek.com/palo-alto-networks-patches-fragnesia-flaw-used-to-root-firewalls/) · 2026-11-25* `AI`

> A China-linked group chained PHP2Shell with the Fragnesia flaw in AI-assisted attacks to gain root on Palo Alto Networks firewalls, hitting 25 servers; patches are out.

**Affected:** Palo Alto Networks firewalls, per SecurityWeek. The report as collected does not give a CVE identifier for Fragnesia or the affected and fixed PAN-OS versions - check the vendor advisory for the fixed release and the affected version range.  

**What happened.** SecurityWeek reports that Palo Alto Networks has patched a flaw referred to as Fragnesia that was used to gain root on firewalls. A China-linked hacking group chained PHP2Shell with Fragnesia in what the report describes as AI-assisted attacks, affecting 25 servers.

**Why it matters.** Root on a perimeter firewall is the worst case for this device class: the attacker controls policy, sees decrypted traffic where the device decrypts, holds the VPN configuration and can pivot inward from the one device that is trusted by both sides. The chain is the notable part - a PHP-layer foothold followed by privilege escalation, which matches the separate Cisco PHP advisory in this same window and means patching only the second half of the chain is not enough. Exploitation is confirmed and limited in count so far, but the technique is now public.

**Recommended actions**

- Identify your PAN-OS firewalls and their running versions, then confirm each against the Palo Alto Networks advisory's fixed-release table; patch the HA pair secondary first, then fail over.
- Treat this as an emergency change for any firewall whose management interface or administrative web access has ever been reachable from the internet.
- After patching, rotate local admin credentials, API keys, certificates and VPN pre-shared keys, and terminate active administrative and VPN sessions - root access taken before the upgrade survives it.
- Export and review the firewall configuration for unexpected admin users, modified login pages, new scripts, unexplained static routes and changed management-plane permit rules.
- Remove management access (HTTPS/SSH/API) from any untrusted or internet-facing interface and restrict it to a dedicated management VLAN or out-of-band network with MFA enforced on every admin account.

**Legacy / unpatchable gear.** Firewalls on an unsupported PAN-OS branch will not receive the Fragnesia fix. Deny all inbound administrative access, place the device behind a supported inspecting layer, allow management only from a dedicated jump host with session recording, baseline its NetFlow and alert on any new outbound flow, and schedule replacement with a named owner.

**Detection.** Hunt for shell or interpreter execution parented by the firewall's web management process, new or modified files in the management web root, root logins outside change windows, and outbound connections from the firewall's management plane to unfamiliar addresses. Compare the current configuration against the last known-good export rather than reading it fresh.

**AI angle.** The report characterises the campaign as AI-assisted. No AI capability on the firewall was involved and the vulnerabilities are not attributed to AI discovery in the collected text - the defensive implication is tempo, so shorten the patch window for internet-facing firewalls rather than buying a control for an AI threat.

📄 **[Read the full report at SecurityWeek →](https://www.securityweek.com/palo-alto-networks-patches-fragnesia-flaw-used-to-root-firewalls/)**

#### 🟠 HIGH — CVE-2026-123633 Portfolio Status: Impact of NGINX Vulnerabilities (CVE-2026-123633, CVE-2026-123007) on Palo Alto Networks Products (Severity: CRITICAL)

*[Palo Alto Networks PSIRT](https://security.paloaltonetworks.com/CVE-2026-123633) · 2026-11-24* `vendor-advisory`

> Palo Alto Networks opened a critical-severity portfolio-status bulletin for the nginx flaws CVE-2026-123633 and CVE-2026-123007 affecting its products.

**Affected:** Palo Alto Networks products, per the portfolio-status bulletin for CVE-2026-123633, rated Critical. The collected entry carries no body text, so the affected product list and fixed releases are not available here - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-123007](https://nvd.nist.gov/vuln/detail/CVE-2026-123007), [CVE-2026-123633](https://nvd.nist.gov/vuln/detail/CVE-2026-123633)  

**What happened.** Palo Alto Networks published a portfolio status bulletin, rated Critical, covering the impact of the nginx vulnerabilities CVE-2026-123633 and CVE-2026-123007 on its products. Both CVEs are in the nginx ngx_http_v2_module and are part of the same upstream nginx release that Cisco is also assessing in this cycle.

**Why it matters.** A Critical severity rating on a portfolio-status bulletin means the vendor expects real product impact, not a courtesy note. nginx is a front-end web component, so the reachable surface is likely the HTTP listeners on appliances - including management interfaces and GlobalProtect-style portals, if the bulletin lists them. Two vendors assessing the same upstream flaws in the same window is the signal to treat this as an estate-wide question rather than a per-vendor one.

**Recommended actions**

- Read the Palo Alto Networks portfolio-status bulletin for the confirmed affected product list and the fixed releases, and re-check it, since portfolio bulletins are updated as assessment proceeds.
- Match the affected list against your inventory by product and running version, and schedule upgrades per the vendor's fixed-release table - HA pair secondary first.
- Until a fixed build is applied, remove administrative and portal HTTP listeners from untrusted interfaces and apply management-plane ACLs.
- Cross-reference the same two CVEs against every other appliance vendor in the estate; the upstream nginx range affected is wide.
- After upgrading, verify configuration sync across device groups and confirm the running version on each unit.

**Legacy / unpatchable gear.** Out-of-support products will not get the rebuilt nginx component. Terminate untrusted HTTP in front of them on a supported device, deny inbound access from untrusted zones, restrict management to a jump host with full logging, and document the accepted risk against a replacement date.

**Detection.** Monitor appliance web stacks for HTTP/2 frame errors and repeated worker restarts, and alert on any process or outbound connection spawned by the web service on a security appliance.

📄 **[Read the full report at Palo Alto Networks PSIRT →](https://security.paloaltonetworks.com/CVE-2026-123633)**

#### ⚪ WATCH — PAN-SA-2026-0015 Security Bulletin: OSS CVEs Fixed in PAN-OS (Severity: INFORMATIONAL)

*[Palo Alto Networks PSIRT](https://security.paloaltonetworks.com/PAN-SA-2026-0015) · 2026-11-24* `vendor-advisory`

> Palo Alto Networks published an informational bulletin on open-source CVEs fixed in PAN-OS, with no exploitable impact claimed.

**Affected:** PAN-OS, per bulletin PAN-SA-2026-0015. The collected feed entry carries no body text, so affected versions, the CVE list and fixed releases are not available here - check the vendor advisory.  

**What happened.** Palo Alto Networks issued PAN-SA-2026-0015, an informational security bulletin covering open-source CVEs fixed in PAN-OS. The collected entry has no summary text beyond the title and severity rating of Informational.

**Why it matters.** Informational PAN-SA bulletins exist so that scanner findings against bundled open-source components can be answered with a vendor position rather than an emergency change. It matters for your vulnerability-management paperwork and for picking up the fixes on the next scheduled upgrade, not for tonight's change window.

**Recommended actions**

- Read PAN-SA-2026-0015 and record the vendor position against any matching scanner findings on your PAN-OS devices, so the tickets can be closed with evidence.
- Fold the fixed PAN-OS release into your next scheduled maintenance rather than opening an out-of-band change.
- Keep the fixes from this bulletin separate from the Fragnesia patching work in this cycle - do not let an informational item consume the emergency window.

**Legacy / unpatchable gear.** For PAN-OS branches that are out of support, the component fixes in this bulletin will never arrive; carry the finding as an accepted risk against a documented replacement or upgrade date, with management access restricted to a jump host in the meantime.

**Detection.** No specific detection applies; confirm your configuration-management records show the running PAN-OS version so the bulletin can be assessed against it accurately.

📄 **[Read the full report at Palo Alto Networks PSIRT →](https://security.paloaltonetworks.com/PAN-SA-2026-0015)**

#### ⚪ WATCH — PAN-SA-2026-0005 Informational Bulletin: Precautionary Fixes for Non-Exploitable OSS CVEs in PAN-OS (Severity: INFORMATIONAL)

*[Palo Alto Networks PSIRT](https://security.paloaltonetworks.com/PAN-SA-2026-0005) · 2026-11-24* `vendor-advisory`

> A second Palo Alto Networks informational bulletin covers precautionary PAN-OS fixes for open-source CVEs the vendor assesses as non-exploitable.

**Affected:** PAN-OS, per bulletin PAN-SA-2026-0005. The collected entry has no body text, so the CVE list, affected versions and fixed releases are not available here - check the vendor advisory.  

**What happened.** Palo Alto Networks issued PAN-SA-2026-0005, an informational bulletin describing precautionary fixes for open-source CVEs in PAN-OS that the vendor assesses as non-exploitable in its products. Severity is rated Informational.

**Why it matters.** This is the vendor pre-empting scanner noise: the components are updated even though the vendor's position is that the flaws are not exploitable as shipped. Useful for closing vulnerability-management tickets with a documented vendor statement, not a reason to touch a firewall out of band.

**Recommended actions**

- Record the vendor's non-exploitable assessment from PAN-SA-2026-0005 against matching scanner findings so the tickets close with evidence rather than a suppression.
- Pick the fixes up with the next planned PAN-OS maintenance upgrade.
- Do not reprioritise the Fragnesia or nginx portfolio work in this cycle because of this bulletin.

**Legacy / unpatchable gear.** On unsupported PAN-OS branches these precautionary updates will not be delivered; keep the finding as an accepted risk with a documented upgrade or replacement date and management access limited to a jump host.

**Detection.** No detection applies; this is a vulnerability-management record-keeping item.

📄 **[Read the full report at Palo Alto Networks PSIRT →](https://security.paloaltonetworks.com/PAN-SA-2026-0005)**

---

## Check Point

*2 item(s) — 1 critical, 1 watch*

### Management platforms

#### 🔴 CRITICAL — Hackers Compromise 24 Check Point Management Servers in Suspected AI-Driven Attack Campaign

*[The Hacker News](https://thehackernews.com/2026/11/hackers-compromise-24-check-point.html) · 2026-11-25* `AI`

> An unknown actor used AI-driven offensive tooling to compromise 24 Check Point Security Management servers across 22 organisations, per GreyNoise.

**Affected:** Check Point Security Management servers. The report as collected names no CVE, no affected Gaia or management-server version and no fixed release - check the vendor advisory for the fixed release and any mitigation.  

**What happened.** The Hacker News reports that an unknown threat actor has been running a campaign against Check Point Security Management servers using AI-driven offensive tooling, compromising 24 servers and impacting 22 distinct organisations worldwide. GreyNoise tracks the activity as GreyNoise Threat 13 (GT13). Named target sectors include financial services, energy and government.

**Why it matters.** The Security Management server is the policy and credential centre for a Check Point gateway estate: it holds the rule base, administrator accounts, SIC certificates and the ability to push configuration to every gateway it manages. A compromise there is a compromise of the gateways by extension, which outranks any single gateway vulnerability. Twenty-four servers across twenty-two organisations means this is an active campaign with external reconnaissance behind it, and the absence of a named CVE means you cannot patch your way out of the question - you have to check exposure and then check for compromise.

**Recommended actions**

- Determine whether any Security Management server is reachable from the internet or from untrusted internal zones, and remove that reachability immediately - management servers belong on a dedicated management network.
- Check the Check Point advisories for guidance tied to this campaign and for the fixed release or mitigation if one is published; do not assume a version number from the news report.
- Assume credential exposure on any reachable server: rotate administrator credentials, API keys and SIC certificates, re-establish trust with managed gateways, and enforce MFA on the management console with no shared admin accounts.
- Audit the rule base and the management database for unauthorised changes - new or modified admin users, policy changes outside a change record, altered NAT or access rules, new scripts and scheduled jobs - and compare against a known-good configuration backup.
- Verify that policy installations to gateways in the affected period match approved changes, and reinstall a verified policy where they do not.
- Ship management-server logs to the SIEM and alert on administrator login, policy install, object change and software change events.

**Legacy / unpatchable gear.** Management servers on unsupported versions cannot be remediated by upgrade. Isolate them to a management-only segment behind a supported inspecting firewall with inbound access denied from everywhere else, allow administrative access only from a dedicated jump host with full session recording, baseline their NetFlow and alert on any new outbound flow, and set a hard migration date with a budget owner.

**Detection.** Look for administrator logins from unfamiliar source addresses or outside working hours, policy installs with no matching change record, new management API tokens, and outbound connections from the management server to addresses outside your update and licensing ranges. GreyNoise publishes the GT13 tag, so check whether any of your management-server source traffic matches the reported scanning infrastructure.

**AI angle.** The offensive tooling in the campaign is described as AI-driven. Nothing in the report says an AI feature of the Check Point platform was the entry point - the operational lesson is that management-plane exposure is now found and exploited faster, so treat internet reachability of a management server as an incident in itself.

📄 **[Read the full report at The Hacker News →](https://thehackernews.com/2026/11/hackers-compromise-24-check-point.html)**

### Other

#### ⚪ WATCH — Check Point Named a Leader in the 2026 Gartner® Magic Quadrant™ for Email Security

*[Check Point Blog](https://blog.checkpoint.com/security/check-point-named-a-leader-in-the-2026-gartner-magic-quadrant-for-email-security/) · 2026-11-25* `AI` `AI-defense`

> Check Point's email security was named a Gartner Magic Quadrant Leader, with the vendor describing AI-powered protection for users and for the AI assistants reading their mail.

**Affected:** Not a vulnerability. Check Point email and collaboration security, as described in the vendor's own blog post.  

**What happened.** Check Point published a blog post stating it has been named a Leader in the 2026 Gartner Magic Quadrant for Email Security for the third consecutive year. The post describes AI-powered security that helps prevent phishing, ransomware, account takeover, BEC and zero-day attacks, a unified platform covering email, collaboration tools and endpoints, and protection for the AI assistants that now read and act on email.

**Why it matters.** Analyst recognition is not an engineering event, and email security sits outside this digest's device scope. It is recorded here only because the same post documents a vendor defensive position on AI-era threats, which is tracked separately on the AI defences page. Note the contrast with the other Check Point item in this edition: the management-server campaign is the one that needs action today.

**Recommended actions**

- No device action. If you already run Check Point email security, read the post for the described AI protections and decide whether the AI-assistant protection claim is worth a conversation with your account team.
- Keep this separate from the GreyNoise GT13 management-server campaign in this edition, which is the Check Point item that requires work now.

**Legacy / unpatchable gear.** Not applicable - no device fix or end-of-life exposure is involved.

**AI angle.** The post is the vendor's defensive AI position: AI-powered email threat protection, extended to cover AI assistants that read and act on mail. It is a described capability in a marketing post, not a documented launch, and it is captured in the vendor AI defences section.

📄 **[Read the full report at Check Point Blog →](https://blog.checkpoint.com/security/check-point-named-a-leader-in-the-2026-gartner-magic-quadrant-for-email-security/)**

---

## Juniper

*1 item(s) — 1 critical*

### Management platforms

#### 🔴 CRITICAL — CISA KEV addition: Hewlett Packard Enterprise (HPE) Juniper Session Smart Conductor and Session Smart Router - CVE-2026-103406

*[CISA KEV catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?search_api_fulltext=CVE-2026-103406) · 2026-11-25* `KEV`

> CISA added a Juniper Session Smart Conductor and Session Smart Router command-injection flaw to the KEV catalog; a crafted salt minion ID gives a remote attacker root.

**Affected:** Hewlett Packard Enterprise (HPE) Juniper Session Smart Conductor and Session Smart Router, per the CISA KEV entry for CVE-2026-103406. The entry does not state affected or fixed version numbers - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-103406](https://nvd.nist.gov/vuln/detail/CVE-2026-103406) **[KEV]**  

**What happened.** CISA added CVE-2026-103406 to the Known Exploited Vulnerabilities catalog on 2026-11-25 with a remediation due date of 2026-11-28. The vulnerability is an OS command injection in HPE Juniper Session Smart Conductor and Session Smart Router that allows a remote attacker to execute arbitrary commands as root via a specially crafted salt minion ID. CISA's required action is to apply vendor mitigations in accordance with BOD 26-04 and its forensics triage requirements, or to discontinue use of the product where mitigations are unavailable.

**Why it matters.** This is unauthenticated-grade access to the management plane of an SD-WAN fleet. Session Smart Conductor orchestrates Session Smart Routers, so root on the Conductor yields the configuration templates, onboarding secrets and certificates for every router under it - a single compromise scales to the whole WAN edge. The three-day federal due date and the KEV listing mean exploitation is confirmed, not theoretical. The injection arrives through the Salt management channel, so the exposure question is whether that channel is reachable from anywhere untrusted.

**Recommended actions**

- Identify all Session Smart Conductor and Session Smart Router instances from inventory with their running versions, and patch or mitigate the Conductor before the routers - the controller is the higher-value target.
- Check the vendor advisory for the fixed release and the documented mitigation for CVE-2026-103406; do not act on a version number from a news summary.
- Restrict the Conductor's management interface and Salt channel to the management network only, with an explicit ACL, and remove any internet exposure now even if the upgrade has to wait for a window.
- After patching, rotate local admin credentials, API keys, device certificates and router onboarding secrets - root access obtained before the upgrade survives it.
- Audit the Conductor's templates and pushed configuration for insecure overrides, unexpected admin users, new static routes or added tunnels before you trust the fleet again.
- Enforce MFA on Conductor administrative access and ship its logs to the SIEM with alerting on admin login, config change and software change events.

**Legacy / unpatchable gear.** Where a Conductor or router build has no available fix, CISA's own instruction is to discontinue use. Short of that, isolate the device behind a supported inspecting firewall, deny all inbound access from untrusted zones, permit management only from a dedicated jump host with full session logging, monitor it as a high-risk asset with a NetFlow baseline, and set a replacement date with a named budget owner.

**Detection.** Hunt the Salt master and minion logs on the Conductor for minion IDs containing shell metacharacters or unusually long identifiers, and for minion authentication from unknown hosts. Alert on root process execution spawned by the Salt service, new local accounts, modified salt state files, and any new outbound connection from a Conductor or Session Smart Router to an address outside your management ranges.

📄 **[Read the full report at CISA KEV catalog →](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?search_api_fulltext=CVE-2026-103406)**

---

## Cisco

*3 item(s) — 3 high*

### Other

#### 🟠 HIGH — Vulnerabilities in Node.js and Brotli Affecting Cisco Products: October 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-oct2026-node-js-Jj3SQ19u) · 2026-11-24* `KEV` `vendor-advisory`

> Cisco's October advisory covers four Node.js flaws in its products, including the KEV-listed Brotli decoder out-of-bounds write that can lead to remote code execution.

**Affected:** Cisco products that embed Node.js release lines 22.x, 24.x and 26.x. The advisory does not enumerate the affected products in the collected text - consult the Cisco advisory for the product list and fixed releases.  
**CVEs:** [CVE-2026-105482](https://nvd.nist.gov/vuln/detail/CVE-2026-105482), [CVE-2026-105483](https://nvd.nist.gov/vuln/detail/CVE-2026-105483), [CVE-2026-106069](https://nvd.nist.gov/vuln/detail/CVE-2026-106069), [CVE-2026-111497](https://nvd.nist.gov/vuln/detail/CVE-2026-111497) **[KEV]**  

**What happened.** On 2026-10-21 Node.js published fixes for multiple vulnerabilities, and Cisco opened an advisory tracking which of its products are affected. Four are rated Security Impact Rating High: CVE-2026-111497 (Brotli decoder out-of-bounds write, potential remote code execution), CVE-2026-105482 (DNS resolution truncation involving embedded NUL characters), CVE-2026-105483 (TLS hostname validation bypass using fullwidth IP digits) and CVE-2026-106069 (TLS hostname validation bypass with noncanonical IP addresses). Cisco states the advisory will be updated as more information becomes available.

**Why it matters.** CVE-2026-111497 is on the CISA KEV catalog (added 2026-10-27, due 2026-11-10), so it is confirmed exploited somewhere in the wild, and the Brotli decoder is reachable wherever a device accepts compressed HTTP content. The two TLS hostname validation bypasses are quieter but matter on network devices that make outbound TLS calls for updates, telemetry or cloud management: a bypass there means an interception that does not break the trust chain. The practical problem today is scoping - the product list is still moving, so you need the advisory, not an assumption.

**Recommended actions**

- Pull the Cisco advisory and build the affected-product list against your inventory, filtered by model and running version; re-check it, because Cisco says it will be updated.
- Where a product is listed and a fixed release is published, schedule the upgrade; where it is not, check the vendor advisory for the fixed release rather than assuming the Node.js version number maps to a Cisco build.
- Prioritise any affected device that terminates or proxies untrusted HTTP, since the Brotli flaw is KEV-listed and reachable from compressed request or response bodies.
- Restrict management access (HTTPS/SSH/API) on affected devices to a dedicated management VLAN or out-of-band network while you wait for fixed builds.
- Record the decision (patched / mitigated / accepted risk) per device group in the change record, and track CVE-2026-111497 against the KEV due date of 2026-11-10 that has already passed.

**Legacy / unpatchable gear.** Cisco products past end of software maintenance will not receive a Node.js rebuild. Put them behind a supported inspecting firewall, deny inbound access from untrusted zones, remove direct management reachability in favour of a jump host with full session logging, and set a replacement date with a budget owner.

**Detection.** Alert on device syslog for TLS validation failures and unexpected certificate changes on outbound management and telemetry connections, and on crashes or restarts of the HTTP service on affected devices - an out-of-bounds write in a decoder usually shows up as a process restart before it shows up as a compromise.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-oct2026-node-js-Jj3SQ19u)**

#### 🟠 HIGH — Vulnerabilities in nginx Affecting Cisco Products: November 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nginx-nov-2026-e8jska2s) · 2026-11-24* `vendor-advisory`

> Cisco is assessing which of its products ship the nginx releases hit by five critical RCE flaws, fixed upstream in nginx 1.31.9 mainline and 1.30.7 stable.

**Affected:** Cisco products embedding nginx releases 1.9.5 through 1.31.8, per the Cisco advisory. nginx itself is fixed in 1.31.9 (mainline) and 1.30.7 (stable); the Cisco product list and the per-product fixed releases are not in the collected text - check the vendor advisory.  
**CVEs:** [CVE-2026-113425](https://nvd.nist.gov/vuln/detail/CVE-2026-113425), [CVE-2026-121810](https://nvd.nist.gov/vuln/detail/CVE-2026-121810), [CVE-2026-123007](https://nvd.nist.gov/vuln/detail/CVE-2026-123007), [CVE-2026-123633](https://nvd.nist.gov/vuln/detail/CVE-2026-123633), [CVE-2026-123634](https://nvd.nist.gov/vuln/detail/CVE-2026-123634)  

**What happened.** On 2026-11-18 nginx published fixes for multiple critical vulnerabilities that can lead to remote code execution, affecting releases 1.9.5 through 1.31.8. The CVEs are CVE-2026-113425 (ngx_mail_proxy_module), CVE-2026-121810 (ngx_http_v3_module), CVE-2026-123007 and CVE-2026-123633 (ngx_http_v2_module) and CVE-2026-123634 (resolver). Cisco opened an advisory to track the impact on its products and says it will be updated as information becomes available.

**Why it matters.** nginx is embedded in a large amount of network and security appliance software as the front-end web stack, which is exactly the component that faces untrusted traffic. The affected range starts at 1.9.5, so this is a decade of builds rather than a recent regression, and the HTTP/2 and HTTP/3 module flaws are reachable on any listener that negotiates those protocols. Palo Alto Networks has opened its own critical-severity portfolio status bulletin for two of the same CVEs, which is a good indication of how broadly this will land across vendors.

**Recommended actions**

- Track the Cisco advisory for the affected-product list and per-product fixed releases; it is explicitly still being updated, so re-check rather than concluding you are unaffected today.
- Where a Cisco product is confirmed affected, apply the Cisco fixed release - not an upstream nginx version - and verify the running version after upgrade.
- Reduce exposure in the meantime: remove administrative and management web interfaces from untrusted interfaces and put management ACLs on the control plane.
- Where an affected listener does not need HTTP/2 or HTTP/3, consider disabling those protocols as an interim measure if the product supports it, and validate the change in a lab first.
- Check the same CVEs against every other appliance vendor in your estate, not just Cisco - the upstream range is wide enough that most perimeter vendors will publish something.

**Legacy / unpatchable gear.** Unsupported appliances will keep the vulnerable nginx build permanently. Front them with a supported reverse proxy or firewall that terminates untrusted HTTP, deny inbound access from untrusted zones, restrict management to a jump host, and plan replacement with a documented accepted risk until then.

**Detection.** Watch for repeated worker-process restarts or segfaults in the appliance web stack, malformed HTTP/2 and HTTP/3 frame errors in logs, and any new process or outbound connection parented by the web service. Alert on DNS resolution anomalies from appliances, given the resolver CVE.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nginx-nov-2026-e8jska2s)**

#### 🟠 HIGH — Vulnerability in PHP Affecting Cisco Products: November 2026

*[Cisco PSIRT](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-phpaftershell-nov-2026-kT9pL4Xe) · 2026-11-24* `vendor-advisory`

> A critical PHP remote code execution flaw dubbed phpAfterShell (CVE-2026-126714) is being assessed for impact across Cisco products.

**Affected:** Cisco products embedding PHP, per the Cisco advisory, which carries a Security Impact Rating of Critical. The affected product list and fixed releases are not in the collected text - check the vendor advisory for the fixed release.  
**CVEs:** [CVE-2026-126714](https://nvd.nist.gov/vuln/detail/CVE-2026-126714)  

**What happened.** On 2026-11-19 the PHP Foundation released security fixes for a critical vulnerability that can lead to remote code execution, tracked as CVE-2026-126714 and referenced as phpAfterShell with details in GHSA-fm2m-fjf5-v3cp. Cisco published an advisory with a Critical Security Impact Rating to track which of its products are affected, and says it will be updated as further information becomes available.

**Why it matters.** PHP shows up in the administrative web interfaces of network and security appliances, where it runs with high privilege and often on the same management plane you log into. A critical RCE in that layer is an appliance takeover, and the name is already circulating publicly, which shortens the gap between advisory and scanning. In this same window a China-linked group was observed chaining a tool called PHP2Shell against Palo Alto Networks firewalls, so PHP-layer footholds on network devices are not hypothetical right now.

**Recommended actions**

- Monitor the Cisco advisory for the affected-product list and fixed builds, and map it against inventory by model and running version as entries appear.
- Apply the Cisco fixed release once published; check the vendor advisory for the fixed release rather than inferring it from the PHP release number.
- Pull management web interfaces off untrusted and internet-facing interfaces now - this class of flaw is only reachable if the admin interface is reachable.
- Enforce MFA on all administrative accounts on appliances with PHP-based management consoles, and remove shared local admin accounts.
- Review appliance configuration and web root for unexpected admin users, scripts or modified login pages, since a web-layer RCE is usually followed by a dropped file.

**Legacy / unpatchable gear.** End-of-life appliances with a PHP management interface should have that interface unreachable from anything but a dedicated jump host, with strict ACLs and full session logging, behind a supported inspecting firewall. Set a replacement date and document the accepted risk.

**Detection.** Alert on new or modified files under the appliance web root, on the web server process spawning a shell or interpreter, and on outbound connections initiated by the management web service. Review admin interface access logs for requests from addresses outside the management network.

📄 **[Read the full report at Cisco PSIRT →](https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-phpaftershell-nov-2026-kT9pL4Xe)**

---

## Unspecified

*1 item(s) — 1 medium*

### VPN gateways

#### 🟡 MEDIUM — Webworm APT Slithers Into Networks With Customized SoftEther VPN

*[Dark Reading](https://www.darkreading.com/perimeter/webworm-slithers-customized-softether-vpn) · 2026-11-24* 

> The China-aligned Webworm group is deploying a recompiled SoftEther VPN build stripped of detection artifacts to tunnel into victim networks.

**Affected:** No vendor appliance vulnerability. The subject is the open-source SoftEther VPN software, rebuilt from source by the attacker and deployed inside victim networks as tunnelling infrastructure.  

**What happened.** Dark Reading reports that the China-aligned espionage group Webworm has rebuilt SoftEther VPN from source, removing the artifacts that would normally let defenders identify it, and is using the customised build to establish access into target networks.

**Why it matters.** This is an attacker-deployed VPN concentrator inside your perimeter, not a flaw in your gateway - and it defeats the common detection approach of matching known SoftEther binaries or strings. Once established, it is an encrypted tunnel that egresses as ordinary outbound traffic and bypasses whatever controls your sanctioned remote-access path enforces, including MFA and conditional access. For network teams this is an egress-control and anomaly-detection problem rather than a patching one.

**Recommended actions**

- Enforce egress filtering so that only authorised hosts can open outbound tunnels: default-deny outbound from servers, and allow VPN and tunnelling protocols only from the devices that are supposed to use them.
- Block unsanctioned VPN and tunnelling protocols at the perimeter firewall, including SoftEther's HTTPS, UDP and ICMP/DNS transport modes, and alert rather than silently drop so you see the attempts.
- Audit for rogue tunnel endpoints: unexpected listening services on servers, unauthorised port forwards and NAT entries on firewalls and routers, and new site-to-site or client tunnels in device configuration.
- Make sure all remote access goes through the sanctioned gateway with phishing-resistant MFA and geo/ASN conditional access, so an alternative tunnel stands out by definition.
- Keep a NetFlow baseline per server segment and alert on long-lived, high-volume encrypted sessions to new external destinations.

**Legacy / unpatchable gear.** Legacy segments that cannot run modern endpoint controls should be constrained at the network layer instead: default-deny egress at the nearest supported firewall, no direct internet route, DNS forced through an inspecting resolver, and NetFlow monitoring for any new outbound flow.

**Detection.** Do not rely on binary or string signatures for SoftEther - the build was specifically stripped. Hunt behaviour instead: persistent outbound TLS sessions from servers that have no business initiating them, tunnel-shaped traffic volumes, unexpected virtual network adapters on hosts, and new services configured for automatic start. Review firewall logs for long-duration sessions to hosting-provider ranges.

📄 **[Read the full report at Dark Reading →](https://www.darkreading.com/perimeter/webworm-slithers-customized-softether-vpn)**

---

## 🛡️ Vendor AI defences introduced

AI capabilities the vendors shipped this cycle - the answer side of the AI story.

### Check Point

**AI-powered email threat protection on Check Point's unified email, collaboration and endpoint platform** — Check Point describes AI-powered security that helps prevent phishing, ransomware, account takeover, business email compromise and zero-day attacks, delivered on one platform covering email, collaboration tools and endpoints. The post states the protection extends to the AI assistants that now read and act on email, not only to employees.

- *Defends against:* AI-era email threats - AI-generated phishing and BEC, and abuse of AI assistants that read and act on mailbox content on a user's behalf.
- *Availability:* described-only - the post is analyst-recognition and capability description, with no launch, release or availability date stated.
- *Worth evaluating if:* Teams already running Check Point email security, or anyone about to let an AI assistant read and act on corporate mail: the assistant becomes a new path into the mailbox, and this is one of the few vendor statements that addresses protecting it. Network teams can skip it - it touches no device in this digest's scope.
- 📄 [Read the announcement at Check Point Blog →](https://blog.checkpoint.com/security/check-point-named-a-leader-in-the-2026-gartner-magic-quadrant-for-email-security/)

---

## 🧠 AI & network devices — AI-driven offensive tooling turns up in two separate campaigns against network gear

Two items in this window are AI-assisted attacks on network infrastructure rather than AI product news. A China-linked group chained PHP2Shell with the Fragnesia flaw in what SecurityWeek describes as AI-assisted attacks to gain root on Palo Alto Networks firewalls, hitting 25 servers. Separately, GreyNoise tracked an unknown actor using AI-driven offensive tooling against Check Point Security Management servers, compromising 24 servers across 22 organisations in financial services, energy and government.

What matters operationally is not the novelty of the tooling but the tempo. Both campaigns targeted management and perimeter devices, chained more than one weakness, and moved faster than a monthly patch cycle. Neither report, as collected, attributes the vulnerabilities themselves to AI discovery, and no CVE is given for Fragnesia - so plan against the speed, not against a specific AI capability.

**Vendors with an AI angle in this edition:** Check Point, Palo Alto Networks

**Preventing it on current systems**

- Shorten the patch window for internet-facing firewalls, VPN gateways and management platforms to days: exploit chaining against these two product families moved inside a normal monthly cycle.
- Take firewall and security-management consoles off the public internet entirely and front them with a jump host, management ACLs and MFA - both campaigns hit management-plane and perimeter surfaces.
- Assume chaining: patch the auxiliary components on a security appliance (web stack, PHP, scripting runtimes, management agents) at the same priority as the appliance OS, because one of them was the first hop here.
- Baseline NetFlow and management-plane syslog and alert on deviation - new admin logins, config pushes outside change windows, new outbound flows from an appliance - rather than relying on signature updates to keep pace.
- Treat any AI/LLM or AIOps integration on network gear as a privileged admin path: scoped read-only credentials, no unattended config write, full audit trail.

**Preventing it on legacy / end-of-life systems**

- End-of-life firewalls and management servers that will not receive a fix must come off any untrusted path: isolate behind a supported inspecting firewall and deny inbound access from untrusted zones.
- Where the device cannot be replaced soon, give it compensating controls - strict management ACLs, dedicated jump host, no direct administrative reachability, full session logging.
- Monitor unpatchable appliances as high-risk assets with a NetFlow baseline and alerting on any new outbound flow, since you cannot close the underlying hole.
- Set a replacement date and a budget owner, and document the accepted risk until the device is gone.

---

<!--index
{
 "url": "/2026/11/25/network-security-digest/",
 "date": "2026-11-25",
 "entries": [
  {
   "title": "CISA KEV addition: Hewlett Packard Enterprise (HPE) Juniper Session Smart Conductor and Session Smart Router - CVE-2026-103406",
   "link": "https://www.cisa.gov/known-exploited-vulnerabilities-catalog?search_api_fulltext=CVE-2026-103406",
   "source": "CISA KEV catalog",
   "relevance": "critical",
   "device_types": [
    "management-platform",
    "router",
    "sdwan"
   ],
   "vendors": [
    "Juniper"
   ],
   "cves": [
    "CVE-2026-103406"
   ],
   "kev": true,
   "ai": false,
   "ai_angle": "",
   "summary": "CISA added a Juniper Session Smart Conductor and Session Smart Router command-injection flaw to the KEV catalog; a crafted salt minion ID gives a remote attacker root.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Palo Alto Networks Patches Fragnesia Flaw Used to Root Firewalls",
   "link": "https://www.securityweek.com/palo-alto-networks-patches-fragnesia-flaw-used-to-root-firewalls/",
   "source": "SecurityWeek",
   "relevance": "critical",
   "device_types": [
    "firewall"
   ],
   "vendors": [
    "Palo Alto Networks"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "The report characterises the campaign as AI-assisted. No AI capability on the firewall was involved and the vulnerabilities are not attributed to AI discovery in the collected text - the defensive implication is tempo, so shorten the patch window for internet-facing firewalls rather than buying a control for an AI threat.",
   "summary": "A China-linked group chained PHP2Shell with the Fragnesia flaw in AI-assisted attacks to gain root on Palo Alto Networks firewalls, hitting 25 servers; patches are out.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Hackers Compromise 24 Check Point Management Servers in Suspected AI-Driven Attack Campaign",
   "link": "https://thehackernews.com/2026/11/hackers-compromise-24-check-point.html",
   "source": "The Hacker News",
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
   "ai_angle": "The offensive tooling in the campaign is described as AI-driven. Nothing in the report says an AI feature of the Check Point platform was the entry point - the operational lesson is that management-plane exposure is now found and exploited faster, so treat internet reachability of a management server as an incident in itself.",
   "summary": "An unknown actor used AI-driven offensive tooling to compromise 24 Check Point Security Management servers across 22 organisations, per GreyNoise.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Vulnerabilities in Node.js and Brotli Affecting Cisco Products: October 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-oct2026-node-js-Jj3SQ19u",
   "source": "Cisco PSIRT",
   "relevance": "high",
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
   "summary": "Cisco's October advisory covers four Node.js flaws in its products, including the KEV-listed Brotli decoder out-of-bounds write that can lead to remote code execution.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Vulnerabilities in nginx Affecting Cisco Products: November 2026",
   "link": "https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-nginx-nov-2026-e8jska2s",
   "source": "Cisco PSIRT",
   "relevance": "high",
   "device_types": [
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
   "summary": "Cisco is assessing which of its products ship the nginx releases hit by five critical RCE flaws, fixed upstream in nginx 1.31.9 mainline and 1.30.7 stable.",
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
   "summary": "A critical PHP remote code execution flaw dubbed phpAfterShell (CVE-2026-126714) is being assessed for impact across Cisco products.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "CVE-2026-123633 Portfolio Status: Impact of NGINX Vulnerabilities (CVE-2026-123633, CVE-2026-123007) on Palo Alto Networks Products (Severity: CRITICAL)",
   "link": "https://security.paloaltonetworks.com/CVE-2026-123633",
   "source": "Palo Alto Networks PSIRT",
   "relevance": "high",
   "device_types": [
    "firewall"
   ],
   "vendors": [
    "Palo Alto Networks"
   ],
   "cves": [
    "CVE-2026-123007",
    "CVE-2026-123633"
   ],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Palo Alto Networks opened a critical-severity portfolio-status bulletin for the nginx flaws CVE-2026-123633 and CVE-2026-123007 affecting its products.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Webworm APT Slithers Into Networks With Customized SoftEther VPN",
   "link": "https://www.darkreading.com/perimeter/webworm-slithers-customized-softether-vpn",
   "source": "Dark Reading",
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
   "summary": "The China-aligned Webworm group is deploying a recompiled SoftEther VPN build stripped of detection artifacts to tunnel into victim networks.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "PAN-SA-2026-0015 Security Bulletin: OSS CVEs Fixed in PAN-OS (Severity: INFORMATIONAL)",
   "link": "https://security.paloaltonetworks.com/PAN-SA-2026-0015",
   "source": "Palo Alto Networks PSIRT",
   "relevance": "watch",
   "device_types": [
    "firewall"
   ],
   "vendors": [
    "Palo Alto Networks"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "Palo Alto Networks published an informational bulletin on open-source CVEs fixed in PAN-OS, with no exploitable impact claimed.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "PAN-SA-2026-0005 Informational Bulletin: Precautionary Fixes for Non-Exploitable OSS CVEs in PAN-OS (Severity: INFORMATIONAL)",
   "link": "https://security.paloaltonetworks.com/PAN-SA-2026-0005",
   "source": "Palo Alto Networks PSIRT",
   "relevance": "watch",
   "device_types": [
    "firewall"
   ],
   "vendors": [
    "Palo Alto Networks"
   ],
   "cves": [],
   "kev": false,
   "ai": false,
   "ai_angle": "",
   "summary": "A second Palo Alto Networks informational bulletin covers precautionary PAN-OS fixes for open-source CVEs the vendor assesses as non-exploitable.",
   "ai_defense": false,
   "defense": null
  },
  {
   "title": "Check Point Named a Leader in the 2026 Gartner\u00ae Magic Quadrant\u2122 for Email Security",
   "link": "https://blog.checkpoint.com/security/check-point-named-a-leader-in-the-2026-gartner-magic-quadrant-for-email-security/",
   "source": "Check Point Blog",
   "relevance": "watch",
   "device_types": [
    "other"
   ],
   "vendors": [
    "Check Point"
   ],
   "cves": [],
   "kev": false,
   "ai": true,
   "ai_angle": "The post is the vendor's defensive AI position: AI-powered email threat protection, extended to cover AI assistants that read and act on mail. It is a described capability in a marketing post, not a documented launch, and it is captured in the vendor AI defences section.",
   "summary": "Check Point's email security was named a Gartner Magic Quadrant Leader, with the vendor describing AI-powered protection for users and for the AI assistants reading their mail.",
   "ai_defense": true,
   "defense": {
    "solution": "AI-powered email threat protection on Check Point's unified email, collaboration and endpoint platform",
    "what_it_does": "Check Point describes AI-powered security that helps prevent phishing, ransomware, account takeover, business email compromise and zero-day attacks, delivered on one platform covering email, collaboration tools and endpoints. The post states the protection extends to the AI assistants that now read and act on email, not only to employees.",
    "defends_against": "AI-era email threats - AI-generated phishing and BEC, and abuse of AI assistants that read and act on mailbox content on a user's behalf.",
    "availability": "described-only - the post is analyst-recognition and capability description, with no launch, release or availability date stated."
   }
  }
 ]
}
-->

## How this was produced

- Feeds polled: 23 ok, 0 failed
- Raw items: 739 → in window: 66 → network-device relevant: 14 → published: 11
- Enrichment: CISA KEV, FIRST EPSS
- Analysis: `claude-code-action`

_Automated digest. Verify every version number against the vendor advisory before you schedule a change._
