# ShopFast PCI DSS Evidence Tracker

This log tracks the evidence required to substantiate each gap and, eventually, each remediation. Status moves from `Missing` to `Requested` to `Collected` to `Verified` as an engagement progresses. In this fictional scenario all evidence is `Missing`, since no interviews or artifact collection have actually occurred.

## Evidence Log

| Evidence ID | Gap ID | Description | Source | Status | Date Collected | Collection Method |
|---|---|---|---|---|---|---|
| EVD-001 | GAP-001 | RDS schema screenshot/export showing the `card_number` column and sample row structure | DBA interview / RDS console | Missing | - | Request read-only DB console access from the DBA and capture the schema with `\d orders` in `psql`. |
| EVD-002 | GAP-001 | Sample of S3 transaction log objects showing card data in query strings | S3 console / log export | Missing | - | Pull 5-10 recent log objects from the transaction log bucket and grep for PAN-pattern strings. |
| EVD-003 | GAP-002 | Lambda database connection string / environment configuration | Code review | Missing | - | Review the Lambda function's environment variables and DB client initialization code in the repo. |
| EVD-004 | GAP-002 | RDS parameter group settings showing `rds.force_ssl` value | RDS console | Missing | - | Export the current RDS parameter group configuration via AWS Console or `aws rds describe-db-parameters`. |
| EVD-005 | GAP-003 | VPC subnet and security group configuration for Lambda and RDS | AWS console / Infra-as-code repo | Missing | - | Export VPC diagram and security group rules via AWS Console or `aws ec2 describe-security-groups`. |
| EVD-006 | GAP-004 | MFA configuration screenshot for the admin portal / IAM or IdP | IAM console / IdP admin panel | Missing | - | Capture the MFA enforcement policy screen from Cognito, Okta, or the equivalent identity provider. |
| EVD-007 | GAP-005 | Most recent ASV scan report and/or penetration test report | Vendor deliverable | Missing | - | Request the latest scan/pentest report from the ASV or pentest firm; confirm scope covers CDE components. |
| EVD-008 | GAP-006 | Patch management policy and recent patch history | Policy docs / change log | Missing | - | Request the patch management SOP and the last 2 quarters of patch/change records from engineering. |
| EVD-009 | GAP-007 | CloudWatch log review policy or SOP, and alarm configuration | Policy docs / CloudWatch console | Missing | - | Request the log review SOP document and export configured CloudWatch alarms/metric filters. |
| EVD-010 | GAP-008 | RDS and Lambda configuration baseline / hardening checklist | Infra-as-code repo / policy docs | Missing | - | Request the Terraform/CloudFormation templates or manual configuration checklist used for CDE components. |
| EVD-011 | GAP-009 | IAM policy exports and most recent access review record | IAM console / access review log | Missing | - | Export IAM policies attached to admin/portal roles via `aws iam list-attached-role-policies`; request the last access review sign-off. |
| EVD-012 | GAP-010 | SAQ-A (or equivalent) or AOC for each of the three checkout plugin vendors | Vendor portal / vendor compliance contact | Missing | - | Email each vendor's compliance/security contact requesting their current SAQ-A or AOC document. |

## Notes on Evidence Discipline

Every gap in the gap analysis should trace to at least one evidence item here, and every evidence item should trace back to a gap. Evidence status should never sit at `Missing` past the gap's target remediation date without an explanation logged in the Notes area of the gap analysis workbook. In a live engagement, this tracker would also record the evidence reviewer's name and any follow-up questions raised during review, since a QSA or auditor will ask not just "do you have this," but "who collected it, when, and how was it verified."
