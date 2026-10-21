# MAS AI Risk Management Guidelines: VCF 9 Implementation Architecture

Level 400 mapping of the Monetary Authority of Singapore (MAS) *Guidelines on Artificial Intelligence Risk Management* (7 October 2026) to VMware Cloud Foundation 9: vSphere, vSAN, NSX / vDefend / Avi, VCF Operations / Automation, and VKS / Private AI Services.

## Key dates

- 7 October 2027: MAS Sections 3-4 expected.
- 7 October 2028: MAS Sections 5-6 expected.

## Scope boundary

- MAS is technology-agnostic. No product complies. The FI complies. VCF supplies controls and evidence.
- VCF covers infrastructure controls. It does not cover fairness, explainability, drift detection, or independent model validation.
- Items tagged **[V]** are unverified. Lab-verify before customer use (see `docs/16-*.md`).
- Source links were taken from search results and not read in full.

## Contents

- [Full document in one file](docs/00-full-document.md)
- [1. Summary and scope boundary](docs/01-summary-and-scope-boundary.md)
- [2. Corrections to the two source analyses](docs/02-corrections-to-the-two-source-analyses.md)
- [3. MAS timeline, applicability and proportionality](docs/03-mas-timeline-applicability-and-proportionality.md)
- [4. Reference architecture and risk-tier model](docs/04-reference-architecture-and-risk-tier-model.md)
- [5. Oversight, identification, inventory, materiality (MAS 3.x, 4.x)](docs/05-oversight-identification-inventory-materiality.md)
- [6. Lifecycle controls, contingency and kill switch (MAS 5.1-5.3)](docs/06-lifecycle-controls-contingency-and-kill-switch.md)
- [7. Data management, encryption and erasure (MAS 5.4)](docs/07-data-management-encryption-and-erasure.md)
- [8. Transparency, fairness and human oversight (MAS 5.5-5.9)](docs/08-transparency-fairness-and-human-oversight.md)
- [9. Third-party AI and selection (MAS 5.10-5.13)](docs/09-third-party-ai-and-selection.md)
- [10. Evaluation, testing and pre-deployment review (MAS 5.14-5.15, 5.18-5.22)](docs/10-evaluation-testing-and-pre-deployment-review.md)
- [11. Technology and cybersecurity (MAS 5.16, 5.22)](docs/11-technology-and-cybersecurity.md)
- [12. Reproducibility, monitoring and re-validation (MAS 5.17, 5.23, 5.24)](docs/12-reproducibility-monitoring-and-re-validation.md)
- [13. Change management and decommissioning (MAS 5.25-5.26)](docs/13-change-management-and-decommissioning.md)
- [14. Capacity, capability and infrastructure design (MAS 6.1-6.3)](docs/14-capacity-capability-and-infrastructure-design.md)
- [15. Traceability matrix](docs/15-traceability-matrix.md)
- [16. Gap register and verification list](docs/16-gap-register-and-verification-list.md)
- [17. Implementation roadmap](docs/17-implementation-roadmap.md)
- [Sources](docs/99-sources.md)

## Reference material

- [MAS Guidelines on AI Risk Management (PDF)](reference/MAS-Guidelines-on-AI-Risk-Management-2026-10-07.pdf). Publisher: Monetary Authority of Singapore. Copyright remains with MAS. Original source: mas.gov.sg.
- [Original draft 1](reference/original-drafts/draft-1-vcf-architecture-analysis.md) and [draft 2](reference/original-drafts/draft-2-vcf-analysis.md). Starting analyses. Section 2 of this repo lists 17 corrections to them. Kept for traceability.

## Status

Draft. Not a regulatory opinion.
