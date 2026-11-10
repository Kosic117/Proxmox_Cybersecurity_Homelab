# Proxmox Cybersecurity Homelab — Troubleshooting & Incident Documentation

**Project:** Secure Proxmox Cybersecurity Homelab  
**Platform:** Proxmox VE 9.2  
**Hardware:** GMKtec NucBox G3 Plus — Intel N150, 8 GB RAM  
**Author:** Michael Schoenbeck  
**Document Status:** Initial troubleshooting record

---

## 1. Overview

This document records technical issues encountered while deploying, configuring, and maintaining a Proxmox-based cybersecurity homelab.

The environment includes a Proxmox virtualization host, Debian Linux gateway, isolated virtual laboratory network, Metasploitable 2 testing VM, and Tailscale overlay connectivity.

Troubleshooting activities focused on restoring service, identifying network configuration problems, resolving DNS failures, and maintaining remote access to laboratory systems.

Each case includes the observed symptoms, investigation, corrective actions, verification, and lessons learned.

**Security note:** All addresses used in examples are documentation-only addresses. Actual LAN, laboratory, and Tailscale IP addresses are intentionally excluded.

---

## 2. Incident 01 — Proxmox Host Connectivity Following ISP Migration

**Category:** Network Connectivity / Infrastructure Recovery  
**Status:** Resolved  
**Impact:** Loss of normal access to the Proxmox management interface

### Background

The home network was migrated from Spectrum to Breezeline, introducing changes to the network environment.

Following the migration, the Proxmox host was no longer accessible through its previous management address.

### Observed Symptoms

- Proxmox management interface was unreachable using the previous network configuration.
- The host initially produced no usable local display output.
- Normal remote management was unavailable.
- The host retained network settings associated with the previous environment.

### Investigation

Troubleshooting began by determining whether the issue involved the host itself or network connectivity.

The system was powered down for an extended period and restarted.

Following startup, the host became accessible for further troubleshooting, allowing its network configuration to be investigated.

The previously configured management address no longer matched the current network environment.

### Corrective Actions

1. Performed a full shutdown and restart of the Proxmox host.
2. Re-established access to the operating system.
3. Identified the outdated management network configuration.
4. Adjusted network connectivity for the updated environment.
5. Configured Tailscale on the Proxmox host to
