# BGP-first implementation milestones

## M1 — Schema and topology (in progress)
- [x] Document opt-in Cilium BGP advertisement templates.
- [x] Document dual-peer planning inventory for ACI L3Out boundary.
- [x] Document BGP preflight and recovery runbook.
- [ ] Choose exact APIC release and validated L3Out peering topology.
- [ ] Confirm Cilium and NAC versions and validate against vendor schemas.

## M2 — Deployable configuration
- [ ] Build version-pinned NAC tenant data model with real node/interface profiles and routing policies.
- [ ] Build cluster-specific Cilium Helm values and BGP CRD overlays.
- [ ] Establish encrypted remote Terraform state, locks and independent secrets recovery.
- [ ] Add automated schema/plan checks without unattended applies.

## M3 — Isolated integration tests
- [ ] Establish both BGP sessions and advertise a single service VIP.
- [ ] Verify route filtering, return traffic and externalTrafficPolicy behavior.
- [ ] Validate failover, withdrawal, MTU and policy isolation.
- [ ] Capture Git revision, APIC backup, state version and test evidence.

## M4 — Rebuild and recovery
- [ ] Destroy isolated test resources after confirmed backup and approval.
- [ ] Recreate ACI network, management and service clusters from Git.
- [ ] Restore secrets/state/application data from independently secured storage.
- [ ] Prove BGP convergence, DNS/NTP, observability and application availability.

**Never mark M2–M4 complete based on committed YAML alone.**
