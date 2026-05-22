# 01 — Proxmox VLAN Setup

This guide documents the VLAN architecture used in Project Deathstar and the design decisions behind the Proxmox + UniFi network layout.

Project Deathstar is built around a single Proxmox VE host connected to a UniFi Dream Machine Pro Max. The environment is segmented across 10 VLANs so that security tooling, management systems, offensive testing, IoT devices, guest traffic, and Active Directory resources do not share a flat network.

## Goals

This design is intended to:
- separate systems by trust level and function
- reduce blast radius between services
- isolate higher-risk lab segments from management and daily-use traffic
- support realistic firewalling and east-west visibility
- make the environment easier to monitor, troubleshoot, and document

## Platform Summary

| Component | Value |
|---|---|
| Hypervisor | Proxmox VE |
| Host hardware | Supermicro H11SSL-i / AMD EPYC 7532 / 256GB ECC |
| Edge router / firewall | UniFi Dream Machine Pro Max |
| Switching approach | VLAN trunk between UniFi and Proxmox |
| Bridge model | VLAN-aware Proxmox Linux bridge |
| Lab style | Segmented home cyber lab |

## Network Design Overview

The Proxmox host acts as the compute platform for VMs and LXCs, while UniFi handles the routed networks, VLAN definitions, and inter-VLAN firewall policy.

At a high level:
- UniFi defines each network and VLAN ID
- a trunk/uplink carries tagged VLAN traffic to the Proxmox host
- Proxmox presents those VLANs to guest systems through a VLAN-aware bridge
- each VM or LXC is attached to the appropriate segment based on its role

This keeps network policy centralized at the router while allowing workloads on the hypervisor to live on cleanly separated virtual networks.

## VLAN Plan

| VLAN | ID | Subnet | Purpose |
|---|---|---|---|
| LAN_MAIN | 10 | 192.168.10.0/24 | Primary LAN |
| LAB_DMZ | 20 | 192.168.20.0/24 | Security tooling and monitoring |
| IOT | 30 | 192.168.30.0/24 | IoT devices and Home Assistant |
| GUEST | 40 | 192.168.40.0/24 | Guest access |
| MGMT | 50 | 192.168.50.0/24 | Proxmox host and management interfaces |
| HONEYPOT | 60 | 192.168.60.0/24 | Deception systems and vulnerable targets |
| ATTACK | 70 | 192.168.70.0/24 | Kali and offensive tooling |
| AD_LAB | 80 | 192.168.80.0/24 | Isolated Windows domain environment |
| BLACKHOLE | 99 | 192.168.99.0/24 | Sink / null route network |
| NORDVPN | 200 | 192.168.200.0/24 | VPN-routed traffic |

## Why the Segmentation Matters

Each VLAN exists for a reason:

### LAN_MAIN
The primary internal network for general-purpose internal services that do not belong in management or security-specific segments.

### LAB_DMZ
Used for the monitoring stack, including systems such as Suricata, Splunk, OpenCanary, Wazuh, and TheHive. Keeping these systems together simplifies log flow and allows tighter control over who can reach them.

### IOT
Keeps smart-home and appliance-style devices out of the primary LAN and away from sensitive management or lab systems.

### GUEST
Provides isolated internet access for non-trusted client devices without granting visibility into lab infrastructure.

### MGMT
Reserved for administrative interfaces such as Proxmox, IPMI/BMC, and other privileged infrastructure services. This should be treated as the highest-trust segment in the lab.

### HONEYPOT
Dedicated to deception and intentionally exposed or monitored systems. Treat this segment as noisy and potentially hostile by default.

### ATTACK
Used for Kali and offensive testing tools. This segment should be tightly restricted and should not be able to move freely into the rest of the environment.

### AD_LAB
Contains the isolated Windows domain environment used for administration practice and attack-path simulation.

### BLACKHOLE
Useful as a sink or null-route segment for containment, testing, or policy enforcement scenarios.

### NORDVPN
Reserved for workloads that should exit through a VPN-routed path rather than the normal WAN path.

## Proxmox Design

The exact interface names in the live lab may vary, but the architectural model should look like this:

1. A physical NIC on the Proxmox host uplinks to UniFi
2. That uplink carries tagged VLAN traffic
3. Proxmox uses a VLAN-aware bridge to pass traffic to guests
4. Guests are assigned VLAN tags based on their role
5. The host management IP lives on the management network, not the flat default LAN

### Recommended Proxmox Approach

- Use one main bridge for the trunked uplink
- Enable VLAN awareness on that bridge
- Attach VMs/LXCs to the bridge and set their VLAN tag individually
- Keep management on a dedicated management subnet
- Avoid mixing management with guest or attack traffic

### Example Guest Placement Logic

| Workload Type | Suggested VLAN |
|---|---|
| Proxmox management | MGMT (50) |
| Splunk / Suricata / Wazuh / TheHive | LAB_DMZ (20) |
| OpenCanary / vulnerable decoys | HONEYPOT (60) or LAB_DMZ depending on design |
| Kali Linux | ATTACK (70) |
| Windows DC / workstation | AD_LAB (80) |
| Home Assistant / internal user apps | IOT (30) or LAN_MAIN (10) depending on service purpose |
| VPN-exit workloads | NORDVPN (200) |

## UniFi Design

On the UniFi side, this lab depends on a clean separation between network definition and policy enforcement.

### UniFi Responsibilities

- define each VLAN and subnet
- provide gateway and DHCP services where appropriate
- assign the correct trunk/native profile to the Proxmox uplink
- enforce inter-VLAN firewall rules
- route selected traffic through the VPN segment where needed

### Recommended UniFi Configuration Pattern

- Create a separate network object for each VLAN
- Use clear names matching the lab documentation
- Apply a trunk profile to the Proxmox uplink port
- Limit administrative access to the MGMT segment
- Explicitly define rules for ATTACK, HONEYPOT, AD_LAB, and IOT rather than relying on permissive defaults

## Firewall Policy Baseline

The exact rules will depend on what is currently deployed, but the design intent is straightforward.

### Recommended Baseline

- deny inter-VLAN access by default where practical
- explicitly allow only required service paths
- allow management access only from trusted admin systems
- prevent GUEST from reaching lab or management networks
- prevent IOT from reaching MGMT and sensitive lab systems
- restrict ATTACK VLAN egress and lateral movement
- isolate HONEYPOT from trusted segments except for logging and monitoring paths
- keep AD_LAB isolated from the rest of the environment except where intentional testing requires otherwise

### Example Allowed Flows

- MGMT → Proxmox / infrastructure management services
- LAB_DMZ → logging, update paths, and internal telemetry collection
- HONEYPOT → Splunk or logging destination only
- Wazuh agents → Wazuh manager / Splunk inputs
- Open WebUI → Ollama backend on approved internal paths

## Operational Notes

A few design principles make this lab easier to maintain:

- Name VLANs consistently across Proxmox, UniFi, and documentation
- Keep ID, subnet, and purpose aligned in every diagram and guide
- Reserve room in each subnet for static assignments and growth
- Treat management and offensive segments as special cases with stricter policy
- Document every exception to the default traffic model

## Validation Checklist

- [ ] All VLANs are defined in UniFi with the correct IDs and subnets
- [ ] The Proxmox uplink is using the intended trunk profile
- [ ] The Proxmox bridge is VLAN-aware
- [ ] The host management IP is placed on MGMT rather than a flat default LAN
- [ ] VMs and LXCs receive addresses from the correct segment
- [ ] GUEST and IOT cannot reach sensitive infrastructure by default
- [ ] ATTACK cannot move freely across trusted VLANs
- [ ] HONEYPOT traffic is visible to the monitoring stack
- [ ] AD_LAB remains isolated unless a specific test requires connectivity

## What Still Needs to Be Added

This guide now reflects the Deathstar network design, but it still needs live-environment specifics to become a full build document.

Add these next:
- actual Proxmox interface names and bridge config
- UniFi screenshots or export details
- exact DHCP/gateway assignments
- real firewall rule names and ordering
- notes on which systems use static IPs versus reservations
- any exceptions required for AI services, AD testing, or remote access
