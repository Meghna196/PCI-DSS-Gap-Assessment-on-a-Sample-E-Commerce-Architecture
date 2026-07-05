# ShopFast PCI DSS Remediation Plan

**Assessment Date:** July 5, 2026
**Assessor:** Meghna Suresh
**PCI DSS Version:** 4.0
**Next Review Date:** October 3, 2026 (90 days from assessment date)

## Executive Summary

ShopFast's current architecture is not PCI DSS v4.0 compliant. The assessment identified 10 gaps across 10 requirement areas, including 1 Critical, 4 High, 4 Medium, and 1 Low severity finding. The most urgent issue is that full, unencrypted PANs are stored at rest and leaking into application logs, which puts ShopFast at immediate risk of a reportable card data breach. The remediation plan below is sequenced so that the highest-risk exposure closes first, followed by access control and monitoring gaps, then longer-horizon process and vendor governance work.

---

## Critical Priority (Resolve within 30 days)

### GAP-001: Unencrypted PAN Storage and Log Contamination

**Requirement:** PCI DSS v4.0 Req 3.3, 3.5
**Current State:** Full PANs are stored in the `card_number` column of the RDS `orders` table, and raw card data is leaking into S3 transaction logs via query string parameters.
**Target State:** No sensitive authentication data (SAD) retained post-authorization; PANs replaced with tokens; no card data present in logs.
**Remediation Steps:**
1. Engage a PCI-compliant tokenization provider (e.g., Stripe, Braintree) and migrate the checkout form to their hosted fields or client-side SDK so raw PANs never touch ShopFast's servers.
2. Update the `orders` table schema to store only a token reference and `card_last4`; drop the `card_number` column.
3. Export and securely archive any legally required historical data, then purge the `card_number` column contents and confirm removal with `VACUUM FULL` in PostgreSQL so the data is not recoverable from disk.
4. Audit all S3 log objects for PAN-pattern strings (regex `\b\d{13,19}\b` combined with Luhn validation), quarantine affected objects, and enable a log-scrubbing Lambda or Firehose transform to redact card-pattern strings before they reach S3 going forward.
5. Enable S3 Object Lock (WORM) on the log bucket to prevent tampering with logs during and after cleanup.
6. Update ShopFast's data retention policy documentation to explicitly prohibit PAN storage and define the token-based data model.

**Verification:** Run a regex + Luhn-check scan across the full RDS database and all S3 log objects post-remediation and confirm zero matches. Re-run the scan as part of the 90-day review.

---

## High Priority (Resolve within 14-45 days)

### GAP-002: Unencrypted Lambda-to-RDS Connection

**Requirement:** PCI DSS v4.0 Req 4.2
**Current State:** Internal traffic between AWS Lambda and RDS PostgreSQL does not enforce TLS; PANs traverse the VPC in cleartext.
**Target State:** All Lambda-to-RDS traffic is encrypted with TLS 1.2+.
**Remediation Steps:**
1. Set `rds.force_ssl = 1` on the RDS parameter group associated with the `orders` database instance.
2. Update the Lambda database connection string/config to include `?ssl=true` (or the equivalent `sslmode=require` for `pg` / `node-postgres` clients) and load the AWS RDS CA bundle for certificate validation.
3. Reboot or apply the parameter group change during a maintenance window, since `force_ssl` changes require a restart on some engine versions.
4. Verify with AWS VPC Flow Logs or a packet capture (Wireshark on a bastion host) that the connection now negotiates TLS.

**Verification:** Confirm `pg_stat_ssl` shows `ssl = true` for all active Lambda connections; confirm no plaintext PostgreSQL wire protocol packets appear in a VPC traffic mirror sample.

**Target Date:** 14 days

---

### GAP-003: No Network Segmentation Around the CDE

**Requirement:** PCI DSS v4.0 Req 1.3
**Current State:** No segmentation exists between the CDE (API Gateway, Lambda, RDS, log path) and the rest of the AWS VPC.
**Target State:** The CDE is isolated in dedicated subnets with security groups and NACLs restricting traffic to only what is required.
**Remediation Steps:**
1. Define CDE-specific private subnets for Lambda (VPC-attached) and RDS, separate from any non-CDE workloads (marketing site, internal tools, etc.).
2. Create dedicated security groups: allow inbound to RDS only from the Lambda security group on port 5432; deny all other inbound traffic by default.
3. Apply NACLs at the subnet level as a defense-in-depth control restricting both inbound and outbound traffic to documented, business-justified rules.
4. Remove any broad `0.0.0.0/0` or overly permissive security group rules currently attached to CDE components.
5. Document the network diagram and ruleset in the architecture repo so future changes go through review.

**Verification:** Run an AWS Security Hub or manual security group audit confirming no CDE resource accepts unrestricted inbound traffic; confirm non-CDE resources cannot reach RDS directly.

**Target Date:** 30 days

---

### GAP-004: No MFA on Admin Portal

**Requirement:** PCI DSS v4.0 Req 8.4
**Current State:** Admin portal is protected by username and password only.
**Target State:** MFA is required for all administrative access into the CDE.
**Remediation Steps:**
1. Integrate the admin portal with an identity provider that supports MFA (AWS Cognito with MFA enabled, Okta, or Auth0).
2. Enforce MFA at login for all admin accounts; disable legacy password-only login paths.
3. Require re-authentication with MFA for sensitive actions (e.g., viewing order/PAN-adjacent data, changing account settings).
4. Communicate the change to all admin users and set a grace period for MFA enrollment (7 days), after which non-enrolled accounts are locked.

**Verification:** Attempt a login with valid credentials but no MFA token and confirm access is denied. Screenshot the IAM/Cognito MFA policy configuration for the evidence tracker.

**Target Date:** 14 days

---

### GAP-005: No Vulnerability Scanning or Penetration Testing

**Requirement:** PCI DSS v4.0 Req 11.3
**Current State:** No evidence of quarterly ASV scans or an annual penetration test.
**Target State:** Recurring vulnerability scanning and annual penetration testing program in place with tracked remediation.
**Remediation Steps:**
1. Engage an Approved Scanning Vendor (ASV) for quarterly external vulnerability scans of all public-facing components (frontend, API Gateway endpoint).
2. Stand up internal vulnerability scanning (e.g., AWS Inspector) covering Lambda dependencies and any remaining compute.
3. Schedule an annual penetration test covering the checkout flow, API layer, and admin portal, plus a re-test after the Critical/High gaps above are closed.
4. Create a vulnerability remediation SLA (e.g., 30 days for critical, 60 for high) and track findings to closure.

**Verification:** First ASV scan report on file with a passing result (or documented remediation plan for any findings); signed penetration test report and remediation tracker.

**Target Date:** 45 days

---

## Medium Priority (Resolve within 60 days)

### GAP-006: Insufficient Patch Cadence

**Requirement:** PCI DSS v4.0 Req 6.3
**Current State:** OS and dependency patching happens roughly quarterly, on an ad hoc basis.
**Target State:** Risk-based patch management with defined SLAs (e.g., 30 days for critical/high severity vulnerabilities).
**Remediation Steps:**
1. Enable automated dependency scanning (e.g., GitHub Dependabot, Snyk) on the Node.js Lambda codebase to flag vulnerable packages continuously.
2. Define a patch SLA policy: critical within 30 days, high within 60 days, medium/low on the next release cycle.
3. Automate RDS minor version patching via the AWS-managed maintenance window; track major version upgrades on a planned schedule.
4. Report patch compliance status monthly to engineering leadership.

**Verification:** Dependabot/Snyk dashboard shows zero unaddressed critical vulnerabilities older than 30 days; patch SLA policy document signed off by engineering leadership.

**Target Date:** 60 days

---

### GAP-007: Logs Not Reviewed, No Alerting

**Requirement:** PCI DSS v4.0 Req 10.4
**Current State:** CloudWatch logs exist but are never reviewed; no alerting on suspicious access patterns.
**Target State:** Logs are actively monitored with automated alerting on anomalous activity.
**Remediation Steps:**
1. Configure CloudWatch metric filters and alarms for indicators such as repeated failed admin logins, unusual RDS query volume, and access from unexpected IP ranges.
2. Route alerts to an on-call channel (PagerDuty, Slack, or email distribution) with defined escalation.
3. Define and document a daily/weekly log review SOP, including who reviews, what they look for, and how findings are escalated.
4. Consider forwarding logs to a lightweight SIEM (e.g., AWS Security Lake, Datadog) for correlation across CloudWatch, VPC Flow Logs, and RDS logs.

**Verification:** Trigger a test alarm condition (e.g., simulated failed login burst) and confirm the alert fires and reaches the on-call channel; log review SOP document on file.

**Target Date:** 45 days

---

### GAP-008: No Documented Secure Configuration Baseline

**Requirement:** PCI DSS v4.0 Req 2.2
**Current State:** No documented hardening baseline for Lambda or RDS; default parameter groups and configurations appear to be in use.
**Target State:** A documented, applied secure configuration baseline for all CDE components.
**Remediation Steps:**
1. Create a custom RDS parameter group replacing the default, enabling `rds.force_ssl`, disabling unused extensions, and setting appropriate logging parameters.
2. Document a Lambda hardening checklist: least-privilege execution role, no hardcoded secrets in environment variables (use AWS Secrets Manager), minimal IAM permissions.
3. Remove any default credentials, sample data, or example configurations left over from initial setup.
4. Store the baseline as infrastructure-as-code (Terraform/CloudFormation) so future deployments inherit it automatically.

**Verification:** Confirm RDS is using the custom parameter group; confirm Lambda environment variables contain no plaintext secrets (spot-check via AWS Console or CLI).

**Target Date:** 60 days

---

### GAP-009: No Least-Privilege Access Model

**Requirement:** PCI DSS v4.0 Req 7.2
**Current State:** No documented least-privilege access control model or periodic access review for admin portal users or IAM roles touching the CDE.
**Target State:** Role-based access aligned to business need-to-know, reviewed quarterly.
**Remediation Steps:**
1. Inventory all current admin portal users and IAM roles/policies with access to CDE resources.
2. Define role-based access tiers (e.g., read-only support, order management, full admin) mapped to job function.
3. Rewrite IAM policies to grant only the permissions each role requires; remove any wildcard (`*`) resource or action permissions.
4. Establish a quarterly access review process with sign-off from a designated owner.

**Verification:** IAM Access Analyzer report shows no unused or overly broad permissions; first quarterly access review completed and documented.

**Target Date:** 60 days

---

## Low Priority (Resolve within 90 days)

### GAP-010: Unverified Third-Party Checkout Plugins

**Requirement:** PCI DSS v4.0 Req 12.8
**Current State:** Three third-party checkout plugins are in use with no verified SAQ-A (or equivalent) compliance documentation on file.
**Target State:** All third-party service providers touching the checkout flow have current, verified compliance attestations on file.
**Remediation Steps:**
1. Contact each of the three plugin vendors and request their current SAQ-A (or applicable SAQ type) or Attestation of Compliance (AOC).
2. Maintain a service provider inventory listing each vendor, the service provided, PCI DSS responsibility matrix, and compliance status.
3. For any vendor unable to provide current compliance documentation, evaluate replacement with a verified alternative or removal from the checkout flow.
4. Add a contractual requirement for annual compliance attestation renewal in future vendor agreements.

**Verification:** Service provider inventory on file with current AOC/SAQ for all three vendors, or documented remediation/replacement plan for any that cannot provide one.

**Target Date:** 90 days
