# 02 — VM & LXC Inventory

This guide tracks the virtual machines and Linux containers that make up Project Deathstar.

## Purpose

Use this file to document:
- active systems
- planned systems
- CPU and memory allocation
- storage placement
- VLAN assignment
- role and dependency mapping

## Recommended Inventory Fields

| ID | Type | Hostname | VLAN | OS | vCPU | RAM | Storage | Status | Purpose |
|---|---|---|---|---|---|---|---|---|---|
| CT100 | LXC | suricata | 20 | Debian/Ubuntu | TBD | TBD | TBD | Active | Network IDS |
| VM200 | VM | splunk | 20 | Linux | TBD | TBD | TBD | Active | SIEM |
| CT101 | LXC | opencanary | 20 | Linux | TBD | TBD | TBD | Active | Honeypot |

## Suggested Sections

### Active Systems
Document all currently deployed workloads with:
- ID
- hostname
- IP or DHCP reservation reference
- VLAN
- platform type (VM or LXC)
- core function
- current operational state

### Planned Systems
List systems not yet deployed but expected in the final design.

Potential examples:
- Wazuh manager
- TheHive
- Windows Server 2022 domain controller
- Windows 10 workstation
- Kali Linux attack VM
- Metasploitable
- DVWA host
- Ollama / Open WebUI inference node
- internal application services

### Resource Budget

| Resource | Available | Allocated | Notes |
|---|---|---|---|
| CPU threads | 64 | TBD | Track overcommit carefully |
| RAM | 256GB | TBD | Prioritize Splunk, Windows, and AI workloads |
| NVMe | 256GB | TBD | Use for OS / latency-sensitive workloads |
| ZFS mirror | 4TB | TBD | Main VM/LXC data pool |
| GPU VRAM | 24GB | TBD | Reserved for local inference workloads |

## Placement Strategy

Recommended documentation topics:
- which workloads live on NVMe vs ZFS
- which workloads require snapshots/backups
- which workloads should remain lightweight LXCs
- which workloads need full VMs for compatibility or isolation

## Operational Notes

Track details such as:
- startup order dependencies
- monitoring coverage
- backup status
- exposure level
- maintenance caveats

## Validation Checklist

- [ ] All active workloads documented
- [ ] Planned workloads separated from active workloads
- [ ] VLAN assignment recorded for every system
- [ ] CPU/RAM/storage budget updated
- [ ] Backup and recovery priorities identified

## Notes

This is a starter inventory. Replace the placeholders with exact allocations, operating systems, and deployment status from the live lab.
