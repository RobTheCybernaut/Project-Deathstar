# 03 — Security Stack

This guide documents the security monitoring and response tooling deployed in Project Deathstar.

## Purpose

The goal of this stack is to provide visibility across:
- network telemetry
- host telemetry
- deception/honeypot events
- centralized search and analysis
- basic incident workflow and case tracking

## Core Components

| Service | ID | Role |
|---|---|---|
| Suricata | CT100 | Network intrusion detection |
| Splunk Enterprise | VM200 | SIEM, search, dashboards, alerting |
| OpenCanary | CT101 | Honeypot / deception alerts |
| Wazuh | CT102 | Host monitoring, FIM, vulnerability visibility |
| TheHive | CT104 | Case management and incident workflow |

## Alert Flow

```text
Network traffic
    → Suricata
        → eve.json
            → Splunk
                → TheHive

Host events
    → Wazuh agent
        → Wazuh manager
            → Splunk

Honeypot hits
    → OpenCanary
        → logs
            → Splunk
```

## Suricata

Document here:
- deployment model
- monitored interface(s)
- rule source and update method
- log location
- forwarding path into Splunk
- tuning notes and noisy signatures

## Splunk

Document here:
- installation method
- data inputs
- indexes used
- dashboards/searches created
- alerting use cases
- resource impact and storage retention notes

## OpenCanary

Document here:
- enabled decoy protocols
- log handling
- VLAN placement
- alert routing
- intentional exposure boundaries

## Wazuh

Document here:
- manager deployment
- agent coverage
- file integrity monitoring scope
- vulnerability scanning scope
- integrations with Splunk or other tooling

## TheHive

Document here:
- deployment notes
- case creation workflow
- analyst process
- data sources feeding investigations
- future automation plans

## Detection and Triage Goals

Examples of activities this stack should support:
- suspicious network activity review
- honeypot hit validation
- host change visibility
- investigation workflow practice
- blue-team dashboarding and alert correlation

## Validation Checklist

- [ ] Suricata is generating network events
- [ ] Splunk is ingesting all major telemetry sources
- [ ] OpenCanary alerts are visible in central logging
- [ ] Wazuh agents are reporting successfully
- [ ] TheHive workflow is documented or integrated
- [ ] Alert flow is tested end to end

## Notes

This is a starter guide. Replace placeholders with screenshots, config snippets, pipeline details, and example detections from the live environment.
