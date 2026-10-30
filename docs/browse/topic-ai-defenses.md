---
layout: default
title: "Vendor AI defences"
permalink: /browse/ai-defenses/
---

# Vendor AI defences

*6 vendor AI capability announcement(s), all editions.*

What the network vendors have shipped to answer AI-era threats: AI security products and features, AI-assisted detection, guardrails for AI running on network gear, and vendor guidance for defending against AI-enabled attacks.

## Check Point (4)

### Maestro Hyperscale Orchestrator (MHO) 200

Check Point's most powerful Maestro orchestrator, with new 400GbE ports and four times the capacity of its predecessor, intended to secure AI traffic in the data center without slowing it down.

*Defends against:* Loss of inspection coverage as AI training, inference and agentic workloads drive up perimeter and east-west traffic, much of it encrypted and all of it needing inspection.  
*Availability:* Announced in this post as being introduced; no general-availability date stated.

- 🟡 [Secure AI-Scale Traffic with the Maestro MHO 200](https://blog.checkpoint.com/security/secure-ai-traffic-maestro-mho-200/)  
  <small>2026-10-30 · Check Point Blog · [in digest]({{ '/2026/10/30/network-security-digest/' | relative_url }})</small>

### audio guardrails

Screens the sound of a voice interaction itself rather than a derived transcript, because an attacker can speak an instruction and tamper with the recording so the transcript is garbled or empty while the model still receives the original audio.

*Defends against:* Audio prompt injection against voice and multimodal agents, including transcript-blinding that defeats text-layer guardrails.  
*Availability:* described-only

- ⚪ [Audio Prompt Injection: What Voice Agents Hear That Text Guardrails Miss](https://blog.checkpoint.com/ai-security/audio-guardrails-voice-agents/)  
  <small>2026-10-30 · Check Point Blog · [in digest]({{ '/2026/10/30/network-security-digest/' | relative_url }})</small>

- ⚪ [AI Models Broke Their Own Containment: Key Findings from the July-August 2026 AI Threat Landscape](https://blog.checkpoint.com/artificial-intelligence/ai-models-broke-their-own-containment-key-findings-from-the-july-august-2026-ai-threat-landscape/)  
  <small>2026-09-18 · Check Point Blog · [in digest]({{ '/2026/09/18/network-security-digest/' | relative_url }})</small>

### Agentic network security management

Check Point describes agentic AI taking on day-to-day security operations work in hybrid mesh environments, setting out four ways agentic network security management improves security operations where environments change faster than teams can manage them manually. The post describes the approach; it does not state a product name, release or availability date.

*Defends against:* Operational drift and slow response in hybrid networks changing faster than human review can keep pace with, which Check Point frames as worsening as organisations adopt AI.  
*Availability:* described-only

- ⚪ [When Security Operations Can’t Keep Up: 4 Ways Agentic Network Security Management Improves Security Operations](https://blog.checkpoint.com/hybrid-mesh/when-security-operations-cant-keep-up-4-ways-agentic-network-security-management-improves-security-operations/)  
  <small>2026-09-18 · Check Point Blog · [in digest]({{ '/2026/09/18/network-security-digest/' | relative_url }})</small>

## Cisco (1)

### Project Quarry

An open-sourced rule language for detecting AI agent attacks that only emerge across sequences of individually benign actions.

*Defends against:* Multi-step AI agent attacks that evade per-action policy checks because each step is permitted on its own.  
*Availability:* Open sourced, per the source; no release date or version stated.

- ⚪ [Cisco Open Sources Quarry to Catch AI Agent Attacks Hidden in Harmless Steps](https://www.securityweek.com/cisco-open-sources-quarry-to-catch-ai-agent-attacks-hidden-in-harmless-steps/)  
  <small>2026-10-30 · SecurityWeek · [in digest]({{ '/2026/10/30/network-security-digest/' | relative_url }})</small>

## Palo Alto Networks (1)

### AI Application Security

Palo Alto Networks describes its work on AI security and states it was named a Market Shaper in the 2026 September Gartner Emerging Market Quadrant for AI Application Security - Established Vendors. The collected text names no specific product, feature or capability detail.

*Defends against:* AI application security risk generally; the source does not specify which threats the offering addresses.  
*Availability:* not stated

- ⚪ [Defining the Standard for AI Security](https://www.paloaltonetworks.com/blog/2026/09/defining-the-standard-for-ai-security/)  
  <small>2026-09-18 · Palo Alto Networks Blog · [in digest]({{ '/2026/09/18/network-security-digest/' | relative_url }})</small>


<p><a href="{{ "/" | relative_url }}">← All editions</a> · <a href="{{ "/browse/" | relative_url }}">Browse index</a></p>
