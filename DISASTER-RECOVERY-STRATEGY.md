# Disaster recovery: rebuild from Git + encrypted recovery assets

**Status:** architecture contract. Do not claim a recovery time until a complete isolated restore has been tested.

## Recovery equation
**Git revision + pinned binaries/images + secrets + Terraform state + cluster datastore/application data + reachable bootstrap infrastructure = recoverable environment.**

Git alone does not recreate secrets, external identities, persistent volumes, Terraform state, DNS delegation or APIC state.

## Failure domains
1. Single workload: reconcile GitOps application, restore its PV/data if needed.
2. Service cluster loss: rebuild service nodes/K3s/Cilium, restore data, reconnect GitOps.
3. Management cluster loss: use independent bootstrap host, external secret/backup access, restore management, then reconnect service.
4. Both clusters lost: restore minimum network/DNS/NTP/secret access, management first, service second.
5. ACI fabric loss: use independently stored APIC/fabric backups and OOB recovery; **Terraform state alone is not an APIC configuration backup**.
6. GitHub unavailable: use an encrypted off-platform Git mirror/bundle of tested revisions.

## Tier-0 bootstrap dependencies (must work without either Kubernetes cluster)
- OOB/bastion access, administrator identity and break-glass procedures
- Independent DNS/NTP or documented fallback with reachable endpoints
- Artifact/image mirror or verified access to pinned artifacts
- External secret manager or recoverable encrypted secret bundle
- External Terraform state backend and immutable/versioned backup
- Off-cluster encrypted cluster snapshots, tokens and PV/application backups
- APIC access and controller/fabric configuration backups

## Recovery sequence
1. Declare incident; freeze automation and preserve evidence.
2. Identify last known-good tested Git commit and matching state/backup set.
3. Verify access to independent OOB, DNS/NTP, artifact registry, secrets and state.
4. Restore fabric connectivity and APIC control plane if needed.
5. Recreate INFRA networks and bootstrap management K3s with pinned versions.
6. Restore management datastore or bootstrap clean; restore secrets and application data as appropriate.
7. Bootstrap Argo CD, reconcile management apps and verify Prometheus/Grafana.
8. Recreate service network/cluster and restore application data.
9. Reconnect service cluster to Argo CD and validate policies/traffic/telemetry.
10. Record achieved RPO/RTO and deviations.

## Integrity and security
- Encrypt backups at rest and in transit with independently managed keys.
- Use separate identities and least-privilege access for writers and restore operators.
- Keep at least one immutable/offsite backup copy and test recovery without the primary account.
- Version Terraform state and use locking; never edit state by hand without a documented incident procedure.
- Back up K3s server tokens together with compatible datastore snapshots, in a separately protected location.
- Restore secrets before reconciling applications that require them.
- Preserve recovery manifests/checksums and test artifacts against pinned versions.

## Acceptance
A deployment package is **not disaster-recovery ready** until an operator can follow its runbook in a clean environment, recover services and data, and prove integrity and access boundaries.
