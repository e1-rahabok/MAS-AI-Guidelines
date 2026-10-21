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
