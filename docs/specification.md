# Cluster Specification

## Overview

This is a specification of everything that should be included in the cluster implementation.

## Good links

- https://geek-cookbook.funkypenguin.co.nz/
- https://bjw-s.github.io/helm-charts/docs/
- https://github.com/buroa/k8s-gitops
- https://github.com/gabe565/home-ops
- https://github.com/onedr0p/home-ops
- https://github.com/onedr0p/cluster-template

## Network

- Network: 192.168.222.0/24 "Server"
- DHCP range: 192.168.222.21-99
- Control plane shared IP: 192.168.222.20
- MetalLB load balancer IP-range: 192.168.222.100-192.168.222.130
- Network manufacturer: Ubiquiti Unifi
- Public domain name: myk8s.se
- Most services will be published over authenticated (Entra ID) Cloudflare tunnel.
  - Might change this later to unauthenticated tunnel and use Authentik instead.

Nodes will get their IP from DHCP, flagged as static in router.

## Hardware

### Kubernetes Nodes (Talos/CozyStack)

Three schedulable control plane nodes.
Operating System: Talos 1.10.6 opionated with CozyStack 0.36.0 (2025-09-17)
All nodes have Nut-Client installed and configured against TrueNAS-Stor.

1. k8s01

- Hostname: k8s01
- IP: 102.168.222.21
- Model: Dell Optiplex 5090M
- CPU: Intel i5-10500T 2,3GHz (10:th gen, 6 cores/12 threads Q2 2020)
- GPU: Intel UHD 630 iGPU
- RAM: 64GB (2x 32GB Kingston Fury DDR4 3200Mhz)
- Boot drive: 250GB SATA-SSD (PNY CS900)
- Storage drive: 1TB NVME Storage Drive (Kingston Fury Renegade PCIe 4.0)

2. k8s02

- Hostname: k8s02
- IP: 102.168.222.22
- Model:
- CPU: Intel i5-10500T 2,3GHz (10:th gen, 6 cores/12 threads Q2 2020)
- GPU: Intel UHD 630 iGPU
- RAM: 64GB (2x 32GB Kingston Fury DDR4 3200MHz)
- Boot drive: 250GB SATA-SSD (PNY CS900)
- Storage drive: 1TB NVME (Kingston Fury Renegade PCIe 4.0)

3. k8s03

- Hostname: k8s03
- IP: 192.168.222.23
- Model: Intel NUC NUC10i7FNH
- CPU: Intel i7-10710U 1,1GHz CPU (10:th gen, 6 cores/12 threads Q3 2019)
- GPU: Intel® UHD Graphics for 10th Gen Intel® Processors (UHD630 successor)
- RAM: 64GB (2x 32GB Kingston Fury DDR4 3200MHz)
- Boot drive: 250GB SATA-SSD (PNY CS900)
- Storage drive: 1TB NVME (Kingston Fury Renegade PCIe 4.0)

### External storage, TrueNAS Scale:

1. nas (nas.myk8s.se)

- Operating System: TrueNAS Scale Fangtooth-25.04.2.4
- Hostname: nas
- IP Address: 192.168.222.13
- Model: homebuild (AsRock Z370M-ITX/ac motherboard)
- CPU: i7-8700 3,2Ghz (8:th gen, 6cores, 12 threads, Q4 2017)
- RAM: 32GB (2133Mhz)
- Boot drive: 120GB SATA-SSD (OCZ-VERTEX2)
- Storage:
  - 4x12TB 7200rpm HGST/WD DC HC520 in 2 x MIRROR | 2 wide ZFS configuration
  - 1 256GB NVME (Gen3) SLOG drive - not working!

Also runs as NUT and Minio (S3) server.

2. backup (backup.myk8s.se) - Remote location backup

- Operating System: TrueNAS Scale Fangtooth-25.04.2.1
- Hostname: backup
- IP: 192.168.1.126 (DHCP) (at remote location, connected using Tailscale)
- Model: homebuild (ASRock Z87E-ITX motherboard)
- CPU: i5-4570S 2.9Ghz (4:th gen, 4 cores, 4 threads, Q1 2014)
- RAM: 16GB (1333 Mhz)
- Boot drive: 240GB SATA-SSD (SanDisk SSD Plus)
- Storage: 4x4GB 5400rpm WD40EFRX (1 x RAIDZ1 | 3 wide and 1 spare VDEV ZFS configuration)

## Kubernetes

### Talos System Extensions required

- drbd
- intel-ucode
- i915
- intel-ice-firmware
- nut-client (needs config)
- zfs
- lldpd (likely to be added to Cozystack soon, needs config)

### CozyStack ✅

Nice hyper-converged stack on top of Talos!

- Flux CD Operator (ControlPlane) ✅
- Cilium, Kube-OVN - Network CNI ✅
- Linstor DRDB Operator (Piraeus) - Storage ✅
- Cert-Manager Operator - Let´s Encrypt/Cloudflare ✅
- MetalB - Load Balancer ✅
- Nginx/HA Proxy - Cache och Http Loadbalancer ✅
- HA Proxy - TCP Loadbalancer
- PostgreSQL Operator (CNPG) - Database ✅
- Maria DB Operator - Database ✅
- Clickhouse Operator (Altinity) - Database ✅
- Redis Operator (Spotahome) - Key-Value database ✅
- Kafka Operator (Strimzi) - Distributed event streaming platform ✅
- RabbitMQ Operator - Messaging and streaming broker ✅
- Nats - Messaging
- Victoria Metrics ✅
- Grafana Operator (Integreatly) - Data visualization ✅
- Goldpinger - Debugging tool for Kubernetes, cluster internal network ✅
- External Secrets Operator - Not being updated in CozyStack, installed separately.
- External DNS Operator - Not being updated in CozyStack, installed separately.
- Nginx proxy ✅
- Kamaji Operator (ControlPlane) ✅
- Vertical Pod Autoscaler Operator ✅
- Velero Operator - Backups to Minio on Stor (Needs to be enabled in cozy config "bundle-enable") ✅
- and more...

### Initial cluster settings - Components/Bootstrap ✅

Settings to complete the cluster setup.

- Cozy patches, to enable etcd, ingress, monitoring, dashboard etc. (https://cozystack.io/docs/get-started/#install-cozystack) ✅
- Linstor storage classes (local, replicated) ✅
- MetalLB IP Pool (192.168.222.101-192.168.222.130) ✅
- MetalLB L2Advertisement for the pool above ✅
- Cert-Manager cluster-issuer for Cloudflare ✅
- Cert-Manager wildcard certificate for myk8s.se ✅
- External Secrets Operator, connected to against Infisical SAAS. ✅
- Intel Device Plugin Operator for iGPUs, will be removed when K8s 1.34 with stable DRA is deployed ✅
- Cozy NFS Driver (CSI NFS Driver for Kubernetes) - For being able to connect to storage on truenas-stor. ✅
- Media NFS PVs (Read Write/Read Only) ✅
- Reflector - Tool for syncing certs and config maps between namespaces, mainly use for syncing wildcard TLS cert. ✅

#### Apps

#### GitOps - Flux CD ✅

Public Github site that describes the cluster. ✅

### Update and configuration management

- Renovate - Cluster dependency management (on GitHub) ✅
- Reloader - Rolling upgrades on ConfigMap/Secret changes (in cluster) ✅
- Reflector - Copies secrets between namespaces (mainly wildcard Lets Encrypt Cert). ✅

#### Storage/Networking

- Minio (on nas) For backups. ✅
- S3 Manager (Part of Cozystack, needed when Minio has a Gui?)

#### Web Publishing

- External-DNS - Operator, Manage DNS names against CloudFlare ✅
- Cloudflared - Cloudflare Tunnel ✅
- Authentik - Authentication of basically all apps (may be some exceptions.) ☑️
- Cloudflare DDNS - Updates Cloudflare DNS with public IP Address ✅

#### Media

- Jellyfin ✅
- Plex ✅
- Bazarr ✅
- Sonarr ✅
- Radarr ✅
- Prowlarr ✅
- Recyclarr - Trash Guides best practices sync with Radarr/Sonarr ✅
- FlareSolverr - Proxy server to bypass Cloudflare and DDoS-GUARD protection ✅
- QBitTorrent with GlueTun for NordVPN ✅
- Autobrr - Allows for better seeding ✅
- Sabnzbd with GlueTun for NordVPN (new)

#### Virtual Machines

- Kubevirt Manager (https://github.com/kubevirt-manager) - Will I need this?

#### PXE Boot

- Netboot.xyz or Matchbox or Tinkerbell or...?

#### Backup

- Velero configuration (https://cozystack.io/docs/kubernetes/backup-and-recovery/)
- Velero UI - UI for Velero that is included in CozyStack

#### Maybe in the future

- InfluxDB (Currently included in HAOS VM, Will Clickhouse do?)

#### Startpage

- Homepage (http://gethomepage.dev/)

#### Development Server

- Code-Server
  - Atuin client.
  - Helm
  - Talosctl
  - Talm
  - KubeCtl
  - VirtCtl
  - etcdctl
  - Flux CLI
  - Cilium CLI
  - VSCode GitOps VSCode extension
  - Git
  - VSCode Yaml Extension
  - VSCode Kubernetes Extension
  - VSCode SQL Tools with Postgres and Maria DB drivers
  - VSCode HomeAssistant extension
  - More???

#### DNS/Ad Blocking ✅

- Blocky + Redis (Sync) + Postgres (logging), DNS request over DoH over NordVPN (GlueTun). ✅
- Blocky Grafana Postgres Datasource ✅
- Blocky Grafana Dashboards ✅

#### AI Workflow/Automation engine
- n8n - Automation against different services with workflow and AI Agents
  [Helm Chart](https://github.com/8gears/n8n-helm-chart)
- nocodb - Postgres backed simple database for use with n8n
- qdrant operator - vector database
- graphiti - GraphRAG, requires FalkorDB (redis variant)
- knative - Environment For spinning up serverless MCP server pods on demand.

##### MCP Servers
- nocodb
- qdrant
- graphiti
- Apple Find My - Create own service based on FindMy.py (https://github.com/malmeloo/FindMy.py)

#### Home Automation

- Home Assistant OS (kubevirt) - Virtual Machine - currently on truenas-stor.
- Generic Device Plugin (for connecting node USB device to pod)
  https://github.com/gabe565/home-ops/tree/main/kubernetes/gabernetes/apps/generic-device-plugin
- Zigbee2MQTT (USB passtrough) (currently on HAOS VM)
- ESP Home
- Frigate
  [Helm Chart](https://github.com/blakeblackshear/blakeshome-charts/tree/master/charts/frigate)
- NodeRed (currently on HAOS VM)

#### 3D Printing

- Mainsail
- Obico
  https://github.com/gabe565/home-ops/tree/main/kubernetes/gabernetes/apps/obico

#### Management/Security

- Goldilocks ?
- Polaris ?
- Kyverno ?

#### Maybe in the future

- Immich - Photo management (requires postgres and redis)
  https://github.com/gabe565/home-ops/tree/main/kubernetes/gabernetes/apps/immich
- Immich Drop - Allows dropping photos easily into Immich.
- Streamio Server (https://github.com/tsaridas/stremio-docker) + MediaFusion (Debrid support) + GlueTun
- Gatus - Uptime monitoring
- Paperless-ngx - document management
- Headscale - Local implementation of Tailscale
  https://github.com/gabe565/home-ops/tree/main/kubernetes/gabernetes/apps/paperless-ngx
- Gaseous Server - Old games (ROMS) in the browser
- Invidious - YouTube UI - no adds - Will I need this, Brave is good enough???
- Brave + VPN (https://github.com/linuxserver/docker-brave) - Browser with ad blocking and privacy in mind
- Vaultwarden
- Guacamole
- TDArr (transcoding , could save a lot of diskspace for media library)
  https://github.com/onedr0p/home-ops/tree/main/kubernetes/apps/default/autobrr
- Unpackerr
- exportarr - exports metrics for Sonarr, Radarr, Lidarr, Prowlarr, Readarr, Bazarr and Sabnzbd
- mathesar - Fantastic Postgres UI
  https://github.com/mathesar-foundation/mathesar
- Atuin - Shell history server. ✅
- Peanut - Network UPS Tools GUI and Prometheus metrics. ✅
- Cilium Hubble - Cilium Network Policy observation. ✅
- ...more to come
