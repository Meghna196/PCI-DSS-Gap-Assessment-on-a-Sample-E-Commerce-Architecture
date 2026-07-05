# PCI DSS Gap Assessment on a Sample E-Commerce Architecture

> A structured PCI DSS v4.0 gap assessment against a fictional cloud-native e-commerce architecture, producing a scored findings register, risk-ranked remediation roadmap, and executive-ready compliance documentation.

## Overview

This project simulates the kind of compliance work a GRC Analyst performs before a formal PCI DSS audit. I designed a representative e-commerce architecture hosted on AWS, then assessed it against all 12 PCI DSS v4.0 requirements, identifying control gaps, assigning risk ratings, and producing actionable remediation guidance.

The goal was to demonstrate end-to-end GRC thinking: not just finding what is broken, but understanding why it matters, how to prioritize fixes, and how to communicate findings to both technical and non-technical stakeholders. That balance is exactly what a GRC role at a developer-focused infrastructure company like Vercel demands, where compliance work must translate cleanly across engineering and leadership audiences.

The sample architecture reflects a realistic deployment pattern: a Next.js storefront, a Node.js payments API, a managed PostgreSQL database, and third-party payment processor integration via Stripe. This mirrors the kind of stack Vercel customers commonly deploy, making the findings directly relevant rather than purely academic.

All artifacts in this repository are original work produced from publicly available PCI DSS standards, AWS documentation, and open-source compliance frameworks.

## What I Built / Key Features

- **Scoped Architecture Diagram:** A documented system boundary defining the cardholder data environment (CDE), including data flows, trust zones, and out-of-scope components.
- **Requirements Traceability Matrix:** A spreadsheet mapping all 12 PCI DSS v4.0 requirements and sub-requirements to the sample architecture, with a `Met / Partial / Not Met` status for each control.
- **Findings Register:** A structured log of every identified gap, including requirement reference, observation, risk rating (Critical / High / Medium / Low), and evidence notes.
- **Remediation Roadmap:** A prioritized, time-boxed plan grouping findings into 30/60/90-day remediation waves based on exploitability and compliance impact.
- **Executive Summary Report:** A one-page PDF-ready summary translating technical findings into business risk language suitable for a board or compliance committee.
- **Control Narrative Templates:** Reusable policy and procedure document stubs covering network segmentation, access control, logging, and encryption requirements.

## Skills & Tools Demonstrated

**Standards and Frameworks**
- PCI DSS v4.0 (all 12 requirements)
- NIST SP 800-53 (referenced for control mapping)
- Shared Responsibility Model (AWS)

**Cloud and Architecture**
- AWS (VPC, Security Groups, RDS, CloudTrail, KMS, WAF)
- Stripe API integration and scope-reduction via tokenization

**GRC and Documentation**
- Gap assessment methodology
- Risk rating and prioritization
- Compliance documentation and report writing
- Requirements traceability matrices

**Tooling**
- `draw.io` for architecture and data flow diagrams
- `Google Sheets` / `Excel` for the traceability matrix and findings register
- `Markdown` for all narrative documentation in this repository
- `git` for version-controlled documentation management

## Architecture & Approach

The assessment targets a simplified but realistic e-commerce CDE. The architecture places the Next.js frontend on a CDN edge layer outside the CDE, while the payments API and database sit inside a private VPC subnet. Stripe handles raw card capture via hosted payment fields, which significantly reduces CDE scope under PCI DSS requirement 1 and 4.

```text
[ Customer Browser ]
        |
  [ CDN / Edge ]  <-- Out of scope (no CHD)
        |
  [ Payments API ]  <-- In scope (processes auth tokens, order metadata)
        |
  [ RDS PostgreSQL ]  <-- In scope (stores transaction records)
        |
  [ Stripe API ]  <-- Third-party, assessed via SAQ A eligibility review
```

I worked through each requirement systematically, using the PCI DSS v4.0 ROC Reporting Template as a reference for what assessors look for. Gaps were documented with enough detail that an engineering team could act on them without needing a follow-up conversation. Risk ratings follow a likelihood-times-impact model consistent with NIST guidance.

## Suggested Repository Structure

```text
pci-dss-gap-assessment/
├── architecture/
│   ├── cde-boundary-diagram.drawio
│   └── data-flow-diagram.drawio
├── assessment/
│   ├── requirements-traceability-matrix.xlsx
│   └── findings-register.xlsx
├── reports/
│   ├── executive-summary.md
│   └── full-gap-assessment-report.md
├── remediation/
│   └── remediation-roadmap.md
├── templates/
│   ├── network-segmentation-policy.md
│   ├── access-control-policy.md
│   ├── logging-and-monitoring-procedure.md
│   └── encryption-standards.md
└── README.md
```

## What This Demonstrates to Employers

- **Shows ability to scope a CDE accurately**, including identifying what qualifies as in-scope versus out-of-scope under PCI DSS v4.0 using compensating controls and tokenization strategies.
- **Demonstrates familiarity with PCI DSS v4.0**, including the new customized approach option and changes from v3.2.1, which are active topics in compliance programs today.
- **Proves cross-functional communication skills**, producing separate artifacts for engineering teams (findings register, remediation roadmap) and leadership (executive summary).
- **Reflects cloud-native GRC thinking**, assessing controls within an AWS shared responsibility model rather than treating the cloud as an opaque black box.
- **Shows structured risk prioritization**, using a repeatable scoring methodology to sequence remediation work in a way that reduces the highest-impact gaps first.
- **Demonstrates documentation discipline**, producing version-controlled, audit-ready artifacts that could be handed directly to a QSA or internal audit team.

## Getting Started

**Prerequisites:** No code runtime is required. You need `draw.io` (free, browser-based) to view diagrams and a spreadsheet application to open `.xlsx` files.

```bash
# Clone the repository
git clone https://github.com/your-username/pci-dss-gap-assessment.git

# Navigate to the project root
cd pci-dss-gap-assessment
```

Open `reports/full-gap-assessment-report.md` for the complete assessment narrative, or start with `reports/executive-summary.md` for a high-level overview. The findings register at `assessment/findings-register.xlsx` is the authoritative source for all identified gaps and their current remediation status.
