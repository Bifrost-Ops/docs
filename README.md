# Bifrost Ops — Documentation

**From Code to Infrastructure.**

Bifrost Ops is a reproducible, documentation-first infrastructure-as-code reference for Cisco ACI, Kubernetes and observability.

> **Safety:** The repositories start as templates. No production credentials, real endpoints, or automatic deployment are included.

## Repository map

| Repository | Responsibility |
| --- | --- |
| [fabric](https://github.com/Bifrost-Ops/fabric) | ACI fabric and access configuration (Day 0) |
| [tenants](https://github.com/Bifrost-Ops/tenants) | ACI tenants, VRFs, bridge domains, EPGs and contracts (Day 1) |
| [kubernetes](https://github.com/Bifrost-Ops/kubernetes) | K3s, Cilium and Argo CD |
| [observability](https://github.com/Bifrost-Ops/observability) | Hubble, Prometheus and Grafana |
| [docs](https://github.com/Bifrost-Ops/docs) | Architecture, runbooks, change management and recovery |

## Start here

1. Read [Architecture](ARCHITECTURE.md) and understand the dependency order.
2. Read [Operations](OPERATIONS.md) before configuring any credentials.
3. Use [Bootstrap](BOOTSTRAP.md) for a clean-room installation.
4. Follow [Recovery](RECOVERY.md) before any destructive operation.
5. Review [Documentation standards](DOCUMENTATION-STANDARDS.md) for required YAML comments and runbook structure.

## Operating rules

- Every change must be reviewable as a diff.
- Validate syntax and produce a plan before any apply.
- Never commit credentials, kubeconfigs or Terraform state.
- Keep a known-good Git revision and record the last successful deployment.
- Distinguish **rollback of configuration** from **restoration of persistent data**.
- ACI Sandbox capabilities may differ from physical fabric capabilities; unsupported steps must be marked explicitly.
