## 1. Summary and scope boundary

VCF 9 supplies the infrastructure controls and audit evidence for MAS clauses 4.5, 5.4(e), 5.16, 5.22, 5.23(d), 5.25(b) and 6.3. It does not supply model-level controls. Those need an MLOps, GRC and data-governance layer on top.

- MAS is technology-agnostic. No product complies. The FI complies. VCF provides controls and evidence.
- Infrastructure-complete: encryption, segmentation, privileged access, logging, capacity, change control on the platform.
- Partial: AI inventory (4.5-4.9), kill switch (5.3, 5.23b), monitoring (5.23a), reproducibility (5.17), third-party AI (5.10-5.11).
- Not covered by VCF: fairness (5.7-5.8), explainability (5.5-5.6), data and concept drift (5.23a), independent model validation (5.19), model evaluation (5.14), data representativeness (5.4b).
- Section 2 lists 17 corrections to the two source drafts. Read it first. Several drafted claims would not survive a regulator or VCDX panel review.
- Product naming follows VCF 9.x: VCF Operations, VCF Automation, VKS, vDefend, Avi Load Balancer, Private AI Services (PAIS).
- Items tagged **\[V\]** are from memory or a single secondary source. Lab-verify before customer use. Section 16 consolidates them.

**Document structure**

- Sections 3-4: timeline, tier model, reference architecture.
- Sections 5-14: clause-by-clause design. Each gives MAS requirement, control design, configuration, evidence, gap.
- Section 15: traceability matrix.
- Sections 16-17: gap register, verification list, roadmap.
