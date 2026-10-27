# Operations and change control

## Standard change procedure
1. Identify the target environment and capture current health, backups, Git revision and Terraform state version.
2. Describe intent, blast radius, dependencies, validation checks and a **specific rollback decision point**.
3. Open a pull request with linked issue, updated YAML comments, runbook and expected plan.
4. Run static checks and `terraform plan` using pinned providers/modules and the correct state backend.
5. Inspect every `create`, `update`, `delete` and `replace` action. Unexpected destruction blocks approval.
6. Obtain explicit approval before applying. Never apply a speculative plan to a shared ACI sandbox or production.
7. Apply only the reviewed plan artifact when possible; avoid planning again with changed inputs.
8. Verify APIC health, endpoint learning, contracts and connectivity; for Kubernetes check nodes, CNI and workloads.
9. Record the deployed commit, plan, timestamp, approver, verification results and any drift.

## Rollback decision tree
- **Configuration error, state healthy:** revert the change in Git, generate a new plan, inspect destructive effects and apply only after approval.
- **Terraform state corruption:** stop all writes; restore a verified state backup and reconcile with real infrastructure before planning.
- **Kubernetes application error:** roll back the GitOps revision; confirm reconciliation and health.
- **Persistent data loss:** restore from application/datastore backup; Git alone cannot recover data.
- **Connectivity to management plane lost:** use the documented out-of-band recovery procedure, not repeated blind applies.

## Destructive actions
No automatic destroy. Require a separately reviewed teardown checklist, inventory of resources and dependencies, and backup confirmation.

## Evidence
Store sanitized plan summaries and verification results in pull requests or change records. Never publish secret-bearing logs, state files or kubeconfigs.
