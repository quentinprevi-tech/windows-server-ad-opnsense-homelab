# Active Directory

This document describes the Active Directory deployment used in the lab.

## Domain Controller

- Hostname: SRV-AD01
- IP address: 10.10.20.10/24
- Gateway: 10.10.20.1
- DNS: 10.10.20.10
- Network zone: SERVERS
- Proxmox VM ID: 320

SRV-AD01 is installed on Windows Server and provides the following roles:

- Active Directory Domain Services
- DNS Server
- File sharing for the lab network share
- Group Policy management

## Domain Information

- Active Directory domain: homelab.local
- NetBIOS name: HOMELAB
- Domain controller: SRV-AD01
- DNS zone: homelab.local

The domain was created as a new forest.

## DNS Role

SRV-AD01 acts as the DNS server for the lab domain.

Domain clients use SRV-AD01 for DNS resolution instead of public DNS servers.

This is required because Active Directory depends on DNS records such as SRV records to locate domain controllers and services.

Examples of validated DNS records:

- homelab.local
- SRV-AD01.homelab.local
- _ldap._tcp.dc._msdcs.homelab.local
- web01.homelab.local

## Organizational Units

The following OU structure was created:

- Workstations
  - Contains the Windows 11 domain client.

- Servers
  - Reserved for server computer objects.

- Lab-Users
  - Contains lab user accounts such as test.user.

- Groups
  - Contains lab security groups.

## User Accounts

The main test user is:

- Username: test.user
- Domain logon: HOMELAB\\test.user
- UPN logon: test.user@homelab.local
- OU: Lab-Users

This user was used to validate domain login, GPO application, file share access and drive mapping.

## Security Groups

The following security group was created:

- GG_Lab_Share_RW
  - Scope: Global
  - Type: Security
  - Purpose: grant read/write access to the lab file share.

The user test.user was added to this group.

## Domain Client

The Windows 11 client was joined to the domain:

- Hostname: WIN11-LAB
- IP address: 10.10.10.105
- Network zone: LAN
- Proxmox VM ID: 310
- Domain: homelab.local
- OU: Workstations

Validated client-side commands:

- whoami
- nslookup homelab.local
- nltest /dsgetdc:homelab.local
- gpupdate /force
- gpresult /scope computer /r
- gpresult /scope user /r

## Time Synchronization

Time synchronization is important in Active Directory because Kerberos authentication depends on time accuracy.

SRV-AD01 was configured to use external NTP sources.

OPNsense allows SRV-AD01 outbound NTP traffic on UDP 123.

## Design Reasoning

The domain controller is placed in the SERVERS network instead of the LAN network.

This allows the firewall to control which services LAN clients can access on the server.

The DMZ is only allowed to query DNS on SRV-AD01 and cannot freely access Active Directory services.

This reflects a more realistic segmented enterprise design.
