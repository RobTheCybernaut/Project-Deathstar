# 05 — Self-Hosted Apps

This guide documents the internal services hosted in Project Deathstar and the role they play in the larger lab.

The self-hosted side of Deathstar is important for two reasons:
- it makes the environment useful day to day instead of being just a pile of security tooling
- it creates real infrastructure that has to be segmented, secured, monitored, maintained, and backed up

That makes these services part of the learning value, not a side quest.

## Service Overview

Project Deathstar includes or plans to include the following internal services:
- Nextcloud
- Nginx Proxy Manager
- Jellyfin
- Vaultwarden
- Immich
- Home Assistant
- Ollama
- Open WebUI

## Design Goals

The self-hosted app layer is intended to:
- provide practical internal services
- create realistic admin and security responsibilities
- support reverse proxy, storage, access-control, and backup workflows
- give the lab real user-facing systems to protect
- support local AI experimentation through GPU-backed inference tools

## Why This Layer Matters

A lot of labs focus only on “security tools talking to security tools.” That is useful, but not enough. Real environments contain normal applications, user data, media, admin panels, secrets, and convenience services.

By hosting real services, Deathstar becomes a better training environment for:
- segmentation decisions
- access control
- patching and maintenance
- storage planning
- backup prioritization
- monitoring coverage
- internal service hardening

## Current / Planned Services

| Service | Role | Likely Exposure | Notes |
|---|---|---|---|
| Nextcloud | File sync and collaboration | Internal or proxied | Useful for private cloud workflows |
| Nginx Proxy Manager | Reverse proxy / ingress layer | Internal admin, selective service exposure | Central entry point for web apps |
| Jellyfin | Media streaming | Internal | Lower sensitivity, but still useful to segment |
| Vaultwarden | Password management | Internal or tightly controlled proxied access | Higher sensitivity than most user apps |
| Immich | Photo management | Internal | Storage-heavy service |
| Home Assistant | IoT automation | Internal | Naturally tied to the IOT segment |
| Ollama | Local LLM backend | Internal | Main inference engine |
| Open WebUI | AI front end | Internal | User-facing interface for local AI access |

## Likely Network Placement

Exact placements should be confirmed from the live lab, but the architecture suggests a model like this:

| Service Type | Suggested VLAN | Reason |
|---|---|---|
| General internal apps | LAN_MAIN (10) | Fits standard internal-service role |
| IoT-related management | IOT (30) | Keeps automation close to device segment |
| Sensitive admin-facing services | LAN_MAIN (10) or restricted management path | Easier to control tightly |
| AI services | LAN_MAIN (10) or dedicated internal app zone | Keeps them accessible without exposing management systems |

### Special Cases
- Home Assistant belongs near the IOT environment from a functional standpoint, but admin access should still be controlled
- Vaultwarden deserves stricter access controls than a media service like Jellyfin
- Ollama and Open WebUI may need placement decisions based on GPU access, storage, and who should be allowed to use them

## Reverse Proxy and Access Layer

Nginx Proxy Manager is one of the most important support services in the stack because it shapes how internal apps are reached.

Typical responsibilities include:
- routing internal hostnames to the correct services
- centralizing TLS handling
- simplifying service publication
- acting as a narrow control point for which apps are reachable and how

From a security perspective, that matters because you do not want every internal app inventing its own access model like a committee of caffeinated raccoons.

## Storage and Persistence Considerations

Not all services deserve the same storage treatment.

### Higher-Value Persistent Data
- Nextcloud
- Vaultwarden
- Immich
- Home Assistant
- AI model storage for Ollama

### Lower-Sensitivity but Still Important Data
- Jellyfin metadata and media indexes
- reverse proxy configuration
- Open WebUI configuration and state

### Planning Considerations
- large media and photo services can consume storage quickly
- password and document services should have stronger backup discipline
- AI models may occupy substantial disk space even when the service itself is simple
- some services are easy to redeploy, but their data is not

## Security Considerations

Self-hosted services are useful, but they expand the attack surface.

### Recommended Security Priorities
- keep administrative interfaces off casual-access segments
- restrict externally reachable services to only what is necessary
- use segmentation to separate higher-trust from lower-trust apps
- monitor exposed services through the security stack where practical
- protect secrets, tokens, and admin credentials carefully
- patch services on a regular cadence

### Risk Differences Between Services

Not all apps carry the same operational risk.

Higher-sensitivity services:
- Vaultwarden
- Nextcloud
- Nginx Proxy Manager
- Home Assistant

Higher storage-growth services:
- Jellyfin
- Immich
- Ollama model directories

Potentially high-interest services for attackers or misconfiguration:
- reverse proxies
- password managers
- automation platforms
- anything exposed beyond the lab

## AI Services

The AI layer is one of the more distinctive parts of Deathstar.

### Ollama
Ollama is the local inference backend and the main reason the Tesla P40 matters to the project.

### Open WebUI
Open WebUI provides the user-facing interface for interacting with local models.

Together, they demonstrate:
- self-hosted AI service deployment
- GPU-backed inference design
- internal service integration
- the overlap between infrastructure, performance, and security

## Operational Model

A sensible operating model for this layer is:

1. deploy apps to the correct internal segment
2. place reverse proxy access in front of the services that need friendly URLs
3. document which apps are internal-only versus selectively exposed
4. define backup priority based on data sensitivity and recovery value
5. monitor the important services through logs, health checks, and security tooling where possible

## Validation Checklist

- [ ] Each service is documented with host, VLAN, and purpose
- [ ] Exposure level is recorded for every app
- [ ] Reverse proxy and naming strategy are documented
- [ ] Backup priority is defined for critical data stores
- [ ] Higher-sensitivity services have tighter access controls
- [ ] Monitoring or logging coverage is identified for important apps
- [ ] AI services are mapped clearly to the GPU-backed backend

## What Still Needs to Be Added

This guide now reflects the Deathstar service architecture, but it still needs live deployment details.

Add these next:
- exact host placement for each service
- whether each service is a VM or LXC
- internal DNS and reverse proxy naming conventions
- storage paths and backup targets
- auth model for exposed services
- which services are active today versus planned next
