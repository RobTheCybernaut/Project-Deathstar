# 04 — AD Lab & Attack VM

This guide documents the isolated Windows Active Directory environment and associated offensive-testing systems used in Project Deathstar.

This part of the lab exists to do two jobs at once:
- build practical Windows and identity-administration skills
- create a safe place to study attacker behavior and defensive visibility

That combination matters because a lot of junior cybersecurity work sits right at the intersection of systems administration, authentication, endpoint behavior, and security monitoring.

## Goals

The AD and attack portion of Deathstar is intended to support:
- Windows Server administration practice
- domain services and identity fundamentals
- Group Policy learning
- workstation management
- credential and privilege-path awareness
- controlled attack simulation
- validation of detections across the monitoring stack

## Environment Overview

The environment is built around two closely related segments:

### Active Directory Lab
Used for normal Windows administration and identity workflows.

Expected components:
- Windows Server 2022 domain controller
- Windows 10 domain workstation
- supporting DNS/domain services as needed

### Attack / Testing Environment
Used for safe offensive testing against isolated targets.

Expected components:
- Kali Linux
- DVWA
- Metasploitable
- other intentionally vulnerable or test-only systems as needed

## Network Placement

| Zone | VLAN | Purpose |
|---|---|---|
| AD_LAB | 80 | Domain services and Windows workstation testing |
| ATTACK | 70 | Kali Linux and offensive tooling |
| HONEYPOT | 60 | Vulnerable or deceptive systems where applicable |

This segmentation is one of the most important controls in the lab. The point is not just to have attack tools, but to keep them fenced into an intentional practice environment.

## Why This Matters

For portfolio and skills-development purposes, this section of the lab demonstrates practical exposure to:
- Active Directory basics
- Windows endpoint administration
- privilege and identity concepts
- attack-path awareness
- lab-safe offensive testing
- defender visibility into common attack behavior

That is a much better signal than simply saying “I installed Kali once and made the Wi-Fi uncomfortable.”

## Active Directory Design Intent

The AD lab is meant to model a small but realistic Windows environment where identity, policy, and workstation behavior can be practiced repeatedly.

### Likely Core Roles

| System | Role | Notes |
|---|---|---|
| Windows Server 2022 | Domain controller | Core identity, DNS, and policy services |
| Windows 10 workstation | Domain-joined client | User/admin workflow validation |
| Optional future systems | Additional member servers or test clients | Useful for expansion once the base domain is stable |

### Administration Focus Areas

This environment supports learning around:
- user and group creation
- OU structure and delegation concepts
- Group Policy fundamentals
- DNS and domain join troubleshooting
- local vs domain admin boundaries
- service accounts and permission scoping
- basic Windows hardening decisions

## Offensive Testing Design Intent

The attack side of the lab is there to safely study what offensive activity looks like and how defenders can spot it.

### Attack Segment Role

Kali Linux belongs on a dedicated ATTACK VLAN so it can be used to:
- enumerate the AD lab and vulnerable hosts
- test exposed services and weak configurations
- simulate common recon and credential-oriented activity
- validate that the monitoring stack sees what it should

### Vulnerable Targets

Targets such as DVWA and Metasploitable are useful because they create known-bad activity in a controlled way. That allows you to practice both offense and visibility without putting real systems at risk.

## Learning Objectives

This part of the lab is especially strong for building junior-level security skills.

### Blue-Team Learning
- recognize suspicious authentication and host behavior
- understand how attacker activity appears in logs and telemetry
- validate whether segmentation and firewalling work as intended
- connect host and network activity during an investigation

### Admin Learning
- build and manage a small Windows domain
- troubleshoot name resolution and join issues
- apply baseline policy controls
- understand where identity and endpoint management overlap

### Attack-Awareness Learning
- enumeration workflows
- privilege and trust-boundary awareness
- credential attack concepts
- why segmentation and monitoring matter in practice

## Recommended Workflow

A good operating model for this segment is:

1. Build the domain cleanly
2. Join at least one workstation
3. Apply and test baseline policy
4. Launch controlled attack activity from Kali
5. Observe traffic and alerts in Suricata, Splunk, Wazuh, and related tools
6. Record what was visible, what was missed, and what needs tuning

That loop turns the AD lab from a build project into a repeatable training environment.

## Safety Boundaries

This environment should remain:
- isolated from everyday devices and personal systems
- segmented from management infrastructure unless explicitly needed
- limited to intentional practice activity
- monitored when offensive testing is being performed

Recommended safety posture:
- ATTACK should not have broad access to trusted segments
- AD_LAB should not be routable to the wider environment by default
- vulnerable targets should never be treated as trusted internal systems
- any internet exposure should be deliberate and minimal

## Example Documentation Sections to Fill In Later

To turn this into a full operational guide, add live details for:
- domain name and structure
- OU layout
- local admin and delegated admin model
- baseline GPOs
- workstation build and join steps
- Kali network restrictions and tooling scope
- which vulnerable targets are active and where they live
- which detections are expected during common test scenarios

## Validation Checklist

- [ ] Domain controller deployed and reachable on the intended VLAN
- [ ] Workstation joined to the domain successfully
- [ ] DNS and authentication flows are working as expected
- [ ] Kali Linux is isolated on the ATTACK segment
- [ ] Vulnerable targets are separated from trusted lab resources
- [ ] Offensive test activity is visible in the monitoring stack
- [ ] Administrative and attack workflows are documented clearly

## What Still Needs to Be Added

This guide now reflects the Deathstar architecture and intent, but it still needs the real environment details.

Add these next:
- actual domain name and functional design
- VM specs and placement
- account structure and baseline GPO notes
- specific attack tools and scenarios used from Kali
- which systems have Wazuh agents or other host visibility
- screenshots, detection examples, and tuning notes from the live lab
