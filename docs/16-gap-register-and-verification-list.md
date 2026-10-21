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
