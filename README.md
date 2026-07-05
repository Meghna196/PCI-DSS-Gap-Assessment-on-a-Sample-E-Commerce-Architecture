# ShopFast PCI DSS Gap Assessment

This repository contains a structured PCI DSS v4.0 gap assessment conducted against ShopFast, a fictional mid-size e-commerce company processing card payments directly through a custom checkout form on a Vercel-like edge platform, backed by an AWS Lambda API, RDS PostgreSQL database, and S3 log storage. The scenario was designed to mirror a realistic, under-documented merchant environment: full card numbers are stored at rest and leak into logs, internal traffic is unencrypted, the admin portal has no MFA, there is no network segmentation around the cardholder data environment, and third-party checkout plugins have no verified compliance status. The goal was to practice the full analyst workflow of scoping a cloud-native architecture, mapping it to compliance requirements, and producing findings a real engineering team could act on.

The methodology follows a standard GRC gap assessment lifecycle: an architecture diagram (`shopfast-architecture-v1.drawio`) establishes the CDE boundary and traces card data flow; a requirement mapping (`shopfast-pci-mapping.xlsx`) walks all 12 PCI DSS v4.0 requirement domains against ShopFast's components, including explicit reasoning for requirements that shift to AWS under the cloud shared responsibility model; a risk-rated gap analysis (`shopfast-gap-analysis.xlsx`) scores each finding using a 3x3 likelihood/impact matrix and assigns an owner and target date; a remediation plan (`remediation-plan.md`) translates each gap into specific, engineer-actionable steps; an evidence tracker (`evidence-tracker.md`) documents how each piece of supporting evidence would be collected in a live engagement; and an executive report (`shopfast-pci-gap-report.md`) summarizes the posture, findings, and risk narrative for a non-technical audience.

## Contents

| File | Purpose |
|---|---|
| `shopfast-architecture-v1.drawio` | Architecture diagram with CDE boundary and data flow |
| `shopfast-pci-mapping.xlsx` | Requirement-by-requirement scoping across all 12 PCI DSS v4.0 domains |
| `shopfast-gap-analysis.xlsx` | Risk-rated gap analysis (10 gaps, Critical to Low) |
| `remediation-plan.md` | Prioritized, technical remediation steps per gap |
| `evidence-tracker.md` | Evidence log with collection method per item |
| `shopfast-pci-gap-report.md` | Executive summary report for leadership |

**Disclaimer:** ShopFast is a fictional scenario built for portfolio and training purposes. This assessment does not represent an actual audit of any real company or system.
