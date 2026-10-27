# Dual-cluster architecture — proposed target design

**Status: proposed; not yet deployed or validated against an APIC.**

## Topology

One Cisco ACI fabric hosts two **independent** Kubernetes clusters:

- **Management cluster:** deployed in an application tenant named `INFRA` (not the APIC built-in `infra` tenant). Runs Argo CD, Grafana, Prometheus, Alertmanager, APIC read-only exporter and operational tooling.
- **Service cluster:** runs application workloads, Cilium and local telemetry agents. Its tenant placement is **to be confirmed**; a dedicated `SERVICES` tenant is recommended for policy isolation, but not a prerequisite.

Both clusters have separate Kubernetes API endpoints, identities, CNI/IPAM configuration, upgrade schedules, backup policies and failure domains. They share the physical ACI fabric, so fabric-wide outages remain a common dependency.

## Logical connectivity

```text
GitHub (source of truth)
  |                        Cisco ACI fabric
  |                    +---------------------------+
  +--> Mgmt Argo CD -->| INFRA application tenant  |
                       | Management K3s cluster   |
                       | Argo CD, Grafana,         |
                       | Prometheus, Alertmanager |
                       | APIC read-only exporter  |
                       +-------------+-------------+
                                     |
                        Explicit routed connectivity
                        + ACI contracts (least privilege)
                                     |
                       +-------------+-------------+
                       | Service tenant (TBD)      |
                       | Service K3s cluster       |
                       | Cilium, apps, telemetry   |
                       +---------------------------+
```

## Integration boundaries

1. **ACI provides underlay connectivity** to cluster node networks and explicitly approved routed connectivity between management and service networks.
2. **Cilium manages Kubernetes pod/service networking and policy** inside each cluster. Choose routing mode, encapsulation, IPAM and load-balancer advertisement only after validating compatibility with ACI routing and the chosen K3s configuration. Do not assume native ACI CNI and Cilium can coexist.
3. **Argo CD in management** may manage the service cluster through a scoped Kubernetes API credential. Management bootstrap remains reproducible **without** relying on its own Argo CD instance.
4. **Prometheus in management** receives service-cluster telemetry using a documented secure transport and authorization pattern (e.g. remote write or a controlled scrape endpoint); choose the pattern before deployment. No blanket cross-tenant connectivity.
5. **APIC integration is read-only**, through a dedicated least-privilege API identity. Metrics may cover faults, fabric health, node/interface state and endpoint summaries. API compatibility, query cost and cardinality must be tested.

## Proposed policy matrix — review before implementation

| Source | Destination | Purpose | Restriction |
| --- | --- | --- | --- |
| Management Argo CD | Service Kubernetes API | GitOps deployment | TLS, scoped RBAC, explicitly approved API endpoint |
| Service telemetry | Management metrics ingress | Metrics/alerts | TLS/authentication, limited ports and source subnets |
| Management APIC exporter | APIC API | Fabric telemetry | HTTPS, read-only account, API rate controls |
| Operators | Management entry points | Dashboards and administration | VPN/bastion, SSO/MFA where available |
| Service workloads | Management control plane | Not generally required | Deny by default |

The precise ports, source addresses, VRF routing and contracts are deliberately deferred until topology/IPAM is agreed.

## Bootstrap order

1. Record fabric topology, tenant/VRF/BD/EPG design, node IPs, APIC capabilities, DNS/NTP and routing.
2. Configure ACI fabric/access and the required application tenant policies with reviewed Terraform plans.
3. Bootstrap the management cluster independently using pinned artifacts and secured external backups.
4. Install and validate Cilium, then bootstrap management Argo CD from Git.
5. Bootstrap the service cluster independently; validate Cilium networking and workloads.
6. Register service cluster in management Argo CD with least-privilege credentials.
7. Deploy monitoring, dashboards, alerts and APIC exporter declaratively.
8. Test inter-cluster reachability, policy enforcement, telemetry, and both failure scenarios.

## Recovery scenarios

### Management cluster lost
- Service cluster should continue serving existing workloads and policies.
- Restore management hosts/datastore from tested backups, or bootstrap a clean cluster from pinned versions and Git.
- Recover credentials from an external secret store; reinstall Argo CD and monitoring; reconnect service cluster.
- Historical metrics require separate data backups if retention is needed.

### Service cluster lost
- Management monitoring/Argo CD remain available.
- Rebuild service K3s/Cilium, restore data and secrets separately, register the new cluster identity and reconcile applications from Git.
- Verify ACI routing, contracts and telemetry before declaring recovery complete.

### ACI fabric outage
- Both clusters may lose underlay connectivity. Kubernetes separation does **not** protect against a shared fabric failure.
- Use APIC/fabric-specific out-of-band recovery procedures and verified fabric state backups.

## Decisions required before runnable manifests

- Physical/virtual node placement and HA expectations for both clusters.
- Exact tenant/VRF separation and inter-VRF routing approach.
- ACI/APIC version and lab versus production restrictions.
- K3s/Cilium networking (native routing versus tunnel), PodCIDR/ServiceCIDR and IPAM.
- Service exposure/load balancer strategy and DNS/TLS.
- Argo CD cluster credential lifecycle and secret manager.
- Prometheus cross-cluster collection pattern, storage and retention.
- Backup destinations, restore objectives and external bootstrap dependencies.

See [ARCHITECTURE.md](ARCHITECTURE.md), [BOOTSTRAP.md](BOOTSTRAP.md) and [RECOVERY.md](RECOVERY.md).
