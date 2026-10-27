# Bifrost Ops — standing engineering policy

**Applies to all five Bifrost Ops repositories and future packages.**

## Non-negotiable design goals
- Vendor documentation and documented best practices take precedence over assumptions.
- Every YAML/configuration block explains intent, dependencies, scope, impact and recovery.
- Each deployable component has step-by-step install, verify, upgrade, rollback, backup, restore and clean rebuild instructions.
- Configuration, GitOps manifests, monitoring rules and Grafana dashboards are version controlled.
- Secrets, live Terraform state and persistent data never go into public Git.
- Maintain a reusable catalog for on-premises lab, on-premises HA and cloud/hybrid deployments.
- Include compute, memory, storage/IOPS, interfaces, primary/secondary connectivity, backup, BMC/iLO/iDRAC and independent OOB requirements.
- INFRA is a user-created ACI application tenant for shared DNS, NTP, management, observability and centralized GitOps coordination.
- SERVICE supports multiple workloads/teams; additional ACI tenants or dedicated clusters are allowed for stronger isolation and dedicated compute.
- Kubernetes control planes remain independent per cluster; management Argo CD is a centralized reconciler, not the service API server.
- Validate vendor compatibility and pin tested versions before turning reference designs into deployable manifests.
- All applies, destructive changes and production access require explicit approval.

## Acceptance gate
A package is not marked `ready` until vendor compatibility, preflight, syntax/schema checks, isolated deployment, recovery rehearsal and security review have recorded evidence.

## Documentation discipline
Whenever implementation changes, update architecture diagrams, configuration comments, package sizing, protocol matrices and recovery steps **in the same pull request**. Keep explicit `TBD` fields rather than guessing.
