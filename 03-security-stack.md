# 03 — Security Stack

This guide documents the security monitoring architecture used in Project Deathstar.

The Deathstar security stack is designed to give visibility across three major areas:
- network traffic
- host activity
- deception-based signals

Rather than relying on a single tool, the lab uses multiple telemetry sources and a central SIEM workflow so alerts can be reviewed, correlated, and eventually escalated into incident-handling workflows.

## Stack Overview

| Service | ID | Role |
|---|---|---|
| Suricata | CT100 | Network intrusion detection |
| Splunk Enterprise | VM200 | SIEM, dashboards, searching, alerting |
| OpenCanary | CT101 | Honeypot and deception telemetry |
| Wazuh | CT102 | Host monitoring, FIM, vulnerability visibility |
| TheHive | CT104 | Case management and incident workflow |

## Security Design Goals

The stack is built to support:
- visibility into suspicious network behavior
- centralized collection and review of security events
- host-level change detection and endpoint monitoring
- high-signal alerting from honeypot interactions
- repeatable investigation and triage practice
- a blue-team workflow that can grow over time

## Alert Flow

```text
Network traffic
    → Suricata CT100
        → eve.json
            → Splunk VM200
                → TheHive CT104

Host events
    → Wazuh agents
        → Wazuh CT102
            → Splunk VM200

Honeypot hits
    → OpenCanary CT101
        → opencanary logs
            → Splunk VM200
```

## How the Components Fit Together

### Suricata
Suricata provides network-layer visibility. In the Deathstar design, it is positioned to watch traffic relevant to the lab and generate event data for analysis.

Key functions:
- inspect network traffic
- generate IDS alerts
- produce structured event output
- feed detections into the SIEM pipeline

From the top-level project description, Suricata is already active and using a large rule set. That makes it the primary network detection sensor in the lab.

### Splunk
Splunk is the central analysis platform and the heart of the current monitoring workflow.

Key functions:
- collect telemetry from multiple sources
- normalize and search event data
- provide dashboards and basic alerting
- act as the main place for triage and investigation

Splunk matters here because it turns separate tools into one usable workflow instead of a pile of logs nobody reads.

### OpenCanary
OpenCanary adds deception-based signal generation. Honeypots are useful in a lab because they create highly interesting events when touched.

Key functions:
- emulate exposed services
- generate alerts when decoys are probed or accessed
- provide high-confidence signals for review in Splunk

In a segmented environment, OpenCanary is especially useful for validating whether attack simulation or unexpected access attempts are reaching systems they should not.

### Wazuh
Wazuh is the host-visibility layer.

Key functions:
- file integrity monitoring
- agent-based host visibility
- vulnerability visibility
- central endpoint telemetry collection

This gives the lab something that network-only tools cannot: awareness of what is changing on the systems themselves.

### TheHive
TheHive is the incident and case-management layer.

Key functions:
- track investigations
- document analyst actions
- move from alert review toward a structured IR workflow
- support repeatable response habits

Even in a home lab, this is useful because it shifts the mindset from “I saw an alert” to “I worked a case.”

For Project Deathstar, the current design target is CT104: a dedicated Debian 12 Proxmox LXC on LAB_DMZ (20) running a standalone TheHive deployment. StrangeBee’s current package-install documentation supports Debian 12, Java 11, Cassandra 4.1.x, and Elasticsearch 7.11.x through 9.1.x for recent TheHive 5 releases. Their hardware guidance for small package-based deployments starts around 3 CPU cores / 4 GB RAM per major service at under 10 concurrent users, with 100 GB storage recommended for most use cases.

That makes CT104 a good fit as its own container rather than something bolted onto an existing node. In Deathstar, TheHive should sit behind the internal management/access model, receive escalated alerts from Splunk-driven workflows, and act as the place where detections become analyst notes, tasks, and case records.

## Current Active Components

Based on the repository README, these components are already active:

| Service | Status |
|---|---|
| Suricata | Active |
| Splunk | Active |
| OpenCanary | Active |

These components are included in the intended architecture but appear to still be in progress or being expanded:

| Service | Status |
|---|---|
| Wazuh | In progress / expanding |
| TheHive | Planned / integration phase |

## Recommended Data Flow Model

### Network Telemetry Path
1. Traffic crosses the relevant monitored interface or segment
2. Suricata evaluates packets and flows against its configured rules
3. Events are written to `eve.json`
4. Splunk ingests those events for searching, dashboards, and alerting
5. High-value detections can be escalated into TheHive workflows

### Host Telemetry Path
1. Wazuh agents run on supported systems
2. Host events, integrity changes, and vulnerability data report back to the Wazuh manager
3. Relevant telemetry is forwarded into Splunk for central review
4. Findings can be tied back to network events or case activity

### Honeypot Telemetry Path
1. OpenCanary exposes monitored decoy services
2. Probes or connection attempts generate log entries
3. Those events are forwarded into Splunk
4. Suspicious or repeatable events can be turned into dashboard views, detections, or cases

## What This Stack Demonstrates

From a junior cybersecurity perspective, this stack shows practical exposure to:
- IDS concepts
- SIEM workflow
- centralized logging
- host telemetry collection
- deception technology
- alert triage
- case-management workflow design

That matters because it shows not just tool familiarity, but an understanding of how multiple tools combine into a defensive monitoring process.

## Suggested Detection Use Cases

These are strong use cases for the lab as currently designed:

### Network-Focused
- suspicious scans or noisy connections observed by Suricata
- traffic into restricted or sensitive segments
- protocol anomalies or unwanted service access

### Host-Focused
- unexpected file changes
- suspicious package or service changes
- visibility into endpoint state and vulnerability posture

### Honeypot-Focused
- SSH, FTP, SMB, HTTP, or database-style touches against decoy services
- unexpected traffic reaching systems that should not be used in normal workflows

### Correlation-Focused
- correlate a honeypot hit with traffic patterns in Suricata
- correlate host changes with suspicious network activity
- validate whether offensive lab activity is visible across multiple layers

## Operational Considerations

A few design principles matter here:

- high-noise tools need tuning or they become wallpaper
- deception alerts are valuable because they are usually higher signal
- SIEM storage and indexing can become resource-heavy quickly
- host agents add visibility, but they also increase operational overhead
- incident tooling is most useful when tied to actual triage habits

## Validation Checklist

- [ ] Suricata is producing usable event data
- [ ] Splunk is ingesting events from all active telemetry sources
- [ ] OpenCanary alerts are visible in Splunk
- [ ] Wazuh agents are reporting from intended hosts
- [ ] TheHive workflow is documented or integrated
- [ ] End-to-end alert flow has been tested intentionally
- [ ] At least one attack simulation has been validated across multiple tools

## Good Next Enhancements

To make this guide fully operational, add:
- exact installation methods for each component
- index names and sourcetypes in Splunk
- Suricata rule-update process and tuning notes
- Wazuh agent coverage by host
- OpenCanary enabled protocols and placement details
- TheHive case taxonomy or workflow notes
- example detections and screenshots from the live lab

## Why This Guide Matters

This is one of the strongest parts of the project from a portfolio perspective. A lot of homelabs stop at “I installed a tool.” Deathstar is more interesting because the tools are part of a layered monitoring design:

- Suricata watches the wire
- Wazuh watches the hosts
- OpenCanary creates high-signal bait
- Splunk ties telemetry together
- TheHive turns events into workflow

That is a much better story than “I have a SIEM because I saw one on YouTube.”
