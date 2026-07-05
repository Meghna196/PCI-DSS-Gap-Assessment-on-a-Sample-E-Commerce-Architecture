# PCI DSS v4.0 Gap Assessment Report

**Client:** ShopFast (Fictional Scenario)
**Prepared by:** Meghna Suresh
**Date:** July 5, 2026
**Confidentiality:** For internal use only

---

## 1. Scope

This assessment covers the ShopFast e-commerce platform as described in architecture version 1.0, including the Next.js frontend, AWS Lambda API layer, RDS PostgreSQL database, and S3 log storage. Physical security controls are considered the shared responsibility of AWS under their PCI DSS AOC. The cardholder data environment (CDE) is defined as the customer-facing checkout form, the API Gateway/Lambda API layer, the RDS `orders` table, and the S3 transaction log bucket, since raw card data flows through and is currently retained in all four locations. The admin portal and third-party checkout plugins are treated as CDE-adjacent, since they connect directly to CDE components without any documented network segmentation.

## 2. Overall Compliance Posture

ShopFast is currently **not compliant** with PCI DSS v4.0. The assessment identified 10 gaps across 10 requirement areas, including 1 rated Critical, 4 rated High, 4 rated Medium, and 1 rated Low. Two additional requirement areas (physical security and anti-malware for serverless compute) were scoped as Not Applicable, with the reasoning documented in the requirement mapping. The most urgent finding is that full, unencrypted card numbers are stored in the production database and are also leaking into application logs, an issue that alone would likely place ShopFast out of compliance and at risk of a reportable data exposure event if left unaddressed.

## 3. Key Findings Summary

| Priority | Count | Example Finding |
|---|---|---|
| Critical | 1 | Unencrypted PAN storage at rest and log leakage (Req 3.3, 3.5) |
| High | 4 | No MFA on admin portal (Req 8.4); unencrypted Lambda-to-RDS traffic (Req 4.2); no network segmentation (Req 1.3); no vulnerability scanning or pentest (Req 11.3) |
| Medium | 4 | Insufficient patch cadence (Req 6.3); logs not reviewed (Req 10.4); no configuration baseline (Req 2.2); no least-privilege access model (Req 7.2) |
| Low | 1 | Missing third-party vendor SAQs (Req 12.8) |

## 4. Risk Narrative

The most serious exposure ShopFast faces today is straightforward: full card numbers sit unencrypted in the production `orders` table and also show up in plaintext inside transaction logs stored in S3. Anyone with read access to the database, or to that log bucket, can view complete card numbers with no additional effort. Combine that with the lack of network segmentation and the absence of MFA on the admin portal, and a single compromised credential or a misconfigured access policy could expose the full card data of every ShopFast customer who has ever checked out. That is the realistic worst case: a mass card-data breach traceable directly to storage and logging practices rather than to a sophisticated attack.

The consequences of that scenario extend well beyond a technical incident. Payment card brands can levy substantial fines through the acquiring bank, and ShopFast would very likely lose the ability to process card payments directly until it demonstrates full remediation and passes a follow-up assessment. Under most state breach notification laws, ShopFast would also be required to notify every affected customer, and the reputational damage from "an e-commerce company stored my full card number in plaintext" tends to be more lasting than the direct financial penalty. Customers whose cards are exposed also bear real cost and stress, from fraudulent charges to the burden of replacing cards and monitoring accounts.

The good news is that the single highest-risk finding also has the clearest fix: moving card capture to a tokenization provider removes raw PANs from ShopFast's systems almost entirely, which shrinks the CDE, reduces audit scope, and closes the most dangerous gap in one coordinated project. The remaining findings, MFA, segmentation, patching, logging, and vendor management, are standard security hygiene items that most engineering teams can implement within the 90-day remediation window without significant new tooling spend.

## 5. Recommended Next Steps

1. Immediately migrate card capture to a hosted payment field/tokenization solution to remove raw PANs from ShopFast's systems and shrink the CDE.
2. Purge existing plaintext PAN data from RDS and scrub S3 transaction logs, then confirm removal with a full data scan.
3. Enforce TLS on all internal Lambda-to-RDS traffic and enable MFA on the admin portal, both low-effort, high-impact fixes.
4. Stand up network segmentation between the CDE and the rest of the AWS VPC.
5. Establish recurring vulnerability scanning, an annual penetration test, and a documented patch management SLA.
6. Implement CloudWatch alerting and a formal log review process.
7. Document a secure configuration baseline and a least-privilege access model for all CDE-adjacent systems.
8. Collect verified compliance attestations (SAQ-A or AOC) from all three third-party checkout plugin vendors.
9. Re-assess against PCI DSS v4.0 at the 90-day mark to confirm closure of all Critical and High findings before considering a formal SAQ or ROC submission.

## 6. Appendices

- Appendix A: Full Gap Analysis (shopfast-gap-analysis.xlsx)
- Appendix B: Remediation Plan (remediation-plan.md)
- Appendix C: Evidence Tracker (evidence-tracker.md)
- Appendix D: Architecture Diagram (shopfast-architecture-v1.drawio)
- Appendix E: Requirement Mapping (shopfast-pci-mapping.xlsx)
