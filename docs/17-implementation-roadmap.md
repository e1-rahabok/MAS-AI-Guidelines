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
