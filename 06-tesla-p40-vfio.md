# 06 — Tesla P40 VFIO Passthrough

This guide documents how the Tesla P40 is assigned for local AI inference workloads in Project Deathstar.

## Purpose

The Tesla P40 enables:
- local LLM inference
- self-hosted AI experimentation
- GPU-backed Ollama workloads
- Open WebUI integration
- reduced dependence on cloud-hosted AI services

## Hardware Summary

| Component | Value |
|---|---|
| GPU | NVIDIA Tesla P40 |
| VRAM | 24GB |
| Host platform | Proxmox VE on AMD EPYC |
| Intended services | Ollama, Open WebUI, local inference workflows |

## Recommended Documentation Sections

### BIOS / Firmware Settings
Capture:
- IOMMU / virtualization settings
- PCIe and Above 4G Decoding notes if relevant
- any host firmware requirements

### Proxmox Host Preparation
Capture:
- IOMMU kernel parameters
- VFIO modules
- blacklisting or driver binding changes
- initramfs updates
- reboot requirements

### Device Isolation
Capture:
- IOMMU group layout
- GPU and audio function pairing if applicable
- verification commands used before passthrough

### VM Configuration
Capture:
- target VM or workload host
- machine type / BIOS mode
- PCIe passthrough settings
- ROM/bar/reset caveats if encountered
- CPU and RAM sizing for inference workloads

### Guest OS Setup
Capture:
- NVIDIA driver installation
- CUDA/runtime notes if needed
- validation steps such as `nvidia-smi`
- model-serving prerequisites

### AI Stack
Capture:
- Ollama installation
- Open WebUI integration
- model storage path
- access model
- resource monitoring approach

## Validation Checklist

- [ ] IOMMU enabled on host
- [ ] VFIO binding configured correctly
- [ ] GPU isolated from host as intended
- [ ] Guest OS detects Tesla P40
- [ ] `nvidia-smi` works inside the guest
- [ ] Ollama uses the GPU successfully
- [ ] Open WebUI reaches the inference backend

## Common Pitfalls

Potential issues to document if encountered:
- incomplete IOMMU isolation
- driver conflicts on the host
- BAR / reset issues
- guest driver mismatch
- insufficient power/cooling considerations
- model performance limits versus VRAM availability

## Notes

This is a starter guide. Replace placeholders with the exact host configuration, passthrough steps, troubleshooting notes, and validation outputs from the live Deathstar build.
