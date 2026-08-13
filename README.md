# Splunk Upskilling: SOC, SIEM and Splunk Fundamentals

Personal notes from learning security operations and Splunk from scratch. Written for someone seeing these terms for the first time, so jargon is explained the first time it appears rather than assumed.

---

## Contents

**Part 1: Security Operations**
1. [What is a SOC?](#what-is-a-soc)
   - [Key functions](#key-functions)
   - [Common technologies](#common-technologies)
   - [Processes](#processes)
   - [Challenges](#challenges)
   - [Best practices](#best-practices)
   - [Roles](#roles)
   - [Threat hunting](#threat-hunting)
   - [Common event types to look out for](#common-event-types-to-look-out-for)

**Part 2: SIEM**

2. [What is SIEM?](#what-is-siem)
3. [An analogy for SIEM](#an-analogy-for-siem)
4. [Examples of SIEM software in 2026](#examples-of-siem-software-in-2026)

**Part 3: Splunk**

5. [What is Splunk?](#what-is-splunk)
6. [What can Splunk be used for, and why use it?](#what-can-splunk-be-used-for-and-why-use-it)
7. [What is a Splunk / SOC analyst?](#what-is-a-splunk--soc-analyst)
8. [Versions and products](#versions-and-products)
9. [Splunk architecture](#splunk-architecture)
10. [Deployment options](#deployment-options)
11. [Basic Splunk terms](#basic-splunk-terms)
12. [What data does Splunk ingest?](#what-data-does-splunk-ingest)
13. [How does data get into Splunk?](#how-does-data-get-into-splunk)
14. [What is SPL?](#what-is-spl)
15. [SPL examples](#spl-examples)
16. [What can you build in Splunk?](#what-can-you-build-in-splunk)
17. [Apps vs add-ons](#apps-vs-add-ons)
18. [Securing data in Splunk](#securing-data-in-splunk)
19. [Case studies](#case-studies)
20. [Certification path](#certification-path)

21. [Glossary](#glossary)

---

# Part 1: Security Operations

## What is a SOC?

A **Security Operations Centre (SOC)** is the team that watches over a company's computers, networks, and accounts, looking for signs that someone is trying to break in or has already gotten in. When they spot something, it is their job to deal with it before it turns into a real breach.

Despite the name, a SOC is not always a physical room. Some companies do have a room full of screens and analysts. Others have a fully remote team, or pay an outside company to do this job for them (this is called an **MSSP**, a managed security service provider, or **MDR**, managed detection and response). What actually makes it a SOC is not the room, it is the job: watch constantly, investigate anything suspicious, and respond.

Most SOCs run 24 hours a day, all year, because attackers do not keep office hours. Plenty of attacks are deliberately launched at 3am on a bank holiday weekend, precisely because fewer people are watching.

### Key functions

| Function | What it means in practice |
|---|---|
| **Asset discovery and inventory** | Keeping an up to date list of every laptop, server, cloud system, and user account the company has. You cannot defend something you do not know exists. |
| **Log collection and monitoring** | Pulling all the activity records from every device and app into one searchable place, normally the SIEM. These records are called **logs**, or **telemetry**, which just means the data a system automatically writes down about what it is doing: a login attempt, a file being opened, a connection to a website. |
| **Detection engineering** | Writing and improving the rules that decide when normal activity becomes an alert worth a human looking at. |
| **Alert triage** | Working through incoming alerts and deciding which are real threats and which are noise. **Triage** is borrowed from hospitals: sort quickly, deal with the serious cases first. |
| **Incident response** | When something is confirmed as real, stopping it spreading, removing the attacker, and getting systems back to normal. |
| **Threat intelligence** | Keeping up with who is attacking organisations like yours and how, then using that knowledge to build better detections. |
| **Threat hunting** | Actively going looking for attackers who have slipped past the automated alerts. |
| **Vulnerability oversight** | Tracking known weaknesses in software and chasing the IT team to patch them. |
| **Compliance and reporting** | Producing evidence for regulations and standards such as ISO 27001, PCI DSS, GDPR, and SOC 2. Keeping logs for a set period is often a legal requirement, not a choice. |
| **Post-incident review** | After an incident, working out what happened, why it was not caught sooner, and what to change. |

### Common technologies

You will hear these acronyms constantly. Each row is one category of tool.

| Category | What it does | Examples |
|---|---|---|
| **SIEM** (Security Information and Event Management) | The central system that collects all the logs, searches them, and raises alerts. This is where Splunk sits. | Splunk Enterprise Security, Microsoft Sentinel, Elastic Security |
| **EDR** (Endpoint Detection and Response) | Software installed on laptops and servers that watches what programs do and can block or isolate them. An **endpoint** just means an individual device. | CrowdStrike Falcon, Microsoft Defender for Endpoint, SentinelOne |
| **XDR** (Extended Detection and Response) | EDR widened out to also cover email, network, and cloud. | Cortex XDR, Microsoft Defender XDR |
| **SOAR** (Security Orchestration, Automation and Response) | Automates the boring repeated steps of an investigation using **playbooks**, which are pre-written sequences of actions. | Splunk SOAR, Cortex XSOAR, Tines |
| **UEBA** (User and Entity Behaviour Analytics) | Learns what normal behaviour looks like for each user and machine, then flags anything unusual. | Splunk UBA, Exabeam, Securonix |
| **NDR / IDS / IPS** | Watches network traffic for attacks. IDS detects, IPS also blocks. | Zeek, Suricata, Darktrace |
| **Firewalls and proxies** | Control what traffic can enter and leave. They also produce very useful logs. | Palo Alto, Fortinet, Zscaler |
| **TIP** (Threat Intelligence Platform) | Stores and distributes lists of known-bad IP addresses, domains, and file signatures. | MISP, Anomali, Recorded Future |
| **Vulnerability management** | Scans for out of date and weak software. | Tenable, Qualys, Rapid7 |
| **IAM / PAM** | Identity and access management, and privileged access management. Controls who can log in to what, and protects admin accounts specifically. | Okta, Microsoft Entra ID, CyberArk |
| **CSPM / CNAPP** | Checks cloud accounts for risky configuration, such as a storage bucket accidentally made public. | Wiz, Prisma Cloud, Defender for Cloud |
| **Email security** | Blocks phishing and malicious attachments. | Proofpoint, Mimecast, Defender for Office 365 |
| **Ticketing / ITSM** | Where incidents get logged and tracked. | ServiceNow, Jira Service Management |
| **Attack simulation** | Safely mimics attacker behaviour to check your detections actually fire. | Atomic Red Team, Caldera |

### Processes

A SOC runs on a small number of written, repeatable processes. Documentation matters more than individual heroics, because staff change shift, go on holiday, and leave.

**1. Monitoring and triage (the tiered model)**

Work escalates upwards as it gets harder:

- **Tier 1** watches the alert queue, checks each alert against a **runbook** (a step by step guide for that alert type), closes the obvious false alarms, and passes anything real upwards.
- **Tier 2** investigates properly. They dig across multiple systems to work out how far the problem spreads and start shutting it down.
- **Tier 3** handles the worst incidents, does forensics and malware analysis, and runs proactive hunts.

Worth knowing: many modern SOCs have dropped strict tiers in favour of **swarming**, where a mixed group tackles an incident together. Tiering is efficient but Tier 1 work can be repetitive enough to drive people out of the profession.

**2. Incident response lifecycle**

Two standard frameworks describe the same journey. **NIST SP 800-61** uses four phases: Preparation, Detection and Analysis, Containment / Eradication / Recovery, and Post-Incident Activity. **SANS** splits it into six: Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned.

**3. Detection engineering lifecycle**

Research a threat, write a rule, test it, deploy it, tune it based on how many false alarms it causes, then review or retire it later. Increasingly rules are stored in version control and peer reviewed like software, an approach called **Detection as Code**.

**4. Escalation and communication**

Agreed severity levels, clear rules on who gets woken up at night, and response time targets (**SLAs**, service level agreements), for example acknowledge a critical alert within 15 minutes. Knowing who contacts legal, the regulator, and the press is part of the plan too.

**5. Shift handover**

Written notes passed between shifts so nothing in progress gets dropped.

**6. Continuous improvement**

Practice exercises, **purple teaming** (attack and defence teams working together to test detections), and checking detection coverage against MITRE ATT&CK.

### Challenges

- **Alert fatigue.** The number one complaint. Analysts can face thousands of alerts a day, most of them false alarms, so real threats get lost or dismissed out of habit.
- **Skills shortage and burnout.** Experienced analysts are hard to find and expensive. Repetitive work drives people out, and their knowledge leaves with them.
- **Data volume and cost.** Most SIEM licensing charges by how much data you send it, so teams end up choosing what to monitor partly on budget grounds. Uncomfortable, but real.
- **Visibility gaps.** Unapproved software, personal devices, contractor laptops, and sprawling cloud accounts all create blind spots.
- **Too many tools.** A dozen separate consoles that do not talk to each other force analysts to jump between screens to answer one question.
- **Attacker speed.** **Breakout time**, meaning how long it takes an attacker to move from the first machine to others, is now often measured in minutes. Detection is often measured in days.
- **Encryption and cloud.** Traditional network monitoring sees less than it used to, because most traffic is encrypted and most systems are somebody else's.
- **Insider threat.** Someone doing something they are technically allowed to do is very hard to detect with rules.
- **Proving value.** Success looks like nothing happening, which is a difficult thing to put in a budget request.

### Best practices

1. **Know what you own.** An accurate asset and user list underpins everything else.
2. **Prioritise the log sources that actually catch attackers.** Authentication, endpoint, DNS, web proxy, and cloud admin logs earn their cost. Verbose application debug logs usually do not.
3. **Map your coverage to MITRE ATT&CK.** ATT&CK is a free public catalogue of the techniques real attackers use. Mapping your detections against it turns a vague question ("are we secure?") into a specific gap list.
4. **Tune constantly.** Every alert that fires repeatedly for no reason costs analyst attention. Track your ten noisiest rules and fix them.
5. **Automate the repetitive parts.** Looking up whether an IP address is known-bad, or pulling a user's job title, are perfect automation candidates. Automate information gathering before you automate any blocking action.
6. **Write runbooks.** Every alert should tell the analyst what it means, what to check, and what to do.
7. **Treat detections like code.** Version control, review, and testing.
8. **Test your detections.** Safely simulate attacks and check the alert fires. A rule that has never triggered is not necessarily good news.
9. **Measure sensible things.** MTTD and MTTR (mean time to detect and mean time to respond), dwell time, and the percentage of alerts that turn out to be real. Raw alert counts tell you nothing useful.
10. **Defence in depth.** The SOC is one layer. It does not make up for missing MFA or unpatched servers.
11. **Look after people.** Rotate analysts into hunting and engineering work, fund training, and take burnout seriously.
12. **Practise.** Tabletop exercises expose broken processes long before a real incident does.
13. **Keep logs long enough.** Attackers are sometimes found months after they got in. Short retention destroys your ability to investigate. Keep recent data fast to search and older data in cheaper storage.

### Roles

| Role | Focus |
|---|---|
| **SOC Analyst (Tier 1)** | Alert monitoring and first-pass triage. The most common way into the industry. |
| **SOC Analyst (Tier 2)** | Deeper investigation across multiple systems, working out scope and impact. |
| **Incident Responder (Tier 3)** | Major incidents, forensics, malware analysis, coordinating recovery. |
| **Threat Hunter** | Proactively searching for attackers the alerts missed. |
| **Detection Engineer** | Builds and maintains the detection rules and searches. |
| **Security Engineer / Splunk Engineer** | Owns the platform itself: architecture, getting data in, integrations, performance. |
| **Threat Intelligence Analyst** | Tracks attacker groups and turns that into usable detections and briefings. |
| **SOC Manager** | Staffing, shifts, metrics, process ownership, reporting upwards. |
| **CISO** | Chief Information Security Officer. Executive accountability for security strategy and budget. |
| **DFIR Specialist** | Digital Forensics and Incident Response. Evidence collection, disk and memory analysis. |
| **GRC Analyst** | Governance, Risk and Compliance. Maps controls to regulations and produces audit evidence. |
| **Red Team** | Plays the attacker to test the defenders. Sits alongside the SOC, not inside it. |

### Threat hunting

Threat hunting is proactively going looking for attackers, rather than waiting for an alert. The starting assumption is called **assume breach**: work as though someone is already inside and your job is to find them.

**Why it is needed:** rules only catch what somebody already thought to write a rule for. New techniques, and attackers using legitimate built-in Windows tools, often generate no alert at all.

**Types of hunt**

- **Hypothesis-driven.** Start with an educated guess, usually from MITRE ATT&CK. For example: "if an attacker were moving between machines here, what would that look like in our logs?"
- **Intelligence-driven.** Start with **IOCs** (Indicators of Compromise, meaning specific known-bad file hashes, IPs, or domains) from a new threat report and search your history for them.
- **Analytics-driven.** Use statistics to find outliers, for example one user logging in from two countries an hour apart.
- **Situational.** Driven by your own context, such as a particularly sensitive system or an employee who has just resigned.

**A typical hunt loop**

1. Form a hypothesis.
2. Work out which logs would prove or disprove it, and check you actually collect them.
3. Search, pivot between data sources, and analyse.
4. Reach one of three conclusions: you found an attacker, you found a harmless explanation, or you found a gap in your logging. All three are useful.
5. Turn anything worth finding into a permanent detection rule so nobody has to hunt for it manually again.
6. Write it up.

**Models worth knowing**

- **MITRE ATT&CK**, a catalogue of real attacker techniques.
- **Cyber Kill Chain**, the stages of an attack from reconnaissance to objective.
- **Diamond Model**, links adversary, capability, infrastructure, and victim.
- **Pyramid of Pain**, ranks indicators by how annoying they are for the attacker to change. File hashes are trivial to change. Behaviours and techniques are not, so detections built on behaviour last longer.

### Common event types to look out for

Two different things share the name "event type" here, so it is worth separating them.

#### First, the Splunk meaning of `eventtype`

In Splunk, an **eventtype** is a saved piece of search logic given a name, so you can refer to a whole category of events without retyping the search.

```spl
# Definition (set in Settings > Event types, or in eventtypes.conf)
[windows_failed_login]
search = source="WinEventLog:Security" EventCode=4625

# Use it like this
eventtype=windows_failed_login | stats count by user, src_ip
```

Eventtypes feed the **Common Information Model (CIM)**, which is Splunk's way of giving fields the same names regardless of which vendor produced the log. Without it, one firewall calls a field `src_ip`, another calls it `source_address`, and your search only works on one of them. Splunk Enterprise Security depends on CIM being done properly.

#### Second, the activity categories worth alerting on

**Authentication and identity**

- Many failed logins followed by a success (**brute force**, guessing one account's password repeatedly, or **password spraying**, trying one common password against many accounts)
- **Impossible travel**: two successful logins from locations too far apart for the time between them
- Logins at unusual hours or from unexpected countries
- **MFA fatigue**: repeated login prompts sent until the tired user approves one
- Dormant or service accounts suddenly becoming active
- Logins from Tor or commercial VPNs used to hide origin

**Privilege and account changes**

- New accounts created outside the normal joiner process
- Accounts added to admin groups such as Domain Admins
- Password resets on privileged accounts
- Changes to security tool settings, especially new exclusions

**Process and endpoint activity**

- PowerShell run with encoded or hidden commands
- **LOLBins** (Living Off the Land Binaries), meaning legitimate Windows tools abused by attackers: `certutil`, `mshta`, `rundll32`, `regsvr32`, `wmic`, `bitsadmin`
- Word or Excel launching a command prompt, the classic malicious macro pattern
- Programs running from temporary folders such as `%TEMP%`, `%APPDATA%`, or `C:\Users\Public`
- Anything reading LSASS memory, which is how attackers steal passwords from a Windows machine
- New scheduled tasks, services, or registry startup entries, all used for **persistence** (surviving a reboot)

**Network**

- **Beaconing**: regular, clockwork-like outbound connections to one address, typical of malware checking in with its operator
- DNS oddities: very long or random-looking domain names, which can indicate data being smuggled out inside DNS queries
- Connections to known-bad or newly registered domains
- Large uploads to cloud storage, a common sign of data theft
- Internal scanning and movement between machines (**lateral movement**)

**Data and files**

- Huge numbers of files modified or renamed quickly, the signature of ransomware
- Deletion of volume shadow copies, which ransomware does to stop you restoring
- Access to sensitive file shares by people with no business reason
- Files being bundled into archives just before a big upload

**Cloud and SaaS**

- Admin logins without MFA
- Root or global admin account usage
- Permission and role changes
- Storage buckets made publicly readable
- Audit logging being switched off
- New API keys or third party apps granted wide access
- Resources created in regions you never use, often cryptomining

**Defence evasion**

- Security logs cleared
- Antivirus or EDR stopped, uninstalled, or given new exclusions
- A log source going silent. Silence is itself worth alerting on.

#### Useful Windows Event IDs

| Event ID | Meaning |
|---|---|
| 4624 | Successful logon (check the Logon Type: 3 is network, 10 is remote desktop) |
| 4625 | Failed logon |
| 4634 / 4647 | Logoff |
| 4648 | Logon using explicit credentials, often "run as" |
| 4672 | Special privileges assigned, meaning an admin just logged in |
| 4688 | New process created (turn on command line logging to make this genuinely useful) |
| 4697 | Service installed |
| 4720 | User account created |
| 4728 / 4732 / 4756 | User added to a security group |
| 4740 | Account locked out |
| 4768 / 4769 / 4771 | Kerberos ticket activity, relevant to credential attacks |
| 5140 / 5145 | Network share accessed |
| 1102 | Audit log cleared |
| 7045 | New service installed (System log) |
| Sysmon 1 | Process creation, more detail than 4688 |
| Sysmon 3 | Network connection |
| Sysmon 8 | Remote thread created, a sign of process injection |
| Sysmon 10 | Process accessed another process, relevant to password theft |
| Sysmon 11 | File created |
| Sysmon 13 | Registry value set |

**Sysmon** is a free Microsoft tool that produces much richer endpoint logs than Windows does by default. Installing it is one of the cheapest detection upgrades available.

---

# Part 2: SIEM

## What is SIEM?

**Security Information and Event Management (SIEM)**, pronounced "sim", is the system that collects logs from everything in the organisation, puts them into a consistent format, looks for patterns that suggest an attack, and raises alerts. It is the main tool a SOC analyst works in all day.

The name is a merger of two older product types: **SIM**, which was about storing logs for compliance, and **SEM**, which was about real-time alerting. Gartner combined them in 2005 and the name stuck.

### Core capabilities

- **Log aggregation.** Pulling data in from endpoints, servers, firewalls, cloud platforms, identity systems, and applications.
- **Normalisation.** Rewriting all those different formats into one consistent structure, so `src_ip`, `srcip`, and `source_address` all become one searchable field name.
- **Enrichment.** Adding context, such as how critical a server is, which department a user works in, where an IP address is located, or whether it appears on a threat intelligence list.
- **Correlation.** Connecting events across different systems and points in time. This is the real value. One failed login means nothing. Five hundred failed logins, then a success, then a connection to a known-bad address, is an incident.
- **Alerting.** Raising a notification when a rule matches, with a severity attached.
- **Search and investigation.** Digging through historical data to understand what actually happened.
- **Dashboards.** Visual summaries for analysts and for management.
- **Retention.** Keeping data long enough to satisfy regulators and to investigate old activity.
- **Reporting.** Evidence for auditors.

### How a SIEM works

```
   Data sources                 SIEM pipeline                    Outputs
   ------------                 -------------                    -------
   Endpoints    \
   Servers       \
   Firewalls      \   collect    parse and      enrich and     alerts
   Proxies         >  -------->  normalise  ->  correlate  ->  dashboards
   Cloud / SaaS   /   (agents,   (tidy into     (add context,  investigations
   Identity      /    forwarders, consistent    apply rules)   reports
   Applications /     APIs,       fields)                      stored logs
                      syslog)
```

### SIEM compared to nearby tools

| Tool | What it covers | How it relates to SIEM |
|---|---|---|
| **SIEM** | Anything that produces a log | The central place where it all comes together |
| **EDR** | Endpoints only, but in great depth | A very valuable data source that feeds the SIEM |
| **XDR** | Endpoint, network, email, cloud, usually from one vendor | Overlaps with SIEM. Deeper detail, narrower coverage |
| **SOAR** | Automation | Takes SIEM alerts and runs automated playbooks |
| **UEBA** | Behaviour baselining | Now usually a SIEM feature rather than a separate product |
| **Log management** | Storage and search only | A SIEM without the security analysis on top |

---

## An analogy for SIEM

**A SIEM is the control room of a large airport.**

Every part of the airport produces its own stream of information. Radar, runway sensors, gate scanners, baggage handling, passport control, CCTV, ground crew radios. On its own each stream is routine noise. Nobody can watch all of them at once, and nobody would want to.

The control room does four things:

- **It brings every feed into one room.** That is log aggregation. Radar, boarding scans, and CCTV all arrive on the same wall of screens.
- **It translates everything into one language.** One system reports in local time, another in UTC. One says "stand 14", another says "gate B14". The control room reconciles all of it so controllers can compare like with like. That is normalisation.
- **It joins the dots between feeds.** A passenger checking in is normal. A bag being loaded is normal. A bag loaded onto a plane whose passenger never boarded is a serious problem, and you can only see it by comparing two separate systems. That is correlation, and it is the entire point of a SIEM.
- **It keeps the recordings.** When something goes wrong, investigators replay the tapes to reconstruct exactly what happened and in what order. That is retention and investigation.

The rest maps over neatly:

- **Sensors and cameras** are your log sources.
- **Air traffic controllers** are your SOC analysts. The control room does not fly the planes. It gives humans the picture they need to decide.
- **The alarm when two aircraft are converging** is a correlation rule firing.
- **Nuisance alarms**, a flock of birds tripping the same sensor for the tenth time this week, are false positives. Enough of them and controllers start ignoring the alarm entirely, which is exactly how alert fatigue kills real detections.
- **Emergency services on standby** are incident response. SOAR is the automated part, the barrier that lowers itself without anyone pressing a button.
- **The corner of the apron with no camera** is a missing log source. The control room is only as good as its coverage.
- **You cannot keep every camera at full resolution forever**, so you decide deliberately what to keep and for how long. That is your data ingest licensing and retention tiers.

The most important thing the analogy captures: **a control room prevents nothing by itself.** It provides visibility and coordination. If nobody is watching, if the alarms are badly set up, or if half the airport has no sensors, it is just an expensive room full of monitors. Exactly the same is true of a SIEM, which is why tuning, coverage, and staffing matter far more than which logo is on the product.

*Shorter versions if you need one in a sentence:* a SIEM is the **flight recorder plus cockpit warning system** for your network. Or, it is the **detective who interviews every witness in the building, notices that three unrelated stories only make sense if the same person was involved, and keeps the case file for seven years.**

---

## Examples of SIEM software in 2026

The market has consolidated a lot. Two acquisitions in particular reshaped it: Cisco's acquisition of Splunk, and Palo Alto's acquisition of IBM QRadar's software assets. The broader trend is convergence, with SIEM, XDR, SOAR, and UEBA merging into single platforms rather than separate products.

### Enterprise platforms

| Product | Vendor | Notes |
|---|---|---|
| **Splunk Enterprise Security** | Cisco | Market share leader and the reason this repo exists. Very flexible search language (SPL) and a huge library of add-ons. Best suited to teams willing to build and tune their own content. Historically expensive at high data volumes. |
| **Microsoft Sentinel** | Microsoft | Cloud-only, pay for what you use, tightly integrated with Microsoft 365 and Entra ID. Often the default choice in Microsoft-heavy organisations. Uses KQL rather than SPL. |
| **Google Security Operations** (was Chronicle) | Google | Built on Google's own data infrastructure. Priced per user rather than per gigabyte, which suits very large log volumes. |
| **Cortex XSIAM** | Palo Alto Networks | Marketed as an AI-driven replacement for traditional SIEM, combining SIEM, XDR, and SOAR. Also where QRadar customers are being migrated. |
| **IBM QRadar** | IBM / Palo Alto | Still widely installed on-premises, but the software has moved to Palo Alto. Mostly a migration story now. |
| **CrowdStrike Falcon Next-Gen SIEM** | CrowdStrike | Appealing where Falcon endpoint agents are already deployed. |

### Analytics-led specialists

| Product | Notes |
|---|---|
| **Exabeam** | Strong behaviour analytics heritage. Merged with LogRhythm, so the product lines are converging. |
| **Securonix** | Cloud-native and analytics-first. Lets you use your own cloud storage to control cost. |
| **Sumo Logic Cloud SIEM** | Cloud-native, popular with engineering-led teams. |
| **Devo** | Fast searching at large scale, keeps data quickly searchable for longer. |

### Open source and budget options (good for home labs)

| Product | Notes |
|---|---|
| **Wazuh** | Free and open source. Combines endpoint monitoring, log analysis, and compliance. Very popular for learning. |
| **Elastic Security** | Built on the Elastic Stack. Free tier available, paid tiers for advanced detection. |
| **Graylog Security** | Approachable log management with a security layer on top. |
| **Security Onion** | Free Linux distribution bundling Zeek, Suricata, Elastic, and hunting tools. Excellent for practice. |

> **How to actually compare these:** every one of them does log collection, threat detection, and compliance reporting. The real differences are how it is deployed, how it is priced, how much tuning effort it demands, and whether it matches your team's experience level. Splunk and QRadar reward experienced teams who can tune rules and handle high alert volumes. Cloud-native platforms usually deploy faster with less maintenance.

---

# Part 3: Splunk

## What is Splunk?

**Splunk is a platform for collecting, storing, and searching machine data.**

**Machine data** means the records that computers write automatically as they run. Every time someone logs in, a website is visited, a file is saved, an app crashes, or a firewall blocks a connection, something somewhere writes a line of text about it. Those lines are scattered across thousands of separate systems in different formats, and on their own they are unreadable.

Splunk takes all of that, puts it in one place, gives it a common structure, and lets you search it with a query language. A useful shorthand: **Splunk is a search engine for log data**. People often describe it as "Google for your logs", which is close enough to be a helpful starting point.

The important thing to understand early is that **Splunk is not only a security tool**. It is a general purpose data platform. Security is its biggest and best known use case, delivered through an add-on product called **Splunk Enterprise Security**, but the same underlying engine is used for IT troubleshooting, website performance, and business reporting.

A short history worth knowing: Splunk was founded in 2003, went public in 2012, and was acquired by **Cisco in 2024**. It is one of the longest-standing leaders in the SIEM market.

---

## What can Splunk be used for, and why use it?

### Main use cases

| Use case | What it looks like |
|---|---|
| **Security (SIEM)** | Detecting attacks, investigating alerts, threat hunting, compliance reporting. Powered by Splunk Enterprise Security. |
| **IT operations and troubleshooting** | Why did the payment service fail at 2am? Search every log from every server in one query rather than logging into twenty machines. |
| **Observability and monitoring** | Tracking application performance, error rates, and infrastructure health in real time. |
| **DevOps** | Monitoring builds, deployments, and releases. |
| **Business analytics** | Because logs record real user behaviour, they can answer business questions too: how many orders failed at checkout, which regions are growing, how long deliveries take. |
| **Fraud detection** | Spotting unusual transaction patterns using the same correlation techniques as security detection. |
| **Compliance and audit** | Proving to an auditor that logs were kept, monitored, and reviewed. |

### Why organisations choose it

- **It takes almost any data.** Splunk does not require you to define a rigid database structure before loading data in. It handles messy, unstructured text, which most other databases hate. This is the single biggest reason it gets used.
- **Schema-on-read.** Traditional databases force you to decide the structure when you store data (schema-on-write). Splunk applies structure when you *search* it instead. In practice this means you can load data first and work out what the fields mean later, and you can reinterpret old data with new logic without reloading it.
- **Speed at scale.** It handles terabytes per day and still returns searches quickly.
- **Real time.** Alerts can fire within seconds of an event arriving.
- **A very large ecosystem.** Splunkbase has thousands of free apps and add-ons for common products, so getting data from a Palo Alto firewall or AWS is usually a matter of installing something rather than writing code.
- **One platform, many teams.** Security, IT, and business teams can all use the same data.

### Honest drawbacks

- **Cost.** Traditional licensing charges by how many gigabytes you send per day, which gets expensive fast. Newer workload-based pricing exists but cost management is still a real skill.
- **Learning curve.** SPL is powerful but takes time.
- **It needs care.** Splunk rewards teams who invest in tuning and data onboarding. It is not a set-and-forget appliance.

---

## What is a Splunk / SOC analyst?

A **SOC analyst** is the person who monitors alerts, investigates suspicious activity, and escalates or responds to real threats. A **Splunk analyst** is a SOC analyst whose main working tool is Splunk, which given its market share is extremely common.

### What the job actually involves day to day

- Working through the alert queue in Splunk Enterprise Security, where alerts are called **notable events**
- Deciding whether each one is real or a false alarm, following a runbook
- Writing SPL searches to answer investigation questions such as "what else did this user do in the hour before this alert?"
- Pivoting between data sources: the alert came from the firewall, so now check the endpoint logs, then the authentication logs
- Documenting findings in a ticket and escalating what needs escalating
- Suggesting tuning changes when a rule is too noisy
- Building dashboards and saved searches
- Over time, moving into writing detections rather than only consuming them

### Skills that matter

**Technical:** SPL, log analysis, Windows and Linux fundamentals, networking basics (TCP/IP, DNS, HTTP), understanding of common attack techniques, and familiarity with MITRE ATT&CK.

**Non-technical, and genuinely just as important:** curiosity, attention to detail, clear written communication (your investigation notes are read by other people under pressure), and the discipline to stay methodical when everything is urgent.

### Career progression

Tier 1 analyst, then Tier 2, then a fork: **detection engineering** (writing the content), **incident response and forensics** (handling the worst cases), **threat intelligence**, **threat hunting**, or **Splunk engineering and administration** (running the platform itself). The platform route pays well and is often overlooked.

---

## Versions and products

The word "version" covers two things here: the different Splunk *products*, and the software *release numbers*. The products are what matters most.

### Products

| Product | What it is | Typical use |
|---|---|---|
| **Splunk Enterprise** | The full platform, installed on your own servers (**on-premises**) or your own cloud infrastructure. You manage everything. | Organisations that need full control, or have data that cannot leave their premises. |
| **Splunk Cloud Platform** | The same platform, run and maintained by Splunk as a service (**SaaS**). Splunk handles upgrades and infrastructure. | Most new customers. Faster to start, less to maintain, slightly less freedom to install anything you like. |
| **Splunk Free** | A free licence of Splunk Enterprise, limited to 500MB of data per day. No authentication, no alerting, no clustering. | Learning and home labs. This is the one to install while studying. |
| **Splunk Enterprise Security (ES)** | A **premium app** that sits on top of Splunk Enterprise or Cloud and turns it into a full SIEM. Adds correlation searches, notable events, risk-based alerting, incident review, and threat intelligence. | The actual SOC product. When someone says "we use Splunk as our SIEM", they usually mean ES. |
| **Splunk SOAR** | Automation and playbooks. Formerly called Phantom. | Automating repetitive response actions. |
| **Splunk Observability Cloud** | Metrics, traces, and application monitoring. | IT and engineering teams rather than security. |
| **Splunk UBA** | User Behaviour Analytics, machine learning to spot unusual behaviour. | Insider threat and account compromise detection. |
| **Splunk Attack Analyzer** | Automated analysis of suspicious files and URLs. | Phishing and malware triage. |

Two you may see referenced in older material: **Splunk Light**, a cut-down version, which was retired; and **Splunk Phantom**, which was renamed Splunk SOAR.

### Release numbers

Splunk Enterprise is currently on the **9.x** line. The main thing to know is that release numbers matter for compatibility: Enterprise Security requires a minimum Splunk Enterprise version, and Universal Forwarders should generally not be newer than the indexers they send to. Always check the compatibility matrix in the docs before upgrading anything.

---

## Splunk architecture

Splunk splits its work across three main roles. On a laptop install, one program does all three. In a real deployment they run on separate machines.

```
   DATA SOURCES              COLLECTION            STORAGE            USERS
   ------------              ----------            -------            -----

   Servers      [UF] \
   Laptops      [UF]  \
   Firewalls  syslog   \                        +-----------+     +-------------+
   Cloud APIs           ----> Heavy Forwarder -->  INDEXERS  <---->  SEARCH HEAD  <-- analysts
   Apps        [HEC]   /      (optional)         +-----------+     +-------------+
   Databases           /                          parse, store,      run searches,
                      /                           and index data     dashboards, alerts

   [UF]  = Universal Forwarder (small agent installed on the machine)
   [HEC] = HTTP Event Collector (apps send data directly over HTTP)
```

### Universal Forwarder (UF)

A **small, lightweight agent installed on the machine that produces the logs**. Its only job is to pick up data and send it on to the indexers. It uses very little CPU and memory, which matters when you are installing it on thousands of production servers.

Key point: a Universal Forwarder does almost no processing. It just ships data. That is deliberate, so it does not slow down the machine it is monitoring.

There is also a **Heavy Forwarder (HF)**, which is a full Splunk install configured to forward. It *can* parse, filter, and route data before sending it on. Use one when you need to drop unwanted data before it costs you licence, mask sensitive fields, or pull data from an API that requires an app to be installed.

### Indexer

The **indexer receives the data, processes it, and stores it**. This is where the heavy lifting happens.

What it does on arrival:
1. Breaks the incoming stream into individual **events** (usually one per line)
2. Identifies the **timestamp** for each event
3. Assigns metadata: `host`, `source`, `sourcetype`
4. Compresses the raw data and builds an index of the terms in it, so searching is fast
5. Writes it into an **index**, which is stored on disk in files called **buckets**

Indexers also run the searches against their own data and send results back to the search head. In larger environments, multiple indexers form an **indexer cluster**, which spreads data across machines and keeps duplicate copies so nothing is lost if one dies. That duplication is controlled by the **replication factor** (how many copies of the raw data) and **search factor** (how many copies are searchable).

### Search Head

The **search head is what users actually log into**. It is the web interface where you type SPL, view dashboards, and manage alerts.

It does not store the data. When you run a search, the search head sends the request to all the indexers, they each search their own data in parallel, and the search head merges the results and presents them. This is called **distributed search**, and it is why Splunk scales.

### Supporting components

| Component | Purpose |
|---|---|
| **Deployment Server** | Pushes configuration out to hundreds or thousands of forwarders centrally, so you are not editing config files by hand on every server. |
| **Cluster Manager** (formerly master node) | Coordinates an indexer cluster and manages replication. |
| **Deployer** | Pushes apps and configuration to a search head cluster. |
| **Licence Manager** | Tracks how much data you are ingesting against your licence. |
| **Monitoring Console** | Built-in dashboards showing the health of your Splunk environment itself. |

---

## Deployment options

Deployments scale up in stages as data volume and user count grow.

**1. Single instance (standalone)**

Everything (forwarding, indexing, searching) runs on one machine. This is what you get when you install Splunk on a laptop. Fine for learning, testing, and very small environments.

**2. Distributed deployment**

One search head, several indexers. The most common starting architecture for a real organisation. Searching and storing are separated so each can be scaled independently.

**3. Search Head Cluster (SHC)**

Multiple search heads working as a group, with a minimum of three members. They share configuration, saved searches, and dashboards automatically, and a **captain** coordinates them. This gives you two things: **high availability**, meaning the system keeps working if one search head fails, and **load balancing** across more concurrent users. Needed once you have a large analyst team all running searches at once.

**4. Indexer clustering**

Multiple indexers holding replicated copies of the data, so no data is lost if a machine fails.

**5. Multi-site clustering**

Clusters spread across data centres or regions, for disaster recovery and to keep searches local to users.

**6. Splunk Cloud Platform**

Splunk runs all of the above for you. You get a search head and send data to their indexers. Less operational work, less flexibility, and you cannot install absolutely anything you want.

**7. Hybrid**

Some data indexed on-premises, some in Splunk Cloud, searched together. Common where regulations require certain data to stay in a specific country.

---

## Basic Splunk terms

Learn these first. Almost every Splunk conversation uses them.

### Data terms

| Term | Meaning |
|---|---|
| **Event** | A single record. Usually one line of a log file. The basic unit of everything in Splunk. |
| **Index** | Where events are stored. Like a folder or database table. You often search a specific index, for example `index=wineventlog`. Splitting data into indexes also controls who can see what. |
| **Sourcetype** | The *format* of the data, which tells Splunk how to interpret it. For example `access_combined` for Apache web logs, or `WinEventLog:Security`. Getting sourcetypes right at onboarding is one of the most important things you can do. |
| **Source** | Where the data came from specifically, usually a file path or port, such as `/var/log/auth.log`. |
| **Host** | The machine that produced the event. |
| **Field** | A name and value pair extracted from an event, such as `user=jsmith` or `status=404`. Fields are what you search and calculate on. |
| **_time** | The timestamp field. Splunk is built around time, and every event has one. |
| **_raw** | The original, unmodified text of the event. |
| **Bucket** | The physical directory an index is stored in. Buckets age through stages: **hot** (being written to), **warm** (recent, searchable), **cold** (older, on cheaper disk), **frozen** (deleted or archived), **thawed** (restored from archive). |

### Search and knowledge terms

| Term | Meaning |
|---|---|
| **SPL** | Search Processing Language. Splunk's query language. |
| **Knowledge object** | Anything you create to make raw data more useful: field extractions, eventtypes, tags, lookups, macros, data models. |
| **Field extraction** | A rule that pulls a value out of raw text and names it as a field. |
| **Eventtype** | A saved search condition given a name, so you can categorise events. |
| **Tag** | A label applied to a field value, letting you group things across different data sources. |
| **Lookup** | A file or external source used to enrich events, for example a CSV mapping machine names to their owner and criticality. |
| **Macro** | A reusable chunk of SPL, so common logic can be written once and called by name. |
| **Data model** | A structured layer over your data that powers fast reporting and the `tstats` command. The CIM is built from data models. |
| **CIM** | Common Information Model. Splunk's standard field naming scheme, which lets one search work across many vendors' logs. Essential for Enterprise Security. |
| **Saved search / report** | A search stored for reuse or scheduled to run automatically. |
| **Alert** | A saved search that runs on a schedule and takes an action when its condition is met. |
| **Notable event** | An alert inside Splunk Enterprise Security, appearing in the analyst's incident review queue. |
| **Dashboard** | A page of visual panels built from searches. |
| **App** | A packaged bundle of Splunk content. |
| **Add-on (TA)** | A bundle focused on getting data in and normalising it, rather than on displaying it. |

---

## What data does Splunk ingest?

Splunk's main selling point is that it accepts **almost any text-based machine data**, without you having to define its structure in advance.

### By format

- **Plain text log files.** Most common. Web server logs, application logs, syslog.
- **Structured formats.** JSON, XML, CSV. Splunk parses these automatically.
- **Windows Event Logs.** Collected via the Universal Forwarder.
- **Syslog.** The standard log format from network devices, sent over UDP or TCP port 514.
- **Metrics.** Numeric time-series data, stored in a special metrics index for efficiency.
- **Database records.** Via the Splunk DB Connect add-on.
- **API and cloud data.** Pulled directly from AWS, Azure, GCP, Microsoft 365, Okta, and similar.
- **Scripted output.** The result of running a script, for example a command that lists open ports.

### By source, in a typical SOC

| Source | Why it matters |
|---|---|
| Windows Security event logs | Logins, account changes, privilege use |
| Sysmon | Detailed process, network, and file activity on endpoints |
| Linux `auth.log` and `syslog` | Logins, sudo use, service activity |
| EDR alerts and telemetry | Endpoint threat detection |
| Firewall logs | Allowed and blocked connections |
| Web proxy logs | Which websites users visited |
| DNS logs | Every domain lookup, which is excellent for spotting malware |
| VPN logs | Remote access, geography |
| Email gateway logs | Phishing |
| Cloud audit logs (AWS CloudTrail, Azure Activity, GCP) | Who changed what in the cloud |
| Identity provider logs (Entra ID, Okta) | Authentication and MFA |
| Application logs | Business-specific activity |

Note what Splunk generally does **not** ingest: video, images, and binary files. It is built for text and numbers.

---

## How does data get into Splunk?

Getting data in properly is the single most important stage. Bad onboarding produces wrong timestamps, missing fields, and detections that silently do not work.

### The methods

| Method | How it works | When to use it |
|---|---|---|
| **Universal Forwarder** | Small agent installed on the source machine, monitors files and Windows event logs, ships data to indexers. | The default for servers and endpoints. |
| **Heavy Forwarder** | Full Splunk install that can parse, filter, and route before forwarding. | When you need to filter out data to save licence cost, mask sensitive fields, or run an add-on that pulls from an API. |
| **Monitor input** | Splunk watches a file or folder and reads new lines as they appear. | Local log files. |
| **Network input** | Splunk listens on a TCP or UDP port, typically for syslog. | Network devices that cannot run an agent, such as firewalls and switches. Best practice is to put a syslog server in front rather than pointing devices straight at Splunk. |
| **HTTP Event Collector (HEC)** | Applications send events directly to Splunk over HTTP using a token for authentication. | Modern applications, containers, and cloud services. No agent needed. |
| **Scripted input** | Splunk runs a script on a schedule and ingests its output. | Data with no native log file. |
| **Modular input** | A packaged input, usually installed as an add-on, that pulls from an API. | Cloud services such as AWS, Azure, and Okta. |
| **Upload** | Manually upload a single file through the web interface. | Testing, one-off analysis, and learning. |

### What happens to data on arrival: the pipeline

1. **Input.** Data is collected and tagged with `host`, `source`, and `sourcetype`.
2. **Parsing.** The stream is split into individual events, timestamps are identified and converted, and character encoding is handled.
3. **Indexing.** Events are compressed, an index of their terms is built, and everything is written to disk in buckets.
4. **Searching.** When you run a search, fields are extracted from the raw text at that moment. This is **schema-on-read**, and it means you can change how you interpret data later without re-ingesting it.

### What are events?

An **event is a single record of something that happened**, and it is the fundamental unit of data in Splunk. Usually it is one line from a log file.

A raw event looks like this:

```
2026-08-13 09:14:22 host=WEB01 user=jsmith src_ip=10.4.2.19 action=login status=failure
```

Splunk stores that raw text and attaches:

- **`_time`**: the timestamp, `2026-08-13 09:14:22`, parsed into a proper time value
- **`host`**: which machine sent it
- **`source`**: which file or input it came from
- **`sourcetype`**: what format it is, which tells Splunk how to read it
- **Extracted fields**: `user=jsmith`, `src_ip=10.4.2.19`, `action=login`, `status=failure`

Once those fields exist, you can search, filter, count, and chart on them.

Two things frequently go wrong with events, and both are worth knowing about early:

- **Line breaking.** Splunk has to decide where one event ends and the next begins. Usually one line equals one event, but stack traces and XML can span many lines, so this sometimes needs configuring manually.
- **Timestamp recognition.** If Splunk picks the wrong timestamp, or gets the timezone wrong, events land in the wrong place in time and your searches quietly miss them. This is one of the most common onboarding problems.

---

## What is SPL?

**SPL (Search Processing Language)** is Splunk's query language. It is how you ask questions of your data.

The core idea is the **pipe**, written `|`. Each command passes its results to the next command, which transforms them further. If you have used a Linux command line, this will feel familiar.

```
search terms | command1 | command2 | command3
   |              |          |          |
   find the       then       then       then
   events         reshape    calculate  display
```

Everything reads left to right, and a good habit is to **filter as early as possible**, because everything after that is working on a smaller set of data and runs faster.

A basic search has this shape:

```spl
index=web sourcetype=access_combined status=404
| stats count by uri_path
| sort - count
| head 10
```

Read as: find 404 errors in the web index, count them grouped by page, sort highest first, show the top ten.

---

## SPL examples

### Basic searches

```spl
# Everything in one index (fine for exploring, avoid on large data)
index=main

# Filter by field values
index=wineventlog EventCode=4625

# Multiple conditions with AND, OR, NOT
index=wineventlog EventCode=4625 AND user="administrator"
index=firewall (action=blocked OR action=denied) NOT src_ip=10.0.0.0/8

# Wildcards
index=web uri_path="/admin*"

# The IN operator, tidier than lots of ORs
index=wineventlog EventCode IN (4624, 4625, 4720)

# Search the raw text for a keyword
index=main "failed password"

# Time is normally set with the time picker, but can be typed
index=main earliest=-24h latest=now
index=main earliest=-7d@d latest=@d
```

### Basic transformations

```spl
# Count events grouped by a field
index=wineventlog EventCode=4625
| stats count by user

# Multiple statistics at once
index=web
| stats count, avg(response_time) AS avg_time, max(response_time) AS slowest BY host

# Count unique values (distinct count)
index=firewall
| stats dc(dest_ip) AS unique_destinations BY src_ip

# Quick frequency check of the most common values
index=web
| top limit=10 uri_path

# The least common values, useful in threat hunting because
# attacker activity is often rare rather than frequent
index=sysmon EventCode=1
| rare limit=20 process_name

# Create a new field with eval
index=web
| eval response_kb = round(bytes/1024, 2)

# Conditional logic
index=wineventlog
| eval severity = case(EventCode=4625, "low", EventCode=1102, "critical", true(), "info")

# Pick which columns to display
index=wineventlog EventCode=4625
| table _time, host, user, src_ip

# Rename a field for readability
| rename src_ip AS "Source IP"

# Filter after a transformation (where works on calculated fields, search does not)
index=wineventlog EventCode=4625
| stats count BY user
| where count > 50

# Deduplicate
index=main | dedup user

# Enrich with a lookup file
index=main
| lookup asset_inventory host OUTPUT owner, criticality

# Group events into time buckets, the basis of most time charts
index=web
| bin _time span=1h
| stats count BY _time
```

### Basic visualisations

```spl
# timechart automatically plots a metric over time
index=web
| timechart span=1h count BY status

# Compare failed logins per hour across hosts
index=wineventlog EventCode=4625
| timechart span=1h count BY host

# chart creates a table suited to bar and column charts
index=firewall
| chart count OVER src_ip BY action

# A single big number, good for dashboard headline panels
index=wineventlog EventCode=4625 earliest=-24h
| stats count AS "Failed Logins (24h)"

# Geographic map of source IP addresses
index=firewall action=blocked
| iplocation src_ip
| geostats count BY Country

# Pie chart data (choose the pie visualisation in the UI)
index=web
| stats count BY status
```

Once a search returns results, the **Visualization** tab in the Splunk UI lets you switch between line, column, pie, map, and gauge without changing the SPL. The rule of thumb is: `timechart` for anything over time, `chart` or `stats` for comparisons between categories.

### A worked SOC example

```spl
# Find accounts with many failed logins followed by a success,
# a classic brute force pattern
index=wineventlog EventCode IN (4624, 4625)
| stats count(eval(EventCode=4625)) AS failures,
        count(eval(EventCode=4624)) AS successes,
        values(src_ip) AS source_ips
        BY user
| where failures >= 10 AND successes > 0
| sort - failures
```

---

## What can you build in Splunk?

| Thing | What it is | Typical use |
|---|---|---|
| **Reports** | A saved search you can rerun or schedule. | Weekly failed login summary. |
| **Alerts** | A scheduled search that triggers an action when a condition is met. Actions include email, a webhook, running a script, or creating a notable event. | Notify the SOC when audit logs are cleared. |
| **Dashboards** | A page of panels driven by searches, with filters (**inputs**) so viewers can change the time range or pick a host. | A SOC overview screen. |
| **Reports with drilldown** | Clicking a chart runs a more detailed search. | Click a spike, see the underlying events. |
| **Data models and accelerated reports** | Pre-structured, pre-computed data for very fast searching using `tstats`. | Powering Enterprise Security dashboards. |
| **Lookups** | Reference tables that add context. | Mapping IP ranges to office locations. |
| **Knowledge objects** | Field extractions, eventtypes, tags, macros. | Making raw data usable for everyone. |
| **Custom apps** | Packaged dashboards, searches, and configuration you can share or reuse. | A team's own toolkit. |
| **Correlation searches** (ES) | The detection rules in Enterprise Security that generate notable events. | The core of SIEM detection content. |
| **Risk-based alerting** (ES) | Assigns risk scores to users and machines, and alerts when a total crosses a threshold rather than on each individual signal. | Cutting alert volume dramatically while catching slow-burn attacks. |
| **Glass tables** (ES) | Visual diagrams of a system with live metrics overlaid. | Executive and service-health views. |
| **SOAR playbooks** | Automated response sequences. | Auto-disable an account, auto-enrich an IP. |

**Dashboard Studio** is the current dashboard builder, offering more design control than the older **Simple XML** dashboards. You will meet both in the wild.

---

## Apps vs add-ons

Both are downloaded from **Splunkbase**, Splunk's app store, and both are technically just folders of configuration. The difference is what they are for.

| | **App** | **Add-on (TA)** |
|---|---|---|
| **Purpose** | Provides a user interface and analysis | Gets data in and cleans it up |
| **Has a UI?** | Yes, dashboards and views | Usually not, it works in the background |
| **Contains** | Dashboards, reports, searches, navigation | Inputs, field extractions, sourcetype definitions, CIM mapping |
| **Answers the question** | "How do I look at this data?" | "How do I collect this data and name its fields correctly?" |
| **Examples** | Splunk Enterprise Security, Splunk Security Essentials, Splunk App for AWS | Splunk Add-on for Microsoft Windows, Splunk Add-on for AWS, Palo Alto Networks Add-on |

**TA** stands for Technology Add-on, and the naming convention is usually `Splunk_TA_<vendor>`.

In practice they work together. To monitor AWS, you install the **add-on** to pull the data in and normalise the field names, and optionally the **app** to get ready-made dashboards on top of it. Add-ons usually need to be installed in several places (forwarder, indexer, and search head) depending on what part of the job they do, which trips people up early on.

There is also **Splunk Security Essentials**, a free app worth knowing about: it contains hundreds of example detections mapped to MITRE ATT&CK, and is one of the better free learning resources available.

---

## Securing data in Splunk

Splunk holds a complete record of everything that happens in an organisation, which makes it an extremely attractive target. An attacker with Splunk admin access can read your data and, worse, delete the evidence of their own activity.

### Access control

- **Use role-based access control (RBAC).** Splunk ships with `admin`, `power`, and `user` roles. Create your own roles rather than handing out `admin`.
- **Restrict access at the index level.** Roles can be limited to specific indexes, so the HR log index is not visible to the whole company.
- **Use search filters per role.** A role can have a filter automatically appended to every search it runs, restricting it to certain hosts or field values.
- **Least privilege.** Give people the minimum they need. Very few people need to install apps or change configuration.
- **Integrate with SSO.** Connect to LDAP, Active Directory, or SAML so accounts are managed centrally and disabled when people leave. Enforce **MFA**.
- **Change all default credentials** on installation.

### Protecting the data itself

- **Encrypt data in transit.** Enable TLS between forwarders and indexers, and on the web interface. This is not on by default in older setups.
- **Encrypt data at rest.** Usually handled with disk-level encryption.
- **Mask sensitive data at index time.** Use `SEDCMD` in `props.conf` to strip or obscure card numbers, national insurance numbers, and passwords *before* they are written to disk. Once it is indexed, it is very hard to remove selectively.
- **Filter out what you do not need.** Data you never ingest cannot leak, and does not cost licence either.

### Protecting the platform

- **Secure the management port** (8089) and restrict network access to Splunk components.
- **Keep Splunk patched.** It is software with vulnerabilities like any other.
- **Monitor Splunk with Splunk.** The internal `_audit` index records who searched for what, who changed configuration, and who logged in. Alert on suspicious admin activity, especially anyone disabling logging or deleting data.
- **Protect HEC tokens** as credentials, and rotate them.
- **Back up configuration** as well as data, including your knowledge objects and apps.
- **Set retention deliberately.** Regulations often set a minimum, and privacy law sometimes sets a maximum.

---

## Case studies

Splunk publishes customer stories at [splunk.com/en_us/customers.html](https://www.splunk.com/en_us/customers.html). The patterns below are the common ones you will see referenced, described generically so you can recognise the shape of each use case.

### Security and SOC

- **Financial services fraud and threat detection.** Banks correlate login activity, transaction records, and device data to spot account takeover in real time. The same platform serves both the fraud team and the security team, which is a common driver for choosing Splunk.
- **Ransomware detection.** Detecting mass file modification and shadow copy deletion within minutes rather than discovering it when staff arrive in the morning.
- **Insider threat.** Correlating HR data (such as a resignation date) with file access and USB usage to spot data being taken on the way out.
- **Government and defence.** Long log retention for compliance combined with threat hunting across very large estates.
- **Healthcare.** Monitoring access to patient records for both security and regulatory reasons, where inappropriate access by an authorised user is the main risk.

### IT operations

- **Retail and e-commerce uptime.** Tracing a checkout failure across web servers, payment gateways, and databases in a single search rather than logging into each system separately. During peak trading periods, minutes of downtime are extremely expensive, so mean time to resolution is the metric that matters.
- **Telecoms network monitoring.** Correlating equipment logs across huge geographic networks to find faults before customers report them.

### Business and data analysis

- **Delivery and logistics.** Using operational logs to track order flow and delivery times, and to spot bottlenecks in real time.
- **Customer experience.** Analysing web and app logs to find where users abandon a purchase, which is a business question answered with the same log data the security team uses.
- **Manufacturing and IoT.** Ingesting sensor data from equipment to predict failures before they cause a shutdown.

### The pattern worth noticing

The recurring theme is that **one platform serves multiple teams**. The security team, the IT team, and the business intelligence team are often all querying the same underlying data for different reasons. That shared-value argument is usually how Splunk gets justified financially, given the licence cost.

---

## Certification path

Certifications are only part of the picture. A portfolio of your own detections, lab notes, and worked investigations carries at least as much weight with employers. That said, certs get you past HR filters.

Exam codes and names do change, so always confirm the current list at [splunk.com/training](https://www.splunk.com/en_us/training/learning-paths-certifications.html).

### Core Splunk track

| Certification | Code | Level | Covers |
|---|---|---|---|
| **Splunk Core Certified User** | SPLK-1001 | Entry | Basic searching, fields, lookups, simple reports and dashboards. Assumes no prior knowledge. **Start here.** |
| **Splunk Core Certified Power User** | SPLK-1002 | Intermediate | Deeper searching and reporting, plus creating knowledge objects such as eventtypes, tags, and data models. This is the practical working level for an analyst. |
| **Splunk Core Certified Advanced Power User** | SPLK-1004 | Advanced | Complex searches, search optimisation, advanced dashboards. |

### Administration and architecture

| Certification | Code | Covers |
|---|---|---|
| **Splunk Enterprise Certified Admin** | SPLK-1003 | Installing and managing Splunk, data inputs, users, licensing, configuration files. |
| **Splunk Cloud Certified Admin** | SPLK-1005 | The same for Splunk Cloud. |
| **Splunk Enterprise Certified Architect** | SPLK-2002 | Designing and scaling large distributed deployments. Requires the Admin cert first. |

### Security track, most relevant for SOC work

| Certification | Code | Covers |
|---|---|---|
| **Splunk Certified Cybersecurity Defense Analyst** | SPLK-5001 | The analyst-focused security cert: threat detection, investigation, and using Splunk in a SOC. The most directly relevant to a SOC analyst role. |
| **Splunk Certified Cybersecurity Defense Engineer** | SPLK-5002 | Building detections, tuning, and engineering security content. |
| **Splunk Enterprise Security Certified Admin** | SPLK-3001 | Installing and configuring Enterprise Security itself, including correlation searches, risk analysis, and threat intelligence. |
| **Splunk SOAR Certified Automation Developer** | SPLK-2003 | Building automation playbooks. |

### A sensible order for a SOC career

```
Core Certified User  ->  Core Certified Power User  ->  Cybersecurity Defense Analyst
                                     |
                                     +-> (platform route) Enterprise Certified Admin
                                                                    |
                                                          Enterprise Security Admin
```

### Broader SOC certifications worth knowing

Splunk certs prove you can drive the tool. These prove you understand the job:

| Certification | Notes |
|---|---|
| **CompTIA Security+** | The standard entry-level security qualification. Often an HR requirement. |
| **CompTIA CySA+** | Specifically about security analysis and monitoring. Sits well beside Splunk certs. |
| **Blue Team Level 1 (BTL1)** | Practical, hands-on, well regarded for junior SOC roles. |
| **Microsoft SC-200** | Security Operations Analyst, focused on Sentinel and Defender. Useful if the employer is Microsoft-heavy. |
| **HTB Certified Defensive Security Analyst (CDSA)** | Hands-on, exam involves a real investigation. |
| **GIAC GCIH / GCIA** | Incident handling and intrusion analysis. Expensive but highly respected. |

### Free ways to build experience

- **Splunk Free** licence, 500MB a day, installed locally
- **Splunk Search Tutorial** and free eLearning on Splunk's own site
- **Boss of the SOC (BOTS)** datasets, free realistic security data with an investigation scenario attached. The best available way to practise Splunk for security specifically.
- **Splunk Security Essentials** app, hundreds of example detections mapped to MITRE ATT&CK
- **TryHackMe** has a Splunk and SOC Level 1 path
- **Security Onion** or **Wazuh** in a home lab to generate your own data

---

## Glossary

| Term | Meaning |
|---|---|
| **Add-on (TA)** | Splunk package that collects and normalises data, usually with no user interface. |
| **App** | Splunk package that provides dashboards and analysis views. |
| **Bucket** | The directory on disk where indexed data is stored. Ages hot, warm, cold, frozen. |
| **CIM** | Common Information Model. Splunk's standard field naming scheme. |
| **DFIR** | Digital Forensics and Incident Response. |
| **Dwell time** | How long an attacker was present before being detected. |
| **Endpoint** | An individual device such as a laptop or server. |
| **Event** | A single record of something that happened. Splunk's basic unit of data. |
| **HEC** | HTTP Event Collector. Lets applications send data straight to Splunk over HTTP. |
| **Index** | Where Splunk stores events. |
| **IOC** | Indicator of Compromise, for example a known-bad IP, domain, or file hash. |
| **Knowledge object** | Anything you create to make data more useful: fields, eventtypes, tags, lookups, macros, data models. |
| **LOLBin** | Living Off the Land Binary. A legitimate system tool abused by an attacker. |
| **Machine data** | The records computers write automatically as they operate. |
| **MDR / MSSP** | Outsourced detection and response, or outsourced security services generally. |
| **MITRE ATT&CK** | Public catalogue of attacker tactics and techniques. |
| **MTTD / MTTR** | Mean Time To Detect / Mean Time To Respond. |
| **Notable event** | An alert in Splunk Enterprise Security requiring analyst review. |
| **On-premises** | Software running on your own servers rather than a vendor's cloud. |
| **RBA** | Risk-Based Alerting. Builds up a risk score across many small signals instead of alerting on each one. |
| **Runbook** | Step by step instructions for handling a specific alert type. |
| **Schema-on-read** | Applying structure to data when you search it, not when you store it. Splunk's defining characteristic. |
| **SOAR** | Security Orchestration, Automation and Response. |
| **Sourcetype** | The format of a set of data, telling Splunk how to interpret it. |
| **SPL** | Search Processing Language. Splunk's query language. |
| **Sysmon** | Free Microsoft tool that produces detailed Windows endpoint logs. |
| **Telemetry** | The activity data a system automatically records about itself. |
| **TTP** | Tactics, Techniques and Procedures. How an attacker operates. |
| **UEBA** | User and Entity Behaviour Analytics. |
| **Universal Forwarder** | Lightweight Splunk agent installed on machines to ship their logs. |


