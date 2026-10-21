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
