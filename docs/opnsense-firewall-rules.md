# OPNsense Firewall Rules

This document summarizes the main firewall and NAT rules used in the lab.

The goal is to replace broad default allow rules with targeted rules based on the least-privilege principle.

## Interfaces

- WAN
  - Network: 192.168.0.0/24
  - OPNsense IP: 192.168.0.57

- LAN
  - Network: 10.10.10.0/24
  - OPNsense IP: 10.10.10.1

- SERVERS
  - Network: 10.10.20.0/24
  - OPNsense IP: 10.10.20.1

- DMZ
  - Network: 10.10.30.0/24
  - OPNsense IP: 10.10.30.1

## Aliases

- SRV_AD01
  - 10.10.20.10
  - Windows Server domain controller, DNS server and file server.

- WEB01_DMZ
  - 10.10.30.10
  - Debian/Nginx web server in the DMZ.

- AD_DNS_PORTS
  - TCP/UDP 53
  - DNS queries to the Active Directory DNS server.

- WEB_PORTS
  - TCP 80 and 443
  - HTTP and HTTPS traffic.

- NTP_PORTS
  - UDP 123
  - Time synchronization.

## Disabled Broad Rules

The following broad rules were disabled after targeted rules were created and tested:

- Default allow LAN to any IPv4 rule.
- Default allow LAN IPv6 to any rule.
- Temporary SERVERS / OPT1 net to any rule.

These rules were useful during initial setup, but they were too permissive for a segmented lab.

## LAN Rules

The LAN contains the Windows 11 domain client.

Allowed traffic:

- LAN net to SRV_AD01: any protocol
  - Required for Active Directory, DNS, GPO, SMB and RPC services.
  - This is limited to the domain controller, not the whole SERVERS network.

- LAN net to any: TCP 80/443
  - Allows client web access and updates.

- LAN net to any: ICMP
  - Allows ping for diagnostics.

Blocked by default:

- LAN to DMZ except explicitly allowed services.
- LAN to other SERVERS hosts unless a rule is created.
- LAN to arbitrary destinations on any port.

## SERVERS Rules

The SERVERS network contains SRV-AD01.

Allowed traffic from SRV_AD01:

- SRV_AD01 to any: TCP/UDP 53
  - Allows external DNS resolution or DNS forwarding.

- SRV_AD01 to any: TCP 80/443
  - Allows Windows updates, downloads and certificate checks.

- SRV_AD01 to any: UDP 123
  - Allows NTP time synchronization.

- SRV_AD01 to any: ICMP
  - Allows ping diagnostics.

Blocked by default:

- General SERVERS net to any traffic.
- Any traffic from other servers unless explicitly allowed.

## DMZ Rules

The DMZ contains the Debian/Nginx web server.

Allowed traffic from DMZ:

- DMZ net to SRV_AD01: TCP/UDP 53
  - Allows DNS queries to the internal DNS server only.

- DMZ net to any: TCP 80/443
  - Allows package updates and web downloads.

- DMZ net to DMZ gateway: ICMP
  - Allows ping to 10.10.30.1 for diagnostics.

- DMZ net to 8.8.8.8: ICMP
  - Allows a simple external ping test.

Blocked by default:

- DMZ to LAN.
- DMZ to SERVERS, except DNS to SRV_AD01.
- DMZ to SRV_AD01 ping.
- DMZ to internal SMB, AD or management services.

## WAN to DMZ NAT

A Destination NAT rule publishes the DMZ web server to the home network:

- Source: any
- Destination: WAN address
- External port: TCP 8080
- Redirect target: 10.10.30.10
- Redirect port: TCP 80

Result:

- http://192.168.0.57:8080 forwards to http://10.10.30.10:80

This exposes only the DMZ web server, not the internal LAN or SERVERS network.

## Internal Access

Internal clients access the DMZ web server using:

- http://web01.homelab.local
- http://10.10.30.10

Accessing the WAN address from inside the lab is not required and would need NAT reflection / hairpin NAT.

## Security Summary

The firewall policy follows this model:

- LAN can access required AD services and the DMZ web service.
- SERVERS traffic is restricted to required outbound services.
- DMZ can access the Internet for updates and internal DNS only.
- DMZ cannot freely access LAN or SERVERS.
- WAN publishes only one controlled service to the DMZ.
