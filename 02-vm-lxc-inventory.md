# 02 — VM & LXC Inventory

This guide tracks the systems that make up Project Deathstar, including active workloads, planned workloads, and the resource strategy behind the environment.

Project Deathstar runs on a Proxmox VE host built on a Supermicro H11SSL-i with an AMD EPYC 7532, 256GB ECC RAM, a 256GB NVMe device for fast system workloads, a 4TB ZFS mirror for VM/LXC storage, and an NVIDIA Tesla P40 for local AI inference.

## Inventory Goals

This document is intended to show:
- which systems currently exist
- which systems are planned next
- how they are segmented across VLANs
- whether they are VMs or LXCs
- which services are security-critical, user-facing, or higher risk
- how CPU, memory, storage, and GPU resources are expected to be used

## Platform Capacity

| Resource | Capacity | Notes |
|---|---|---|
| CPU | 32 cores / 64 threads | Good fit for many mixed VMs/LXCs |
| Memory | 256GB ECC | Supports Windows, Splunk, containers, and AI workloads in parallel |
| Fast storage | 256GB NVMe | Best reserved for Proxmox OS and latency-sensitive workloads |
| Main data storage | 4TB ZFS mirror | Primary VM/LXC and service data pool |
| GPU | Tesla P40 24GB | Reserved for local inference and AI services |

## Workload Strategy

The overall platform design favors:
- LXCs for lightweight Linux infrastructure services
- VMs for Windows systems, Splunk, and anything that benefits from stronger isolation or broader compatibility
- ZFS-backed storage for most persistent data
- dedicated GPU-backed compute for Ollama and Open WebUI workflows

## Active Systems

These systems are explicitly referenced as currently active in the repository README.

| ID | Type | Hostname | VLAN | Role | Status | Notes |
|---|---|---|---|---|---|---|
| CT100 | LXC | suricata | LAB_DMZ (20) | Network intrusion detection | Active | Monitors network traffic for security visibility |
| VM200 | VM | splunk | LAB_DMZ (20) | SIEM / dashboards / alerting | Active | Central logging and analysis platform |
| CT101 | LXC | opencanary | LAB_DMZ (20) | Honeypot / deception alerts | Active | Generates high-signal telemetry from suspicious service interaction |

## Planned Security Systems

These systems are part of the intended security architecture but are not clearly listed as active yet.

| ID | Type | Hostname / Role | VLAN | Status | Notes |
|---|---|---|---|---|---|
| CT102 | LXC or VM | wazuh | LAB_DMZ (20) | Planned / expanding | Host monitoring, FIM, vulnerability visibility |
| CT104 | LXC or VM | thehive | LAB_DMZ (20) | Planned / integration phase | Case management and incident workflow |

## Planned Active Directory and Testing Systems

| Type | Hostname / Role | VLAN | Status | Notes |
|---|---|---|---|---|
| VM | Windows Server 2022 domain controller | AD_LAB (80) | Planned / in progress | Core AD, DNS, policy, identity practice |
| VM | Windows 10 workstation | AD_LAB (80) | Planned / in progress | Domain-joined endpoint for admin and testing |
| VM | Kali Linux | ATTACK (70) | Planned / in progress | Offensive tooling and validation platform |
| VM or LXC | Metasploitable | HONEYPOT (60) or test segment | Planned | Vulnerable target for safe testing |
| VM or LXC | DVWA | HONEYPOT (60) or test segment | Planned | Web exploitation practice target |

## Planned Self-Hosted Services

| Service | Type | Likely VLAN | Status | Notes |
|---|---|---|---|---|
| Nextcloud | VM or LXC | LAN_MAIN (10) | Planned / expanding | Private cloud and file sync |
| Nginx Proxy Manager | LXC | LAN_MAIN (10) | Planned / expanding | Reverse proxy and access layer |
| Jellyfin | VM or LXC | LAN_MAIN (10) | Planned / expanding | Media streaming |
| Vaultwarden | LXC | LAN_MAIN (10) | Planned / expanding | Password management |
| Immich | VM or LXC | LAN_MAIN (10) or IOT (30) | Planned / expanding | Photo management |
| Home Assistant | VM or LXC | IOT (30) | Planned / expanding | Smart-home control |
| Ollama | VM with GPU or dedicated VM/LXC design | LAN_MAIN (10) | Planned / in progress | Local LLM inference |
| Open WebUI | VM or LXC | LAN_MAIN (10) | Planned / in progress | Front end for AI access |

## Resource Planning Notes

The exact allocations still need to be documented from the live environment, but the intended priorities are already fairly clear.

### Higher-Priority Resource Consumers

- Splunk will likely be one of the heavier RAM and storage consumers
- Windows AD systems will need dedicated VM resources and full virtualization
- Ollama / GPU-backed inference workloads will consume GPU memory and can also demand significant CPU/RAM
- media and photo services may grow quickly in storage usage

### Good LXC Candidates

Likely good fits for containers:
- Suricata, depending on deployment preference
- OpenCanary
- Vaultwarden
- Nginx Proxy Manager
- lightweight utility or management services

### Good VM Candidates

Likely better as full VMs:
- Splunk
- Windows Server 2022 domain controller
- Windows 10 workstation
- Kali Linux
- AI workloads that need passthrough or stronger isolation

## Storage Placement Strategy

Based on the platform design, a sensible storage split is:

### NVMe
Use for:
- Proxmox OS
- lightweight latency-sensitive services
- possibly small but performance-critical container roots

### ZFS Mirror
Use for:
- primary VM/LXC disks
- application data
- snapshots
- self-hosted media and service storage
- AD and security tooling persistence

### GPU Allocation
Use for:
- Ollama inference
- Open WebUI-backed model access
- future local AI workflows that benefit from the Tesla P40

## Dependency Notes

Some systems naturally depend on others and should be documented that way.

Examples:
- Splunk depends on upstream telemetry sources such as Suricata, Wazuh, and OpenCanary
- TheHive is most useful once Splunk and detection workflows are feeding incident use cases
- Open WebUI depends on an available Ollama backend
- Windows workstation testing depends on the AD domain controller being in place
- Home Assistant becomes more useful as the IOT segment matures

## Operational Priorities

A practical order of importance for the lab looks like this:

1. Core hypervisor and storage stability
2. Management access and segmentation
3. Security visibility stack
4. Active Directory lab systems
5. Self-hosted app expansion
6. AI/GPU workflow maturity

## Validation Checklist

- [ ] All active systems are documented with exact IDs, VLANs, and roles
- [ ] Planned systems are clearly separated from deployed systems
- [ ] VM versus LXC choice is recorded for every workload
- [ ] CPU, RAM, and storage allocations are filled in from the live host
- [ ] GPU consumers are identified clearly
- [ ] Backup priority is documented for critical services
- [ ] High-risk systems are flagged by exposure level and segment

## What Still Needs to Be Added

This guide is now structured around the actual Deathstar architecture, but it still needs live-environment specifics.

Add these next:
- exact vCPU and RAM assignments
- disk sizes and datastore placement
- IP assignments or reservation references
- startup order and backup notes
- whether each service is truly active, planned, or retired
- which AI services are running directly with GPU access versus proxied access
