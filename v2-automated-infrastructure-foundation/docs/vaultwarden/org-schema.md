# Vaultwarden Organization Schema: Platform Engineering

This document outlines the enterprise-grade secret management structure for our internal platform infrastructure. Secrets are isolated within the **Platform Engineering** organization and categorized chronologically from the bare-metal foundation up to the automation and observability layers using **Collections**.

## Organization Workspace Layout

### [Org] Platform Engineering
Shared workspace hosting all enterprise infrastructure secrets, isolated from personal and household operations credentials.

---

## Collection Structure & Asset Directory

### 01-compute-fabric (bare-metal, hypervisors, oobm)
*The physical and virtual computing power of the platform.*
* **Sub-items & Asset Types:**
  * **Out-of-Band Management (OOBM):** IPMI, Dell iDRAC, HPE iLO, Pi-KVM web console admin credentials.
  * **Bare-Metal Hosts:** Base OS root logins, local admin user passwords, and SSH keys for physical nodes.
  * **Hypervisors:** Proxmox VE, VMware ESXi, or Hyper-V standalone management interfaces.
  * **Cluster Bootstrap:** Provisioning tokens, hardware MAC address lists, and IPMI encryption keys.

### 02-ingress-network (firewalls, routing, reverse-proxies)
*The network fabric and edge perimeter handling traffic routing and application delivery.*
* **Sub-items & Asset Types:**
  * **Edge Security:** OPNsense, pfSense, or enterprise gateway firewall administrative logins.
  * **Core Switching:** Managed switch UI/SSH credentials, VLAN configurations, and console passwords.
  * **Reverse Proxies & Ingress Controllers:** Traefik, Nginx Proxy Manager, HAProxy, or Caddy admin keys.
  * **External DNS & Edge Networking:** Cloudflare API tokens, Let's Encrypt/ACME DNS challenge keys, and domain registrar logins.
  * **VPN / Mesh Gateways:** WireGuard private keys, Tailscale Auth Keys, or OpenVPN server credentials.

### 03-storage-data (nas-san, backups, databases)
*Persistent volume storage, stateful data backends, and disaster recovery assets.*
* **Sub-items & Asset Types:**
  * **Network Attached Storage (NAS):** TrueNAS SCALE/Core, Unraid, or Synology DSM admin portals.
  * **Storage Protocols:** iSCSI targets, NFS export privileges, and SMB/CIFS service user accounts.
  * **Cloud Storage & Offsites:** Backblaze B2 API keys, AWS S3 bucket credentials, or Wasabi access keys.
  * **Central Databases:** Root credentials for platform-wide database engines (PostgreSQL, MySQL/MariaDB, Redis, MongoDB).
  * **Backup Automation:** Restic, Kopia, or Velero repository passwords and encryption passphrases.

### 04-control-plane (kubernetes, container-runtimes, portainer)
*The container orchestration runtimes and application scheduling environments.*
* **Sub-items & Asset Types:**
  * **Kubernetes Orchestration:** `kubeconfig` text contents (stored as Secure Notes), service accounts, and K3s/K8s cluster join tokens.
  * **Container Managers:** Portainer CE/EE, Rancher Management Server, or Nomad admin interfaces.
  * **Container Registries:** Private Docker Registry logins, GitHub Packages tokens, or Harbor credentials.
  * **Node Orchestration:** Cloud-init default user passwords and automated cluster-scaling service keys.

### 05-deployment-cicd (gitops, automation, api-tokens)
*Infrastructure-as-Code (IaC), deployment automation engines, and pipeline integrations.*
* **Sub-items & Asset Types:**
  * **Source Control:** GitHub/GitLab Personal Access Tokens (PATs) and SSH deploy keys.
  * **GitOps Controllers:** ArgoCD or FluxCD admin dashboards and repository access keys.
  * **Configuration Management:** Ansible Vault passphrases, Terraform Cloud API tokens, or OpenTofu state backend keys.
  * **Automation Runners:** GitHub Actions self-hosted runner registration tokens or GitLab CI runner tokens.

### 06-observability (metrics, logging, alerting)
*Centralized telemetry, log aggregation, and automated alerting systems.*
* **Sub-items & Asset Types:**
  * **Visualization & Dashboards:** Grafana admin/editor user accounts.
  * **Time-Series Databases:** Prometheus, InfluxDB, or VictoriaMetrics API authorization tokens.
  * **Log Aggregation:** Loki, FluentBit, or ElasticSearch cluster access credentials.
  * **Synthetics & Status Pages:** Uptime Kuma dashboards or Cachet admin credentials.
  * **Alert Routing:** Discord, Slack, or Telegram webhook URLs; SMTP credentials for email alerts.

---

## Operational Best Practices
1. **Never Mix Layers:** If a database is running as a container inside Kubernetes (04), its *root administrative password* should still go to 03-storage-data. Keep boundaries strict.
2. **URI Matching Strategy:** Set the URI match detection to **Host** or **Exact** in Vaultwarden settings. This prevents browser extensions from autofilling the wrong credentials when multiple containers share the same local IP or domain name via different ports.
3. **Use Custom Fields:** For deep infrastructure interfaces (like Proxmox or Kubernetes dashboards), leverage Vaultwarden's "Custom Fields" to store specific parameters like `Realm`, `VLAN ID`, or `Service Account Name` directly within the login item.
