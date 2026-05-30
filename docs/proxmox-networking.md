# Proxmox Networking

This lab uses Proxmox VE virtual bridges to simulate separate physical network segments.

Each Proxmox bridge acts like a virtual switch. Virtual machines connected to the same bridge are placed in the same Layer 2 network.

## Bridge Overview

- vmbr0
  - Purpose: Home network / WAN
  - Network: 192.168.0.0/24
  - Proxmox management IP: 192.168.0.156/24
  - Gateway: 192.168.0.1
  - This is the only bridge connected to the physical home network.

- vmbr10
  - Purpose: LAN clients
  - Network: 10.10.10.0/24
  - Proxmox IP: none
  - Used by the Windows 11 domain client.

- vmbr20
  - Purpose: internal servers
  - Network: 10.10.20.0/24
  - Proxmox IP: none
  - Used by the Windows Server domain controller.

- vmbr30
  - Purpose: DMZ
  - Network: 10.10.30.0/24
  - Proxmox IP: none
  - Used by the Debian/Nginx web server.

## Why only vmbr0 has a Proxmox IP

Only vmbr0 has an IP address on the Proxmox host because it is used for Proxmox management.

The internal bridges vmbr10, vmbr20 and vmbr30 do not have Proxmox IP addresses. They are used only as isolated virtual switches.

This keeps the Proxmox host out of the internal lab networks and leaves routing and firewalling to OPNsense.

## OPNsense Interfaces

OPNsense is connected to all network zones:

- WAN: vmbr0, 192.168.0.57/24
- LAN: vmbr10, 10.10.10.1/24
- SERVERS: vmbr20, 10.10.20.1/24
- DMZ: vmbr30, 10.10.30.1/24

OPNsense acts as the router and firewall between all zones.

## VM Placement

- WIN11-LAB is connected to vmbr10.
- SRV-AD01 is connected to vmbr20.
- web01 is connected to vmbr30.
- OPNsense is connected to vmbr0, vmbr10, vmbr20 and vmbr30.

## Design Reasoning

The goal is to simulate a real enterprise network using virtual machines only.

Instead of using physical switches or VLANs, Proxmox bridges are used to create isolated virtual networks.

Traffic between LAN, SERVERS and DMZ must pass through OPNsense, where firewall rules control what is allowed or blocked.

This design makes it possible to practice routing, firewalling, Active Directory, DMZ publishing and network segmentation inside a single Proxmox host.
