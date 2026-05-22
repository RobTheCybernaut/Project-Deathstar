# 07 — TheHive CT104 Proxmox Deployment

This guide documents the recommended design and deployment path for CT104, the dedicated TheHive container in Project Deathstar.

TheHive is the case-management and incident-workflow layer in the stack. In Deathstar, its job is not to replace Splunk, Suricata, or Wazuh. Its job is to turn interesting detections into trackable analyst work.

## Purpose

CT104 exists to provide:
- case creation and tracking
- analyst notes and tasking
- observable and alert enrichment workflow
- repeatable incident documentation
- a more realistic SOC-style workflow inside the lab

## Recommended Role in Deathstar

Suggested placement for CT104:

| Field | Value |
|---|---|
| Proxmox ID | CT104 |
| Hostname | thehive |
| Platform | Proxmox LXC |
| OS | Debian 12 |
| VLAN | LAB_DMZ (20) |
| Role | Case management and incident workflow |
| Exposure | Internal only |

## Why LXC for CT104

For this lab, a dedicated LXC makes sense because:
- TheHive is an internal service, not an internet-facing appliance
- the platform already uses LXCs for several Linux services
- keeping TheHive isolated in its own container is cleaner than bolting it onto Splunk or another node
- it keeps storage, Java services, and troubleshooting scoped to one workload

That said, TheHive is not a tiny throwaway container. It depends on JVM-backed services and deserves real resources.

## Official Compatibility Notes

Based on current StrangeBee documentation for TheHive 5 package installs:
- Debian 12 is a supported operating system
- Java 11 is required
- Cassandra 4.0.x to 4.1.x is supported
- Elasticsearch 7.11.x through 9.1.x is supported for recent 5.5.10 to 5.7.2 releases
- around 100 GB storage is recommended for most use cases

StrangeBee also notes small package-based deployments with fewer than 10 concurrent users at roughly:
- TheHive: 3 CPU cores / 4 GB RAM
- Cassandra: 3 CPU cores / 4 GB RAM
- Elasticsearch: 3 CPU cores / 4 GB RAM

Because Deathstar is a homelab rather than a multi-analyst production SOC, you can start smaller than a full enterprise footprint if performance is acceptable, but do not starve the container and then act surprised when Java starts filing complaints.

## Recommended CT104 Sizing

For a practical homelab starting point:

| Resource | Recommendation |
|---|---|
| vCPU | 4 |
| RAM | 8 GB minimum, 12 GB preferred |
| Root disk | 100 GB |
| Filesystem | ZFS-backed storage preferred |
| Network | Static DHCP reservation or documented static IP |

Why this sizing:
- 4 vCPU gives enough room for TheHive plus its local dependencies
- 8-12 GB RAM is a more realistic floor for a JVM-heavy single-node setup than pretending 2 GB will be fine
- 100 GB lines up with StrangeBee’s general storage guidance

## Architecture Choice

For Deathstar, the cleanest first deployment path is:
- one Debian 12 LXC
- standalone package-based install
- all required services hosted on the same CT104 container
- internal-only exposure
- reverse proxy/TLS later if desired

This matches StrangeBee’s package-install guide scope for a single-server deployment and keeps the lab easy to understand.

## Proxmox Build Plan

Create CT104 as a Debian 12 container with these characteristics:
- privileged or unprivileged according to your normal lab standard, but make sure required services run cleanly
- static hostname: `thehive`
- attach to LAB_DMZ VLAN 20
- allocate 4 vCPU and at least 8 GB RAM
- place disk on the main ZFS-backed VM/LXC storage
- keep it internal-only behind your normal firewall policy

Suggested operational notes:
- snapshot before major package changes
- back up configuration and application data after initial setup
- keep firewall rules tight; TheHive should be reachable only from trusted admin systems and any explicitly approved internal integrations

## Installation Paths

StrangeBee documents two realistic paths for this style of deployment.

### Option A — Quickest Path
Use StrangeBee’s one-command install script.

Command:
```bash
wget -q -O /tmp/install_script.sh https://scripts.download.strangebee.com/latest/sh/install_script.sh ; sudo -v ; bash /tmp/install_script.sh
```

What it does:
- installs TheHive and required dependencies
- uses predefined standalone settings
- gets you to a working lab instance faster

Important warning from StrangeBee:
- this quick script does not configure authentication for Cassandra or Elasticsearch
- that is fine for a fast lab bootstrap, but not the version of the story you want to brag about forever

### Option B — Better for Documentation and Learning
Use the package-install method and configure the pieces intentionally.

That path gives you better visibility into:
- Java setup
- Cassandra setup
- Elasticsearch setup
- service configuration
- future troubleshooting

For a portfolio lab, Option B is the better long-term writeup even if Option A is the faster bootstrapping move.

## Package-Install Highlights

From StrangeBee’s current Linux standalone package guide, the important setup pieces are:

### Base dependencies
```bash
sudo apt update
sudo apt install wget curl gnupg coreutils apt-transport-https git ca-certificates ca-certificates-java software-properties-common python3-pip lsb-release unzip
```

### Java 11
StrangeBee currently recommends Amazon Corretto Java 11.

```bash
wget -qO- https://apt.corretto.aws/corretto.key | sudo gpg --dearmor -o /usr/share/keyrings/corretto.gpg
echo "deb [signed-by=/usr/share/keyrings/corretto.gpg] https://apt.corretto.aws stable main" | sudo tee -a /etc/apt/sources.list.d/corretto.sources.list
sudo apt update
sudo apt install java-common java-11-amazon-corretto-jdk
echo JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto" | sudo tee -a /etc/environment
export JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto"
java -version
```

### Cassandra 4.1.x
StrangeBee’s guide currently targets Cassandra 4.1.x for supported installs.

```bash
wget -qO -  https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor  -o /usr/share/keyrings/cassandra-archive.gpg
echo "deb [signed-by=/usr/share/keyrings/cassandra-archive.gpg] https://debian.cassandra.apache.org 41x main" |  sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
sudo apt update
sudo apt install cassandra
sudo chown -R cassandra:cassandra /var/lib/cassandra
```

Recommended Cassandra hardening from StrangeBee:
- set a real cluster name
- configure JVM heap explicitly
- enable authentication instead of leaving it open

### TheHive and Elasticsearch
Follow the current StrangeBee package guide for the exact repository and service configuration steps for your chosen TheHive release. Keep versions inside the supported matrix.

## Recommended Security Posture

For Deathstar, CT104 should be treated as a sensitive internal system.

Recommended baseline:
- internal-only access
- no direct guest or public access
- only trusted admin systems can reach the UI
- keep Cassandra and Elasticsearch local to the container or otherwise tightly restricted
- use strong admin credentials at initial setup
- add TLS through your reverse proxy later if you want a cleaner internal access model

## Integration Role in the Stack

CT104 should sit after detection tooling, not in front of it.

Suggested workflow:
1. Suricata, Wazuh, and OpenCanary generate alerts
2. Splunk becomes the main aggregation and triage point
3. notable or validated events are escalated into TheHive cases
4. TheHive stores case notes, tasks, observables, and workflow state

That makes TheHive the place where “interesting alert” becomes “documented investigation.”

## Validation Checklist

- [ ] CT104 created in Proxmox as a dedicated Debian 12 LXC
- [ ] Container attached to LAB_DMZ VLAN 20
- [ ] CPU, RAM, and disk match intended sizing
- [ ] Java 11 installed and verified
- [ ] Cassandra installed and running
- [ ] TheHive installed successfully
- [ ] The web UI is reachable from trusted internal admin hosts only
- [ ] Initial admin setup completed
- [ ] Internal firewall policy documented
- [ ] Case workflow tested with at least one sample alert or manual case

## Good Next Steps After Install

Once CT104 is up:
- document the exact Proxmox resource allocation in `02-vm-lxc-inventory.md`
- update `03-security-stack.md` from “planned” to “active” when it is live
- record the actual management URL and access path
- decide whether Splunk will feed TheHive manually, by email, or through a later automation path
- add screenshots of case creation, tasking, and observables for the GitHub repo

## References

Official StrangeBee docs used for this guide:
- TheHive system requirements: https://docs.strangebee.com/thehive/installation/system-requirements/
- TheHive software requirements: https://docs.strangebee.com/thehive/installation/software-requirements/
- Quick install script: https://docs.strangebee.com/thehive/installation/automated-installation-script-linux/
- Linux standalone package install: https://docs.strangebee.com/thehive/installation/installation-guide-linux-standalone-server/
