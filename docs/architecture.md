# Architecture

This lab simulates a small enterprise network using Proxmox VE and virtual machines.

The goal is to separate client workstations, internal servers and exposed services into different network zones, then control traffic between these zones with OPNsense.

## Network Zones

| Zone | Network | Description |
|---|---:|---|
| WAN / Home Network | 192.168.0.0/24 | External network used by OPNsense WAN |
| LAN | 10.10.10.0/24 | Domain client workstations |
| SERVERS | 10.10.20.0/24 | Internal services such as Active Directory and DNS |
| DMZ | 10.10.30.0/24 | Exposed services such as the Nginx web server |

## Main Components

| Component | Role |
|---|---|
| Proxmox VE | Hypervisor hosting all virtual machines |
| OPNsense | Firewall and router between all network zones |
| SRV-AD01 | Windows Server domain controller, DNS server and file server |
| WIN11-LAB | Windows 11 domain-joined client |
| web01 | Debian/Nginx web server located in the DMZ |

## Traffic Flow

- LAN clients use SRV-AD01 as their DNS server.
- LAN clients access Active Directory, GPOs and the file share on SRV-AD01.
- LAN clients can access the DMZ web server over HTTP.
- The DMZ web server can access the Internet for updates.
- The DMZ web server can query internal DNS on SRV-AD01.
- The DMZ cannot freely access the LAN or internal servers.
- WAN traffic on port 8080 is forwarded to the DMZ web server on port 80.

## Logical Diagram

- Home Network / WAN: 192.168.0.0/24
  - OPNsense WAN: 192.168.0.57

- LAN: 10.10.10.0/24
  - WIN11-LAB: 10.10.10.105

- SERVERS: 10.10.20.0/24
  - SRV-AD01: 10.10.20.10

- DMZ: 10.10.30.0/24
  - web01: 10.10.30.10

OPNsense routes and filters traffic between WAN, LAN, SERVERS and DMZ.

## Design Goals

- Practice enterprise-style network segmentation.
- Deploy and validate Active Directory services.
- Apply Group Policy to domain users and computers.
- Place exposed services in a DMZ instead of the internal server network.
- Harden firewall rules using a least-privilege approach.
- Validate every major service with practical tests.
- Protect the lab state with Proxmox snapshots and backups.
