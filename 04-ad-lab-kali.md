# 04 — AD Lab & Attack VM

This guide documents the isolated Windows Active Directory environment and the associated offensive-testing systems in Project Deathstar.

## Purpose

This segment exists to support:
- Windows administration practice
- domain services configuration
- GPO and identity management learning
- attack simulation in a controlled environment
- defensive validation against known offensive techniques

## Environment Scope

### Active Directory Lab
- Windows Server 2022 domain controller
- Windows 10 domain workstation
- supporting DNS/DHCP/domain services as applicable

### Attack / Testing Systems
- Kali Linux
- Metasploitable
- DVWA

## Network Placement

| Zone | VLAN | Purpose |
|---|---|---|
| AD_LAB | 80 | Domain controller and workstation |
| ATTACK | 70 | Kali Linux and offensive tooling |
| HONEYPOT | 60 | Vulnerable / deceptive targets as applicable |

## Recommended Documentation Sections

### Domain Build
Capture:
- domain name used in the lab
- forest/domain functional level
- OU structure
- administrative account model
- baseline GPOs

### Workstation Join Process
Capture:
- workstation build details
- join procedure
- DNS configuration
- validation steps

### Kali Linux Role
Capture:
- toolset used
- network restrictions
- access controls
- no-egress or limited-egress policy

### Vulnerable Targets
Capture:
- which targets are deployed
- which VLAN they live on
- what attack classes they support for practice
- how they are isolated from the rest of the lab

## Learning Objectives

Suggested focus areas:
- domain administration basics
- user and group management
- Group Policy fundamentals
- BloodHound or AD enumeration workflows
- Kerberoasting awareness
- credential hygiene and exposure paths
- attacker behavior versus defender visibility

## Safety Boundaries

This environment should remain:
- isolated from normal user systems
- tightly controlled from a routing perspective
- limited to intentional practice activity only

## Validation Checklist

- [ ] Domain controller deployed and documented
- [ ] Workstation joined successfully
- [ ] Attack VM segmented appropriately
- [ ] Vulnerable targets isolated from primary LAN
- [ ] Administrative and attack workflows documented
- [ ] Defensive visibility mapped to offensive activity

## Notes

This is a starter guide. Replace the placeholders with the actual domain design, workflows, and validation steps used in Deathstar.
