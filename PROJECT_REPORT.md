# Secure Proxmox Cybersecurity Homelab
## Technical Project Report

**Author:** Michael Schoenbeck  
**Platform:** Proxmox Virtual Environment 9.2  
**Project Type:** Virtualization, Network Security, Linux Administration  
**Status:** Operational — Further security validation planned

---

## 1. Executive Summary

This project involved designing, deploying, and maintaining a self-hosted cybersecurity laboratory using Proxmox VE on a compact, resource-constrained computer.

The primary objective was to establish a persistent virtualization environment for authorized cybersecurity testing while separating intentionally vulnerable systems from the primary home network.

The solution incorporates a dedicated Debian Linux gateway, isolated virtual networking, and Tailscale overlay connectivity to support remote laboratory access.

The project also involved troubleshooting infrastructure problems following an internet service provider migration, including outdated network settings, remote connectivity failures, and DNS resolution issues.

The completed environment provides a foundation for continued security testing, infrastructure experimentation, and network administration practice.

## 2. Project Objectives

The project was developed around the following objectives:

1. Deploy a dedicated virtualization host using available hardware.
2. Establish an isolated virtual network for vulnerable testing systems.
3. Configure a Linux gateway to support controlled laboratory connectivity.
4. Enable encrypted remote administration without directly publishing vulnerable services to the internet.
5. Deploy Metasploitable 2 as an authorized testing target.
6. Develop practical experience troubleshooting Linux networking and virtualization problems.
7. Document the environment in a reproducible and security-conscious manner.

## 3. Hardware and Software Environment

### 3.1 Physical Hardware

| Component | Specification |
|---|---|
| Manufacturer | GMKtec |
| Model | NucBox G3 Plus |
| Processor | Intel N150 |
| CPU Configuration | 4 Cores / 4 Threads |
| Memory | 8 GB DDR4-3200 |
| Storage | 256 GB SSD |
| Hypervisor | Proxmox VE 9.2 |

The hardware was selected as a low-cost platform for persistent laboratory services.

Its limited memory and storage capacity required consideration of virtual machine sizing, simultaneous workload requirements, and future expansion.

### 3.2 Software Components

**Proxmox VE**

Provides the virtualization platform, virtual networking, storage management, and administrative interface.

**Debian 13 Linux**

Runs as a dedicated gateway VM responsible for routing between the upstream environment and isolated laboratory network.

**Tailscale**

Provides authenticated, encrypted overlay connectivity for remote administration and authorized access.

**Metasploitable 2**

An intentionally vulnerable Linux distribution used for controlled penetration-testing exercises.

### 3.3 Virtual Machine Inventory

| VM ID | Operating System | Purpose |
|---|---|---|
| 100 | Debian 13 | Laboratory gateway and routing |
| 101 | Metasploitable 2 | Vulnerable testing target |

The Wazuh SIEM environment is maintained as a separate cybersecurity project and is not represented as a workload hosted on this Proxmox system.

## 4. Network Architecture

### 4.1 Design Overview

The environment separates virtualization management and upstream connectivity from vulnerable testing workloads.

A dedicated Debian gateway connects the isolated laboratory network to authorized remote clients through the configured routing and overlay access path.

The Proxmox host also runs its own Tailscale client for administrative access.

### 4.2 Logical Network Diagram

```text
              Authorized Remote Client
                Windows / Linux
                       |
                       |
               Tailscale Overlay
                       |
             +---------+---------+
             |                   |
             |                   |
      Proxmox Host         Debian Gateway
      Administration          VM 100
                                  |
                                  |
                          Isolated Lab Bridge
                                vmbr1
                                  |
                                  |
                           Metasploitable 2
                                VM 101


      Proxmox VE Host
      GMKtec NucBox G3 Plus
              |
       vmbr0 — Upstream
              |
        Home Network
```

*Simplified logical diagram. The gateway also has an upstream-facing interface. The Tailscale overlay provides authenticated connectivity; forwarding and firewall configuration determine permitted traffic paths.*

### 4.3 Virtual Network Bridges

**vmbr0 — Upstream and Management**

Provides connectivity to the physical home network and supports Proxmox management.

The Debian gateway uses this upstream network for connectivity outside the isolated laboratory.

**vmbr1 — Isolated Laboratory**

Provides an internal virtual network for testing workloads.

This bridge is not directly connected to a physical network interface.

Metasploitable 2 is attached to this isolated network, with the Debian gateway providing the intended routed access path.

### 4.4 Network Addressing

Actual IP addresses and subnets are excluded from the public documentation.

For explanatory purposes, the following documentation-only networks may be used:

| Network | Example Address Range |
|---|---|
| Upstream Management | 192.0.2.0/24 |
| Isolated Laboratory | 198.51.100.0/24 |
| Remote Overlay | Tailscale-managed addressing, not disclosed |

These are illustrative documentation ranges, not actual deployed values.

## 5. Proxmox Deployment

### 5.1 Hypervisor Installation

Proxmox VE was installed directly on the GMKtec mini PC.

Initial deployment included:

- Installing the hypervisor.
- Configuring the management network.
- Establishing administrative access.
- Enabling the community software repository.
- Preparing virtual networking and storage.
- Creating the initial virtual machines.

### 5.2 Virtualization Configuration

Proxmox provides KVM-based virtual machine hosting and virtual network bridge management.

The laboratory uses separate VMs for network infrastructure and security testing.

This approach allows the gateway to be managed independently from vulnerable testing targets.

### 5.3 Resource Considerations

With 8 GB of physical RAM, resource allocation is an important design constraint.

The environment prioritizes lightweight Linux systems and limited concurrent workloads.

Potential future expansion may require additional memory, storage, or a separate virtualization host.

## 6. Debian Gateway Implementation

### 6.1 Purpose

The Debian gateway provides a controlled connection point between the isolated laboratory network and authorized remote clients.

The gateway was configured to support:

- Multiple network interfaces.
- IPv4 forwarding.
- Routing between designated networks.
- Tailscale overlay connectivity.
- Firewall-based traffic control.

### 6.2 IPv4 Forwarding

IPv4 forwarding allows the Linux operating system to route packets between interfaces.

The gateway's forwarding status can be inspected using:

```bash
sysctl net.ipv4.ip_forward
```

A value of `1` indicates that IPv4 forwarding is enabled.

Forwarding alone does not enforce secure segmentation. Firewall rules and routing restrictions must also be configured appropriately.

### 6.3 Routing

The gateway uses network routes to direct traffic toward the isolated laboratory subnet.

Relevant diagnostic commands include:

```bash
ip -br address
ip route show
```

These display interface addresses and routing information.

### 6.4 Security Considerations

The gateway is a critical network boundary.

Its configuration should be evaluated to ensure:

- Only authorized remote clients can access laboratory systems.
- Laboratory systems cannot initiate unauthorized connections to production devices.
- Unnecessary forwarding paths are disabled.
- Firewall rules are maintained and reviewed.
- Network changes do not unintentionally expand access.

A formal negative-access validation procedure remains planned.

## 7. Tailscale Remote Access

### 7.1 Requirements

The environment needed to remain accessible while away from the local network.

Remote access was required for both:

- Proxmox infrastructure administration.
- Authorized interaction with isolated laboratory virtual machines.

### 7.2 Implementation

Tailscale was configured on the Proxmox host and Debian gateway.

The Proxmox host's Tailscale client provides a remote administrative access path.

The Debian gateway provides the intended access path to the isolated laboratory network.

This separation distinguishes hypervisor administration from access to testing targets.

### 7.3 Security Benefits

The overlay approach provides:

- Encrypted connectivity between participating devices.
- Authenticated device membership.
- Remote access without directly publishing the vulnerable target's services.
- Reduced dependence on conventional inbound port forwarding.

However, Tailscale membership alone does not guarantee complete network isolation. Access policies, routing, firewall configuration, and endpoint security remain relevant.

## 8. Metasploitable 2 Deployment

### 8.1 Purpose

Metasploitable 2 was deployed as a deliberately vulnerable Linux system for authorized security testing.

It supports practice with:

- Network discovery.
- Service enumeration.
- Vulnerability identification.
- Exploitation in a controlled environment.
- Post-exercise technical documentation.

### 8.2 Virtual Disk Import

Metasploitable 2 was deployed by importing an existing VMDK virtual disk into Proxmox.

The imported disk was attached to a Proxmox virtual machine and configured as its boot device.

### 8.3 Network Placement

The VM was connected to the isolated laboratory bridge.

This avoids directly placing the vulnerable target on the upstream home network.

### 8.4 Operational Verification

The VM was started successfully and later confirmed reachable from an authorized client.

This established that the virtual disk import and intended access path were functional.

### 8.5 Security Risk

Metasploitable 2 contains intentionally vulnerable services.

It should remain restricted to authorized testing environments and should not be exposed directly to the public internet.

Its connectivity should be reviewed after any network or hypervisor configuration changes.

## 9. Infrastructure Troubleshooting

### 9.1 Internet Provider Migration

The home network was migrated from Spectrum to Breezeline.

Following the change, the Proxmox host was no longer reachable through its previous management configuration.

The host also initially failed to provide usable local display output.

After an extended shutdown and restart, the host became accessible for further investigation.

Outdated management network settings were identified, and remote access was subsequently restored.

The initial display issue was not conclusively diagnosed.

### 9.2 DNS Resolution Failure

After basic network connectivity was restored, the host experienced DNS resolution problems.

The investigation established that external IP connectivity worked while hostname resolution failed.

This distinction indicated that the issue was related to DNS configuration rather than a complete loss of network access.

The DNS configuration was corrected and normal name resolution restored.

### 9.3 Remote Administration Recovery

Tailscale was configured directly on the Proxmox host, providing an additional remote management path.

Successful remote connectivity was verified.

### 9.4 Laboratory Connectivity Verification

Following network recovery, connectivity to Metasploitable 2 was confirmed from an authorized client.

This demonstrated that the primary laboratory access path remained operational.

For detailed symptoms, diagnostic commands, corrective actions, and lessons learned, see [Troubleshooting Documentation](TROUBLESHOOTING.md).

## 10. Testing and Validation

### 10.1 Functional Validation

| Test | Result |
|---|---|
| Proxmox management access | Passed |
| Debian gateway deployment | Passed |
| Metasploitable 2 disk import and startup | Passed |
| Tailscale host administration | Passed |
| Remote access to laboratory target | Passed |
| DNS recovery following migration | Passed |
| Remote access after network changes | Passed |

### 10.2 Validation Limitations

The testing completed to date establishes operational functionality.

It does not constitute a complete security assessment of the network segmentation design.

The following tests have not yet been documented as completed:

- Attempts to access production home-network devices from the isolated laboratory.
- Attempts to access the public internet from the vulnerable target.
- Unauthorized client access attempts through the overlay network.
- Firewall rule review against a defined least-privilege policy.
- Recovery testing following simultaneous host and gateway restarts.

These tests are planned to provide stronger evidence of the intended security boundaries.

## 11. Security Analysis

### 11.1 Implemented Design Measures

The environment incorporates several security-oriented architectural decisions:

**Virtual Network Separation**

Vulnerable workloads are placed on a dedicated virtual bridge rather than directly connected to the home network.

**Dedicated Routing Gateway**

A Debian VM provides a defined point for controlling traffic entering or leaving the isolated laboratory.

**Encrypted Remote Connectivity**

Tailscale provides authenticated overlay connectivity for authorized devices.

**Separation of Administrative Functions**

Hypervisor administration and laboratory routing are handled through distinct components.

### 11.2 Residual Risks

Potential risks include:

- Firewall misconfiguration allowing unintended network access.
- Incorrect routing or forwarding rules.
- Excessive permissions for overlay-network participants.
- Exposure of vulnerable services through configuration changes.
- Resource exhaustion on limited hardware.
- Loss of management access following network changes.

### 11.3 Recommended Improvements

- Perform formal negative-access testing.
- Review gateway firewall policy.
- Document approved traffic flows.
- Establish configuration backup procedures.
- Implement routine patching and maintenance.
- Review Tailscale access policies.
- Record configuration changes and verification results.

## 12. Challenges and Lessons Learned

### 12.1 Resource-Constrained Virtualization

The Intel N150 system demonstrated that useful cybersecurity infrastructure can operate on relatively inexpensive hardware.

However, the limited memory capacity requires careful workload planning.

### 12.2 Network Design Before Deployment

Intentionally vulnerable systems should be placed in an appropriately segmented environment before being started.

Virtual networking simplifies this process but still requires routing and access-control validation.

### 12.3 Distinguishing DNS from Connectivity Failures

The ISP migration provided a practical example of why network troubleshooting should proceed systematically.

Successful communication with an external IP address helped narrow the investigation to hostname resolution.

### 12.4 Maintaining Administrative Access

Remote infrastructure requires reliable access paths and recovery procedures.

The experience reinforced the value of local console access, documented network settings, and secure overlay administration.

### 12.5 Documentation and Reproducibility

Recording architecture, configuration decisions, and troubleshooting outcomes makes the environment easier to maintain and provides evidence of practical technical experience.

## 13. Future Development

Planned improvements include:

1. Complete formal laboratory network isolation testing.
2. Document sanitized firewall policies and approved traffic flows.
3. Produce a detailed network architecture diagram.
4. Implement configuration backup and recovery procedures.
5. Review host and gateway hardening.
6. Add repeatable deployment and recovery documentation.
7. Evaluate additional security testing workloads as hardware resources permit.
8. Explore integration with broader monitoring and detection projects.

## 14. Conclusion

This project established a functional, remotely accessible Proxmox cybersecurity laboratory using resource-constrained hardware.

The implementation combined virtualization, Linux routing, isolated virtual networking, and encrypted overlay access to support authorized security testing.

The project also provided hands-on troubleshooting experience involving network migration, DNS resolution, and remote infrastructure recovery.

While comprehensive security validation remains an ongoing objective, the operational environment demonstrates practical experience with infrastructure deployment, network administration, virtualization, and security-focused system design.

---

**Author:** Michael Schoenbeck  
**GitHub:** https://github.com/Kosic117  
**Related Project:** [SSH Failed Login Analyzer](https://github.com/Kosic117/SSH_Log_Analyzer)
