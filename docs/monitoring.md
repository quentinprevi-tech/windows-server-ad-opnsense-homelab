# Monitoring

This document describes the monitoring extension added to the Windows Server Active Directory + opnsense Homelab.

## Objective

The goal was to add a basic monitoring layer to the lab without redesigning the network architecture.

The monitoring stack uses the existing Prometheus and Grafana containers already present in the Proxmox homelab.

## Monitoring Components

- Prometheus
  - CT: 101
  - IP: 192.168.0.101
  - Port: 9090

- Grafana
  - CT: 102
  - IP: 192.168.0.102
  - Port: 3000

- web01 node_exporter
  - Host: Debian/Nginx DMZ server
  - Internal IP: 10.10.30.10
  - Exporter port: 9100

- SRV-AD01 windows_exporter
  - Host: Windows Server Active Directory
  - Internal IP: 10.10.20.10
  - Exporter port: 9182

## Design

Prometheus and Grafana remain on the home network 192.168.0.0/24.

The AD/OPNsense lab networks are isolated behind OPNsense:

- SERVERS: 10.10.20.0/24
- DMZ: 10.10.30.0/24

Instead of adding Prometheus directly to the lab networks, OPNsense Destination NAT rules publish only the exporter ports needed for monitoring.

This keeps the monitoring access explicit and limited.

## OPNsense NAT Rules

The following NAT rules were created on OPNsense.

### web01 node_exporter

- Interface: WAN
- Protocol: TCP
- Source: PROMETHEUS (192.168.0.101)
- Destination: WAN address
- Destination port: 9100
- Redirect target IP: 10.10.30.10
- Redirect target port: 9100
- Description: NAT Prometheus to web01 node_exporter

### SRV-AD01 windows_exporter

- Interface: WAN
- Protocol: TCP
- Source: PROMETHEUS (192.168.0.101)
- Destination: WAN address
- Destination port: 9182
- Redirect target IP: 10.10.20.10
- Redirect target port: 9182
- Description: NAT Prometheus to SRV-AD01 windows_exporter

## Exporters

### web01

Debian node_exporter was installed on web01.

Validation:

- systemctl status prometheus-node-exporter --no-pager
- ss -lntp | grep 9100

Result:

- node_exporter listens on port 9100
- Prometheus can scrape the metrics through OPNsense NAT

### SRV-AD01

windows_exporter was installed on SRV-AD01.

Validation:

- Get-Service windows_exporter
- Test-NetConnection 127.0.0.1 -Port 9182
- Test-NetConnection 10.10.20.10 -Port 9182

Result:

- windows_exporter runs as a Windows service
- SRV-AD01 exposes metrics on port 9182

## Prometheus Jobs

The following scrape jobs were added to Prometheus:

### web01


- job_name: "ad-lab-web01-node"
  static_configs:
    - targets: ["192.168.0.57:9100"]
      labels:
        lab: "ad-opnsense"
        zone: "dmz"
        host: "web01"

### SRV-AD01


- job_name: "ad-lab-srv-ad01-windows"
  static_configs:
    - targets: ["192.168.0.57:9182"]
      labels:
        lab: "ad-opnsense"
        zone: "servers"
        host: "srv-ad01"

## Validation

The targets were validated in Prometheus.

### Prometheus Targets

### Grafana Dashboard

A Grafana dashboard was created with two status panels:

- web01 node_exporter status
- SRV-AD01 windows_exporter status

Queries used:

- up{job="ad-lab-web01-node"}
- up{job="ad-lab-srv-ad01-windows"}

When the value is 1 the panel displays UP.
When the value is 0 the panel displays DOWN.

The dashboard was also tested by temporarily stopping an exporter and confirming that Grafana changed from UP to DOWN.

## Screenshots

- screenshots/prometheus-ad-lab-targets.png
- screenshots/grafana-ad-lab-status-dashboard.png

## Design Reasoning

This monitoring extension keeps the network segmentation intact.

The monitoring server does not need full access to the lab networks. OPNsense publishes only the required monitoring ports and limits the source to the Prometheus server.

This provides a simple but realistic operational monitoring layer for the AD/DMZ lab.
