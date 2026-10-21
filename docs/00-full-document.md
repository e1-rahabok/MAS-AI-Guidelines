# MAS AI Risk Management Guidelines: VCF 9 Implementation Architecture (Level 400)

Oct 21, 2026 · @Sunrise

## 1. Summary and scope boundary

VCF 9 supplies the infrastructure controls and audit evidence for MAS clauses 4.5, 5.4(e), 5.16, 5.22, 5.23(d), 5.25(b) and 6.3. It does not supply model-level controls. Those need an MLOps, GRC and data-governance layer on top.

- MAS is technology-agnostic. No product complies. The FI complies. VCF provides controls and evidence.
- Infrastructure-complete: encryption, segmentation, privileged access, logging, capacity, change control on the platform.
- Partial: AI inventory (4.5-4.9), kill switch (5.3, 5.23b), monitoring (5.23a), reproducibility (5.17), third-party AI (5.10-5.11).
- Not covered by VCF: fairness (5.7-5.8), explainability (5.5-5.6), data and concept drift (5.23a), independent model validation (5.19), model evaluation (5.14), data representativeness (5.4b).
- Section 2 lists 17 corrections to the two source drafts. Read it first. Several drafted claims would not survive a regulator or VCDX panel review.
- Product naming follows VCF 9.x: VCF Operations, VCF Automation, VKS, vDefend, Avi Load Balancer, Private AI Services (PAIS).
- Items tagged **\[V\]** are from memory or a single secondary source. Lab-verify before customer use. Section 16 consolidates them.

**Document structure**

- Sections 3-4: timeline, tier model, reference architecture.
- Sections 5-14: clause-by-clause design. Each gives MAS requirement, control design, configuration, evidence, gap.
- Section 15: traceability matrix.
- Sections 16-17: gap register, verification list, roadmap.

## 2. Corrections to the two source analyses

17 claims in the drafts are wrong, overgeneralized or incomplete. Corrected positions are used throughout this document.

| # | Source claim | Problem | Corrected position |
| --- | --- | --- | --- |
| 1 | Aria Automation blueprints enforce tags | Product renamed. In VCF 9, Aria Automation is VCF Automation, Aria Operations is VCF Operations. In VCFA 9.0.x, a vCenter and NSX Manager pair is reserved to classic or modern tenancy, not both. Fixed in 9.1 ([Broadcom KB 441498](https://knowledge.broadcom.com/external/article/441498/missing-svcautoattachvksmbroadcomcom-in.html)) | Use VCF Automation. Plan tenancy model per vCenter/NSX pair on 9.0.x. |
| 2 | Tags at provisioning satisfy a real-time AI inventory | Tags cover infrastructure objects only. MAS 4.7 needs purpose, scope, data, owner, risk rating, review status. Tags prove nothing about completeness (4.4, shadow AI) | Tags are one feed into a GRC/CMDB inventory. Add discovery and reconciliation (section 5). |
| 3 | Admission control policies restrict high-risk AI to clusters | vSphere HA admission control reserves failover capacity. It is not a placement or security control | Use DRS VM-Host rules, compute policies, Supervisor namespace constraints, NSX groups, policy engine in VKS. |
| 4 | 100% CPU and RAM reservations for AI workloads | Reservation guarantees MHz, not scheduling latency or co-stop avoidance. It removes consolidation and consumes HA failover capacity. Memory reservation is already mandatory for vGPU and passthrough | Tiered reservation policy: full memory for GPU VMs, CPU reservation by measured demand and tier (section 14). |
| 5 | Right-size to exact vCPU count always beats topology symmetry (7 not 8) | Overgeneralized. True when the VM fits inside one pNUMA node. False when it spans nodes, where vCPU count must divide cleanly across vNUMA nodes. GPU and NIC NUMA locality also matters | Right-size first. Align to NUMA boundaries once the VM exceeds one node. \[V\] on exact vNUMA split rules. |
| 6 | Deleting the vSAN key cryptographically shreds model weights and training data | vSAN data-at-rest encryption is cluster-scoped. Deleting its key destroys the whole datastore, not one dataset. It protects against device removal, not a privileged reader ([DISA STIG V-258929](https://stigviewer.com/stig/vmware_vsphere_8.0_vcenter/2023-10-11/finding/V-258929)). Snapshots, replicas, backups and DR copies remain | Per-workload erasure needs VM Encryption with segmented key providers plus copy discovery (section 7). |
| 7 | vSAN File Services ACLs ensure only authorized VKS containers mount datasets | NFS exports are host-IP based. CSI file volumes mount at node level. Container identity is not enforced at the share | Enforce at three layers: K8s RBAC and namespaces, node-pool-scoped exports, NetworkPolicy. |
| 8 | DFW on every AI node vNIC plus IDS/IPS stops lateral movement and exfiltration | DFW sees the node vNIC, not pod-to-pod traffic inside a node. VKS uses a CNI. vDefend 9.1 extends IDS/IPS to VKS pods via CNI integration ([vDefend 9.1](https://blogs.vmware.com/security/2026/05/vdefend-vcf-9-1-zero-trust.html)). IDS/IPS is signature and behavior based, not a DLP | Add CNI-level policy. Add gateway egress allowlists. Treat exfiltration as a layered control. \[V\] on 9.0 pod coverage. |
| 9 | (Omission) DFW enforced on all AI traffic | SR-IOV and DirectPath I/O NICs bypass the vSwitch, so DFW does not see them \[V\] | Physical fabric ACL or VRF for any bypass fabric. Isolate training fabric by tier. |
| 10 | 100% DPDK polling utilization means perfect throughput if no queuing | Wrong inference. Poll-mode drivers spin at high CPU by design, so host CPU is uninformative in both directions. Avi reports actual utilization separately ([Avi doc](https://docs.vmware.com/en/VMware-Avi-Load-Balancer/30.2/Monitoring-Operability-Guide/GUID-36DC0333-DC2E-4DC3-9648-1CD8575A1AB4.html), [KB 438582](https://knowledge.broadcom.com/external/article/438582/virtual-machine-cpu-usage-alarm-continuo.html)) | Use Avi per-SE and per-se\_dp CPU, drops, connection and memory metrics. Host CPU proves nothing. |
| 11 | Avi WAF inspects L7 traffic for adversarial payloads | WAF rules match web attack signatures. Prompt injection and jailbreaks are semantic and pass WAF | WAF covers OWASP-class attacks. Add an LLM guardrail layer in the application path. |
| 12 | MIG ensures failure in one AI system never affects another | MIG-backed vGPU gives spatial partitioning (vGPU 19+ on vSphere 9, [NVIDIA](https://docs.nvidia.com/vgpu/deployment/vmware/latest/mig.html)). Time-sliced vGPU gives none. Host, driver and PCIe faults remain a shared domain \[V\] | Use MIG-backed profiles for isolation. Treat the physical GPU and host as one fault domain. |
| 13 | VM snapshots provide stateful rollback of models | Snapshot creation for VMs with vGPU is reported unsupported ([ISPsystem](https://www.ispsystem.com/docs/vmmanager-kb/errors/6120)) \[V on vSphere\]. VKS nodes are replaced, not snapshotted. Rollback also reverts audit logs | Roll back by traffic shift, GitOps revert, and digest-pinned model versions in Model Store (section 13). |
| 14 | VCF Operations alerts on deviation from behavioral baselines enforce AI oversight | Dynamic thresholds model infrastructure metrics. They cannot detect data drift, concept drift or fairness shifts required by 5.23(a). PCIe bandwidth is not a native VCF Operations metric \[V\] | Add a model observability stack on VKS. Feed its metrics to VCF Operations. |
| 15 | VKS provisions isolated execution environments. Node pools segregate by risk | Containers share a kernel. Node pools share a control plane | High tier: dedicated VKS clusters. Medium: dedicated node pools with taints. |
| 16 | Disjointed NSX overlay networks make an air-gapped test environment | Overlay separation is not an air gap. Routing, edge, DNS, registries and artifact flows remain | Isolated workload domain or VPC with no external route, one-way artifact intake, masked data. |
| 17 | Declarative infrastructure fulfils reproducibility (5.17). vLCM fulfils AI change control (5.25b) | Both cover the environment only. Reproducibility also needs data versions, code, seeds, hyperparameters. vLCM covers hosts, firmware and drivers, not model change | Mark both Partial. Pair with MLOps registry and GitOps. |

## 3. MAS timeline, applicability and proportionality

The Guidelines were issued 7 October 2026. Sections 3-4 can be met from 7 October 2027. Sections 5-6 by 7 October 2028 (clause 1.8). From today, 21 October 2026, that is about 11.5 and 23.5 months.

| Clause | Rule | Design consequence |
| --- | --- | --- |
| 1.2 | Applies on a group basis to locally incorporated FIs under consolidated supervision, or owners of critical information infrastructure | Confirm scope per legal entity before sizing the programme |
| 1.3, fn 6 | AI includes ML, deep learning, NLP, vision, GenAI, LLMs, AI agents. Rule-based logic and fixed-instruction RPA are excluded | Inventory scope includes embedded AI in vendor products, including VCF's own AI assistants |
| 1.5 | Group frameworks accepted if they meet the Guidelines | Group VCF standards can be reused. Local senior management stays accountable (3.6) |
| 1.6, 2.1 | Controls proportionate to size, nature and risk | Tiered control profiles, not one profile for all AI |
| 2.3-2.5 | Basic governance allowed where failure has no material impact | Tier 0 profile. Approved tool list, owner, permitted uses, compliance checks |
| 2.2(b), fn 12 | Lifecycle controls calibrated by relevance. Data management, safety and cybersecurity still apply to low-materiality AI | Even Tier 0 keeps encryption, segmentation and logging |
| 5.2 | Where a third party discloses too little, mitigate or limit use | Hosting open-weight models on VKS is a mitigation option (section 9) |

**Implication for sequencing**

- Sections 3-4 are governance, identification, inventory, materiality. First deadline. VCF contributes the inventory feed and discovery signals.
- Sections 5-6 are lifecycle and capacity. Second deadline. This is where VCF design carries most weight.
- Infrastructure built for Tier 2 in 2027 avoids rework in 2028.

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

## 5. Oversight, identification, inventory, materiality (MAS 3.x, 4.x)

VCF cannot assess risk materiality or set risk appetite. It supplies the inventory feed, discovery signals, and the evidence that tier controls operate.

**5.1 Oversight reporting (3.4, 3.5, fn 18)**

Footnote 18 lists quantitative risk appetite measures. Each maps to a VCF Operations super metric or report over tag-based custom groups.

| MAS appetite measure | VCF source | Implementation |
| --- | --- | --- |
| Number of AI incidents | VCF Operations alert history, ITSM tickets with AI category | Custom group by tag ai.use-case-id. Count of critical alerts per period. Scheduled report |
| Material use cases with single-provider dependency | Inventory attribute plus VCF Operations relationships | Count of Tier 1-2 groups sharing one model, one GPU cluster, one Model Store |
| Use cases in breach of performance thresholds | Symptom definitions on SLO metrics | Super metric: count of Tier 1-2 objects with active critical symptom |
| Control effectiveness | Compliance benchmark results, config drift | Percent of Tier 2 hosts and clusters compliant. Report to board committee |

**5.2 AI identification (4.2-4.4)**

No single control finds all AI. Layer four mechanisms and record residual risk (4.4).

| Layer | Mechanism | VCF component | Limit |
| --- | --- | --- | --- |
| Declared | Mandatory intake fields on every AI template and namespace request | VCF Automation templates and approval policies | Only catches workloads that use the catalog |
| Platform-observed | GPU-attached VMs, VKS clusters with GPU operator, Model Store entries, containers with ML frameworks | VCF Operations, PAIS GPU Monitoring, Harbor scan metadata | Cannot see AI running outside VCF |
| Network-observed | Flows to known AI SaaS FQDNs. Egress allow and deny by FQDN | NSX DFW and Gateway Firewall, vDefend flow analytics. URL category filtering needs the correct vDefend tier \[V\] | Encrypted payload not inspected without decryption |
| Embedded third-party AI | Vendor release notes, feature toggles | Includes Broadcom's own AI assistants. vDefend and Avi both now ship an AI Assistant ([StorageReview](https://storagereview.com/news/vmware-vdefend-75tbps-distributed-firewall-vcf-9-1-update)) | Register each. Document enablement decision \[V on toggles\] |

Residual: AI on endpoints and SaaS outside the private cloud is not visible to VCF. Use endpoint DLP, CASB and policy (fn 22).

**5.3 AI inventory (4.5-4.9)**

System of record: the FI's GRC or CMDB, owned by the designated control function (4.9). VCF is a feed and a reconciler.

| MAS 4.7 attribute | Authoritative source | VCF contribution |
| --- | --- | --- |
| Purpose, description | GRC | Link in tag or annotation |
| Approved scope (jurisdiction) | GRC | Tag ai.jurisdiction. Drives DRS rules and replication limits |
| Model type | Model Store / MLOps registry | Tag ai.model-type. Harbor OCI metadata |
| Data used | Data catalog | Tag ai.data-class. Storage policy binding |
| Dependencies | GRC plus platform | VCF Operations relationships. NSX group membership and flow data |
| Lifecycle status | GRC | Tag ai.lifecycle. VCF Automation lease or expiry |
| Risk materiality rating | Control function | Tag ai.risk-tier. Drives every policy |
| Model review status | Validation team | Tag ai.review-status. Deployment gate |
| Owners, developers | GRC | Tag ai.owner. Project membership |
| Agent attributes (fn 25): identity, tools accessed, guardrails | Agent registry | Service account name, NSX policy export, K8s RBAC export, MCP server list |

**Tag design**

- Categories with single cardinality for tier, lifecycle, review status. Multiple for model type.
- Tag assignment privilege restricted to the control function and automation service accounts.
- Tags drive NSX dynamic groups and DRS rules. A tag change is a security change. Audit it.
- Reconciliation job in VCF Operations: GPU-attached VM or GPU-operator cluster without ai.use-case-id raises an alert. Output feeds the control function weekly.
- Export to CMDB through the VCF Operations or VCF Automation APIs \[V on integration path\].

**5.4 Risk materiality (4.10-4.13)**

- Impact, complexity, reliance are business judgments. VCF holds none of them.
- Reliance (4.12c) includes autonomy granted. For agents, record the tool permissions as infrastructure evidence: NSX egress policy and K8s RBAC.
- Inherent versus residual risk (4.11): residual depends on proof that controls operate. Compliance and drift reports from VCF Operations are that proof.
- Tier change must trigger re-deployment of the control profile. Automate by tag change to policy change.

## 6. Lifecycle controls, contingency and kill switch (MAS 5.1-5.3)

Clause 5.3 requires contingency plans for high-risk AI. Where kill switches exist, activation protocols must be tested regularly. VCF can implement four escalating stop levels. Each is scriptable and leaves evidence.

**6.1 Kill-switch ladder**

| Level | Action | Mechanism | Preserves state | Evidence |
| --- | --- | --- | --- | --- |
| L1 Divert | Stop new requests. Route to fallback or manual process | Avi: disable virtual service or pool member, or switch to a fallback pool or static response | Yes | Avi event log, config audit |
| L2 Quarantine | Cut all network access, keep VM or pods for forensics | Apply NSX tag ai.quarantine to workload. A dynamic group matches it. A DFW rule in the Emergency category drops all traffic for that group | Yes | NSX audit log, DFW rule hit counters |
| L3 Stop | Halt execution | K8s scale to zero via GitOps or automation. VM power-off via VCF Automation day-2 action. Revoke agent tokens and secrets in IdP or vault | Disk yes, memory no | Automation run log, IdP revocation log |
| L4 Retire | Decommission and erase | Section 13 decommission workflow. Section 7 erasure | No | Decommission record |

**Design rules**

- Authority to apply ai.quarantine and run L1-L3 is a named role. It maps to MAS 5.9(b): authority and ability to intervene.
- The quarantine rule sits above all application rules. The DFW Emergency category is evaluated first.
- Trigger sources: VCF Operations alert webhook, ITSM approval, manual run. Tier 2 automated triggers require a human approval step by policy.
- Test on a schedule set by the FI. Capture the run log as evidence. Include a test that an agent cannot reach tools after L2.
- Agent credentials must be revocable independent of infrastructure. Do not embed long-lived keys in images.

**6.2 Fallback and continuity (5.3)**

| Failure | Mitigation | Component | Note |
| --- | --- | --- | --- |
| Host or GPU failure | vSphere HA restart. Keep spare GPU capacity | vSphere HA, DRS | vGPU restart needs a host with a compatible free GPU. Passthrough VM restart behavior \[V\] |
| Zone failure | Multi-zone Supervisor and VKS cluster placement. Replicated model artifacts | VKS, Harbor replication | Verify zone-aware GPU capacity |
| Site failure | DR for stateful components: vector DB, datasets, Model Store | VCF DR tooling \[V on product name in 9.x\], vSAN | Replicate keys or key provider as well |
| Model misbehavior | Revert to last good model digest. Fall back to smaller model or rules engine | Model Runtime, Model Store | The rule-based fallback must be listed in the use-case record |
| Third-party API outage | Local model as fallback. Egress policy supports both | Gateway Firewall, Avi | Concentration control (5.11c) |

**6.3 Partial deployments (fn 28)**

- Pilots need time limits, user limits and success criteria.
- Implement time limits as ai.expiry tag plus automation lease or expiry \[V on lease feature in VCF Automation 9\].
- Cap users with namespace quotas and Avi rate limits.
- Monitor pilot usage patterns with the same alerts as production. Tier is assigned by use case, not by pilot status.

**6.4 Roles across lifecycle (5.1)**

- First line: developers and operators. Roles in VCF Automation projects and vCenter.
- Second line: validation and risk. Read-only roles on production config, full roles in the test domain.
- Third line: internal audit. Read-only everywhere, access to VCF Operations compliance and log history.
- Separation of duties (5.16b): different SSO groups for train and test. Section 11.

## 7. Data management, encryption and erasure (MAS 5.4)

VCF controls clause 5.4(d), (e) and part of (f). Clauses 5.4(a), (b), (c) and (g) need data-governance tooling.

**7.1 Clause 5.4(a)-(g) mapping**

| Clause | VCF control | Configuration | Evidence | Gap |
| --- | --- | --- | --- | --- |
| (a) Fit for purpose | None | Data catalog and use-case review | Review record | Out of VCF scope |
| (b) Representativeness | None | Test data sets in the test domain | Evaluation report | Out of VCF scope |
| (c) Quality, drift | Compute and storage for quality pipelines. PAIS data indexing and retrieval connectors | Pipelines on VKS | Pipeline run logs | Quality metrics need a data-quality tool |
| (d) Classification | Tag ai.data-class drives storage policy, NSX group and key provider | One vSAN storage policy per class. Datastore or share per class | Policy compliance report | Classification itself is a data-owner act |
| (e) Security | Encryption at rest and in transit. Key management. Destruction | Sections 7.2-7.4 | Encryption status export, KMS audit log | Defined by FI data destruction standard |
| (f) Privacy | Residency and access logging | ai.jurisdiction tag with DRS VM-Host rules and no cross-region replication. vCenter and NSX audit logs | Placement report, access logs | Consent management is out of scope |
| (g) Lineage | Artifact provenance in Model Store. Pipeline code in Git | Harbor OCI metadata, signed images | Registry audit log | Data lineage needs a catalog tool |

**7.2 Encryption layers**

| Layer | Control | Scope | Protects against | Does not protect against |
| --- | --- | --- | --- | --- |
| vSAN data-at-rest | AES-256, ESA encrypts after compression ([VMware](https://www.vmware.com/docs/vmw-vsan-encryption-services)) | Cluster | Removed or stolen devices | Authorized vSphere or guest access |
| vSAN data-in-transit | Cluster-level. Remote datastore support added in VCF 9.1 | Cluster and mounts | Network sniffing between hosts | Compromised guest |
| VM Encryption | Storage policy per VM, key from key provider ([Entrust guide](https://nshielddocs.entrust.com/interops-docs/vmware-vsphere-kc/procedures.html)) | Per VM and disk | Datastore-level access, backup copies | Guest-level access |
| vMotion encryption | Per VM setting | In flight | Network capture | None |
| Application TLS | Avi termination, mTLS inside mesh | API paths | Eavesdropping | Endpoint compromise |

- Enabling vSAN encryption on an existing cluster triggers a rolling reformat of all disk groups. Plan capacity headroom and a maintenance window ([vSAN admin guide](https://docs.vmware.com/en/VMware-vSphere/8.0/vsan-administration/GUID-E7CA36B7-D7EB-423A-ADD1-7E410E36F5A7.html)).
- The docs list a Wipe residual data option for devices that hold prior data. Use it when re-purposing hardware.
- VM Encryption on vGPU and passthrough VMs: confirm support and vMotion behavior for the exact vSphere, driver and GPU combination \[V\].
- Audit check: the PowerCLI test `Get-Cluster | Get-VsanClusterConfiguration | Select Name,EncryptionEnabled` is in the DISA STIG for vSphere 8. Run it as a scheduled compliance job.

**7.3 Key architecture**

- External KMS with HSM-backed root for Tier 1-2. Native Key Provider for Tier 0 only, if the FI accepts it. Confirm HSM options for NKP \[V\].
- One key provider per tier. Tier 2 uses a key provider per use case or business unit so a single key destruction is scoped.
- Key backup, dual control, rotation schedule, and break-glass access documented in the FI key management standard.
- KMS availability is a dependency of VM boot and vSAN. Treat the KMS as Tier 2 infrastructure. Size its HA and DR.
- Key provider audit logs go to VCF Operations for logs and SIEM.

**7.4 Destruction and sanitisation (5.4e, 5.26)**

Clause 5.4(e) asks for timely destruction of training data, model artefacts and outputs once decommissioned. Cryptographic erase works only where the key scope equals the data scope.

| Scenario | Method | Scope check | Evidence |
| --- | --- | --- | --- |
| One model or VM retired | Delete VM. If VM Encryption was used with a use-case key, destroy that key after all copies are gone | Key must protect only that use case | Key destruction log, deletion record |
| Cluster retired | Destroy cluster key at KMS. Reformat devices. Vendor secure erase or physical destruction for media leaving the site | Cluster key protects the entire datastore | Destruction certificate |
| Kubernetes volumes | Delete PVC with reclaim policy Delete. Storage class carries encryption policy. Confirm vSAN object removal | PVs removed and no retained volumes | Volume list diff |
| Model artifacts | Delete OCI artifact in Harbor, then run Harbor garbage collection to free blobs. Purge replicas and mirrored registries | Registry replication targets included | GC job log |
| Backups, snapshots, DR replicas | Enumerate every copy. Expire. Destroy backup keys | Copy inventory complete | Copy inventory and expiry log |
| Prompts, responses, logs | Retention policy in log platform. Expiry job | Classification equals source data | Retention config export |

- State in the FI data destruction standard whether cryptographic erase is accepted as sanitisation. MAS requires timely destruction or sanitisation, not a method.
- Do not describe cluster key deletion as dataset erasure. It is not granular.

**7.5 vSAN File Services for datasets**

- One file share per data class and tenant. NFS exports limited to the node pool IPs of authorized VKS clusters.
- Kerberos for NFS 4.1 or SMB with AD where identity is needed. Identity is still node-level for CSI mounts.
- Container-level control comes from namespace RBAC, PV binding rules and NetworkPolicy.
- Read-only mounts for training pipelines that must not modify source data.
- Snapshots of datasets are version evidence for 5.4(g) and 5.17.

## 8. Transparency, fairness and human oversight (MAS 5.5-5.9)

Clauses 5.5-5.8 are model and process controls. VCF contributes hosting, isolation and records. Clause 5.9 has real infrastructure content: authority to intervene and oversight records.

**8.1 Transparency, explainability, fairness (5.5-5.8)**

| Clause | What MAS asks | What VCF provides | What VCF does not provide |
| --- | --- | --- | --- |
| 5.5-5.6 | Transparency and explainability matched to materiality | Hosting for explainability tooling on VKS. GPU capacity for post-hoc methods. Versioned model artifacts and configuration for documentation | Explanations, feature attribution, customer disclosure content |
| 5.7-5.8 | Define fair outcomes. Run fairness assessments on protected attributes | Isolated environment and GPU quota for fairness runs. Separate datastore or share for protected-attribute data with its own key provider and RBAC, so training pipelines cannot read it | Fairness metrics, thresholds, mitigation |

- Fairness testing needs protected attributes (5.8, fn 36). Store them apart from training features. Grant access only to the validation role.
- Log model version with every response. Without it, an explanation or redress request (5.6) cannot be tied to the model that produced the output.
- Prompt and response logging at the Model Runtime or gateway is the raw material for later explainability and redress. See section 12.

**8.2 Human oversight (5.9a-d)**

| Clause | Requirement | VCF implementation | Evidence |
| --- | --- | --- | --- |
| 5.9(a) Roles | Assign roles for oversight, escalation, decisions | vCenter and NSX roles. VCF Automation project roles. K8s RBAC. Mapped to named AI oversight group in the IdP | Role export, group membership report |
| 5.9(b) Capabilities | Authority and ability to intervene | Privilege to run kill-switch levels (section 6) bound to oversight group. Break-glass path documented | Privilege report, drill records |
| 5.9(c) Design | Mechanisms to escalate outputs for human review | Infrastructure side: approval gates in VCF Automation. VCF Operations alert actions set to recommend, not auto-execute, for Tier 2. Agent side: tool-call approval in the agent framework. Network side: unapproved tool endpoints unreachable by DFW and Gateway Firewall policy | Approval logs, policy export |
| 5.9(d) Documentation | Logs of development, deployment, operation, monitoring, incidents. Review of interventions and near misses | vCenter events, NSX and VCF Automation audit, K8s audit, Avi logs to VCF Operations for logs and SIEM. Alert history retains near misses | Retention config, review minutes |

**Automation bias and decision fatigue (5.9)**

- Use tiered alert severities. Keep Tier 2 alert volume low enough for a person to review each one.
- Route informational symptoms to dashboards, not tickets.
- Report overrides and near misses monthly. A zero count is a warning sign, not a success.

## 9. Third-party AI and selection (MAS 5.10-5.13)

Hosting open-weight models on VKS reduces SaaS exposure. It does not remove third-party risk. The FI keeps primary accountability (5.11). Weights, container images, and the VCF stack itself are third-party components.

**9.1 Clause 5.11(a)-(h) mapping**

| Clause | Risk | VCF lever | Residual |
| --- | --- | --- | --- |
| (a) Transparency | Little disclosure of training data and controls | Open-weight hosting gives access to weights, config and license. Model Store uses Harbor, so security teams can scan and control access ([source](https://mind-verse.de/news/vmware-cloud-foundation-ki-anwendungen-private-clouds)) | Training data provenance still opaque. Needs independent assessment or compensating tests |
| (b) Supply chain | Poisoned weights, unsafe serialization, vulnerable dependencies | Harbor vulnerability scanning, image signing, digest pinning, mirrored artifacts for air-gapped intake (Artifact Mirroring Tool, [William Lam](https://williamlam.com/2026/09/vcf-9-1-1-deploying-your-own-ai-models-with-vcf-private-ai-services-pais-3-0.html)) | CVE scanners do not detect malicious pickle files. Prefer safetensors. Add model-file scanning \[V on tool choice\] |
| (c) Concentration | Reliance on a few providers | Internal hosting removes SaaS provider dependency. Multi-accelerator Model Runtime supports AMD and NVIDIA GPUs without app rewrite ([source](https://mind-verse.de/news/vmware-cloud-foundation-ki-anwendungen-private-clouds)) | New concentration on Broadcom and NVIDIA stack. Record in register. Document exit plan |
| (d) Change management | Silent updates | Pin model digests. Block mutable tags in Model Runtime config. Track VCF release notes as vendor change feed | Vendor-side behavior change in managed SaaS APIs cannot be pinned |
| (e) Contingency | Provider failure | Local fallback model. Egress policy supports a switch between local and external | Fallback quality needs testing |
| (f) Legal | Audit rights, notifications | Outside VCF | Include Broadcom contract clauses on AI feature introduction and notification |
| (g) Capabilities | Staff skills | Outside VCF | Training plan (section 14.6) |
| (h) Complexity | New AI types | Test domain for compensatory testing | Agents from third parties need deeper tests |

**9.2 VCF as a third party**

- Under MAS outsourcing and technology-risk expectations (fn 39), Broadcom is a material provider for the platform. Apply the FI's third-party process.
- Register embedded AI features in VCF Operations, Avi and vDefend as AI under 4.2 when enabled.
- Record the data those features see. Document the enablement decision.

**9.3 Controlling external AI APIs (5.10, 5.16c)**

- Default deny outbound from AI tiers. Allow named FQDNs through Gateway Firewall or DFW FQDN rules.
- Route approved calls through an egress proxy or Avi in forward mode \[V\] so requests and responses can be logged and DLP-scanned.
- Mask or tokenise customer data before it leaves. Confidential and client data in public AI tools is a prohibited use under 2.5(b).
- Test the third-party model with FI data in the test domain before onboarding (5.10).

**9.4 Selection (5.12-5.13)**

- Infrastructure constrains model choice: GPU memory, interconnect, latency budget. Record these in the selection document.
- Benchmark candidate models, quantization levels and vGPU profiles on production-identical hosts. Record tokens per second and p95 latency per profile.
- Prefer simpler models where performance is equal (5.12, 5.14c). The infrastructure cost comparison supports that justification.
- Selection records live in the model registry entry. Link from the inventory.
- For newer, less understood models (5.13), require a higher test depth and a lower autonomy ceiling in the use-case record.

## 10. Evaluation, testing and pre-deployment review (MAS 5.14-5.15, 5.18-5.22)

Model evaluation is not a VCF function. VCF provides the isolated test environment, the infrastructure test methods, the separation of duties and the configuration evidence that reviewers need.

**10.1 Test and adversarial domain**

| Property | Design |
| --- | --- |
| Isolation | Dedicated workload domain or NSX VPC. No route to production or internet. Own DNS, NTP, registry |
| Intake | One-way artifact transfer through Harbor replication or mirroring tool. Scan on entry. Quarantine until approved |
| Data | Masked or synthetic copies. Production data in test inherits production controls, including encryption tier and retention (5.4) |
| Tooling on VKS | Evaluation harnesses, adversarial test tools, IMDA LLM testing starter kit referenced in MAS fn 43, AI Verify tooling referenced in fn 3 |
| Capacity | Own GPU quota so tests do not starve production |
| Egress | None. Results exit by reviewed export |

**10.2 Infrastructure test methods (5.14b)**

- Load and stress: concurrent requests, long contexts, streaming sessions, until GPU memory or queue limits. Record the knee point per vGPU profile.
- Failure injection: host failure, GPU host driver fault, vSAN device failure, NSX DFW drop of a dependency, KMS unreachable, Harbor unavailable. Observe fallback and alert behavior.
- Edge and out-of-time conditions are model tests. Infrastructure supplies repeatable environments for them.
- Threshold definition (5.14a): latency p95 and p99, tokens per second, error rate, queue depth, GPU memory headroom. Business owners, developers and reviewers sign off.
- Test on production-identical hosts: same BIOS, driver, vGPU profile, NUMA layout. Results from different hardware do not transfer.

**10.3 Pre-deployment review (5.18-5.21)**

| Requirement | Implementation | Evidence |
| --- | --- | --- |
| Parties not involved in development | Separate SSO groups and vCenter roles for developers, operators, validators. Validators have read-only production roles | Role export |
| High-risk: formal independent validation (5.19) | Validation team receives the evidence pack. Gate in VCF Automation or Git approvals blocks promotion until sign-off recorded | Approval record |
| Other use cases: documented peer review (5.20) | Pull request approval by a qualified peer who is not an operator | Git history |
| Findings reported to approval body (5.21) | Findings tracked in ITSM. Deployment gate checks closure | Ticket links |

**Evidence pack contents (automated export)**

- vSphere configuration and drift report for the cluster. VCF Operations compliance result.
- NSX policy export for the use case groups. DFW rule list. IDS/IPS profile.
- vSAN encryption status and key provider mapping.
- Avi virtual service config: TLS, WAF policy, rate limits, auth.
- VKS cluster spec, Pod Security settings, image digests, RBAC export.
- Load and failure test results with thresholds.

**10.4 Technology and cybersecurity review (5.22)**

| Clause | Check | VCF control used |
| --- | --- | --- |
| 5.22(a) Secure design, access control, deployment checklist | Encryption, DLP, firewalls, access restrictions, logging operational before go-live | Automated checks: vSAN encryption status, DFW default-deny present, log forwarding verified, RBAC export, Avi TLS config |
| 5.22(b) Vulnerability assessment, penetration test, red team | Run against the full stack in the test domain | vDefend IDS/IPS in detect mode during the exercise to measure detection. Harbor scan results. Avi WAF tested with OWASP-class attacks |
| 5.22(b) Adversarial threats: manipulation, evasion, poisoning | Model-level tests | Not VCF. Prompt injection and jailbreak tests need an LLM test suite |
| 5.22(b) Safeguards: input validation, throttling, anomaly detection | Throttling and anomaly detection | Avi rate limiting and WAF. Application-level guardrails for semantic input validation |

**10.5 Gap**

- Evaluation metrics, overfitting controls (5.14c), explainability and fairness assessments sit outside VCF.
- Include them in the validation pack, produced on VKS in the test domain.

## 11. Technology and cybersecurity (MAS 5.16, 5.22)

This is VCF's strongest area. Clause 5.16 lists security, access control and third-party controls. The design below applies to Tier 1-2 and scales down for Tier 0.

**11.1 Platform hardening (5.16a)**

- ESXi: lockdown mode, secure boot, TPM 2.0 attestation, execInstalledOnly, host config encryption. Follow the vSphere Security Configuration Guide and the DISA STIG.
- vCenter: TLS, no local accounts except break-glass, event retention, restricted API access.
- Run a scheduled compliance benchmark in VCF Operations. Benchmark list in 9.x: confirm which standards ship \[V\].
- vLCM desired-state image with a compliance check. Drift raises an alert.
- NSX Manager, Avi Controller, VCF Operations, VCF Automation: management plane on a dedicated segment, admin access through jump host with MFA.

**11.2 Identity and access (5.16b)**

| Control | Implementation | Note |
| --- | --- | --- |
| Federated login with MFA | vCenter Identity Federation to the corporate IdP (OIDC or SAML). MFA enforced at the IdP. VCF 9 identity broker for fleet SSO \[V\] | No local admin for daily use |
| Least privilege | Custom vSphere roles per function: developer, operator, validator, auditor. NSX roles by tenant | Tag assignment privilege isolated (section 5.3) |
| Privileged access management | Break-glass accounts vaulted. Session recording through PAM jump host | PAM product is outside VCF |
| Separation of duties (5.16b) | Train and test teams in separate SSO groups, separate folders and resource pools, separate namespaces | Validators cannot modify production |
| K8s access | OIDC to the same IdP. RBAC by namespace. No shared kubeconfig | Review quarterly |
| Agent and service identities | One service account per agent. Short-lived tokens. Secrets from a vault via CSI. Tool access by NSX group and RBAC | Revocation is part of the kill switch |

**11.3 NSX and vDefend segmentation (5.16a)**

**Policy model**

- Groups by tag: ai.tier and ai.role (gateway, model-runtime, vector-store, trainer, agent, tool-server).
- Default deny for all Tier 1-2 groups in both directions.
- Allow rules by role pair. Example: gateway to model-runtime on the model API port only. Model-runtime to vector-store on its port. Agent to named tool-servers only. No direct client to model-runtime.
- Egress: none by default. Named FQDN allow for approved external services.
- Quarantine rule in the Emergency category (section 6).
- VPC-aware policy and delegated self-service are available in VCF 9 ([vDefend for VCF 9](https://blogs.vmware.com/security/2025/06/announcing-vdefend-for-vcf-9.html)). VCF Automation 9.1 delegates vDefend firewall services to organization admins.

**Layered controls**

| Layer | Control | Detail | Limit |
| --- | --- | --- | --- |
| VM vNIC | DFW | Enforced at the vNIC of VMs on NSX segments or VPC subnets | Not enforced on SR-IOV or DirectPath I/O NICs \[V\] |
| Pod level | CNI policy | Antrea network policy for pod-to-pod traffic. vDefend 9.1 adds pod-level IDS/IPS via CNI integration ([source](https://blogs.vmware.com/security/2026/05/vdefend-vcf-9-1-zero-trust.html)) | 9.0 pod coverage \[V\] |
| Lateral threat detection | Distributed IDS/IPS | Detect mode first, then prevent for tuned signatures. Custom signatures supported ([source](https://blogs.vmware.com/security/2024/10/vmware-vdefend-latest-enhancements-in-advanced-threat-prevention.html)). Turbo Mode raises throughput from 3 to 9 Gbps per host in 9.1 | Do not inspect bulk training traffic. Scope to inference and control planes |
| Advanced detection | Advanced Threat Prevention: network traffic analysis, NDR, sandbox, malware prevention | On-prem malware prevention for air-gapped sites ([source](https://blogs.vmware.com/security/2024/08/vmware-vdefend-innovations-elevating-enterprise-lateral-threat-defense-for-genai-era.html)) | Add-on licensing. Encrypted traffic coverage limited \[V\] |
| North-south | Gateway Firewall | FQDN and geo-IP filtering for egress. Ingress allow lists | Policy applies on the NSX T0, T1 or VPC gateway only |

**Bypass fabrics**

- Training clusters often use RoCE with SR-IOV or passthrough NICs. DFW cannot see that traffic \[V\].
- Compensate: dedicated fabric per tier, VRF or ACL on physical switches, no shared fabric between tiers, and no external route from the training fabric.
- Document the control in the architecture record as a compensating control.

**11.4 Avi Load Balancer for inference APIs (5.16a, 5.22b)**

| Function | Configuration |
| --- | --- |
| TLS | Termination with managed certificates. Re-encrypt to backend for Tier 2 |
| API authentication | Client certificate, JWT or OIDC validation on the virtual service \[V on exact profile names\] |
| WAF | Detection mode first, then enforcement. OWASP-class signatures. Does not stop prompt injection |
| Rate limiting and DoS | Per-client and per-service limits. Supports throttling (5.22b) |
| LLM streaming | Long idle timeouts. Disable response buffering for streamed tokens \[V\] |
| Logging | Significant and analytics logs to VCF Operations for logs and SIEM. Do not log full prompt bodies at the load balancer unless the data class permits |
| Isolation | SE group per tier. Dedicated SEs for Tier 2 |
| CPU | Avi docs state CPU is not reserved by default on VMware deployments. Reserve SE CPU for Tier 2 ([Avi docs](https://docs.vmware.com/en/VMware-Avi-Load-Balancer/30.2/Monitoring-Operability-Guide/GUID-9909AF52-4E48-4873-9673-0BCD78D96D5A.html)) |

**Avi monitoring rules**

- Data path uses DPDK poll-mode drivers. Hypervisor and Linux CPU reads high by design ([Avi docs](https://docs.vmware.com/en/VMware-Avi-Load-Balancer/30.2/Monitoring-Operability-Guide/GUID-36DC0333-DC2E-4DC3-9648-1CD8575A1AB4.html)). Do not alert on vCenter VM CPU for SE VMs. Broadcom documents disabling that alarm for SE VMs ([KB 438582](https://knowledge.broadcom.com/external/article/438582/virtual-machine-cpu-usage-alarm-continuo.html)).
- Use Avi-reported CPU: UI SE metrics or `show serviceengine <name> cpu`. The UI shows the average across se\_dp processes.
- Track dispatcher versus proxy core load separately. Dedicated dispatcher is configurable.
- Alert on memory above 90% sustained, connection limits, packet or connection drops, and latency. The docs call out the 90% memory guideline.
- Host CPU near 100% neither proves loss nor proves health. Do not use it either way.

**11.5 VKS hardening (5.16a, 5.22a)**

- Dedicated clusters for Tier 2. Node pools with taints for Tier 1.
- Pod Security Admission at restricted level. Add an admission policy engine for image source, signatures, resource limits, no privileged pods, no host networking \[V on package availability\].
- Images only from the internal Harbor. Digest pins. Scan on push and on schedule.
- Network policy default deny per namespace.
- Secrets from a vault. No secrets in images or ConfigMaps.
- GPU operator and driver versions pinned and change-controlled.
- K8s audit logs forwarded off-cluster.
- Treat MCP servers and tool plugins as third-party components (5.16c). Separate NSX group, named allow list, versioned, logged.

**11.6 Data leakage controls (5.16a)**

- Egress allow lists and Gateway Firewall are the network-level control.
- Content-aware DLP is not a native VCF function. Use an egress proxy or application-level filters for prompt and response content.
- Output filtering and PII redaction sit in the application or guardrail layer.

**11.7 GPU and tenant isolation**

- MIG-backed vGPU gives hardware-level partitioning for compute and memory ([NVIDIA](https://docs.nvidia.com/vgpu/deployment/vmware/latest/mig.html)). Time-sliced vGPU does not.
- Never share one physical GPU across tiers or across data classes where leakage risk is not accepted.
- Confirm framebuffer clearing between assignments for the GPU and driver in use \[V\].
- Evaluate confidential computing support for the GPU and vSphere version before relying on it \[V\].

## 12. Reproducibility, monitoring and re-validation (MAS 5.17, 5.23, 5.24)

VCF Operations covers infrastructure and platform signals. Clause 5.23(a) also requires drift, stability, data quality and fairness monitoring. Those need a model observability layer. Both layers must feed one alerting and incident process.

**12.1 Monitoring stack by layer**

| Layer | Signals | Source | MAS clause | Native to VCF |
| --- | --- | --- | --- | --- |
| Host and VM | CPU ready, co-stop, memory ballooning and swap, NUMA locality, storage and network latency | VCF Operations | 5.23a | Yes |
| vSAN | Latency, throughput, resync, congestion, encryption state | VCF Operations | 5.23a | Yes |
| GPU | Utilization, memory, temperature, power, errors | PAIS GPU Monitoring ([source](https://www.storagereview.com/zh-CN/news/broadcom-positions-vcf-as-an-ai-native-platform-for-the-modern-private-cloud)). NVIDIA DCGM exporter for profiling counters \[V on PCIe counters in VCF Operations\] | 5.23a, 6.3 | Partial |
| Load balancer | SE CPU (Avi-reported), memory, drops, connections, latency | Avi, forwarded to VCF Operations | 5.23a | Yes |
| Kubernetes | Pod restarts, OOM kills, node pressure, queue depth, tokens per second | VKS packages and VCF Operations container monitoring \[V on exact pack\] | 5.23a | Yes |
| Model quality | Data drift, concept drift, stability, fairness, hallucination and toxicity rates | Model observability tool on VKS | 5.23a | No |
| Agent behavior | Reasoning steps, tool calls, actions taken | Agent traces (OpenTelemetry) to a trace backend | 5.23a last sentence | No |
| Prompts and responses | Content, model version, reasoning, timestamps | Model Runtime or gateway logging to log platform | 5.23d | Partial |

- Integration path for non-native metrics: Prometheus scrape into VCF Operations, the VCF Operations Suite API, or a management pack. Your VCF-vCommunity adapter work is one such path. State its supported metric scope in the design record \[V\].
- Do not describe VCF Operations baselines as drift detection. Infrastructure dynamic thresholds do not measure data or concept drift.

**12.2 Thresholds, tiers and incidents (5.23a-c)**

- Tiered thresholds (5.23a): use two severities. Warning is the early-warning level. Critical is the breach level. Static thresholds for SLOs. Dynamic thresholds only for infrastructure anomaly baselines.
- Notifications: webhook or ITSM integration into an AI incident category. The accountable person is named in the alert routing (5.23c).
- Tier 2 remediation: recommendation actions with approval, not automatic execution (5.9c).
- Incident path: alert, ticket, triage, kill-switch decision (section 6), resolution, post-incident review. Resolution options in 5.23b include retraining, rework or decommission.
- User feedback channel is an application feature. Route submissions to the same ITSM category.

**12.3 Documentation and records (5.23d)**

- Access to AI systems, models, training data, pipelines and config files must be strictly controlled and logged. Sources: vCenter events, NSX audit, VCF Automation audit, K8s audit, Harbor audit, KMS audit.
- Forward to VCF Operations for logs and the SIEM. Retain per FI policy. Use write-once storage for Tier 2 logs \[V on target\].
- Prompt and response logs carry the same data class as the underlying data. Apply the same encryption and retention.
- Include model version and, where applicable, reasoning traces in each record.

**12.4 Re-validation triggers (5.24)**

| Trigger in MAS 5.24 | Automated signal |
| --- | --- |
| Alerts or breaches from monitoring | Critical symptom on a Tier 2 object creates a review ticket |
| Significant change to the AI or its environment | vLCM remediation of GPU host driver, K8s version change, vGPU profile change, NSX policy change on the use-case group |
| Issues on interconnected systems | VCF Operations relationship query from the affected object to dependents |
| New external risks | Manual input from risk function |

**12.5 Reproducibility (5.17)**

VCF reproduces the environment. It does not reproduce training.

| 5.17 item | Captured by | VCF role |
| --- | --- | --- |
| Data sources, processing, quality checks | Data catalog and pipeline code | Compute, storage, dataset snapshots |
| Code versions, environments, hyperparameters | Git, MLOps tracker | VKS cluster spec in Git, container digests, vLCM image, GPU driver and vGPU profile, VCF Automation template version |
| Evaluation and test results | Validation pack | Test domain |
| Selection rationale, assumptions, limits | Registry entry | Link only |

- GPU kernels can be non-deterministic. Bitwise reproduction may fail even with identical environments. Document the tolerance. 5.17 asks for an independent party to understand and potentially replicate, not bit-exact output.
- Snapshot dataset versions on vSAN or use a versioned data tool. Record the snapshot ID in the registry.

## 13. Change management and decommissioning (MAS 5.25-5.26)

MAS 5.25 governs changes to the AI, including training data, parameters and configuration. Infrastructure changes matter when they alter AI behavior or performance. The framework below classifies both.

**13.1 Change significance (5.25a)**

| Change | AI significance | Required action |
| --- | --- | --- |
| Model weights, quantization, prompt template, guardrail config | Significant for Tier 1-2 | Re-validation, re-approval before release |
| GPU host driver or guest driver version | Can change numerics and performance | Regression test in test domain. Re-baseline latency and output checks |
| vGPU profile or MIG layout | Changes capacity and isolation | Re-test. Update inventory and tier evidence |
| K8s version or GPU operator upgrade | Platform change | Run test suite. Staged rollout |
| ESXi patch without driver change | Low | Standard change. Compliance re-scan |
| NSX policy change on use-case groups | Security change | Review by security. Evidence updated |
| Reservation or placement rule change | Performance change | Capacity and latency check |
| Model Store tag or digest change | Significant | Blocked unless approved. Digest pinning |

Where a significant change occurs without prior review on a high-materiality use case, apply compensating controls such as enhanced monitoring (5.25a).

**13.2 Change control mechanisms (5.25b)**

- Everything as code: VKS cluster specs, NSX policy, VCF Automation templates, Avi config, model deployment manifests in Git. Pull request approval with human reviewer.
- Cluster desired state through vLCM images and cluster configuration profiles \[V on name in 9.x\]. Drift alerts in VCF Operations.
- Change records link to the inventory entry and to the evidence pack.
- Dynamic updates (5.25c): no mutable tags such as latest. No automatic cluster upgrades for Tier 2. Auto-retraining pipelines may run only within defined parameters and must pass the gate before promotion. Autoscaling of replicas is not a model update. Autoscaling that pulls a new image is.

**13.3 Rollback hierarchy**

Fastest and safest first.

1. Traffic rollback: Avi pool switch, blue/green or weighted. No state change.
2. GitOps revert or K8s rollout undo of the deployment.
3. Model rollback: re-point Model Runtime to the previous digest in Model Store.
4. Infrastructure rollback: vLCM image revert and remediation. Slower. Test it.
5. VM snapshot revert only for non-GPU VMs such as vector database nodes. Snapshots of vGPU VMs are reported unsupported \[V on vSphere 9\]. Keep audit logs off the VM. A revert erases them.
6. Restore from backup for data stores.

- Snapshots are short-lived rollback points, not backups. Do not leave them on Tier 2 databases.
- VKS nodes are replaced by the cluster controller. They are not snapshotted.

**13.4 Decommissioning (5.26)**

| Step | VCF mechanism | Evidence |
| --- | --- | --- |
| Identify dependencies | VCF Operations relationships. NSX flow data | Dependency report |
| Notify stakeholders | ITSM change record | Ticket |
| Disable and drain | Kill-switch L1-L3 | Run log |
| Revoke access | Remove IdP groups, service accounts, tokens, vault secrets | Revocation log |
| Remove network policy | Delete use-case NSX groups and DFW rules. Check no orphan allow rules | Policy diff |
| Remove workload | VCF Automation destroy. Delete namespaces, PVCs, VMs | Destroy log |
| Erase data and artifacts | Section 7.4 | Destruction record |
| Retain required records | Logs, validation pack, decision record under retention policy | Retention config |
| Update inventory | Set ai.lifecycle to retired. Close GRC record | Inventory history |
| Return GPU | Free vGPU or passthrough device for reallocation. Confirm no residual data per vendor guidance \[V\] | Capacity record |

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

## 15. Traceability matrix

Coverage key: **Infra** = VCF can implement the infrastructure part fully. **Partial** = VCF supplies a feed, environment or evidence. Other tools needed. **Out** = not a VCF function.

| MAS clause | Requirement | vSphere | vSAN | NSX / vDefend / Avi | VCF Operations / Automation | VKS / PAIS | Coverage |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 3.4-3.6 | Board and senior management oversight | Roles | n/a | n/a | Appetite metrics, board reports (5.1) | n/a | Partial |
| 4.2-4.4 | AI identification | Tags, GPU VM discovery | n/a | FQDN egress control, flow analytics | Reconciliation alerts, catalog intake | Model Store, cluster discovery | Partial |
| 4.5-4.9 | AI inventory | Tag categories | Storage policy by data class | Group membership as dependency | Relationships, API export | Labels, registry metadata | Partial |
| 4.10-4.13 | Risk materiality assessment | n/a | n/a | n/a | Tier-driven templates and policy | n/a | Out (feeds tier) |
| 5.3 | Contingency, kill switch | HA, DRS, power actions | Replication | Avi divert, DFW quarantine | Day-2 actions, alert webhook | Scale to zero, rollout undo | Partial |
| 5.4(d) | Data classification | Tags | Policy by class | Groups by class | Reports | Storage classes | Infra |
| 5.4(e) | Data security, destruction | VM Encryption | At-rest, in-transit, File Services | TLS at Avi | KMS audit logs | CSI volumes, Harbor GC | Infra (scope caveats) |
| 5.4(f) | Privacy, residency | Placement rules | No cross-region replication | Egress policy | Audit logs | Namespace placement | Partial |
| 5.4(a-c, g) | Fit, representativeness, quality, lineage | n/a | Dataset snapshots | n/a | n/a | Pipelines, Model Store provenance | Out / Partial |
| 5.5-5.8 | Transparency, explainability, fairness | n/a | Separate protected-attribute store | n/a | n/a | Hosting only | Out |
| 5.9 | Human oversight | Roles, privileges | n/a | Tool endpoint reachability | Approval gates, recommend-only actions, logs | Namespace RBAC | Partial |
| 5.10-5.11 | Third-party AI | n/a | n/a | Egress allow list, Avi auth | Third-party register input | Open-weight hosting, Harbor scan, pinning | Partial |
| 5.12-5.13 | Selection | vGPU profile benchmarks | n/a | n/a | n/a | Model Runtime benchmarks | Out / Partial |
| 5.14-5.15 | Evaluation and testing | Test environment | Masked data copy | Isolation | Capacity | Test harness hosting | Partial |
| 5.16(a) | Security hardening, segmentation, encryption, DLP | Hardening, TPM, lockdown | Encryption | DFW, IDS/IPS, ATP, Gateway FW, Avi WAF, TLS | Compliance benchmarks | PSA, network policy, image policy | Infra (DLP content Out) |
| 5.16(b) | Access control, MFA, PAM, SoD | Identity federation, roles | n/a | NSX roles | Org roles | OIDC, RBAC | Infra (MFA at IdP) |
| 5.16(c) | Third-party components | n/a | n/a | FQDN allow list | n/a | MCP and plugin isolation | Infra |
| 5.17 | Reproducibility | vLCM image, driver versions | Dataset snapshots | Policy as code | Template versions | Cluster spec, digests | Partial |
| 5.18-5.21 | Pre-deployment review, independent validation | Separate roles | n/a | n/a | Approval gates, evidence export | Git approvals | Partial |
| 5.22 | Technology and cybersecurity review | Config compliance | Encryption check | Policy export, IDS/IPS test | Compliance report | Image scan, RBAC export | Infra |
| 5.23(a) | Monitoring measures, drift | Host metrics | vSAN metrics | Avi metrics | Infra thresholds, alerting | Platform metrics | Partial (drift Out) |
| 5.23(b-c) | Incident process, roles | n/a | n/a | n/a | Alert routing to ITSM | n/a | Partial |
| 5.23(d) | Logging and records | vCenter events | n/a | NSX, Avi logs | Log platform, retention | K8s audit, prompt logs | Infra / Partial |
| 5.23(e) | Training | n/a | n/a | n/a | n/a | n/a | Out |
| 5.24 | Re-validation | n/a | n/a | n/a | Trigger signals | n/a | Partial |
| 5.25 | Change management | vLCM, configuration profiles | n/a | Policy as code | Drift alerts, change records | GitOps, digest pins | Partial |
| 5.26 | Decommissioning | n/a | Erasure | Rule cleanup | Destroy workflows | PVC and registry cleanup | Partial |
| 6.1-6.2 | Capability and capacity of personnel | n/a | n/a | n/a | n/a | n/a | Out |
| 6.3 | Technology infrastructure adequacy | vGPU, NUMA, HA | ESA, Max | Edge, Avi sizing | Capacity management | VKS scale | Infra |

## 16. Gap register and verification list

**16.1 Capabilities outside VCF**

| Gap | MAS clause | Needed layer | Integration point with VCF |
| --- | --- | --- | --- |
| Model drift, data drift, fairness and quality monitoring | 5.23a | Model observability tool | Runs on VKS. Metrics into VCF Operations. Alerts into ITSM |
| Fairness and explainability tooling | 5.5-5.8 | Responsible-AI toolkit | Runs in test domain and production |
| Independent model validation | 5.19, 5.24 | Validation function and tooling | Receives evidence pack. Holds read-only roles |
| AI inventory system of record | 4.5-4.9 | GRC or CMDB | API feed and reconciliation from VCF |
| Data catalog, quality, lineage | 5.4a-c, g | Data governance platform | Dataset snapshot IDs, storage policy mapping |
| Content DLP, PII redaction | 5.16a | Egress proxy, guardrail layer | Placed behind Avi and before Gateway Firewall |
| LLM guardrails, prompt injection defense | 5.15, 5.22b | Application guardrail framework | Hosted on VKS |
| MFA, PAM | 5.16b | IdP, PAM | Federated into vCenter, NSX, K8s |
| Shadow AI on endpoints and SaaS | 4.4 | Endpoint DLP, CASB | Network signals from NSX as supplement |
| Key management and HSM | 5.4e, 5.16 | KMS with HSM | KMIP key providers for vSAN and VM Encryption |
| Staff training | 5.23e, 6.1 | HR, learning | None |

**16.2 Items to verify in a lab before customer use \[V\]**

1. Exact vNUMA topology rules for VMs wider than one pNUMA node, odd vCPU counts.
2. VM Encryption support and vMotion behavior with vGPU and passthrough VMs.
3. Snapshot support for vGPU VMs on vSphere 9.x.
4. vSphere HA restart behavior for passthrough VMs and vGPU VMs, including free-device requirements.
5. DFW non-enforcement on SR-IOV and DirectPath I/O NICs.
6. vDefend pod-level IDS/IPS in VCF 9.0 versus 9.1.
7. IDS/IPS and ATP behavior on TLS traffic. Gateway URL category filtering and entitlement.
8. HSM support for vSphere Native Key Provider.
9. VCF Operations compliance benchmark list in 9.x. Container monitoring pack name. Native GPU and PCIe metrics.
10. Integration path from VCF Operations or VCF Automation to the CMDB.
11. VCF Automation lease or expiry feature in 9.x.
12. vLCM packaging of NVIDIA host driver and firmware. Configuration profile naming.
13. Avi API auth profile names. LLM streaming timeout and buffering settings.
14. VCF DR product naming and key replication behavior in 9.x.
15. Admission policy engine availability as a VKS package.
16. Framebuffer clearing between vGPU assignments. Confidential computing support on the target GPU and vSphere version.
17. Private AI Services entitlement and feature set for the target VCF 9.x release.
18. Your VCF-vCommunity adapter: supported VKS and container metric scope.

**16.3 Confidence of sources**

- Product facts from Broadcom and NVIDIA pages are marked with links. These were taken from search results, not full page reads.
- Facts tagged \[V\] come from engineering knowledge or a single secondary source. They are not confirmed here.

## 17. Implementation roadmap

Two hard gates: 7 October 2027 (MAS Sections 3-4) and 7 October 2028 (Sections 5-6). Windows below are a proposal for an FI starting now.

| Phase | Window | Deliverables | Exit gate |
| --- | --- | --- | --- |
| 0. Decisions | Nov 2026 - Jan 2027 | Tier model and control profiles. Tag taxonomy. Control function named. vDefend and PAIS entitlement check. KMS and HSM design. Lab test of section 16.2 items | Signed architecture decision records |
| 1. Platform baseline | Feb - Jun 2027 | AI-controlled workload domain. Key providers per tier. DFW default-deny and role policy. Log pipeline to SIEM. VCF Automation intake templates with mandatory tags | Tier 0-1 workloads on the baseline. Compliance report clean |
| 2. Governance readiness | Jul - Sep 2027 | Discovery and reconciliation jobs. Inventory feed to GRC. Appetite metrics and board report. Third-party register entries for embedded AI | **Gate 7 Oct 2027**: Sections 3-4 evidence pack |
| 3. Lifecycle controls | Oct 2027 - Mar 2028 | Test and adversarial domain. Evidence-pack automation. Monitoring stack with model observability. Kill-switch build and first drill. Everything-as-code change control | Tier 2 pilot passes pre-deployment review |
| 4. Assurance | Apr - Aug 2028 | Independent validation dry run. Failure-injection tests. DR test with key recovery. Capacity review. Internal audit walkthrough. Decommission and erasure rehearsal | Findings closed. Evidence archived |
| 5. Steady state | Sep 2028 onward | Recurring drills, re-validation triggers, quarterly role review, annual tier review | **Gate 7 Oct 2028**: Sections 5-6 evidence pack |

**First 30 days**

1. Fix the 17 items in section 2 in any customer-facing material.
2. Agree tier definitions with the risk function.
3. Lab-verify the five items most likely to change design: items 2, 3, 5, 6 and 12 in section 16.2.
4. Confirm vDefend and Private AI Services entitlement and VCF Automation tenancy model per vCenter and NSX pair.

## Sources

Links below came from search results and were not read in full. Treat them as pointers. MAS clause references are from the Guidelines PDF dated 7 October 2026 supplied with the request.

- [vSAN Encryption Services, VMware](https://www.vmware.com/docs/vmw-vsan-encryption-services)
- [Enable vSAN data-at-rest encryption, vSAN 8.0 admin guide](https://docs.vmware.com/en/VMware-vSphere/8.0/vsan-administration/GUID-E7CA36B7-D7EB-423A-ADD1-7E410E36F5A7.html)
- [DISA STIG V-258929, vSAN encryption](https://stigviewer.com/stig/vmware_vsphere_8.0_vcenter/2023-10-11/finding/V-258929)
- [vSAN storage clusters in VCF 9.1, VMware blog](https://blogs.vmware.com/cloud-foundation/2026/05/12/vsan-storage-clusters-in-vcf-9-1/)
- [vDefend for VCF 9.1, VMware blog](https://blogs.vmware.com/security/2026/05/vdefend-vcf-9-1-zero-trust.html)
- [vDefend for VCF 9.0, VMware blog](https://blogs.vmware.com/security/2025/06/announcing-vdefend-for-vcf-9.html)
- [vDefend ATP enhancements, VMware blog](https://blogs.vmware.com/security/2024/10/vmware-vdefend-latest-enhancements-in-advanced-threat-prevention.html)
- [vDefend innovations for GenAI era, VMware blog](https://blogs.vmware.com/security/2024/08/vmware-vdefend-innovations-elevating-enterprise-lateral-threat-defense-for-genai-era.html)
- [vDefend licensing, VMware cloud provider blog](https://blogs.vmware.com/cloudprovider/?p=27026)
- [vDefend and Avi 9.1 update, StorageReview](https://storagereview.com/news/vmware-vdefend-75tbps-distributed-firewall-vcf-9-1-update)
- [Private AI Services in VCF 9.0, VMware blog](https://blogs.vmware.com/cloud-foundation/2025/06/19/private-ai-services-new-in-vmware-private-ai-foundation-with-nvidia-in-vcf-9-0/)
- [Private AI Services standard in VCF, StorageReview](https://www.storagereview.com/zh-CN/news/broadcom-positions-vcf-as-an-ai-native-platform-for-the-modern-private-cloud)
- [PAIS 3.0 model deployment, William Lam](https://williamlam.com/2026/09/vcf-9-1-1-deploying-your-own-ai-models-with-vcf-private-ai-services-pais-3-0.html)
- [VCF 9.1 AI features summary, mind-verse.de](https://mind-verse.de/news/vmware-cloud-foundation-ki-anwendungen-private-clouds)
- [Avi SE CPU disparity, Avi docs](https://docs.vmware.com/en/VMware-Avi-Load-Balancer/30.2/Monitoring-Operability-Guide/GUID-36DC0333-DC2E-4DC3-9648-1CD8575A1AB4.html)
- [Avi SE metrics, Avi docs](https://docs.vmware.com/en/VMware-Avi-Load-Balancer/30.2/Monitoring-Operability-Guide/GUID-9909AF52-4E48-4873-9673-0BCD78D96D5A.html)
- [Broadcom KB 438582, Avi SE CPU alarm](https://knowledge.broadcom.com/external/article/438582/virtual-machine-cpu-usage-alarm-continuo.html)
- [Broadcom KB 441498, VCFA tenancy](https://knowledge.broadcom.com/external/article/441498/missing-svcautoattachvksmbroadcomcom-in.html)
- [Broadcom KB 438333, VCFA supervisors per org](https://knowledge.broadcom.com/external/article/438333/vcfa-90x-unable-to-assign-more-than-1-su.html)
- [NVIDIA vGPU on vSphere, MIG-backed vGPU](https://docs.nvidia.com/vgpu/deployment/vmware/latest/mig.html)
- [NVIDIA vGPU configuration on vSphere 9](https://docs.nvidia.com/vgpu/deployment/vmware/latest/vgpu-config.html)
- [vMotion with vGPU, vSphere docs](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vcenterhost.doc/GUID-8FE6A0DA-49E9-472B-815B-D630CF2014AD.html)
- [Enhanced vMotion for vGPU in VCF 9.0, VMware blog](https://blogs.vmware.com/cloud-foundation/2025/06/19/enhanced-vmotion-for-vgpu-vms-in-vmware-cloud-foundation-9-0/)
- [vGPU snapshot limitation, ISPsystem](https://www.ispsystem.com/docs/vmmanager-kb/errors/6120)
- [VM Encryption with KMS, Entrust guide](https://nshielddocs.entrust.com/interops-docs/vmware-vsphere-kc/procedures.html)
