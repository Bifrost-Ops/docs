# Physical, virtual and network interface requirements

**Fill this matrix before ordering hardware or configuring ACI.** Interface counts are design recommendations, not universal Cisco requirements.

## Physical host port inventory

| Interface | LAB | HA on-prem | Purpose / failure behavior |
| --- | --- | --- | --- |
| Data uplink A | 1x 1/10GbE | 1x 10/25GbE | Primary ACI fabric connection |
| Data uplink B | optional | 1x 10/25GbE | Secondary independent ACI leaf/uplink |
| BMC (iLO/iDRAC/IPMI) | recommended | 1x dedicated | Out-of-band host power, console, rescue |
| Backup/storage | shared or dedicated | separate bandwidth allocation; dedicated where justified | Restore throughput and storage traffic |
| Hypervisor management | logical VLAN | isolated management VLAN/EPG | Administrative control, not workload ingress |

**Important:** Do not connect BMC/OOB to a network that requires the broken production fabric to be healthy to access it. Avoid exposing BMC to the internet. ACI OOB management for APIC/switches and server BMC are separate inventories.

## Logical networks (example placeholders, no real subnets)
| Segment | ACI location | Allowed peers | Notes |
| --- | --- | --- | --- |
| Platform management | INFRA tenant | jump hosts, APIs | Restricted administrative access |
| Shared DNS/NTP | INFRA tenant | approved clients | Resilient service endpoints |
| Management cluster nodes | INFRA tenant | mgmt control plane and monitoring | Separate from service cluster |
| Service cluster nodes | SERVICE tenant (initial) | local control plane; scoped management | Multi-tenant workloads |
| OOB/BMC | dedicated OOB network | bastion only | Independent recovery |
| Backup | dedicated network or policy-controlled shared | backup targets | Encrypted, tested restore |
| APIC management | existing fabric management network | read-only exporter/admin | Do not move APIC mgmt into tenant as part of this project |

## Protocol matrix (baseline; adjust for selected CNI and routing)
| Source | Destination | Port/protocol | Reason |
| --- | --- | --- | --- |
| K3s nodes | Cluster API endpoint | TCP 6443 | K3s API/supervisor |
| HA K3s servers | Other servers in same cluster | TCP 2379-2380 | Embedded etcd |
| K3s nodes | Other nodes | TCP 10250 | Kubelet API/metrics where required |
| Management Argo CD | Service API endpoint | TCP 6443 | Remote cluster GitOps |
| Approved clients | DNS | UDP/TCP 53 | Name resolution |
| Approved clients | NTP | UDP 123 | Time synchronization |
| APIC exporter | APIC | TCP 443 | Read-only telemetry |
| Cilium nodes | Cilium nodes | TBD by mode | VXLAN/Geneve/native-routing selection determines requirements |

**No universal blanket allow rules.** Validate Cilium datapath, tunnel ports, pod CIDRs, MTU and APIC contracts before opening network flows.

## Redundancy tests
1. Disable primary leaf uplink; confirm management and service reachability.
2. Disable secondary uplink; confirm reverse path.
3. Lose one physical host; verify etcd quorum, scheduling and metrics.
4. Block management-to-service path; verify service keeps running.
5. Disconnect OOB from production data; verify independent console.
6. Simulate backup restore to clean isolated environment.
