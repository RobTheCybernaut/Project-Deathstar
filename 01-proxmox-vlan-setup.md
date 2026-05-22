# 01 — Proxmox VLAN Setup

This guide documents the network segmentation model used in Project Deathstar and the Proxmox/UniFi configuration required to support it.

## Purpose

This setup is designed to:
- separate lab systems by trust level and function
- isolate management traffic from user and attack traffic
- contain vulnerable and deception systems
- support realistic security monitoring and firewall policy design

## Target Environment

| Component | Value |
|---|---|
| Hypervisor | Proxmox VE |
| Router / firewall | UniFi Dream Machine Pro Max |
| Switching model | VLAN trunk to Proxmox host |
| Bridge mode | VLAN-aware Linux bridge |

## VLAN Plan

| VLAN | ID | Subnet | Function |
|---|---|---|---|
| LAN_MAIN | 10 | 192.168.10.0/24 | Primary LAN |
| LAB_DMZ | 20 | 192.168.20.0/24 | Security tooling |
| IOT | 30 | 192.168.30.0/24 | IoT devices |
| GUEST | 40 | 192.168.40.0/24 | Guest traffic |
| MGMT | 50 | 192.168.50.0/24 | Management interfaces |
| HONEYPOT | 60 | 192.168.60.0/24 | Deception systems |
| ATTACK | 70 | 192.168.70.0/24 | Offensive tooling |
| AD_LAB | 80 | 192.168.80.0/24 | Active Directory lab |
| BLACKHOLE | 99 | 192.168.99.0/24 | Sink / null route |
| NORDVPN | 200 | 192.168.200.0/24 | VPN-routed traffic |

## Design Notes

Recommended trust model:
- MGMT should be limited to administrative systems only
- ATTACK should have tightly restricted east-west and outbound access
- HONEYPOT should be monitored closely and treated as hostile by default
- AD_LAB should remain isolated from the primary LAN
- LAB_DMZ should host telemetry, logging, and response tooling

## Proxmox Side

Document here:
- physical NIC mapping
- Linux bridge configuration
- VLAN-aware bridge settings
- management IP placement
- VM/LXC network tagging model

### Example topics to capture
- which NIC uplinks to UniFi
- whether vmbr0 carries tagged VLAN traffic
- whether management is untagged or explicitly tagged
- how container and VM interfaces are assigned per VLAN

## UniFi Side

Document here:
- networks created in UniFi
- VLAN IDs and subnets
- trunk port profile assigned to the Proxmox uplink
- firewall rules between segments
- VPN-routing behavior for VLAN 200

## Firewall Policy Baseline

Suggested baseline rules:
- deny by default between sensitive VLANs
- allow only required service paths
- block unrestricted ATTACK VLAN egress
- restrict MGMT access to admin devices only
- prevent GUEST and IOT lateral movement into lab infrastructure

## Validation Checklist

- [ ] All VLAN networks created in UniFi
- [ ] Proxmox bridge configured as VLAN-aware
- [ ] Host management reachable only from intended segment
- [ ] VMs/LXCs receive addresses in the correct subnet
- [ ] Inter-VLAN access matches firewall policy
- [ ] ATTACK and HONEYPOT segments are isolated as intended

## Notes

This is a starter document. Replace each section with exact screenshots, configs, interface names, and firewall rules from the live Deathstar environment.
