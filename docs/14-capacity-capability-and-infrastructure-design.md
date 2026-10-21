## 14. Capacity, capability and infrastructure design (MAS 6.1-6.3)

Clause 6.3 requires GPUs, network, memory and secure data pipelines sufficient for performance, scalability and resilience. This section gives the design decisions that carry evidence for that clause.

**14.1 Compute and NUMA**

| Decision | Rule | Rationale |
| --- | --- | --- |
| vCPU count | Size from measured demand. A VM that fits one pNUMA node can use any count that node supports | Over-allocation raises co-stop and scheduling delay |
| Wide VMs | When a VM spans NUMA nodes, choose a count that divides evenly across nodes. Check the resulting vNUMA layout in the guest \[V on exact rules\] | Uneven vNUMA hurts latency-sensitive inference |
| GPU locality | Place VM vCPU and memory on the NUMA node that hosts the GPU and its NIC | Cross-socket PCIe traffic lowers throughput |
| Memory | Full reservation for vGPU and passthrough VMs | Required for these device types |
| CPU reservation | Tier 2: reserve by measured steady demand. Reservation guarantees MHz, not low scheduling latency. Latency sensitivity High requires full reservation and affects consolidation \[V\] | Avoid blanket 100% policy |
| Overcommit | Tier 2 clusters at or near 1:1 on reserved resources. Policy value set by FI | Predictable capacity |
| Hyperthreading | Test with and without for inference VMs | Workload dependent |

**14.2 GPU design**

| Option | Use | Isolation | Operational notes |
| --- | --- | --- | --- |
| Time-sliced vGPU | Tier 0, light inference | Temporal only | Noisy neighbor risk |
| MIG-backed vGPU | Tier 1-2 inference, shared GPUs | Spatial partition of compute and memory. Supported from vGPU 19.0 on vSphere 9.0 ([NVIDIA](https://docs.nvidia.com/vgpu/deployment/vmware/latest/vgpu-config.html)) | Fixed slice sizes. Plan layout |
| Passthrough (DirectPath I/O) | Large models, multi-GPU training | Whole device per VM | PAIS model endpoint examples use passthrough ([William Lam](https://williamlam.com/2026/09/vcf-9-1-1-deploying-your-own-ai-models-with-vcf-private-ai-services-pais-3-0.html)). No vMotion. HA behavior \[V\] |

- vMotion of vGPU VMs works on a subset of GPUs and drivers. Stun time scales with VM memory and used framebuffer ([VMware doc](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vcenterhost.doc/GUID-8FE6A0DA-49E9-472B-815B-D630CF2014AD.html)). vSphere 9.0 improved large-profile vMotion ([VMware blog](https://blogs.vmware.com/cloud-foundation/2025/06/19/enhanced-vmotion-for-vgpu-vms-in-vmware-cloud-foundation-9-0/)). Test with production model sizes.
- Maintenance evacuation of passthrough VMs needs downtime or a second GPU host. Plan N+1 per tier.
- DRS places vGPU VMs at initial placement. Load balancing support for them is limited \[V on 9.x\].
- GPU host driver and firmware managed through vLCM images \[V on vendor add-on packaging\].
- Large-BAR GPUs in passthrough may need 64-bit MMIO advanced settings on the VM \[V\].

**14.3 Network**

| Plane | Design | Control |
| --- | --- | --- |
| Management | Dedicated VLAN or segment | DFW and jump host |
| Inference north-south | NSX VPC or T1. Avi SEs. Edge sized for peak throughput | Gateway Firewall, WAF |
| Inference east-west | Overlay. VMXNET3. DFW and IDS/IPS | Section 11.3 |
| Storage | vSAN network, data-in-transit encryption on | vSAN policy |
| Training and RDMA | Dedicated fabric. RoCE with lossless config. SR-IOV or passthrough NICs. MTU planned end to end | Physical ACL or VRF. No DFW coverage \[V\] |

- Size Avi SEs from measured tests. Published figures date from old lab runs. Do not reuse them for LLM streaming workloads.
- Edge node sizing needs a throughput test with realistic response sizes.

**14.4 Storage and secure data pipelines**

- vSAN ESA on NVMe for datasets, checkpoints, model artifacts. vSAN Max or a storage cluster where compute and storage scale apart. Data-in-transit encryption for remote datastores from VCF 9.1.
- Capacity inputs: datasets, checkpoints, model versions, snapshots, replicas, and protection overhead from the storage policy.

```latex
Raw = (Datasets + Checkpoints + Models + Snapshots) x PolicyOverhead / (1 - SlackSpace)
```

- Checkpoint size is about parameters times bytes per parameter. Training adds optimizer state. Measure it. Checkpoint writes are bursty. Test vSAN write latency under checkpoint load.
- Reads for training: required GB/s per GPU times GPU count, measured from a pilot.

| Pipeline stage | Control |
| --- | --- |
| Ingest | Source authentication, classification tag, malware scan, quarantine share |
| Transform | Pipeline on VKS in Tier-matched namespace. Read-only mount for source |
| Train and test | Separate roles and namespaces. Protected attributes in separate share |
| Publish | Signed artifact into Model Store. Digest recorded in registry |
| Serve | Model Runtime pulls by digest. Gateway in front |
| Retire | Section 7.4 |

**14.5 Capacity management and resilience (6.3)**

- With reservations, capacity becomes allocation-based, not demand-based. Track allocated versus used GPU and memory. Reserved but idle GPU is waste and a cost signal.
- Use VCF Operations capacity and what-if views for GPU hosts, vSAN, and Edge. Set early-warning thresholds on remaining headroom.
- Resilience: HA admission control with spare GPU hosts. Multi-zone Supervisor placement. DR for stateful services. Test with failure injection (section 10.2).
- Keep a GPU supply buffer for lead-time risk. This is a procurement control. MAS 6.1 asks for adequate resources.

**14.6 Capabilities (6.1-6.2)**

- Staff competence is a people control. Infrastructure teams need training on vGPU and MIG operations, VKS, NSX policy, and AI-specific failure modes.
- Review the training programme regularly against new AI types such as agents (6.2).
- Track capacity of the control function and validators. Under-resourced validation is a finding.
