# 06 — Tesla P40 VFIO Passthrough

This guide documents the Tesla P40 passthrough design used for local AI workloads in Project Deathstar.

The Tesla P40 is one of the more distinctive parts of the lab because it turns Deathstar from a normal virtualization and security environment into a platform that can also run meaningful local AI workloads.

## Goals

The Tesla P40 is intended to support:
- local LLM inference
- self-hosted AI experimentation
- GPU-backed Ollama workloads
- Open WebUI integration
- reduced dependence on cloud-hosted AI services

## Hardware Context

| Component | Value |
|---|---|
| GPU | NVIDIA Tesla P40 |
| VRAM | 24GB |
| Host platform | Proxmox VE on AMD EPYC 7532 |
| Intended services | Ollama, Open WebUI, local inference workflows |

## Why This Matters

From a project and portfolio standpoint, this section shows more than just “I added a GPU.” It demonstrates:
- PCIe passthrough concepts
- virtualization tuning
- Linux host preparation
- guest-driver validation
- AI infrastructure deployment
- performance-aware homelab design

That is a strong signal because it connects hardware, hypervisor configuration, guest operating systems, and application-layer outcomes.

## Architecture Intent

The design goal is straightforward:
- the Proxmox host owns the physical hardware
- the Tesla P40 is isolated cleanly for passthrough
- a guest workload receives direct access to the GPU
- Ollama uses the GPU for inference
- Open WebUI connects to that backend to provide a usable front end

This keeps the GPU attached to the workload that needs it, rather than treating the host itself as the long-term application runtime.

## High-Level Workflow

A typical passthrough path looks like this:

1. Enable virtualization and IOMMU features in firmware
2. Configure Proxmox for PCI passthrough
3. Ensure the GPU is isolated into a usable IOMMU group
4. Bind the device for passthrough rather than normal host use
5. Attach the Tesla P40 to the target guest
6. Install NVIDIA drivers in the guest
7. Validate with `nvidia-smi`
8. Deploy Ollama and related AI services

## Proxmox Host Preparation

The live lab specifics still need to be documented, but this section should ultimately capture:
- BIOS/UEFI settings related to virtualization and IOMMU
- kernel parameters enabling IOMMU on the host
- VFIO module configuration
- any driver blacklisting or device binding changes
- initramfs updates and reboot steps

### What to Verify on the Host
- IOMMU is enabled successfully
- the GPU is visible as a PCI device
- the device is isolated cleanly enough for passthrough
- Proxmox is not trying to use the card for unrelated host graphics tasks

## Guest Workload Design

The exact target guest is not yet documented in the repo, but the intended consumer is clear: a workload hosting Ollama and related local AI services.

### Likely Guest Responsibilities
- run the NVIDIA driver stack
- expose GPU-backed inference
- host Ollama
- support Open WebUI integration either locally or over the internal network

### Sizing Considerations

A GPU-backed inference guest should also be planned around:
- enough CPU to feed inference workloads
- enough RAM for models, services, and UI layers
- enough storage for model files and caching
- network placement that keeps the service usable without overexposing it

## AI Service Layer

### Ollama
Ollama is the primary inference backend in Deathstar. It is the service most directly tied to the Tesla P40.

### Open WebUI
Open WebUI is the user-facing layer that makes the local models easier to access and use.

### Combined Value
Together, these services show:
- how local AI can be self-hosted
- how GPU resources can be integrated into a homelab
- how infrastructure and application services intersect
- how the lab extends beyond pure security tooling into broader systems engineering

## Common Design Challenges

Passthrough work is usually where homelab confidence meets reality at speed.

Common problem areas include:
- incomplete IOMMU isolation
- incorrect VFIO binding
- host drivers grabbing the device unexpectedly
- BIOS settings not fully enabling passthrough features
- guest driver mismatch or installation issues
- insufficient cooling, power, or airflow planning
- model-size expectations exceeding practical GPU memory constraints

## Validation Checklist

- [ ] IOMMU is enabled on the Proxmox host
- [ ] VFIO configuration is working as intended
- [ ] The Tesla P40 is isolated for passthrough
- [ ] The guest OS detects the GPU successfully
- [ ] `nvidia-smi` works inside the guest
- [ ] Ollama can use the GPU for inference
- [ ] Open WebUI can reach the inference backend
- [ ] Model storage and access paths are documented

## What Still Needs to Be Added

This guide now reflects the architecture and purpose of the GPU stack, but it still needs the live-environment specifics.

Add these next:
- actual BIOS and kernel settings used
- `lspci` and IOMMU verification notes
- target guest type and specs
- NVIDIA driver version and validation output
- Ollama deployment details
- model storage path and capacity planning notes
- any passthrough quirks or troubleshooting lessons from the live build

## Why This Guide Is Valuable

This is one of the stronger differentiators in the repo because it shows the lab is not just a security sandbox. It is also a serious infrastructure project.

A Proxmox host running segmented networks, security tooling, Windows lab systems, self-hosted services, and a passed-through Tesla P40 for local AI is a pretty solid answer to the question, “so what have you actually built?”
