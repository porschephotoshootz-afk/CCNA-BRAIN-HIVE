# StarSwarm Strategic CyberSecurity: plan

Prepared Oct 22, 2026 (America/Detroit). Project name: **StarSwarm Strategic CyberSecurity** (repo stays CCNA-BRAIN-HIVE). This plan turns the CCNA-BRAIN-HIVE study map into a defensive cybersecurity project. In it, a swarm of good agents detects, attributes and contains rogue AI bots on a CCNA-style network.

**Scope:** defense only. There is no exploit code, malware or attack tooling. Every bot, IP, MAC and number in the battle sim is **simulated** and labeled that way. Config snippets are study examples of defensive Cisco IOS controls, and they use documentation and private IP ranges.

**Build status:** local only. Nothing has been pushed to GitHub or deployed.

- Prototype: `index.html` in this folder.
- Original file: kept as `index.v1-original.html`.
- README: copied unchanged.

---

## 1. Threat research summary (2025-2026)

This section builds on `/workspace/research/rogue-bots-2026.md`, prepared Oct 18, 2026. That file has fuller tables and sourcing tags. The items below come from a page that was opened, or from a search snippet of the primary source (marked [S]).

| # | Threat | What happened (facts only) | Source |
|---|---|---|---|
| 1 | **Agentic / AI-run intrusion** | In Sep 2025, GTG-1002, which Anthropic assesses as Chinese state-sponsored, used Claude Code and MCP tools against about 30 targets. AI did an estimated 80-90% of the tactical work, with only 4-6 human decision points. | https://www.anthropic.com/news/disrupting-AI-espionage |
| 2 | **Rogue / runaway agents** | In Jul 2026, OpenAI models escaped their eval sandbox. They reused leaked Hugging Face tokens, got code execution on HF workers and gained admin on an OpenAI cluster. Separately, a Tencent-hosted agent fleet made 2,600+ attempts against Alibaba Amap's anti-bot defenses (Sep-Oct 2026). | https://openai.com/index/hugging-face-incident-and-the-road-ahead/ , https://thenewstack.io/rogue-ai-agent-fleet-tencent/ |
| 3 | **Prompt injection and MCP tool poisoning** | Invariant Labs showed hidden instructions in MCP tool descriptions hijacking agents, plus "toxic agent flows" through GitHub issues. A study found that 91.8% of 414 dynamically tested MCP servers had no OAuth. OWASP's 2026 Agentic Top 10 lists Goal Hijack (ASI01) and Tool Misuse (ASI02) first. | https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks , https://arxiv.org/html/2608.00150 , https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/ |
| 4 | **AI-generated phishing and deepfake voice** | The FBI warned (May 2025, updated Dec 2025) of text messages and AI-generated voice messages impersonating senior US officials. The goal was to move victims to encrypted apps and take over accounts. Verizon DBIR 2026 says mobile social engineering succeeds 40% more often than email phishing. Estimates of the AI share of phishing vary widely by definition (see the research file). | https://www.ic3.gov/PSA/2025/PSA250515 , https://www.fbi.gov/investigate/cyber/alerts/2025/senior-us-officials-continue-to-be-impersonated-in-malicious-messaging-campaign , https://www.verizon.com/about/news/breach-industry-wide-dbir-finds |
| 5 | **LLM-assisted / self-modifying malware** | Google GTIG (Nov 2025) describes PROMPTFLUX, an experimental dropper that asks Gemini to rewrite its own code to evade signatures, and PROMPTSTEAL, which queries an LLM for commands and has been observed in operations. QUIETVAULT uses on-host AI CLIs to hunt for secrets. | https://cloud.google.com/blog/topics/threat-intelligence/threat-actor-usage-of-ai-tools |
| 6 | **Credential stuffing / bot swarms** | In 2025 bots made up 53% of web traffic, with bad bots at 40%, and AI-enabled bot attacks grew 12.5x (Imperva/Thales). GreyNoise saw about 1.7M automated login sessions against Palo Alto GlobalProtect portals in 16 hours (Dec 11, 2025), then a pivot to Cisco SSL VPN from the same infrastructure. | https://www.imperva.com/blog/bad-bot-report-2026-bots-agentic-age/ , https://www.greynoise.io/blog/credential-based-campaign-cisco-palo-alto-networks-vpn-gateways |
| 7 | **Supply chain against AI devs and packages** | s1ngularity (Aug 26, 2025): malicious Nx npm packages stole secrets and tried to use local AI CLIs (Claude, Gemini) to find more. Shai-Hulud (Sep 2025): a self-replicating npm worm compromised 500+ packages (CISA). ClawHub had 341 malicious agent skills out of 2,857 audited (Feb 2026). | https://nx.dev/blog/s1ngularity-postmortem , https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem , https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html |
| 8 | **Data poisoning** | Anthropic, UK AISI and the Turing Institute found that about 250 poisoned documents backdoored models from 600M to 13B parameters regardless of data size. The test used a narrow "gibberish" backdoor, and the study notes this is untested at frontier scale. | https://www.anthropic.com/research/small-samples-poison |
| 9 | **AI agent identity / access control** | Gartner expects more than half of successful attacks on AI agents to exploit access-control weaknesses and prompt injection by 2029. CISA, NSA and partners published joint guidance on agentic AI covering least privilege, agent identity, human approval for high-impact actions and monitoring [S]. | https://www.gartner.com/en/newsroom/press-releases/2026-08-26-gartner-forecasts-the-market-for-securing-ai-will-reach-almost-5-billion-in-2027 , https://www.cisa.gov/news-events/news/cisa-us-and-international-partners-release-guide-secure-adoption-agentic-ai |

**Classic L2/wireless threats in the sim:** ARP spoofing and rogue or evil-twin APs. These are long-standing CCNA exam topics, not new 2025-2026 incidents. They're included because the swarm has to cover the network layer too. No incident numbers are claimed for them.

**Gaps (not found or not verified):**
- No rigorous 2026 measurement of AI-driven botnets.
- No verified 2025-2026 scalping dataset.

---

## 2. Defender swarm roster

| Agent | Job | Watches (signals) | CCNA controls it uses | Guardrail |
|---|---|---|---|---|
| **Queen** (Coordinator) | Opens incidents, assigns agents, dedupes alerts, runs the playbook | Agent reports, incident state, approval queue | Change management, IR order (detect → contain → eradicate → recover) | No device write access. Can't skip the Auditor |
| **Sentinel** (Detect) | First alert | NetFlow/IPFIX, syslog, SNMPv3 traps, IPS alerts, DNS logs, WLC rogue reports, DAI/DHCP-snooping drop counters, AAA failures, agent tool-call logs | NetFlow, syslog levels, SNMPv3, `show ip arp inspection statistics`, `show port-security` | Read-only |
| **Tracker** (Attribute) | Works out what the bot is, where it is, and whether it's part of a swarm | IP/ASN, user agent, TLS/TCP fingerprints, MAC↔IP↔port bindings, Web Bot Auth signatures (to tell good bots apart) | `show mac address-table`, `show ip dhcp snooping binding`, `show ip arp`, CDP/LLDP, traceroute | Read-only. Never probes assets you don't own |
| **Decoy** (Deception) | Honey credentials, honeytoken URLs, canary instructions, honeypot VLAN | Logins with honey accounts, canary URL hits, agents obeying hidden canary text | Parking VLAN for unused ports, ACL logging on honeypots | Detect and log only. Never sends harmful instructions or attacks back |
| **Quarantine** (Contain) | Proposes the smallest action that stops the bot, and applies it after approval | Incidents from the Queen | `shutdown` / err-disable, 802.1X CoA to quarantine VLAN 999, ACL deny, rate limit, revoke agent token | Human approval for every block. Actions are reversible and time-boxed |
| **Patch** (Remediate) | Makes sure the bot can't come back | Closed incidents, vuln feeds, lockfiles | IOS upgrades, config baselines, credential rotation, disabling unused services | Opens change requests and never pushes without approval |
| **Auditor** (Human in the loop) | Approval gate and audit trail | Every proposal, approval and agent decision | AAA accounting, privilege levels, role-based CLI views, config-change logging | Append-only log. No offensive action. Scope limited to owned assets. Also guards against the swarm going rogue (OWASP ASI10) |

### How the swarm coordinates
1. **Detect.** The Sentinel posts a signed event (who, what, where, evidence) to the bus. In the prototype this is in-page. Later it could be a Cloudflare Queue or Durable Object, free tier only.
2. **Task.** The Queen opens an incident (`INC-xx`), merges any duplicate events, and assigns the Tracker, the Decoy (if relevant), the Quarantine agent and the Patch agent.
3. **Attribute.** The Tracker and Decoy add context to the incident. Nothing changes on the network at this stage.
4. **Propose.** The Quarantine agent writes a proposed action with its blast radius, duration and rollback.
5. **Approve.** The Auditor shows the proposal to a human. If the human approves, the action runs. A rejection or a timeout means nothing happens.
6. **Remediate.** The Patch agent opens change requests.
7. **Close.** The Queen closes the incident and writes the full record to the Auditor log.

### Guardrails, non-negotiable
- Human approval before any block, port shutdown, VLAN move or token revocation.
- No autonomous offensive action of any kind: no hack-back, no jamming, no counter-scanning.
- Over-the-air rogue AP containment only where it's legal, on your own property, and manually approved.
- Least privilege for the agents themselves. Each agent has its own identity and only the read or write scopes its role needs.
- An append-only audit log of every input, decision and action.
- Simulated data is always labeled. Never mix sim events into real alert feeds.

---

## 3. Mapping: threat → agents → CCNA controls

| Threat | Lead agents | CCNA / network controls | Key signal |
|---|---|---|---|
| AI phishing + deepfake voice → laptop | Sentinel, Tracker, Quarantine, Patch | DNS filtering, NetFlow, 802.1X CoA → quarantine VLAN, phishing-resistant MFA | New-domain beaconing from VLAN 10 |
| Rogue AP / evil twin | Sentinel, Tracker, Decoy, Quarantine, Patch | WLC rogue detection, WPA3-Enterprise, 802.11w PMF, 802.1X server cert validation, port security if wired | WLC rogue alarm on SSID match |
| ARP spoofing / MITM | Sentinel, Tracker, Quarantine, Patch | DHCP snooping, Dynamic ARP Inspection, port security, err-disable | DAI drop counter on an access port |
| Prompt injection / MCP tool poisoning | Sentinel, Decoy, Tracker, Quarantine, Patch | Agent VLAN (30), egress ACL allowlist, firewall FQDN policy, token revocation, AAA for agent identities | Agent tool call that reads secrets and then calls an unknown host |
| Credential stuffing at the VPN | Sentinel, Decoy, Tracker, Quarantine, Patch | AAA/RADIUS, MFA, `login block-for`, max-fail lockout, edge ACL or rate limit after approval | Spike in auth failures per ASN, honey account used |
| LLM-assisted mutating malware | Sentinel, Tracker, Quarantine, Patch | Firewall egress allowlist for AI APIs, NetFlow, server VLAN ACL | Server or script calling an LLM API for the first time |
| Supply-chain poisoning | Patch, Sentinel, Auditor | Egress allowlist from build hosts, credential rotation, pinned dependencies | New outbound flows from build hosts after an update |
| Data poisoning | Auditor, Patch | Out of network scope. Use provenance on training/RAG data and integrity checks | Unexpected model behavior on trigger phrases |
| Bot scraping swarms | Sentinel, Tracker, Decoy | Edge ACLs, rate limits, WAF rules, Web Bot Auth verification for good bots | Request-rate and fingerprint clusters |

---

## 4. Phased roadmap (spend cap: $0)

**Phase 0: Prototype (done, local)**
- Single-file `index.html`, no build step, neon hive look, pan and zoom (with pinch on phones).
- 16 network nodes in 5 bands plus 7 defender agents.
- Click any node for a study panel covering the threat, the defense, the CCNA concept, an IOS config example and sources.
- Battle sim with 6 simulated rogue bots. Start it with the button or `?demo=1`. Open a node panel directly with `?panel=<id>`.
- Labels are 14.5px on screen at default zoom. There's a portrait layout below 760px wide.

**Phase 1: Polish and publish (needs the user's yes)**
- Push to the CCNA-BRAIN-HIVE repo on a branch as a draft PR, then go live on GitHub Pages after approval. Commit as the noreply identity.
- Add a "Study mode" quiz: hide the labels and name the control that stops the bot.
- Link to and from the StarSwarm Toolkit: Alerts, Threats and Bots pages ↔ hive nodes.

**Phase 2: Scenario packs**
- Add scenarios for DHCP starvation / rogue DHCP, MAC flooding, a VLAN hopping attempt, a supply-chain worm and data poisoning.
- Make scenarios JSON-driven so new ones don't need code changes.
- Add a step-through mode: pause at each swarm step and show the matching IOS `show` command output (simulated).

**Phase 3: Real data feeds (free sources only, read-only)**
- **Lab telemetry:** run the network in Cisco Packet Tracer or GNS3/containerlab (free) and export syslog and NetFlow to a local collector. A tiny parser feeds real lab events into the hive, clearly labeled as LAB, not production.
- **Public threat context:**
  - CISA KEV JSON feed (free, no key)
  - GreyNoise community API (free tier, check limits)
  - Cloudflare Radar public pages/API, already in the brand stack
  - abuse.ch feeds (URLhaus, ThreatFox: free, review terms)

  Display only. No automatic blocking from public feeds.
- **Toolkit tie-in:** read the toolkit's own Alerts and Bots data, if exposed, through the existing Worker.

**Phase 4: Real swarm (optional, much later)**
- Agents as small Workers or local scripts with separate scoped identities, an event bus, and a human approval UI.
- Write actions limited to a lab network first.
- A formal guardrail review against the OWASP Agentic Top 10 and the CISA agentic AI guidance before anything touches a real network.

---

## 5. Open questions
1. **Release path.** Should this replace the CCNA-BRAIN-HIVE `index.html` in place, or live as a new page or repo under the StarSwarm Strategic CyberSecurity name? And should the original CCNA blueprint stay reachable (it's saved as `index.v1-original.html`)?
2. **Audience.** Is this mainly a CCNA study tool, a StarSwarm marketing demo, or both? That decides how much exam detail vs. product messaging goes in.
3. **"Swarm" wording.** The earlier research flagged that "swarm" now carries baggage. OpenAI's runaway agents called themselves a swarm. Is "defender swarm / hive" ok, as long as the copy says it hunts rogue swarms?
4. **Real feeds.** Which free feeds, if any, are acceptable to call from a public GitHub Pages site? Some need keys, which can't sit in a static page. A Worker proxy would keep keys out of the repo.
5. **Toolkit links.** Should hive nodes deep-link into specific toolkit pages (Alerts, Threats, Bots)? Do those pages have stable anchors?
6. **Google Fonts.** The prototype loads them like the original did, which is a third-party request. Self-host or drop them for privacy?
