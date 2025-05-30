# Cluster Specification

## Overview

This is a spcification of everything that should be included in the cluster implementation.

## Good links

- https://geek-cookbook.funkypenguin.co.nz/
- https://bjw-s.github.io/helm-charts/docs/
- https://github.com/buroa/k8s-gitops
- https://github.com/gabe565/home-ops
- https://github.com/onedr0p/home-ops
- https://github.com/onedr0p/cluster-template

## Network

- Network: 192.168.222.0/24 "Server"
- DHCP range: 192.168.222.21-100
- Control plane shared IP: 192.168.222.20
- MetalLB load balancer IP-range: 192.168.222.101-192.168.222.130
- Network manufacturer: Ubiquiti Unifi
- Public domain name: myk8s.se
- Most services will be published over authenticated (Entra ID) Cloudflare tunnel.
  - Might change this later to unauthenticated tunnel and use Authentik instead.

Nodes will get their IP from DHCP, flagged as static in router.

## Hardware

### Kubernetes Nodes (Talos/CozyStack)

Three schedulable control plane nodes.
Operating System: Talos 9.5 opionated with CozyStack 0.29 (2025-04-03)

1. k8s01

- Hostname: k8s01
- IP: 102.168.222.21
- Model: Dell Optiplex 5090M
- CPU: Intel i5-10500T 2,3GHz (10:th gen, 6 cores/12 threads Q2 2020)
- RAM: 64GB (2x 32GB Kingston Fury DDR4 3200Mhz)
- Boot drive: 250GB SATA-SSD (PNY CS900)
- Storage drive: 1TB NVME Storage Drive (Kingston Fury Renegade PCIe 4.0)

2. k8s02

- Hostname: k8s02
- IP: 102.168.222.22
- Model: Dell Optiplex 5090M
- CPU: Intel i5-10500T 2,3GHz (10:th gen, 6 cores/12 threads Q2 2020)
- RAM: 64GB (2x 32GB Kingston Fury DDR4 3200MHz)
- Boot drive: 250GB SATA-SSD (PNY CS900)
- Storage drive: 1TB NVME (Kingston Fury Renegade PCIe 4.0)

3. k8s03

- Hostname: k8s03
- IP: 192.168.222.23
- Model: Intel NUC NUC10i7FNH
- CPU: Intel i7-10710U 1,1GHz CPU (10:th gen, 6 cores/12 threads Q3 2019)
- RAM: 64GB (2x 32GB Kingston Fury DDR4 3200MHz)
- Boot drive: 250GB SATA-SSD (PNY CS900)
- Storage drive: 1TB NVME (Kingston Fury Renegade PCIe 4.0)

### External storage, TrueNAS Scale:

1. truenas-stor

- Operating System: TrueNAS Scale Dragonfish-24.04.2 (Will be updated when apps are moved to cluster)
- Hostname: truenas-stor
- IP Address: 192.168.222.13
- Model: homebuild (AsRock Z370M-ITX/ac motherboard)
- CPU: i7-8700 3,2Ghz (8:th gen, 6cores, 12 threads, Q4 2017)
- RAM: 32GB (2133Mhz)
- Boot drive: 120GB SATA-SSD (OCZ-VERTEX2)
- Storage:
  - 4x12TB 7200rpm HGST/WD DC HC520 in 2 x MIRROR | 2 wide ZFS configuration
  - 1 256GB NVME (Gen3) SLOG drive

2. truenas-backup

- Operating System: TrueNAS Scale Fangtooth-24.04.0
- Hostname: truenas-backup
- IP: 192.168.1.201 (at remote location, connected using Tailscale)
- Model: homebuild (ASRock Z87E-ITX motherboard)
- CPU: i5-4570S 2.9Ghz (4:th gen, 4 cores, 4 threads, Q1 2014)
- RAM: 16GB (1333 Mhz)
- Boot drive: 240GB SATA-SSD (SanDisk SSD Plus)
- Storage: 4x4GB 5400rpm WD40EFRX (1 x RAIDZ1 | 3 wide and 1 spare VDEV ZFS configuration)

## Kubernetes

### Cozy Stack ✅

Nice hyper-converged stack on top of Talos!

- Flux CD - Operator (ControlPlane)
- Cilium, Kube-OVN - Network CNI
- Linstor, DRDB - Operator (Piraeus), Storage
- Cert-Manager - Operator, Let´s Encrypt/Cloudflare
- MetalB - Load Balancer
- Kubevirt - Operator, Virtual machines in kubernetes
- PostgreSQL - Operator (CNPG), Database
- Maria DB - Operator, Database
- Clickhouse - Operator (Altinity), Database
- Redis - Operator (Spotahome), Key-Value database
- Kafka - Operator (Strimzi), Distributed event streaming platform
- RabbitMQ - Operator, Messaging and streaming broker
- Victoria Metrics
- Grafana - Operator (Integreatly) Data visualization
- Goldpinger - Debugging tool for Kubernetes
- External Secrets - Operator
- Nginx proxy
- Kamaji - Operator (ControlPlane)
- Vertical Pod Autoscaler - Operator
- Reloader
- and more...

## GitOps - Flux CD ✅s

Will use SOPS/age for encrypting secrets.
Public Github site that describes the cluster.

### Setup

- Public GitHub repo for cluster configuration, deployed with Flux.
  - Renovate - Cluster dependency management (on GitHub)
  - Reloader - Rolling upgrades on ConfigMap/Secret changes (in cluster)

### Initial cluster settings

Settings to complete the cluster setup.

- Cozy patches, to enable etcd, ingress, monitoring, dashboard etc. (https://cozystack.io/docs/get-started/#install-cozystack) ✅
- Linstor storage classes (local, replicated) ✅
- MetalLB IP Pool (192.168.222.101-192.168.222.130) ✅
- MetalLB L2Advertisement for the pool above ✅
- Cert-Manager cluster-issuer for Cloudflare ✅
- Cert-Manager wildcard certificate for myk8s.se ✅

### Apps

#### Storage/Networking

- Democratic CSI - For being able to schedule storage on truenas-stor (NFS or iSCSI) ❓
- Minio (On truenas-stor) For backups.

#### Web Publishing

- External-DNS - Operator, Manage DNS names against CloudFlare ✅
- Cloudflared ✅
- Cloudflare DDNS

#### Management

- Goldilocks ?
- Polaris ?
- Kyverno ?

#### Virtual Machines

- Kubevirt Manager (https://github.com/kubevirt-manager) -Will I need this?

#### PXE Boot

- Netboot.xyz? ☑️
- Matchbox?

#### GPU-support

- Intel
  https://github.com/onedr0p/home-ops/tree/main/kubernetes/apps/kube-system/intel-device-plugin-operator

#### Backup

- Velero + Velero UI - Backup to S3 buckets, this requires Minio on TrueNAS Stor.

#### Maybe in the future

- InfluxDB (Currently on NAS)

#### Startpage

- Flame (https://charts.gabe565.com/charts/flame/)

#### Development Server

- Code-Server
  - Helm
  - Talosctl
  - Talm
  - KubeCtl
  - VirtCtl
  - etcdctl
  - Flux CLI
  - GitOps VSCode extension
  - Git
  - Yaml Extension
  - Kubernetes Extension
  - SQL Tools with Postgres and Maria DB drivers
  - HomeAssistant extension
  - More???

#### Virtual Machines

- Home Assistant OS (kubevirt) - currently on truenas-stor.

#### DNS/Ad Blocking

- Blocky - on Postgres and Redis (Configured with DOP, maybe behind NordVPN)
- Grafana "Blocky Dashboard for Postgres"

#### Media

These apps are prepared, works and runs on current RKE2 cluster.

- Jellyfin
- Sonarr (currently on truenas-stor) ☑️
- Radarr (currently on truenas-stor) ☑️
- Readarr (currently on truenas-stor) ☑️
- Prowlarr (currently on truenas-stor) ☑️
- QBitTorrent (currently on truenas-stor) with GlueTun for NordVPN ☑️

#### Home Automation

- Generic Device Plugin (for connecting node USB device to pod)
- Zigbee2MQTT (USB passtrough) (currently on HAOS VM)
- ESP Home
- Frigate [Helm Chart](https://github.com/blakeblackshear/blakeshome-charts/tree/master/charts/frigate)
- NodeRed (currently on HAOS VM)

#### 3D Printing

- Mainsail
- Obico

#### Maybe in the future

- Gaseous Server (Old games (ROMS) in the browser)
- Authentik
- Invidious
- Brave
- Coraza Web Application firewall (Ingress Middleware?)
- Jellyfin (with GPU Passthrough - https://github.com/Pumba98/flux2-gitops/blob/main/apps/jellyfin/hr-jellyfin.yaml)
- Vaultwarden
- Guacamole
- CompreFace/DeepStack
- Plex Debrid
- Streamio
- Immich
- Sabnzbd
- TDArr
- Autobrr
- Unpackerr
- exportarr - exports metrics for Sonarr, Radarr, Lidarr, Prowlarr, Readarr, Bazarr and Sabnzbd
- Atuin (shell history server)
- ...more to come
