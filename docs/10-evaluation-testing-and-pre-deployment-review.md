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
