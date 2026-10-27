# Vendor documentation and verification policy

## Sources of authority
| Component | Official documentation | What to verify |
| --- | --- | --- |
| Cisco ACI NAC | https://netascode.cisco.com/docs/start/aci/introduction/ | Data model, ownership flags, supported APIC version |
| Cisco NAC state splitting | https://netascode.cisco.com/docs/guides/deployment/scaling/ | No duplicate management of tenant root |
| K3s | https://docs.k3s.io/installation/requirements | CPU/RAM, disk and network ports |
| K3s HA | https://docs.k3s.io/datastore/ha-embedded | Odd-number etcd quorum and consistent server flags |
| Cilium on K3s | https://docs.cilium.io/en/stable/installation/k3s/ | Disable default Flannel/network policy; kube-proxy decision |
| Argo CD HA | https://argo-cd.readthedocs.io/en/stable/operator-manual/high_availability/ | HA placement and operational prerequisites |

## Rules
1. Reference vendor documentation alongside each operational decision.
2. Pin versions after testing against the target APIC, K3s and Cilium versions. Avoid `latest` and broad module constraints in deployable artifacts.
3. Prefer upstream-supported installation patterns; document deviations and their consequences.
4. Never treat a lab topology as production-ready merely because it deploys.
5. For every deployment package document preflight, build, verification, backup, rollback, clean rebuild and teardown.
6. Preserve a list of assumptions and **explicitly mark unvalidated design values**.
7. ACI Network as Code root object ownership must not overlap between independently managed Terraform states.
