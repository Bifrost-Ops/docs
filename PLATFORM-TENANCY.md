# Platform tenancy and multi-cluster design

**Status:** agreed architectural direction; implementation details remain subject to APIC, K3s and Cilium validation.

## Roles

### INFRA — shared platform services
`INFRA` is a **user-created ACI application tenant**, distinct from the built-in APIC `infra` tenant.

It provides shared platform services and centralized management:
- DNS resolution and time synchronization (NTP), with independently documented bootstrap/fallback dependencies.
- Management Kubernetes cluster with Argo CD, Grafana, Prometheus/Alertmanager and platform tooling.
- Read-only APIC metrics integration, GitOps coordination, monitoring and administrative access.
- Centralized *management* of service clusters, not a shared Kubernetes API control plane for them.

The INFRA tenant must be recoverable without relying on the service cluster, and essential DNS/NTP must have a bootstrap plan that avoids circular dependencies.

### SERVICE — shared application hosting
A service tenant may host a Kubernetes cluster with multiple isolated application tenants/workloads. Isolation is layered:
- ACI tenant/VRF/BD/EPG and explicit contracts at the fabric boundary.
- Kubernetes namespaces, RBAC, NetworkPolicies/Cilium policies, quotas and workload identity inside the cluster.
- Separate secret and data access boundaries.

**Namespaces alone are not a sufficient hard isolation boundary** for high-trust-separation or dedicated-compute requirements.

### Additional service tenants and clusters
Create additional ACI tenants and/or dedicated Kubernetes clusters when security, compliance, independent lifecycle, routing, resource isolation or dedicated compute justify it. An ACI tenant and Kubernetes cluster do not have to map one-to-one: the mapping is a design decision documented per environment.

## Logical reference topology

```text
                        GitHub (desired state)
                                |
                    +-----------v------------+
                    | ACI: INFRA tenant      |
                    | Shared DNS / NTP       |
                    | Management K3s cluster |
                    | Argo CD / Observability|
                    +------------+-----------+
                                 |
                   explicit routing + ACI contracts
                                 |
          +----------------------+-----------------------+
          |                      |                       |
 +--------v---------+   +--------v---------+   +---------v--------+
 | SERVICE tenant   |   | SECURE tenant    |   | DEDICATED tenant |
 | Shared workloads |   | Higher isolation |   | Dedicated nodes |
 | Service cluster  |   | Separate cluster |   | Separate cluster |
 +------------------+   +------------------+   +------------------+
```

The latter two tenants are optional future patterns, not mandatory day-one deployments.

## Control-plane boundaries

- Each Kubernetes cluster has its **own API server and cluster control plane**.
- Management Argo CD uses scoped credentials to reconcile authorized remote service clusters.
- Service workloads continue running if management Argo CD is unavailable; GitOps changes and central monitoring may pause.
- APIC/fabric control plane remains independent of Kubernetes.
- DNS/NTP service dependencies, failure modes and recovery are explicitly documented.

## Networking and security

1. Define VRF and route-leak decisions separately from ACI tenant naming.
2. Permit only documented source/destination/port flows with ACI contracts; deny other cross-boundary traffic by default.
3. DNS and NTP consumers may be in different tenants/VRFs; use deliberate reachability, resilience and policy.
4. Select Cilium pod routing, IPAM, service exposure and encryption based on a validated ACI integration design.
5. Apply workload policies in Kubernetes in addition to fabric policy; do not treat one as a substitute for the other.
6. Keep administrative and telemetry credentials scoped per cluster and tenant.

## Placement decision guide

| Requirement | Preferred pattern |
| --- | --- |
| Ordinary applications with shared trust and resources | Namespaces and policy within shared SERVICE cluster |
| Distinct team ownership and resource controls | Namespaces, RBAC, quotas, dedicated node pools if adequate |
| Strong security or regulatory separation | Separate cluster and possibly separate ACI tenant/VRF |
| Dedicated compute or independent upgrade lifecycle | Separate cluster and dedicated compute allocation |
| Dedicated external connectivity or routing boundary | Separate ACI tenant/VRF/contract design as required |

## Bootstrap and recovery order

1. Establish fabric reachability and the minimum external/bootstrap DNS/NTP path.
2. Deploy INFRA tenant network policy and foundational services.
3. Bootstrap management cluster independently; then install GitOps and observability.
4. Bootstrap shared SERVICE cluster and register it with management GitOps.
5. Add further tenants/clusters from reusable, documented templates.
6. Rehearse: loss of management cluster, loss of shared DNS/NTP, loss of a service cluster and fabric outage.

**Open decisions:** shared DNS/NTP implementation and HA, number of nodes per cluster, tenant-to-VRF mapping, service exposure, telemetry transport, and cluster placement rules.

See [DUAL-CLUSTER-ARCHITECTURE.md](DUAL-CLUSTER-ARCHITECTURE.md) for the initial two-cluster deployment.
