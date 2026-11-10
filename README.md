# Secure Proxmox Cybersecurity Homelab

**Virtualization | Network Segmentation | Linux Administration | Secure Remote Access**

A self-hosted cybersecurity laboratory built on Proxmox VE to support hands-on security testing, Linux administration, and network security experimentation.

The environment uses an isolated virtual network, a Debian Linux routing gateway, and Tailscale for authenticated remote access. The goal is to maintain a functional testing environment without directly exposing intentionally vulnerable systems to the public internet or placing them on the primary home network.

## Technologies

| Component | Technology |
|---|---|
| Hardware | GMKtec NucBox G3 Plus — Intel N150, 8 GB RAM |
| Hypervisor | Proxmox VE 9.2 |
| Gateway | Debian 13 Linux |
| Remote Access | Tailscale |
| Networking | Linux IP Forwarding, Virtual Bridges, Routing |
| Security Testing | Metasploitable 2 |

## Architecture

The laboratory separates management connectivity from vulnerable testing workloads.

- **Proxmox Host:** Provides virtualization and management services.
- **Debian Gateway (VM 100):** Handles routing between the isolated laboratory and authorized remote clients.
- **Metasploitable 2 (VM 101):** Intentionally vulnerable Linux testing target.
- **Tailscale:** Provides encrypted remote connectivity to the Proxmox host and laboratory gateway.

The environment uses separate virtual bridges for upstream connectivity and isolated laboratory traffic.

## Key Accomplishments

- Deployed and administered Proxmox VE on resource-constrained hardware.
- Imported and configured Metasploitable 2 as a virtual testing target.
- Created an isolated virtual network for vulnerable workloads.
- Configured a Debian Linux gateway with IPv4 forwarding and routing.
- Established authenticated remote administration through Tailscale.
- Recovered Proxmox connectivity following an ISP and network migration.
- Diagnosed and resolved DNS resolution problems.
- Verified remote access to the Proxmox host and laboratory target.

## Security Considerations

The environment is designed to reduce exposure of intentionally vulnerable systems through virtual network separation and controlled routing.

**Current limitation:** Functional remote access has been verified, but comprehensive negative-access testing and firewall policy validation remain future work.

All publicly shared network information is sanitized. Actual LAN addresses, laboratory addresses, Tailscale addresses, and authentication information are excluded.

## Documentation

- [Technical Project Report](PROJECT_REPORT.md) — Architecture, implementation, security design,
