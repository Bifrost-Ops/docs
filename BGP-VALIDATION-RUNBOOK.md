# BGP validation and recovery runbook

**Scope:** disposable lab first. Record evidence for both INFRA and SERVICE independently.

## Preflight
- Record APIC version, ACI tenant/VRF, L3Out interfaces, BGP peer addresses/ASNs, route filtering and max-prefix policies.
- Confirm IPAM, LB VIP pool, Cilium release/CRDs, node selectors and security approval.
- Confirm separate emergency access to node consoles and ACI management; BGP must not be the only recovery path.
- Confirm independent routes and return paths for the service VIPs.

## Checks
1. Apply peer config, advertisement, VIP pool and cluster config in a controlled lab after replacing placeholders.
2. Check Cilium CRD status and BGP session establishment; record advertised routes.
3. Confirm only explicitly labeled LoadBalancer VIPs are exported; ClusterIP and PodCIDR are not exported.
4. Confirm upstream accepts only intended prefixes and installs valid next hops.
5. Exercise an external client connection through ACI to the service.
6. Remove Service export label; observe route withdrawal and client behavior.
7. Restart one Cilium agent, then operator, one at a time; measure convergence.
8. Simulate one peer failure and leaf/path failure; record downtime and ECMP effects.
9. Test service with no ready endpoints and both externalTrafficPolicy modes.
10. Check MTU, route symmetry, ACI contract denies and Cilium policy denies.

## Recovery
- If routes leak: withdraw approved advertisements at upstream routing boundary, preserve console access, inspect RIB/FIB and reconcile selectors.
- If session fails: validate TCP 179 reachability, ASN, authentication, source IP, next hop, policy and Cilium agent/operator logs.
- If blackhole occurs: inspect endpoint readiness, service externalTrafficPolicy, VIP assignment, upstream next hops and node forwarding.
- Roll back via reviewed Git change and confirm route withdrawals. A Git revert is not itself proof of network recovery.

## Evidence record
Record Git SHA, Cilium/APIC releases, timestamps, peer state, advertised/received prefixes, pcap/log extracts, observed convergence and recovery result.
