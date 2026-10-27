# Clean-room bootstrap runbook

**Current status:** preparation guide, not an executable end-to-end installer. Commands and versions must be validated for the selected environment before use.

## 0. Record prerequisites
- [ ] Target APIC endpoint/version and access method are documented.
- [ ] Sandbox restrictions and unsupported features are recorded.
- [ ] Host OS, CPU/RAM/disk, IP addressing, DNS and NTP are documented.
- [ ] Required egress, firewall rules and routing are verified.
- [ ] Terraform, providers, NAC modules, K3s, Cilium and Argo CD versions are pinned.
- [ ] Remote Terraform state, locking, backups and encryption are ready.
- [ ] Credentials are delivered through a secret manager; never committed.
- [ ] Recovery owner and out-of-band access are confirmed.

## 1. Freeze inputs
Record the Git SHA for each of the five repositories. Review environment configuration, dependency versions and any known deviations.

## 2. Bootstrap ACI fabric
Use the `fabric` repository. Validate schema and controller compatibility. Inventory/import existing objects where necessary. Generate a plan and require approval before apply. Verify APIC object state and connectivity.

## 3. Bootstrap tenant policy
Use the `tenants` repository. Validate VRF/BD/EPG/contract and L3Out dependencies. Apply only after reviewed plan approval. Verify endpoint reachability and intended policy.

## 4. Bootstrap Kubernetes
Use `kubernetes`. Confirm topology and CNI choice; capture datastore backup. Install pinned K3s/Cilium components, verify readiness and test service/pod networking. Bootstrap Argo CD only after network validation.

## 5. Bootstrap observability
Use `observability`. Install monitoring after cluster networking is stable. Validate Hubble flows, Prometheus targets, Grafana provisioning, storage limits and alerts.

## 6. Handover evidence
Capture sanitized inventory, deployment SHAs, validation outputs, state/backup references and outstanding exceptions.

## Stop conditions
Stop immediately if the plan unexpectedly deletes/replaces resources, the state lock fails, APIC reports faults, cluster networking is unhealthy, or the recovery path is untested.
