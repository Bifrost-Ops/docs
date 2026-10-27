# Bifrost Ops — deployment packages

**Status: reference design, not a certified hardware bill of materials.** These are planning baselines, **not vendor minimums**. Size for actual workload, storage IOPS, retention, availability and failure-domain targets.

## Package A — ONPREM-LAB
Purpose: inexpensive reproducible lab, no HA guarantee.
- Management: 1 VM, 4 vCPU, 12 GiB RAM, 120 GiB SSD.
- Service: 1 VM, 4 vCPU, 8 GiB RAM, 100 GiB SSD.
- Monitoring retention: short (7 days), separate persistent volume recommended.
- Compute host: minimum 8 physical cores / 32 GiB RAM / SSD; allow hypervisor and network overhead.
- Interfaces per host: 1 production data NIC minimum; recommend 2 NICs for redundancy. Dedicated BMC/iLO/iDRAC if available.
- Network: management VLAN/EPG, service VLAN/EPG, storage/backup path, and independent OOB path if hardware supports it.
- Failure model: losing one host can lose both clusters. Backups must be external.

## Package B — ONPREM-HA
Purpose: resilient shared platform across independent compute failure domains.
- Management: 3 K3s server VMs (each 4 vCPU, 8 GiB RAM, 100 GiB SSD) plus 2 monitoring workers (each 8 vCPU, 24 GiB RAM, 250 GiB SSD + separately sized persistent monitoring storage).
- Service: 3 K3s server VMs (each 4 vCPU, 8 GiB RAM, 100 GiB SSD) plus 3 workers (each 8 vCPU, 16 GiB RAM, 150 GiB SSD + application persistent storage).
- Total requested virtual allocation (excluding hypervisor, storage replicas and growth): **64 vCPU / 144 GiB RAM**. These are summed VM allocations, not physical CPU capacity requirements.
- Minimum 3 independent compute hosts / fault domains for each embedded-etcd quorum; may share physical platform if host placement and anti-affinity are enforced.
- Physical interfaces **recommended**: 2x 10/25GbE data uplinks to distinct ACI leaves (verify vPC/port-channel design), 1x dedicated BMC management port, separate OOB switch/network, and storage/backup bandwidth sized from measured throughput. Two NICs alone do not guarantee path redundancy.
- Primary/secondary: dual power feeds/PDUs and independent leaf/uplink paths where available. API VIP/LB must not be a single point of failure.
- Backup: off-cluster encrypted K3s etcd snapshots and tokens, Git revisions, secrets, Terraform state versions, application/PV backups and monitoring data if required.
- RPO/RTO: **not promised** until restore tests prove targets.

## Package C — CLOUD-HYBRID
Purpose: cloud-hosted management and/or service compute with an ACI on-prem underlay.
- Each cluster: choose 3 K3s server VMs across distinct availability zones **only when network latency, supported datastore topology and inter-AZ costs are validated**; 4 vCPU / 8 GiB per server is a planning start.
- Workers: management 2x (8 vCPU / 24 GiB), service 3x (8 vCPU / 16 GiB), adjustable.
- Cloud network: private subnets, explicit route tables, security groups/firewalls, egress controls, private API endpoint or restricted admin ingress, DNS and NTP.
- Connectivity to ACI: redundant site-to-site VPN or private interconnect with BGP/routing design, encryption and tested failover; avoid overlapping CIDRs.
- No physical iLO in public cloud: use cloud serial console, provider rescue, IAM break-glass and audit logs.
- Primary/secondary: two independent tunnels/links and failure-domain-aware compute; backup in a separate account/region where policy permits.
- Caveat: cloud-hosted nodes are **not literally inside an on-prem ACI tenant**. ACI remains the on-prem network policy boundary; cloud routing/security is an additional boundary.

## Mandatory infrastructure requirements — all packages

| Area | Requirement |
| --- | --- |
| Compute | CPU/RAM per role, hypervisor/OS support, anti-affinity and headroom |
| Storage | SSD for etcd, IOPS/latency validation, encrypted persistent storage and backup |
| Data NIC | Node underlay VLAN/EPG, MTU, redundant uplinks where required |
| Primary/secondary | Distinct uplinks, ACI leaf paths, switches, power and API endpoint resilience |
| OOB/BMC | Dedicated BMC/iLO/iDRAC network for physical nodes, break-glass credentials and access audit |
| Management | Separate management address space and documented access path |
| Backup | Off-cluster snapshots, secrets and tokens, immutable/offsite copy if required |
| DNS/NTP | Redundant reachable services, independent bootstrap fallback |
| Security | RBAC, certificate lifecycle, secret store, tenant/VRF/contract boundaries |
| Operations | Runbooks, patch windows, monitoring, alerting, restore drills |

## How to select
- LAB is intentionally disposable; no single-component HA claims.
- HA is a **design starting point**, not a guaranteed SLA.
- HYBRID introduces latency, routing and cloud operational dependencies. Validate failure scenarios before use.
- Do not deploy until the [site questionnaire](SITE-QUESTIONNAIRE.md) and [interface matrix](INTERFACE-MATRIX.md) are completed.

## Vendor sources
- K3s hardware/network requirements: https://docs.k3s.io/installation/requirements
- K3s HA embedded etcd: https://docs.k3s.io/datastore/ha-embedded
- Cilium K3s integration: https://docs.cilium.io/en/stable/installation/k3s/
- Argo CD HA: https://argo-cd.readthedocs.io/en/stable/operator-manual/high_availability/
- Cisco NAC ACI model: https://netascode.cisco.com/docs/start/aci/introduction/
