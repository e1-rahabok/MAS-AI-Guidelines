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
