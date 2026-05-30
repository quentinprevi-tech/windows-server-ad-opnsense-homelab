# Windows Server Active Directory + OPNsense Homelab

Enterprise-style homelab built on Proxmox VE to practice system and network administration.

The lab simulates a small company network with:
- OPNsense firewall/router
- Segmented LAN / SERVERS / DMZ networks
- Windows Server Active Directory
- Internal DNS
- Windows 11 domain client
- Group Policy Objects
- Network share with AD permissions
- Debian/Nginx web server in a DMZ
- WAN-to-DMZ NAT publication
- Proxmox snapshots and backups

## Network Zones

| Zone | Network | Purpose |
|---|---:|---|
| WAN / Home Network | 192.168.0.0/24 | External network used as OPNsense WAN |
| LAN | 10.10.10.0/24 | Domain client workstations |
| SERVERS | 10.10.20.0/24 | Internal servers and Active Directory |
| DMZ | 10.10.30.0/24 | Exposed web services |

## Main Virtual Machines

| VM | Role | IP |
|---|---|---:|
| OPNsense | Firewall/router | 192.168.0.57 / 10.10.10.1 / 10.10.20.1 / 10.10.30.1 |
| SRV-AD01 | Windows Server AD DS + DNS | 10.10.20.10 |
| WIN11-LAB | Domain-joined Windows 11 client | 10.10.10.105 |
| web01 | Debian/Nginx DMZ web server | 10.10.30.10 |

## Features Implemented

- Proxmox virtual networking with isolated bridges
- OPNsense firewall with WAN, LAN, SERVERS and DMZ interfaces
- Active Directory domain: homelab.local
- AD-integrated DNS
- Windows 11 client joined to the domain
- OU structure, users and security groups
- GPOs for login banner, drive mapping and user restrictions
- File share with AD group-based permissions
- Debian/Nginx web server deployed in the DMZ
- DNS record: web01.homelab.local -> 10.10.30.10
- NAT rule: 192.168.0.57:8080 -> 10.10.30.10:80
- Firewall hardening between LAN, SERVERS and DMZ
- Validation tests for DNS, AD, GPO, SMB, HTTP, NAT and segmentation
- Proxmox snapshots and vzdump backups

## Network Diagram

Home Network / WAN 192.168.0.0/24
        |
        | OPNsense WAN 192.168.0.57
        |
    [ OPNsense ]
      /   |   \
     /    |    \
   LAN  SERVERS DMZ
10.10.10.0/24 10.10.20.0/24 10.10.30.0/24
   |       |       |
WIN11   SRV-AD01  web01
10.10.10.105 10.10.20.10 10.10.30.10

## Validation Summary

| Test | Result |
|---|---|
| Windows 11 domain join | Success |
| nltest /dsgetdc:homelab.local | Success |
| gpupdate /force | Success |
| Drive mapping through GPO | Success |
| web01.homelab.local DNS resolution | Success |
| LAN to DMZ HTTP access | Success |
| WAN to DMZ NAT on port 8080 | Success |
| DMZ to LAN ping | Blocked as expected |
| DMZ to SRV-AD01 ping | Blocked as expected |
| DMZ to SRV-AD01 DNS | Allowed |
| DMZ to Internet HTTP/HTTPS | Allowed |

## Security Approach

The lab follows a least-privilege firewall model.

Default broad rules such as LAN net to any and SERVERS net to any were disabled and replaced with targeted rules.

The DMZ can access the Internet for updates and query internal DNS, but it cannot freely access the LAN or internal servers.

## Documentation

More detailed documentation is available in the docs directory.

## Status

Lab implementation is complete and validated. Documentation is in progress.
