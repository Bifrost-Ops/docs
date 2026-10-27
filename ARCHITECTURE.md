# Architecture and ownership

## Goal
Build a repeatable lab that can be recreated from a clean host and a known Git revision without relying on undocumented manual changes.

## Dependency order
```text
Prerequisites (network, DNS, NTP, secrets, state backend)
  -> ACI fabric (Day 0, physical connectivity)
  -> ACI tenants (Day 1, policy and L3 connectivity)
  -> K3s (cluster)
  -> Cilium (cluster networking)
  -> Argo CD (application reconciliation)
  -> Hubble / Prometheus / Grafana (observability)
```

The exact ACI-to-Kubernetes integration is **not yet selected**. Do not assume that ACI CNI and Cilium can be installed together without a verified integration design.

## Ownership boundaries
| Component | Source of truth | State |
| --- | --- | --- |
| ACI fabric/access | fabric | Dedicated Terraform state |
| ACI tenant policy | tenants | Dedicated Terraform state |
| K3s bootstrap | kubernetes | Cluster bootstrap configuration and secured datastore backup |
| GitOps applications | kubernetes / observability | Git and Kubernetes cluster state |
| Metrics/dashboards | observability | Git for configuration; separate backup for retained data |

Never have two repositories manage the same object or Terraform state. Record object imports and ownership transfers in a change log.

## Environment model
- `lab`: disposable test environment with explicit teardown approval.
- `production`: future environment only; no deployment until architecture, backup and security reviews are complete.
- Environment-specific values must not be silently copied between environments.

## Open design decisions
- APIC version and ACI sandbox/physical-fabric capabilities
- NAC module/provider versions and exact input schema
- ACI integration with K3s/Cilium, routing and IPAM
- Terraform remote state backend, locking and recovery
- K3s topology/datastore and backup storage
- Secret manager and CI identity model

Do not convert placeholders into live configuration until these decisions are documented and validated.
