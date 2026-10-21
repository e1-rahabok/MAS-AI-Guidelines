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
