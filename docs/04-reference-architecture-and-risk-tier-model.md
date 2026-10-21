## 4. Reference architecture and risk-tier model

A three-tier control profile maps MAS risk materiality (4.10-4.13) to infrastructure. The tier tag is the single control input. Every other control derives from it.

**4.1 Tier definitions**

| Control dimension | Tier 0 (basic, clause 2.3) | Tier 1 (medium) | Tier 2 (high, clauses 4.12, 5.3, 5.19) |
| --- | --- | --- | --- |
| Typical use | Drafting, summarising, internal search | Internal decision support with human review | Credit, underwriting, advice, AML, autonomous agents with tool access |
| Placement | Shared workload domain | Dedicated cluster or resource pool, dedicated Supervisor namespace and VPC | Dedicated workload domain, dedicated VKS clusters |
| Compute | No reservation | Full memory reservation on GPU VMs. CPU reservation by measured demand | Full memory and CPU reservation. DRS anti-affinity across failure domains |
| GPU | Time-sliced vGPU | MIG-backed vGPU | MIG-backed vGPU or passthrough. N+1 GPU host headroom |
| Network | DFW baseline default-deny | DFW per-role policy. IDS/IPS in detect mode | DFW, IDS/IPS prevent, gateway egress allowlist, ATP/NDR |
| Encryption | Cluster vSAN data-at-rest | Cluster encryption plus VM Encryption, tier key provider | Cluster encryption plus VM Encryption, external KMS with HSM, per-use-case keys |
| Logging | Platform logs | Platform plus prompt/response logs | Platform plus prompt/response, agent traces, immutable retention |
| Contingency | Documented | Fallback route tested annually | Kill switch and fallback tested at set interval (5.3) |
| Evidence | Compliance report | Compliance report and change log | Plus independent validation pack, drift reports |

Reservation, key and test-interval values are policy decisions for the FI. This table sets the structure.

**4.2 Domain layout**

1. Management domain: VCF Operations, VCF Automation, vCenter, NSX Manager, SDDC Manager as applicable to the VCF 9.x version \[V\].
2. Shared workload domain: Tier 0 and general enterprise workloads.
3. AI-controlled workload domain: Tier 1-2 GPU clusters, vSAN ESA, dedicated NSX transit and VPCs.
4. Test and adversarial domain: no external route. One-way artifact intake. Masked data only.
5. Optional dedicated vSAN Max or storage cluster for training data, mounted to AI clusters with data-in-transit encryption (supported for remote datastores from VCF 9.1, [VMware blog](https://blogs.vmware.com/cloud-foundation/2026/05/12/vsan-storage-clusters-in-vcf-9-1/)).

**4.3 Platform services used**

- vSphere: vCenter, DRS, HA, vLCM, VM Encryption, vGPU, Identity Federation.
- vSAN ESA: NVMe storage, data-at-rest and in-transit encryption, File Services.
- NSX and vDefend: VPC, DFW, IDS/IPS, ATP, Gateway Firewall. vDefend is sold as an add-on to VCF ([Broadcom cloud provider blog](https://blogs.vmware.com/cloudprovider/?p=27026)). Confirm entitlement.
- Avi Load Balancer: inference API front end, WAF, rate limiting.
- VCF Operations: metrics, logs, compliance, capacity, alerting.
- VCF Automation: organizations, projects, templates, approvals.
- VKS: Kubernetes clusters on Supervisor. Private AI Services is included in VCF 9 and provides GPU Monitoring, Model Store (Harbor), Model Runtime with OpenAI-compatible API, Agent Builder, vector database, data indexing and retrieval ([StorageReview](https://www.storagereview.com/zh-CN/news/broadcom-positions-vcf-as-an-ai-native-platform-for-the-modern-private-cloud), [VMware blog](https://blogs.vmware.com/cloud-foundation/2025/06/19/private-ai-services-new-in-vmware-private-ai-foundation-with-nvidia-in-vcf-9-0/)). Entitlement timing per Broadcom: Q1 FY26 \[V\].

**4.4 Tenancy caveat**

On VCF Automation 9.0.x, one vCenter and NSX pair serves either classic or modern tenancy. Separate pairs may be needed for AI tiers and legacy Aria-style tenants. 9.1 removes the restriction ([KB 441498](https://knowledge.broadcom.com/external/article/441498/missing-svcautoattachvksmbroadcomcom-in.html)). VCF Automation 9.0.x also allows one Supervisor per tenant organization ([KB 438333](https://knowledge.broadcom.com/external/article/438333/vcfa-90x-unable-to-assign-more-than-1-su.html)). 9.1 adds region quotas across Supervisors. Design the org model with this constraint.
