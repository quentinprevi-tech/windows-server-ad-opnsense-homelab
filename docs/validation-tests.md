# Validation Tests

This document lists the main validation tests performed during the lab deployment.

The goal is to prove that the infrastructure works as expected and that firewall segmentation is effective.

## Active Directory and DNS

Tests performed from SRV-AD01:

- hostname
  - Expected result: SRV-AD01

- nslookup homelab.local
  - Expected result: homelab.local resolves through SRV-AD01.

- nslookup -type=SRV _ldap._tcp.dc._msdcs.homelab.local
  - Expected result: SRV-AD01 is returned as domain controller.

- dcdiag /test:dns
  - Expected result: DNS tests pass.

Result: Active Directory and DNS were validated successfully.

## Windows 11 Domain Client

Tests performed from WIN11-LAB:

- ipconfig /all
  - Expected result: DNS points to SRV-AD01 at 10.10.20.10.

- nslookup homelab.local
  - Expected result: domain DNS resolution succeeds.

- nltest /dsgetdc:homelab.local
  - Expected result: SRV-AD01 is discovered as the domain controller.

- gpupdate /force
  - Expected result: Group Policy update succeeds.

- gpresult /scope computer /r
  - Expected result: workstation GPO is applied.

- gpresult /scope user /r
  - Expected result: user GPOs are applied.

Result: the Windows 11 client was successfully joined to the domain and receives GPOs.

## File Share and Drive Mapping

Tests performed from WIN11-LAB with the domain user test.user:

- Access network share
  - Path: \\SRV-AD01\Partage-Lab
  - Expected result: access succeeds.

- Create a test file in the share
  - Expected result: write access succeeds through the AD security group.

- net use
  - Expected result: drive P: is mapped to \\SRV-AD01\Partage-Lab.

Result: AD group-based share access and GPO drive mapping were validated.

## DMZ Web Server

Tests performed from WIN11-LAB:

- nslookup web01.homelab.local
  - Expected result: web01.homelab.local resolves to 10.10.30.10.

- Open http://web01.homelab.local
  - Expected result: custom Nginx page is displayed.

- Test-NetConnection 10.10.30.10 -Port 80
  - Expected result: TCP test succeeds.

Result: LAN to DMZ HTTP access was validated.

## WAN to DMZ NAT

Test performed from the physical PC on the home network:

- Open http://192.168.0.57:8080
  - Expected result: traffic is forwarded to web01 on 10.10.30.10:80.

Result: WAN-to-DMZ NAT publication was validated.

## DMZ Segmentation

Tests performed from web01:

- ping -c 4 10.10.30.1
  - Expected result: succeeds.

- ping -c 4 8.8.8.8
  - Expected result: succeeds.

- ping -c 4 10.10.10.105
  - Expected result: blocked.

- ping -c 4 10.10.20.10
  - Expected result: blocked.

- getent hosts deb.debian.org
  - Expected result: DNS resolution succeeds.

- apt update
  - Expected result: package repository access succeeds.

Result: the DMZ can access its gateway, the Internet and internal DNS, but cannot freely access LAN or SERVERS.

## Backup Validation

Proxmox snapshots and vzdump backups were created after key milestones.

Protected systems:

- VM 300: OPNsense firewall
- VM 310: Windows 11 domain client
- VM 320: SRV-AD01 domain controller
- VM 330: Debian/Nginx DMZ web server
