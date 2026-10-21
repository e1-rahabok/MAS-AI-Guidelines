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
