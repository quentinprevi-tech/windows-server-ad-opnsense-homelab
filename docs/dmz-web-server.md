# DMZ Web Server

This document describes the Debian/Nginx web server deployed in the DMZ.

## VM Overview

- VM ID: 330
- VM name: debian-dmz-web01
- Hostname: web01
- Operating system: Debian
- Network zone: DMZ
- Proxmox bridge: vmbr30
- IP address: 10.10.30.10/24
- Gateway: 10.10.30.1
- DNS server: 10.10.20.10

## Purpose

web01 simulates a public-facing web server placed in a DMZ.

The goal is to expose a controlled web service without placing it directly inside the internal LAN or SERVERS network.

If the web server were compromised, firewall rules should limit lateral movement toward the LAN and Active Directory server.

## Nginx Configuration

Nginx was installed on web01 and configured with a dedicated site.

Main paths:

- Web root: /var/www/web01
- Nginx site config: /etc/nginx/sites-available/web01
- Enabled site: /etc/nginx/sites-enabled/web01
- Access log: /var/log/nginx/web01_access.log
- Error log: /var/log/nginx/web01_error.log

The default Nginx site was disabled and replaced with a custom page.

## Internal DNS

An A record was created on SRV-AD01 in the homelab.local DNS zone.

- Record name: web01
- FQDN: web01.homelab.local
- IP address: 10.10.30.10

This allows internal clients to access the web server using:

- http://web01.homelab.local
- http://10.10.30.10

## WAN Publication

The web server was publishlled on web01 and configured with a dedicated site.

Main paths:

- Web root: /var/www/web01
- Nginx site config: /etc/nginx/sites-available/web01
- Enabled site: /etc/nginx/sites-enabled/web01
- Access log: /var/log/nginx/web01_access.log
- Error log: /var/log/nginx/web01_error.log

The default Nginx site was disabled and replaced with a custom page.

## Internal DNS

An A record was created on SRV-AD01 in the homelab.local DNS zone.

- Record name: web01
- FQDN: web01.homelab.local
- IP address: 10.10.30.10

This allows internal clients to access the web server using:

- http://web01.homelab.local
- http://10.10.30.10

## WAN Publication

The web server was publisho LAN
- DMZ to SERVERS except DNS to SRV-AD01
- DMZ to SRV-AD01 ping
- DMZ to SMB or Active Directory services

## Validation

Tests performed from WIN11-LAB:

- nslookup web01.homelab.local
  - Expected result: resolves to 10.10.30.10

- Open http://web01.homelab.local
  - Expected result: custom Nginx page is displayed

- Test-NetConnection 10.10.30.10 -Port 80
  - Expected result: TCP test succeeds

Tests performed from the physical PC:

- Open http://192.168.0.57:8080
  - Expected result: OPNsense forwards the request to web01

Tests performed from web01:

- ping -c 4 10.10.30.1
  - Expected result: succeeds

- ping -c 4 8.8.8.8
  - Expected result: succeeds

- ping -c 4 10.10.10.105
  - Expected result: blocked

- ping -c 4 10.10.20.10
  - Expected result: blocked

- apt update
  - Expected result: succeeds

## Design Reasoning

The DMZ separates exposed services from internal systems.

The web server is reachable from the LAN and through a controlled WAN NAT rule, but it cannot freely access the LAN or the Active Directory server.

This design demonstrates a common enterprise security pattern: public-facing services are isolated in a DMZ and protected by firewall rules.
