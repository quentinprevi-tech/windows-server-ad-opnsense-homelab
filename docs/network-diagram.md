# Network Architecture Diagram

This diagram represents the validated network segmentation used in the Windows Server AD and OPNsense homelab.

```mermaid
flowchart TB
    Internet["Internet / Home Network<br>192.168.0.0/24"]
    OPN["OPNsense Firewall<br>WAN: 192.168.0.57"]

    LAN["LAN<br>10.10.10.0/24"]
    SERVERS["SERVERS<br>10.10.20.0/24"]
    DMZ["DMZ<br>10.10.30.0/24"]

    WIN11["Windows 11 Client<br>10.10.10.105"]
    AD["SRV-AD01<br>AD DS / DNS / DHCP<br>10.10.20.10"]
    WEB["web01<br>Debian / Nginx<br>10.10.30.10"]

    Internet --> OPN
    OPN --> LAN
    OPN --> SERVERS
    OPN --> DMZ

    LAN --> WIN11
    SERVERS --> AD
    DMZ --> WEB

    WIN11 -->|"AD / DNS / GPO / SMB"| AD
    WIN11 -->|"HTTP"| WEB
    WEB -->|"DNS only"| AD
    WEB -. "blocked" .-> WIN11
    WEB -. "blocked" .-> SERVERS

    Internet -->|"NAT 192.168.0.57:8080 to 10.10.30.10:80"| WEB
```

## Security Model

The lab uses segmented networks to separate client, server and DMZ workloads.

- LAN clients can access Active Directory services on SRV-AD01.
- LAN clients can access the DMZ web server over HTTP.
- The DMZ web server can resolve DNS through SRV-AD01.
- The DMZ is not allowed to freely access LAN or SERVERS.
- WAN access to the DMZ web server is published through a controlled NAT rule.
- Broad allow rules were replaced by targeted firewall rules after validation.

## Main Networks

| Zone | Subnet | Purpose |
|---|---:|---|
| WAN / Home network | 192.168.0.0/24 | Upstream home network |
| LAN | 10.10.10.0/24 | Client devices |
| SERVERS | 10.10.20.0/24 | Domain controller and server workloads |
| DMZ | 10.10.30.0/24 | Public-facing services |

## Main Hosts

| Host | Role | IP |
|---|---|---:|
| OPNsense | Firewall/router | 192.168.0.57 / 10.10.10.1 / 10.10.20.1 / 10.10.30.1 |
| win11-client-lab | Domain client | 10.10.10.105 |
| SRV-AD01 | AD DS, DNS, DHCP | 10.10.20.10 |
| web01 | Debian Nginx DMZ web server | 10.10.30.10 |
