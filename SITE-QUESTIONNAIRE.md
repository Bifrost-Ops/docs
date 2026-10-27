# Site discovery and acceptance questionnaire

Copy this document per site. Replace placeholders, attach network diagrams and review with network, compute, security and operations owners.

## Identity and scope
- Site/environment ID:
- Owner / on-call:
- Package: ONPREM-LAB / ONPREM-HA / CLOUD-HYBRID
- ACI fabric/APIC release:
- Hypervisor and OS versions:
- Expected application workload / peak utilization:
- Security tiers / dedicated-compute requirements:

## Fabric and network
- [ ] Leaf pair, port IDs, VLAN pool, domain, AEP and EPG mapping documented
- [ ] INFRA and SERVICE tenant/VRF/BD model reviewed
- [ ] Pod/Service/Node CIDRs do not overlap with ACI or cloud routing
- [ ] MTU and Cilium routing mode tested
- [ ] DNS/NTP endpoints and bootstrap fallback documented
- [ ] API VIP/LB and TLS certificate lifecycle agreed
- [ ] Inter-tenant contract matrix approved

## Compute and hardware
- [ ] Host count, CPU generation, RAM, SSD/IOPS and spare capacity verified
- [ ] Primary and secondary NICs/leaf paths recorded
- [ ] BMC/iLO/iDRAC addresses, access control and independent OOB verified
- [ ] Dual PSU/PDUs and power failure domains recorded
- [ ] Anti-affinity and etcd quorum placement verified

## Storage and backup
- [ ] Snapshot cadence and retention approved
- [ ] Off-cluster encrypted backup location tested
- [ ] K3s server tokens and secret-manager recovery process documented
- [ ] PV/application backup and restore tested
- [ ] Terraform state versioning/locking/recovery tested
- [ ] RPO and RTO established by restore rehearsal, not assumption

## GitOps and monitoring
- [ ] Bootstrap can run without the management cluster
- [ ] Argo CD service cluster RBAC scoped
- [ ] Grafana/Prometheus retention, credentials and access approved
- [ ] APIC exporter uses a read-only account and bounded API polling
- [ ] Dashboards and alert rules version controlled

## Sign-off
- Network:
- Compute:
- Security:
- Operations:
- Recovery drill date:
- Approved deployment revision:
