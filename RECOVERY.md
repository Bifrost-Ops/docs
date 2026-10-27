# Recovery and rebuild

## Recovery priorities
1. Preserve evidence and stop automation that could compound the failure.
2. Establish out-of-band management access.
3. Identify whether the failure concerns configuration, Terraform state, cluster datastore, persistent volumes or external dependencies.
4. Choose rollback versus full rebuild; they are not interchangeable.

## Git configuration rollback
1. Identify the last known-good deployed commit (not merely the previous commit).
2. Create a revert pull request and review the complete resulting diff.
3. Re-run validation and generate a new infrastructure plan.
4. **Do not apply** if the plan would unexpectedly destroy shared ACI objects or storage.
5. Approve, apply, verify health and document outcome.

## Terraform state recovery
1. Pause all writers and preserve the failed state snapshot.
2. Verify backend, lock status, state lineage and serial.
3. Restore a known-good encrypted state version following backend-specific instructions.
4. Compare state against actual APIC resources and reconcile imports/drift.
5. Resume changes only after a reviewed, non-destructive plan.

## Kubernetes rebuild
1. Export inventory and verify backup freshness, secret availability and version compatibility.
2. Recreate host prerequisites and networking.
3. Restore the appropriate K3s datastore backup **or** deliberately create a fresh cluster; do not confuse these operations.
4. Restore required secrets and persistent data from their own backups.
5. Bootstrap Cilium and Argo CD in the documented order.
6. Reconcile workloads and verify DNS, networking, storage, metrics and alerts.

## Verification matrix
| Layer | Checks |
| --- | --- |
| ACI | APIC faults, interface status, endpoint learning, contracts, routing |
| K3s | nodes ready, system pods healthy, API available |
| Cilium | agent health, policy verdicts, pod/service connectivity |
| Argo CD | desired versus live state, sync and health |
| Observability | Prometheus targets, Hubble flows, Grafana dashboards |

## Rehearsals
Perform recovery exercises in a disposable lab before declaring production readiness. Record elapsed time, missing dependencies and any manual steps discovered.
