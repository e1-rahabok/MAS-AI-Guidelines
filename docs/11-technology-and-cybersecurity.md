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
