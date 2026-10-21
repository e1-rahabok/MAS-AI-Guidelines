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
