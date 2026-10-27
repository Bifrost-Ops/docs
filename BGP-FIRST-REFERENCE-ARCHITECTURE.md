# BGP-first ACI–Cilium reference architecture

**Status: proposed design + example manifests, not tested against an APIC.**

## Principle
Use standard eBGP between Cilium's supported BGP Control Plane and an **explicitly designed routed BGP boundary** (ACI L3Out topology, potentially an external router). Avoid coupling Kubernetes lifecycle to a custom APIC/Kubernetes controller.

- ACI: tenant/VRF, node underlay connectivity, L3Out and contracts (macro-segmentation).
- Cilium: pod/service datapath, workload network policy, BGP advertisements (micro-segmentation).
- INFRA: shared DNS/NTP, management cluster, GitOps and observability.
- SERVICE: independent service cluster; more tenants/clusters for isolation or dedicated compute.
- Git: desired state; state, secrets and data are external protected recovery assets.

## Critical distinction
**BGP Control Plane advertises routes; it does not install the required native pod forwarding routes on every node.** Native routing requires node-to-pod and node-to-node reachability. Do not assume a BGP session alone makes the pod network routable.

### Route export phases
1. **Phase 1: service VIP only.** Advertise opt-in LoadBalancer IPs using `CiliumBGPAdvertisement`; restrict upstream prefix import. Choose tunnel/native datapath separately after MTU tests.
2. **Phase 2: PodCIDR export (optional).** Requires Kubernetes IPAM compatibility, native-routing route distribution, return-path design, no overlap and security approval.
3. **Phase 3: dedicated service clusters.** Repeat the same contract using unique ASNs, prefix pools and peer addresses per security domain.

## Physical and logical peering
ACI BGP adjacency is an L3Out concern, **not an ordinary application EPG feature**. A service-node-to-leaf BGP session is not assumed supported just because nodes attach to a BD/EPG. Choose a validated L3Out interface/profile, border leaf or external routed peer, and record how node networks reach it. No unverified BGP direct-peering claims.

## Minimum safety controls
- Two independently reachable peers/failure paths where the network supports it.
- Explicit export/import prefix filters and maximum-prefix limits upstream.
- Separate INFRA and SERVICE address pools; never export ClusterIP by default.
- BGP authentication where supported; store credentials outside Git.
- Distinguish service VIP advertisement from PodCIDR advertisement.
- Test BGP flap, agent/operator restart, route withdrawal, service backend loss, MTU, ECMP and failover.
- Cilium BGP currently does not provide BFD in the referenced operator guide; design convergence tests accordingly.
- Prevent accidental broad selectors: only Services explicitly labeled for BGP advertisement are eligible.

## Implementation references
- https://docs.cilium.io/en/stable/network/bgp-control-plane/bgp-control-plane-configuration/
- https://docs.cilium.io/en/stable/network/bgp-control-plane/bgp-control-plane-operation/
- https://docs.cilium.io/en/stable/network/concepts/routing/
- https://netascode.cisco.com/docs/data_models/apic/tenants/l3out/
- https://www.cisco.com/c/en/us/td/docs/dcn/aci/apic/6x/l3-configuration/cisco-apic-layer-3-networking-configuration-guide-62x/g_routed_connectivity_to_external_networks.html

## Release gate
Validate actual Cilium chart/CRD schema, APIC version, L3Out peering support and node datapath in a disposable lab before promotion. Example values under `kubernetes/networking/bgp` use **RFC 5737 documentation IPs** and must never be applied unchanged.
