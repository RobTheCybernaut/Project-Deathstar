# 05 — Self-Hosted Apps

This guide documents the internal services hosted inside Project Deathstar.

## Purpose

The self-hosted services layer gives the lab practical day-to-day value while also creating additional infrastructure to secure, monitor, back up, and maintain.

## Current / Planned Services

- Nextcloud
- Nginx Proxy Manager
- Jellyfin
- Vaultwarden
- Immich
- Home Assistant
- Ollama
- Open WebUI

## Recommended Inventory Format

| Service | Platform | VLAN | Exposure | Purpose | Status |
|---|---|---|---|---|---|
| Nextcloud | VM/LXC | TBD | Internal / proxied | File sync and collaboration | Planned |
| Vaultwarden | VM/LXC | TBD | Internal / proxied | Password management | Planned |
| Jellyfin | VM/LXC | TBD | Internal | Media streaming | Planned |
| Home Assistant | VM/LXC | TBD | Internal | IoT automation | Planned |
| Ollama | VM/LXC | TBD | Internal | Local LLM serving | Planned |
| Open WebUI | VM/LXC | TBD | Internal | AI front end | Planned |

## Suggested Sections

### Reverse Proxy and Access Layer
Document here:
- reverse proxy design
- internal DNS strategy
- TLS handling
- authentication approach
- which services are exposed externally, if any

### Storage and Persistence
Document here:
- persistent storage locations
- backup requirements
- media/data growth expectations
- dataset mapping for critical services

### Security Considerations
Document here:
- segmentation for user-facing services
- admin access restrictions
- secrets handling
- monitoring coverage
- patching cadence

### AI Services
Document here:
- Ollama deployment model
- Open WebUI front-end setup
- GPU usage path
- model storage location
- who can access the services

## Operational Checklist

- [ ] Each service documented with host, VLAN, and purpose
- [ ] Exposure level recorded for every service
- [ ] Backup strategy noted for critical data
- [ ] Reverse proxy and access model documented
- [ ] Monitoring/logging coverage identified

## Notes

This is a starter guide. Replace placeholders with the exact deployment layout, storage paths, access patterns, and service-specific configs used in the live lab.
