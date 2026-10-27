# ADR: ACI underlay and Cilium networking

**Status:** Proposed. Do not implement BGP or native pod routing without target-platform testing.

## Baseline
- ACI provides node connectivity, VRF/EPG/contract macro-segmentation and underlay reachability.
- Each Kubernetes cluster owns its Cilium policy domain, with workload micro-segmentation.
- Avoid mixing ACI CNI and Cilium as concurrent primary CNIs.
- Choose either a tunnel datapath or native routing based on observed reachability and MTU.
- Prefer a simple, independently verifiable service exposure mechanism for initial lab.

## Options to evaluate
1. **Cilium tunnel mode:** simple underlay dependency, extra encapsulation and MTU considerations.
2. **Cilium native routing:** needs working routes for PodCIDRs and careful ACI boundary design.
3. **Cilium BGP Control Plane:** optional for service VIP and/or pod route advertisement, requires a supported Cilium version and valid BGP adjacency/routing integration. ACI leaf direct peering is **not assumed**; validate L3Out/external router topology and support.

## Mandatory proof
- Pod-to-pod, pod-to-service and node-to-node reachability
- DNS and NTP reachability from both clusters
- ACI macro-policy and Cilium micro-policy denial tests
- Cross-tenant route and contract tests
- MTU and fragmentation tests
- Service VIP advertisement and failover tests if BGP is selected
- Fabric leaf failure, node failure and management outage behavior
- Packet tracing with ACI faults and Hubble flows correlated

## Decision record fields
Target APIC version; leaf topology; node hypervisor/bare metal; K3s/RKE2 release; Cilium release; kernel; MTU; Node/Pod/Service CIDRs; service exposure; IPAM; route ownership; BGP peers if applicable; tested rollback.
