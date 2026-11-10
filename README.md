# Secure Proxmox Cybersecurity Homelab

**Virtualization | Network Segmentation | Linux Administration | Secure Remote Access**

## Project Overview

This project documents the deployment and configuration of a Proxmox Virtual Environment (VE) cybersecurity homelab designed to support hands-on security testing, network administration, and infrastructure experimentation.

The environment was built using a GMKtec NucBox G3 Plus mini PC running Proxmox VE 9.2. The primary objective was to create a persistent, remotely accessible cybersecurity laboratory while maintaining separation between intentionally vulnerable virtual machines and the production home network.

A dedicated Debian Linux gateway provides controlled connectivity between the isolated laboratory network and an authenticated Tailscale overlay network.

The project emphasizes practical infrastructure administration, network isolation, virtualization, troubleshooting, and security-focused system design.

## Project Objectives

- Deploy a dedicated Proxmox virtualization host.
- Create an isolated virtual network for cybersecurity testing.
- Deploy intentionally vulnerable systems without exposing them directly to the home network or public internet.
- Configure a Debian Linux gateway for controlled network routing.
- Implement encrypted remote access through Tailscale.
- Enable authorized remote testing from personal devices.
- Troubleshoot networking, DNS, routing, and virtualization issues.
- Document system architecture, security controls, and lessons learned.

## Hardware Specifications

| Component | Specification |
|---|---|
| Host | GMKtec NucBox G3 Plus |
| Processor | Intel N150 |
| CPU | 4 Cores / 4 Threads |
| Memory | 8 GB DDR4-3200 |
| Storage | 256 GB SSD |
| Hypervisor | Proxmox VE 9.2 |
| Primary Workloads | Debian Gateway, Metasploitable 2 |

The limited hardware resources required careful consideration of virtual machine resource allocation and workload selection.

## Virtualization Environment

### Proxmox VE

Proxmox VE serves as the primary virtualization platform, providing:

- KVM-based virtual machines.
- Virtual network bridges.
- Web-based virtual machine management.
- Virtual disk and resource administration.
- Console access for troubleshooting.
- Support for isolated cybersecurity testing environments.

### Virtual Machines

**VM 100 — Debian Lab Gateway**

A Debian 13-based Linux virtual machine configured to provide connectivity between the upstream network and the isolated laboratory environment.

Key functions:

- IPv4 packet forwarding.
- Network routing between designated interfaces.
- Tailscale overlay connectivity.
- Controlled access to isolated lab systems.
- Linux network and firewall administration.

**VM 101 — Metasploitable 2**

An intentionally vulnerable Linux virtual machine deployed as a penetration-testing target.

Key characteristics:

- Imported from an existing VMDK virtual disk.
- Connected to the isolated Proxmox laboratory bridge.
- Accessible through the configured laboratory access path.
- Used for authorized vulnerability discovery and exploitation exercises.
- Not intended for direct public internet exposure.

## Network Architecture

The environment uses separate virtual network bridges to distinguish host management and upstream connectivity from vulnerable laboratory workloads.

```text
       Authorized Remote Device
       Windows / Linux / Kali
                  |
                  |
         Encrypted Tailscale
           Overlay Network
                  |
                  |
        +-------------------+
        |  Debian Gateway   |
        |      VM 100       |
        |                   |
        |  IP Forwarding    |
        |  Routing/Firewall |
        +-------------------+
                  |
                  |
         Isolated Lab Bridge
                vmbr1
                  |
                  |
        +-------------------+
        |  Metasploitable 2 |
        |      VM 101       |
        +-------------------+


       Proxmox VE Host
       GMKtec NucBox G3 Plus
                  |
          vmbr0 - Management
                  |
          Home Network
```

*Simplified logical architecture. The Debian gateway connects the upstream and isolated lab networks. The Proxmox host also has its own Tailscale client for remote administration.*

### Network Bridges

**vmbr0 — Management / Upstream Network**

- Provides connectivity between the Proxmox host and the home network.
- Supports Proxmox management and upstream connectivity.
- Used by the Debian gateway's upstream-facing interface.

**vmbr1 — Isolated Cybersecurity Laboratory**

- Virtual bridge without a physical network port.
- Provides an internal network for vulnerable test machines.
- Separates laboratory workloads from ordinary home-network devices.
- Connects Metasploitable 2 to the dedicated gateway.

### Remote Access

Tailscale provides authenticated, encrypted overlay connectivity for administration and laboratory access.

The architecture uses two distinct remote-access functions:

1. Proxmox host administration through its Tailscale client.
2. Access to isolated laboratory resources through the Debian gateway.

Routing and firewall controls are used to restrict network traffic between environments.

**Security note:** The absence of a physical bridge connection provides network separation, but the gateway's forwarding and firewall configuration remains important for enforcing isolation.

## Security Architecture

The laboratory was designed around network segmentation, controlled connectivity, and minimizing unnecessary exposure.

### 1. Network Isolation

Intentionally vulnerable systems are placed on a dedicated virtual bridge rather than directly connected to the production home network.

This reduces exposure to unrelated devices and limits accidental interaction with vulnerable services.

### 2. Controlled Routing

The Debian gateway provides a defined routing point between the isolated laboratory network and authorized remote clients.

Linux IP forwarding and firewall configuration control traffic between network interfaces.

### 3. Encrypted Remote Access

Tailscale is used to provide authenticated remote connectivity without relying on direct public exposure of vulnerable laboratory services.

### 4. Separation of Responsibilities

The Proxmox host manages virtualization infrastructure, while the Debian gateway handles laboratory network connectivity.

This separates hypervisor administration from lab routing functions.

### 5. Intentionally Vulnerable Workloads

Metasploitable 2 is deliberately insecure and is used exclusively as an authorized testing target.

Its deployment requires network isolation and careful access management.

## Implementation Summary

### Phase 1 — Hypervisor Deployment

- Installed Proxmox VE on the GMKtec mini PC.
- Configured the management network.
- Enabled the Proxmox community software repository.
- Established access to the web management interface.
- Prepared storage and virtual networking for laboratory workloads.

### Phase 2 — Virtual Network Configuration

- Configured the upstream management bridge.
- Created an isolated virtual laboratory bridge.
- Assigned the laboratory network an internal address range.
- Configured connectivity between virtual machines and their assigned bridges.

### Phase 3 — Virtual Machine Deployment

- Imported Metasploitable 2 from a VMDK image.
- Created the Debian gateway virtual machine.
- Assigned network interfaces according to their required roles.
- Verified virtual machine connectivity.

### Phase 4 — Remote Connectivity

- Installed and configured Tailscale.
- Enabled IPv4 forwarding on the Debian gateway.
- Configured controlled access to laboratory resources.
- Established remote administrative connectivity to the Proxmox host.
- Verified access to the laboratory environment from an authorized client.

### Phase 5 — Troubleshooting and Network Migration

Following a home internet service and network equipment change, the Proxmox host experienced connectivity issues associated with the previous network configuration.

Troubleshooting activities included:

- Investigating Proxmox host connectivity after the network change.
- Recovering host accessibility following startup issues.
- Identifying outdated management network settings.
- Re-establishing remote administrative access.
- Diagnosing DNS resolution failures.
- Verifying connectivity using direct IP communication.
- Correcting DNS configuration.
- Confirming access to the Metasploitable testing environment.

These activities provided practical experience distinguishing DNS failures from broader network connectivity problems.

## Testing and Validation

The following operational checks were performed during the project:

| Test | Result |
|---|---|
| Proxmox host administration | Successful |
| Debian gateway deployment | Successful |
| Metasploitable 2 VM deployment | Successful |
| Tailscale remote access | Successful |
| Connectivity to isolated test target | Successful |
| DNS troubleshooting and recovery | Successful |
| Remote access following network migration | Successful |

**Validation scope:** These results demonstrate functional connectivity and operation. A complete firewall isolation audit and negative-access testing should be documented separately before claiming comprehensive segmentation validation.

## Challenges and Lessons Learned

### Resource Constraints

Running a virtualization platform on an 8 GB RAM system required attention to workload selection and resource allocation.

This reinforced the importance of capacity planning and understanding the overhead of virtualized environments.

### Network Segmentation

Separating vulnerable workloads from production devices demonstrated the importance of designing network boundaries before deploying security testing systems.

### Linux Routing

Configuring the Debian gateway provided practical experience with interfaces, IP forwarding, routing, and firewall-based access control.

### DNS Troubleshooting

During network migration, the Proxmox host was able to reach external IP addresses while domain-name resolution failed.

This helped distinguish a DNS configuration problem from a complete network connectivity failure.

### Remote Infrastructure Management

Using Tailscale provided experience managing infrastructure remotely and maintaining access to laboratory systems across changing network environments.

## Technologies and Skills Demonstrated

**Virtualization**
- Proxmox VE
- KVM
- Virtual machine provisioning
- Virtual disk imports
- Resource allocation

**Networking**
- TCP/IP
- Virtual network bridges
- Network segmentation
- IP forwarding
- Routing
- DNS troubleshooting

**Linux Administration**
- Debian Linux
- Network configuration
- Firewall administration
- Service troubleshooting
- Command-line diagnostics

**Security**
- Encrypted remote access
- Tailscale
- Vulnerable system isolation
- Controlled testing environments
- Defense-in-depth principles

**Documentation**
- Technical architecture
- Troubleshooting methodology
- Security design decisions
- Operational validation

## Future Improvements

- Perform and document formal network isolation tests.
- Review and refine firewall rules.
- Add sanitized configuration examples.
- Create a detailed network architecture diagram.
- Develop repeatable virtual machine deployment procedures.
- Evaluate additional intentionally vulnerable targets within available hardware resources.
- Document integration with the broader cybersecurity monitoring environment.

## Relationship to Other Projects

This Proxmox environment supports hands-on cybersecurity testing and infrastructure experimentation.

It complements other portfolio projects, including:

- **SSH Failed Login Analyzer:** Python-based authentication log analysis and brute-force detection.
- **Wazuh SIEM Homelab:** Centralized security monitoring, endpoint telemetry, and security event investigation.

The Wazuh monitoring environment is a separate deployment and should not be confused with a virtual machine hosted on this Proxmox system.

## Security and Privacy

All publicly shared documentation is intended to use sanitized or illustrative network information.

Actual LAN addresses, isolated laboratory addresses, Tailscale addresses, credentials, authentication tokens, and identifying configuration details are excluded.

The environment is intended exclusively for authorized security research, training, and experimentation.

## Project Status

**Operational — Documentation and security validation ongoing.**

The core virtualization environment, Debian gateway, isolated testing network, and remote access functionality have been deployed and tested.

Additional documentation, security validation, and improvements remain in progress.

---

**Author:** Michael Schoenbeck  
**GitHub:** https://github.com/Kosic117
