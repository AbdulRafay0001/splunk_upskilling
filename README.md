# Splunk Upskilling: SOC and SIEM Fundamentals

Notes and reference material gathered while learning Splunk and security operations. This document covers what a Security Operations Centre is, how it runs day to day, and where SIEM tooling like Splunk fits into that picture.

---

## Contents

1. [What is a SOC?](#what-is-a-soc)
   - [Key functions](#key-functions)
   - [Common technologies](#common-technologies)
   - [Processes](#processes)
   - [Challenges](#challenges)
   - [Best practices](#best-practices)
   - [Roles](#roles)
   - [Threat hunting](#threat-hunting)
   - [Common event types to look out for](#common-event-types-to-look-out-for)
2. [What is SIEM?](#what-is-siem)
   - [Core capabilities](#core-capabilities)
   - [How a SIEM actually works](#how-a-siem-actually-works)
   - [SIEM vs adjacent tooling](#siem-vs-adjacent-tooling)
3. [An analogy for SIEM](#an-analogy-for-siem)
4. [Examples of SIEM software in 2026](#examples-of-siem-software-in-2026)
5. [Glossary](#glossary)

---

## What is a SOC?

A **Security Operations Centre (SOC)** is the team, and the set of processes and tooling that team uses, responsible for continuously monitoring an organisation's IT estate, detecting malicious or suspicious activity, and responding to it before it becomes a breach.

It is best thought of as a capability rather than a room. Some SOCs are a physical space full of screens. Many modern ones are fully distributed, follow-the-sun teams, or an outsourced service (an MSSP or MDR provider). What makes it a SOC is the mandate: continuous detection, investigation, and response.

The SOC usually operates 24/7/365, because attackers do not keep office hours. A large share of intrusions are deliberately timed for weekends, public holidays, and the small hours of the morning.

### Key functions

| Function | What it means in practice |
|---|---|
| **Asset discovery and inventory** | You cannot defend what you do not know exists. Maintaining a live picture of endpoints, servers, cloud workloads, identities, and applications. |
| **Log collection and monitoring** | Centralising telemetry from across the estate into a single searchable platform, normally the SIEM. |
| **Detection engineering** | Writing, testing, and tuning the correlation rules, searches, and analytics that turn raw logs into alerts. |
| **Alert triage** | Reviewing incoming alerts, separating true positives from noise, and escalating what matters. |
| **Incident response** | Containing, eradicating, and recovering from confirmed incidents. |
| **Threat intelligence** | Consuming and producing intel on adversaries, tactics, and indicators, then operationalising it in detections. |
| **Threat hunting** | Proactively searching for adversary activity that existing detections have missed. |
| **Vulnerability and patch oversight** | Tracking exposure and pushing remediation, often shared with IT operations. |
| **Compliance and reporting** | Evidence for standards such as ISO 27001, PCI DSS, GDPR, SOC 2, and NIS2. Log retention is frequently a regulatory requirement, not a preference. |
| **Post-incident review** | Lessons learned, root cause analysis, and feeding improvements back into detections and controls. |

### Common technologies

| Category | Purpose | Examples |
|---|---|---|
| **SIEM** | Central log aggregation, correlation, alerting, search | Splunk Enterprise Security, Microsoft Sentinel, Elastic Security |
| **EDR / XDR** | Endpoint and cross-domain detection and response | CrowdStrike Falcon, Microsoft Defender for Endpoint, SentinelOne |
| **SOAR** | Playbook-driven automation and case management | Splunk SOAR, Cortex XSOAR, Tines |
| **UEBA** | Baselines normal user and entity behaviour, flags deviation | Splunk UBA, Exabeam, Securonix |
| **NDR / IDS / IPS** | Network traffic analysis and intrusion detection | Zeek, Suricata, Darktrace |
| **Firewalls and proxies** | Perimeter and egress control, plus rich log sources | Palo Alto, Fortinet, Zscaler |
| **TIP** | Threat intelligence aggregation and enrichment | MISP, Anomali, Recorded Future |
| **Vulnerability management** | Exposure discovery and prioritisation | Tenable, Qualys, Rapid7 |
| **IAM / PAM** | Identity governance and privileged access control | Okta, Entra ID, CyberArk |
| **CSPM / CNAPP** | Cloud posture and workload protection | Wiz, Prisma Cloud, Defender for Cloud |
| **Email security** | Phishing detection and mail flow control | Proofpoint, Mimecast, Defender for Office 365 |
| **Ticketing / ITSM** | Case tracking and workflow | ServiceNow, Jira Service Management |
| **Attack simulation** | Validating that detections actually fire | Atomic Red Team, Caldera, breach and attack simulation tools |

### Processes

Most SOCs run on a small number of documented, repeatable processes. Documentation matters more than heroics, because analysts rotate, hand over shifts, and need to act consistently under pressure.

**1. Monitoring and triage (the tiered model)**

The classic structure escalates work upwards as complexity increases:

- **Tier 1** picks up alerts from the queue, validates them against a runbook, closes false positives, and escalates anything real.
- **Tier 2** investigates escalated alerts in depth, pivots across data sources, determines scope and impact, and drives containment.
- **Tier 3** handles the hardest incidents, performs forensics and malware analysis, and runs proactive hunts.

Worth noting: many mature SOCs have moved away from strict tiering towards a flatter, swarming model, where a cross-functional group tackles an incident together. Tiering scales headcount efficiently but can bore Tier 1 analysts into leaving.

**2. Incident response lifecycle**

Two frameworks dominate. NIST SP 800-61 defines four phases: Preparation, Detection and Analysis, Containment / Eradication / Recovery, and Post-Incident Activity. SANS uses six: Preparation, Identification, Containment, Eradication, Recovery, and Lessons Learned. They describe the same arc.

**3. Detection engineering lifecycle**

Threat research, then rule development, then testing against simulated attacks, then deployment, then tuning based on false positive rate, then periodic review and retirement. Detections are code and should be treated as such, with version control and peer review. This is often called Detection as Code.

**4. Escalation and communication**

Defined severity levels, clear escalation paths, agreed SLAs (for example, respond to a critical alert within 15 minutes), and out-of-hours contacts. Knowing who calls legal, who calls the regulator, and who talks to the press is part of the process, not an afterthought.

**5. Shift handover**

Structured handover notes so nothing in flight gets dropped between shifts.

**6. Continuous improvement**

Purple team exercises, tabletop simulations, detection coverage mapping against MITRE ATT&CK, and metric review.

### Challenges

- **Alert fatigue.** The single most cited problem. Analysts face thousands of alerts a day, most of them false positives, and genuine threats get lost in the noise or dismissed reflexively.
- **Skills shortage and burnout.** Experienced analysts are scarce and expensive. Repetitive triage work drives high attrition, which in turn erodes institutional knowledge.
- **Data volume and cost.** Ingesting everything is financially and technically painful. Most SIEM licensing is volume-based, so teams end up making risk decisions on the basis of budget, which is uncomfortable but real.
- **Visibility gaps.** Shadow IT, unmanaged SaaS, OT and IoT devices, contractor laptops, and multi-cloud sprawl all create blind spots.
- **Tool sprawl and integration debt.** A dozen consoles that do not talk to each other force analysts to swivel-chair between them.
- **Speed of the adversary.** Attacker breakout time (from initial access to lateral movement) is now frequently measured in minutes, while mean time to detect is often measured in days.
- **Encrypted and cloud-native traffic.** Traditional network inspection sees less than it used to.
- **Insider threat.** Behaviour that is technically authorised and therefore invisible to signature-based detection.
- **Proving value.** Security is a cost centre. Demonstrating return on investment when the desired outcome is "nothing bad happened" is a perennial difficulty.

### Best practices

1. **Know your estate.** Asset and identity inventory is the foundation of every other control.
2. **Prioritise log sources by detection value, not by volume.** Authentication, EDR, DNS, proxy, and cloud control plane logs earn their keep. Verbose debug logs usually do not.
3. **Map coverage to MITRE ATT&CK.** It turns "are we secure?" into a gap analysis you can actually action.
4. **Tune relentlessly.** Every recurring false positive is a tax on analyst attention. Track the top ten noisiest rules and fix them.
5. **Automate the repetitive.** IP reputation lookups, hash enrichment, user context lookups, and account disablement are all good SOAR candidates. Automate enrichment before you automate response.
6. **Write and maintain runbooks.** Every alert type should tell the analyst what it means, what to check, and what to do next.
7. **Treat detections as code.** Version control, peer review, CI testing, staged rollout.
8. **Validate your detections.** Run adversary emulation regularly. A rule that has never fired is not necessarily good news.
9. **Measure the right things.** MTTD and MTTR, dwell time, true positive rate, escalation accuracy, and detection coverage. Avoid vanity metrics like raw alert counts.
10. **Defence in depth.** The SOC is one layer. It does not compensate for weak identity controls, missing MFA, or unpatched perimeter devices.
11. **Invest in people.** Rotate analysts through hunting and detection engineering, fund training and certification, and take burnout seriously.
12. **Practise.** Tabletop exercises and purple teaming reveal process failures long before a real incident does.
13. **Retain logs sensibly.** Hot storage for recent, searchable data. Cheaper cold or frozen tiers for the long tail. Attackers are sometimes discovered months after entry, so short retention destroys your ability to investigate.

### Roles

| Role | Focus |
|---|---|
| **SOC Analyst (Tier 1)** | Alert monitoring and initial triage, runbook execution, escalation. Common entry point into the field. |
| **SOC Analyst (Tier 2)** | Deeper investigation, correlation across data sources, scoping and containment. |
| **Incident Responder (Tier 3)** | Major incident handling, forensics, malware analysis, coordination of eradication and recovery. |
| **Threat Hunter** | Proactive hypothesis-driven search for undetected adversary activity. |
| **Detection Engineer** | Builds and maintains detection logic, correlation searches, and data models. |
| **Security Engineer** | Owns the platform itself: SIEM architecture, data onboarding, parsers, integrations, performance. |
| **Threat Intelligence Analyst** | Tracks adversary groups and campaigns, produces actionable intel for detection and leadership. |
| **SOC Manager / Lead** | Staffing, shift scheduling, metrics, process ownership, stakeholder reporting. |
| **CISO** | Executive accountability for security strategy, risk appetite, and budget. |
| **Forensics / DFIR Specialist** | Evidence acquisition, disk and memory analysis, chain of custody. |
| **Compliance / GRC Analyst** | Maps controls to regulatory frameworks and produces audit evidence. |
| **Red Team** | Simulates adversaries to test detection and response. Sits alongside rather than within the SOC. |

**Splunk-specific note:** in a Splunk shop, the platform work usually splits into a Splunk Admin (indexers, forwarders, licensing, cluster health) and a Splunk Engineer or Content Developer (data onboarding, CIM compliance, correlation searches in Enterprise Security, dashboards).

### Threat hunting

Threat hunting is the proactive, human-driven search for adversary activity that automated detections have missed. The operating assumption is **assume breach**: something has already got in, and the job is to find it rather than to wait for an alert.

**Why it exists:** signature and rule-based detection only catches what someone has already thought to look for. Novel tradecraft, living-off-the-land techniques, and insider activity frequently generate no alert at all.

**Types of hunt**

- **Hypothesis-driven.** Starts with an informed guess, usually derived from ATT&CK, for example: "If an attacker were using WMI for lateral movement here, what would that look like in our logs?"
- **Intelligence-driven / IOC-based.** Starts from indicators, such as hashes, domains, or IPs from a new threat report, and searches historical data for matches.
- **Analytics-driven.** Uses statistics, baselining, and machine learning to surface outliers, for example a user authenticating from two countries within an hour.
- **Situational.** Driven by internal context, such as a crown-jewel system, a recent acquisition, or a departing privileged employee.

**A typical hunt loop**

1. Form a hypothesis, grounded in threat intel or knowledge of the environment.
2. Identify the data sources that would evidence it and confirm you actually collect them.
3. Search, pivot, and analyse.
4. Reach a conclusion: adversary found, benign explanation found, or visibility gap found. All three are useful outputs.
5. Operationalise the result. Anything worth finding once is worth turning into a detection rule so it is never hunted manually again.
6. Document and share.

**Frameworks and models worth knowing:** MITRE ATT&CK, the Cyber Kill Chain, the Diamond Model of Intrusion Analysis, the Pyramid of Pain (hashes are trivial for an attacker to change, TTPs are not), and the Hunting Maturity Model (HMM0 through HMM4).

**What good hunting needs:** broad and deep telemetry, sufficient retention to look back weeks or months, a fast search platform, and analysts who understand what normal looks like in that specific environment.

### Common event types to look out for

Two things share the name "event type" in a Splunk context, and it is worth separating them.

#### A note on Splunk `eventtype`

In Splunk, an **eventtype** is a knowledge object: a saved search that categorises events matching a condition, so you can refer to a whole class of events by name instead of repeating the search logic.

```spl
# Definition (in eventtypes.conf or via Settings > Event types)
[windows_failed_login]
search = source="WinEventLog:Security" EventCode=4625

# Usage
eventtype=windows_failed_login | stats count by user, src_ip
```

Eventtypes underpin the Common Information Model (CIM), which normalises field names across vendors so a search works whether the log came from Palo Alto or Cisco. Splunk Enterprise Security depends heavily on CIM compliance, so getting eventtypes and tags right at onboarding is not optional.

#### Activity categories worth alerting on

**Authentication and identity**

- Repeated failed logons followed by a success (brute force or password spraying)
- Impossible travel: successful authentications from geographically incompatible locations
- Logins outside normal working hours or from unusual countries
- MFA fatigue, meaning repeated push notifications until the user approves one
- Disabled, dormant, or service accounts suddenly becoming active
- Authentication from anonymising infrastructure such as Tor or a commercial VPN

**Privilege and account changes**

- New account creation, especially outside standard provisioning
- Accounts added to privileged groups such as Domain Admins
- Password resets on privileged accounts
- Group Policy modification
- Changes to security tooling configuration or exclusion lists

**Process and endpoint execution**

- PowerShell with encoded commands, download cradles, or execution policy bypass
- Living-off-the-land binaries: `certutil`, `mshta`, `rundll32`, `regsvr32`, `wmic`, `bitsadmin`
- Office applications spawning shells or script interpreters, a classic macro payload pattern
- Unsigned or unusual binaries executing from `%TEMP%`, `%APPDATA%`, or `C:\Users\Public`
- Credential access tooling behaviour, such as LSASS memory access
- Registry Run key persistence, new scheduled tasks, or new services

**Network**

- Beaconing: regular, low-variance outbound connections to a single destination
- DNS anomalies: high-entropy domain names, long subdomains suggesting DNS tunnelling, spikes in NXDOMAIN
- Connections to known-bad IPs, domains, or newly registered domains
- Large outbound transfers, particularly to cloud storage, indicating exfiltration
- Internal port scanning and east-west lateral movement
- Traffic on non-standard ports, or protocol on wrong port

**Data and files**

- Mass file access, modification, or deletion in a short window (ransomware signature)
- Volume shadow copy deletion
- Access to sensitive shares by users with no business reason
- Data staged into archives before egress

**Cloud and SaaS**

- Console logins without MFA
- Root or global admin account usage
- IAM policy or role changes, especially privilege escalation paths
- Storage buckets made public
- CloudTrail, audit logging, or GuardDuty being disabled
- New API keys or OAuth applications granted broad scopes
- Resource creation in unused regions, often cryptomining

**Defence evasion**

- Security or audit logs cleared
- Antivirus or EDR agent stopped, uninstalled, or excluded
- Timestomping
- Log forwarder stopping. A source that goes quiet is itself an alert worth having.

#### Useful Windows Event IDs

| Event ID | Meaning |
|---|---|
| 4624 | Successful logon (check Logon Type: 3 network, 10 RDP) |
| 4625 | Failed logon |
| 4634 / 4647 | Logoff |
| 4648 | Logon using explicit credentials, often runas |
| 4672 | Special privileges assigned, indicates admin logon |
| 4688 | New process created, enable command line auditing |
| 4697 | Service installed |
| 4720 | User account created |
| 4728 / 4732 / 4756 | Member added to a security group |
| 4740 | Account locked out |
| 4768 / 4769 / 4771 | Kerberos TGT and service ticket activity, relevant to Kerberoasting |
| 5140 / 5145 | Network share accessed |
| 1102 | Audit log cleared |
| 7045 | New service installed (System log) |
| Sysmon 1 | Process creation, richer than 4688 |
| Sysmon 3 | Network connection |
| Sysmon 7 | Image loaded, useful for DLL sideloading |
| Sysmon 8 | CreateRemoteThread, process injection |
| Sysmon 10 | Process access, relevant to LSASS dumping |
| Sysmon 11 | File created |
| Sysmon 13 | Registry value set |

#### Example SPL to build on

```spl
# Potential brute force: 10 or more failures then a success from the same source
index=wineventlog EventCode IN (4624, 4625)
| stats count(eval(EventCode=4625)) AS failures,
        count(eval(EventCode=4624)) AS successes
        BY src_ip, user
| where failures >= 10 AND successes > 0
| sort - failures
```

```spl
# Possible beaconing: low variance in the interval between outbound connections
index=firewall action=allowed
| streamstats current=f last(_time) AS prev_time BY src_ip, dest_ip
| eval delta = prev_time - _time
| stats count, avg(delta) AS avg_interval, stdev(delta) AS jitter BY src_ip, dest_ip
| where count > 20 AND jitter < 5
```

```spl
# Encoded PowerShell execution
index=sysmon EventCode=1 Image="*\\powershell.exe"
CommandLine IN ("*-enc*", "*-EncodedCommand*", "*FromBase64String*")
| table _time, host, User, ParentImage, CommandLine
```

---

## What is SIEM?

**Security Information and Event Management (SIEM)** is the platform that collects log and event data from across an organisation, normalises it into a consistent format, correlates it in near real time, and raises alerts when patterns indicate a security problem. It is the SOC's central nervous system and, for most analysts, the primary place they do their work.

The term merges two older product categories: SIM (Security Information Management, focused on long-term log storage and compliance reporting) and SEM (Security Event Management, focused on real-time monitoring and alerting). Gartner combined them in 2005 and the name stuck.

### Core capabilities

- **Log aggregation** from heterogeneous sources: endpoints, servers, network devices, cloud platforms, identity providers, applications, and SaaS.
- **Normalisation and parsing** into a common schema, so `src_ip`, `srcip`, and `source_address` all become one searchable field. In Splunk this is the Common Information Model.
- **Enrichment** with context: asset criticality, user department, geolocation, threat intelligence, vulnerability status.
- **Correlation** across sources and time, which is the core value. One failed login is nothing. Five hundred failed logins, a success, and then a connection to a known-bad IP is an incident.
- **Real-time alerting** with severity and risk scoring.
- **Search and investigation** across historical data, which is what turns an alert into an understood incident.
- **Dashboards and visualisation** for both analysts and executives.
- **Long-term retention** to satisfy regulation and to allow retrospective hunting.
- **Reporting** for auditors and regulators.
- **Increasingly: UEBA, SOAR, and AI-assisted triage** built directly into the platform rather than bolted on.

### How a SIEM actually works

```
   Data sources                 SIEM pipeline                    Outputs
   ------------                 -------------                    -------
   Endpoints    \
   Servers       \
   Firewalls      \   collect    parse and      enrich and     alerts
   Proxies         >  -------->  normalise  ->  correlate  ->  dashboards
   Cloud / SaaS   /   (agents,   (schema,       (context,      investigations
   Identity      /    forwarders, field         rules,         reports
   Applications /     APIs,       extraction)   analytics)     retained logs
                      syslog)
```

In Splunk terms: **forwarders** ship data, **indexers** parse and store it, **search heads** run searches and correlation, and **Enterprise Security** provides the SOC-specific layer of notable events, risk-based alerting, and investigation workbenches.

### SIEM vs adjacent tooling

| Tool | Scope | Relationship to SIEM |
|---|---|---|
| **SIEM** | Everything that produces a log | The central correlation and search layer |
| **EDR** | Endpoints only, but very deep | A high-value data source for the SIEM, and a response mechanism |
| **XDR** | Endpoint, network, email, cloud, usually one vendor's stack | Overlaps heavily with SIEM. Deeper native telemetry, narrower vendor coverage |
| **SOAR** | Automation and orchestration | Consumes SIEM alerts and executes playbooks |
| **UEBA** | Behavioural baselining | Now typically a SIEM feature rather than a separate product |
| **Log management** | Storage and search | SIEM minus the security analytics and correlation |

---

## An analogy for SIEM

**A SIEM is the control room of a large airport.**

Every part of the airport generates its own information. Runway sensors, radar, gate scanners, baggage handling, passport control, CCTV, ground crew radios. Individually each one is just a stream of routine noise. Nobody can watch all of them at once, and nobody would want to.

The control room does four things:

- **It brings every feed into one room.** That is log aggregation. Radar, boarding scans, and CCTV all arrive in the same place, on the same wall of screens.
- **It puts everything into a common language.** One system reports times in local time, another in UTC, another calls it "stand 14" while a fourth calls it "gate B14". The control room reconciles all of it so controllers can compare like with like. That is normalisation and the Common Information Model.
- **It joins the dots between feeds.** A passenger checking in is normal. A bag loaded onto a plane is normal. A bag loaded onto a plane whose passenger never boarded is a serious problem, and you only see it by correlating two separate systems. That is correlation, and it is the whole point of a SIEM.
- **It keeps the tapes.** When something goes wrong, investigators go back through the recordings to reconstruct exactly what happened, in what order. That is retention and investigation.

The rest of the analogy maps neatly too:

- **Sensors and cameras** are your log sources: endpoints, firewalls, cloud platforms.
- **Air traffic controllers** are your SOC analysts. The control room does not fly the planes. It gives the humans the picture they need to make decisions.
- **The alarm that sounds when two aircraft are on a converging course** is your correlation rule firing.
- **Nuisance alarms**, a flock of birds tripping a sensor for the tenth time this week, are false positives. Too many and controllers start ignoring the alarm, which is precisely how alert fatigue kills real detections.
- **Emergency services on standby** are the incident response function. SOAR is the automated part: the barrier that lowers itself without anyone pressing a button.
- **Blind spots**, the corner of the apron with no camera, are your missing log sources. The control room is only as good as its coverage.
- **Recording cost.** You cannot keep every camera at full resolution forever, so you make deliberate decisions about what to retain and for how long. That is your ingest licensing and retention tiering.

The important limitation the analogy captures: **a control room does not prevent anything by itself.** It provides visibility and coordination. If nobody is watching the screens, if the alarms are badly configured, or if half the airport has no sensors fitted, the control room is an expensive room full of monitors. The same is true of a SIEM, which is why tuning, coverage, and staffing matter more than the logo on the product.

*Shorter alternatives if you need one in a sentence:* a SIEM is the **black box flight recorder plus the cockpit warning system** for your network. Or, it is the **detective who interviews every witness in the building, notices that three separate stories only make sense if the same person was involved, and keeps the case file for seven years.**

---

## Examples of SIEM software in 2026

The market has consolidated considerably. Two acquisitions in particular reshaped the landscape: Cisco's acquisition of Splunk and Palo Alto's acquisition of IBM QRadar's software assets. The broader trend is convergence, with SIEM, XDR, SOAR, UEBA, and attack surface management increasingly merging into single platforms.

### Enterprise platforms

| Product | Vendor | Notes |
|---|---|---|
| **Splunk Enterprise Security** | Cisco | The market share leader and the reason this repo exists. Extremely flexible search language (SPL), huge app ecosystem, strong for mature teams willing to engineer their own content. Enterprise Security Premier bundles SIEM, SOAR, UEBA, and Attack Analyzer on a single work surface. Historically expensive at high ingest volumes. |
| **Microsoft Sentinel** | Microsoft | Cloud-native, consumption-priced, deeply integrated with Entra ID and the Defender suite. Often the default where the estate is already Microsoft-heavy. Uses KQL. |
| **Google Security Operations** (formerly Chronicle) | Google | Built on Google's data infrastructure, flat-rate pricing by user count rather than data volume, strong at very large scale retention. |
| **Cortex XSIAM** | Palo Alto Networks | Positioned as an AI-driven successor to traditional SIEM. Combines SIEM, XDR, SOAR, and attack surface management, using machine learning to auto-correlate low-confidence alerts into higher-fidelity incidents. Now also the home of QRadar customers migrating over. |
| **IBM QRadar** | IBM / Palo Alto | Still widely deployed on-premises, but the software assets moved to Palo Alto. Expect migration conversations rather than new deployments. |
| **CrowdStrike Falcon Next-Gen SIEM** | CrowdStrike | Leverages existing Falcon endpoint telemetry, appealing where Falcon EDR is already in place. |

### Analytics-led specialists

| Product | Notes |
|---|---|
| **Exabeam** | Strong UEBA heritage and behavioural analytics. Merged with LogRhythm, so the two product lines are converging. |
| **Securonix** | Cloud-native, analytics-first, uses a bring-your-own-cloud storage model to control cost. |
| **Sumo Logic Cloud SIEM** | Cloud-native, popular with DevOps-adjacent teams. |
| **Gurucul** | Risk analytics and UEBA focused. |
| **Devo** | Fast search at scale, retains hot data for long periods. |

### Open source and budget-conscious

| Product | Notes |
|---|---|
| **Wazuh** | Free and open source, forked from OSSEC. Combines HIDS, log analysis, and compliance. Very popular for home labs and SMEs. Excellent for building experience without licence cost. |
| **Elastic Security** | Built on the Elastic Stack. Free tier available, with paid tiers for advanced detection features. |
| **Graylog Security** | Approachable log management with a security layer on top. Strong mid-market fit. |
| **Security Onion** | Free Linux distribution bundling Zeek, Suricata, the Elastic Stack, and hunting tools. Superb for learning. |
| **OpenSearch / OpenSearch Security Analytics** | AWS-backed Elasticsearch fork with detection rule support. |

> **A note on comparing these:** every one of these products does log management, threat detection, and compliance reporting. The real differentiators are deployment model, pricing structure, how much tuning effort the platform demands, and whether it fits the team's maturity. Platforms like Splunk and QRadar are built for experienced SOC teams that can tune detection rules and manage high alert volumes, whereas cloud-native platforms tend to deploy faster with less overhead.

---

## Glossary

| Term | Meaning |
|---|---|
| **CIM** | Common Information Model. Splunk's normalisation schema. |
| **DFIR** | Digital Forensics and Incident Response. |
| **Dwell time** | How long an attacker is present before being detected. |
| **IOC** | Indicator of Compromise, for example a hash, IP, or domain. |
| **LOLBin** | Living off the Land Binary. A legitimate system tool abused by an attacker. |
| **MDR** | Managed Detection and Response. Outsourced SOC capability. |
| **MITRE ATT&CK** | A knowledge base of adversary tactics, techniques, and procedures. |
| **MTTD / MTTR** | Mean Time To Detect / Mean Time To Respond. |
| **MSSP** | Managed Security Service Provider. |
| **Notable event** | Splunk Enterprise Security's term for an alert requiring analyst review. |
| **RBA** | Risk-Based Alerting. Accumulating risk score across many low-severity signals rather than alerting on each. |
| **SOAR** | Security Orchestration, Automation and Response. |
| **SPL** | Search Processing Language. Splunk's query language. |
| **TTP** | Tactics, Techniques and Procedures. How an adversary operates. |
| **UEBA** | User and Entity Behaviour Analytics. |

---

## Next steps for this repo

- [ ] Add SPL cheatsheet
- [ ] Add notes on Splunk architecture (forwarders, indexers, search heads)
- [ ] Add worked examples from Splunk BOTS datasets
- [ ] Map practice detections to MITRE ATT&CK techniques
- [ ] Document lab setup
