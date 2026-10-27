# External architecture review — decisions and validation gates

**Status:** review assessment; no deployment compatibility claims are made.

## Accepted findings
- Terraform revert is not a guaranteed safe ACI rollback. Require protected plans, deletion review, APIC-native configuration backups, out-of-band recovery and tested change procedures.
- Choose and implement concrete secrets and remote-state backends before deployable environments are released. Preserve independent bootstrap and key recovery.
- Management Argo CD must be bootstrappable without Argo CD; service Kubernetes control planes remain independent.
- Hubble flows are not equivalent to durable logs or full packet capture. Bound metric cardinality and add separate log aggregation/retention architecture where required.
- Infrastructure alerts need a monitoring path outside the fabric/cluster failure domain.
- On-prem ACI NAC objects must not be assumed to map directly to cloud network controllers.

## Findings requiring correction or validation
- K3s is not inherently unsuitable for production. Compare K3s and RKE2 against operational/security/upgrade requirements; enforce 3-server embedded-etcd HA for the HA reference package.
- ACI's internal VXLAN fabric does not automatically imply problematic double encapsulation at the host. Validate Cilium tunnel/native routing against effective path MTU and operational complexity.
- Do not mandate Cilium BGP peering directly with ACI leaves or use legacy `CiliumBGPPeeringPolicy` without checking the chosen Cilium release. Evaluate BGP Control Plane CRDs supported by the pinned version and the actual ACI L3Out/peer topology. Service VIP advertisements and PodCIDR routing are different design decisions.
- ACI should provide macro-segmentation across cluster/node network boundaries; Cilium should provide pod/workload micro-segmentation, unless an explicitly validated alternative is chosen.
- S3-compatible storage does not by itself guarantee supported Terraform locking, immutability, versioning or reliable recovery. Validate backend features for the selected Terraform release and object-store implementation.

## Design gates
1. **Platform choice:** record K3s vs RKE2 decision, supported OS/kernel, cluster upgrade and HA procedure.
2. **Networking:** document ACI tenant/VRF/BD/EPG, Cilium datapath, IPAM, service exposure, MTU, contracts and test results.
3. **Security:** select secrets, state, key escrow and immutable backup products and test restore with management offline.
4. **Recovery:** capture known-good APIC backup, state version, Git revision, cluster snapshot, tokens, PV backups and secret recovery path.
5. **Operations:** test alert delivery during management/fabric outage, upgrades, rollbacks and full clean-room rebuild.

## Proposed first implementation target
A reproducible isolated lab with two independent clusters and separate INFRA/SERVICE application tenants, initially using a conservative networking approach with no unvalidated BGP assumptions. Promote to HA only after tests and architecture review.

## Official vendor references for verification
- https://docs.k3s.io/datastore/ha-embedded
- https://docs.k3s.io/installation/requirements
- https://docs.cilium.io/en/stable/network/bgp-control-plane/bgp-control-plane/
- https://docs.cilium.io/en/stable/network/concepts/routing/
- https://developer.hashicorp.com/terraform/language/backend/s3
- https://netascode.cisco.com/docs/start/aci/introduction/
