# Enterprise Secret Naming Schema: Platform Infrastructure

This document defines the standardized naming convention for all assets, secrets, and credentials within the Platform Engineering organization. Implementing a strict naming schema ensures predictability, simplifies global search, and enforces an enterprise mindset across the infrastructure.

## Core Naming Formula

All vault items must follow this strict, delimited structure:

`[Environment] [System/Asset Type] [Instance/Identifier] - [Secret Component/Role]`

### Component Breakdown
1. **[Environment]**: The operational tier (e.g., `[PROD]`, `[DEV]`, `[STAGE]`, `[LAB]`). Must be enclosed in square brackets.
2. **[System/Asset Type]**: The technology or service category (e.g., `pve` for Proxmox, `k8s` for Kubernetes, `fw` for Firewall, `nas` for Storage).
3. **[Instance/Identifier]**: The specific node number, cluster name, or location (e.g., `node01`, `core01`, `edge`).
4. **- (Delimiter)**: A space, hyphen, space combination separating the asset from the specific secret metadata.
5. **[Secret Component/Role]**: The specific access mechanism or account type (e.g., `Root SSH Key`, `Web UI Admin`, `Replication User`, `API Token`).

---

## Standardized Envs, Systems, and Secret Components

To maintain consistency, only use the following approved nomenclature when building names:

### Approved Environment Prefixes
* `[PROD]` - Production systems handling household operations, active storage, or live services.
* `[DEV]` - Development environments used for testing new configurations, deployments, or scripts.
* `[LAB]` - Temporary sandbox environments or single-use experimental instances.

### Approved System/Asset Types
* `node` - Physical bare-metal server operating systems.
* `oobm` - Out-of-band management interfaces (iDRAC, IPMI, iLO).
* `pve` - Proxmox VE hypervisor nodes or clusters.
* `fw` - Edge or internal firewalls.
* `sw` - Managed network switches.
* `proxy` - Reverse proxies and ingress controllers.
* `nas` - Network attached storage controllers.
* `db` - Database engine instances.
* `k8s` / `k3s` - Container orchestration clusters.
* `git` - Source control and repository management platforms.
* `mon` - Observability, metrics, and alerting systems.

### Approved Secret Components / Roles
* `Web UI Admin` - Main administrative account accessed via a web browser.
* `Root SSH Key` - Administrative private key access over SSH.
* `Local Admin` - Non-root administrative user accounts.
* `API Token` - Programmatic authentication strings.
* `Kubeconfig` - Cluster access configuration manifests.
* `Service Account` - Restricted non-human user accounts for automation.
* `Webhook URL` - Automated notification delivery endpoints.

---

## Production Examples Across Collections

Below is how the naming schema applies to actual items inside each of the 6 platform engineering collections.

### 01-compute-fabric
* `[PROD] oobm-node01 - Web UI Admin`
* `[PROD] node01 - Root SSH Key`
* `[PROD] pve-cluster01 - Web UI Admin`
* `[DEV] node02 - Local Admin`

### 02-ingress-network
* `[PROD] fw-edge - Web UI Admin`
* `[PROD] sw-core01 - SSH Admin`
* `[PROD] proxy-traefik - API Token`
* `[PROD] cloudflare - API Token DNS Challenge`

### 03-storage-data
* `[PROD] nas-truenas01 - Web UI Admin`
* `[PROD] nas-truenas01 - SMB Service Account`
* `[PROD] db-postgres01 - Root Password`
* `[PROD] backblaze-b2 - S3 Access Keys`

### 04-control-plane
* `[PROD] k3s-cluster01 - Kubeconfig`
* `[PROD] k3s-cluster01 - Join Token`
* `[PROD] portainer-mgmt - Web UI Admin`
* `[DEV] k3s-sandbox01 - Kubeconfig`

### 05-deployment-cicd
* `[PROD] github - PAT Platform Automation`
* `[PROD] argocd-mgmt - Web UI Admin`
* `[PROD] ansible - Vault Passphrase`

### 06-observability
* `[PROD] mon-grafana - Web UI Admin`
* `[PROD] mon-uptime-kuma - Discord Webhook URL`
* `[PROD] mon-influxdb - Read Write Token`

---

## Vaultwarden Field Mapping Rules
* **Username Field**: Always input the exact login ID (e.g., `root`, `admin`, `kube-service-account`). Do not leave it blank if the item is a login.
* **Password Field**: Use the built-in generator to enforce a minimum of 24 characters for internal platform items.
* **URI 1**: Input the exact IP address or local FQDN with the port (e.g., `https://192.168.10.50:8006`).
* **URI Options**: Change the match detection from "Default" to **Host** or **Exact** to ensure the browser extension only prompts for the exact service requested.
* **Custom Fields**: For hypervisors or enterprise dashboards requiring specific domains or realms, add a Text Custom Field labeled `Realm` or `Domain` directly under the password block.
