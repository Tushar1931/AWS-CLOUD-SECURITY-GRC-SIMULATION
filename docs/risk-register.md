# Risk Register

**Project:** AWS Cloud Security Risk Assessment & GRC Simulation  
**Author:** Tushar Sharma  
**Date:** 2025  
**Framework:** ISO 27001 / NIST  
**Overall Risk Posture:** HIGH

---

## Risk Assessment Criteria

### Impact Scale
| Rating | Description |
|--------|-------------|
| Critical | Potential for significant data breach, full account compromise, or major regulatory violation |
| High | Substantial risk of data loss, privilege escalation, or compliance failure |
| Medium | Moderate risk of exposure or delayed detection |
| Low | Limited impact, easy to remediate |

### Likelihood Scale
| Rating | Description |
|--------|-------------|
| High | Likely to be exploited given current configuration |
| Medium | May be exploited under certain conditions |
| Low | Unlikely to be exploited in current context |

---

## Risk Register

---

### R1 — Public S3 Bucket Exposure

| Field | Detail |
|-------|--------|
| **Risk ID** | R1 |
| **Risk Title** | Public S3 bucket allowing unauthorised access to stored data |
| **Affected Resource** | `tushar-risk-bucket` (S3) |
| **Impact** | Critical |
| **Likelihood** | High |
| **Severity** | Critical |
| **Control Gap** | Block Public Access disabled; bucket policy grants public read to all objects |
| **Description** | The S3 bucket was configured with public read access enabled and no authentication required. A sample file (`customer_data.txt`) simulating sensitive customer data was accessible via a direct object URL without any credentials. This represents a realistic data exposure scenario. |
| **Potential Impact** | Unauthorised access to sensitive data; potential data breach and regulatory violation (GDPR, etc.) |
| **Recommendation** | Enable S3 Block Public Access at both bucket and account level. Remove the public-read bucket policy. Implement S3 bucket policies with explicit deny for public access. Classify and encrypt sensitive data at rest using SSE-S3 or SSE-KMS. |
| **ISO 27001** | A.9 (Access Control), A.13 (Network Security) |
| **NIST** | AC (Access Control), SC (System and Communications Protection) |

---

### R2 — Over-Permissive IAM Policy

| Field | Detail |
|-------|--------|
| **Risk ID** | R2 |
| **Risk Title** | Over-permissive IAM policy (`*:*`) assigned to dev-user |
| **Affected Resource** | `dev-user` (IAM) — `FullAccessCustomPolicy` |
| **Impact** | Critical |
| **Likelihood** | High |
| **Severity** | Critical |
| **Control Gap** | Direct policy attachment bypasses group-based access control; violates Principle of Least Privilege |
| **Description** | The `dev-user` account was assigned a custom IAM policy granting all actions (`*`) on all resources (`*`). This policy was attached directly to the user, bypassing the `LimitedAccess` group which had a `ReadOnlyAccess` policy. This simulates a common real-world misconfiguration where developer accounts are granted excessive permissions. |
| **Potential Impact** | Full AWS account compromise; privilege escalation; ability to create, modify, or delete any resource including IAM users, S3 buckets, EC2 instances, and billing configurations. |
| **Recommendation** | Replace the `*:*` policy with a scoped, least-privilege policy aligned to dev-user's actual job function. Remove direct policy attachments in favour of group-based access control. Implement IAM Access Analyser to identify overly permissive policies. Use AWS Permission Boundaries to cap maximum permissions. |
| **ISO 27001** | A.9 (Access Control) |
| **NIST** | AC (Access Control) |

---

### R3 — EC2 Instance Exposed to Internet

| Field | Detail |
|-------|--------|
| **Risk ID** | R3 |
| **Risk Title** | EC2 instance exposed to the internet via open SSH (port 22) and HTTP (port 80) |
| **Affected Resource** | `Risk-Instance` (EC2) — Security Group |
| **Impact** | Medium |
| **Likelihood** | Medium |
| **Severity** | Medium |
| **Control Gap** | Overly permissive security group rules allowing inbound traffic from `0.0.0.0/0` |
| **Description** | The EC2 instance `Risk-Instance` was configured with a security group allowing inbound SSH (port 22) and HTTP (port 80) access from any IP address (`0.0.0.0/0`). The instance was assigned a public IP and is directly reachable from the internet. These findings were also confirmed by AWS Security Hub. |
| **Potential Impact** | SSH exposure increases the attack surface for brute-force credential attacks and unauthorised remote access. HTTP exposure risks serving content or revealing server information without encryption. |
| **Recommendation** | Restrict SSH access to specific trusted IP ranges or use AWS Systems Manager Session Manager as a secure alternative. Remove HTTP (port 80) if not required, or redirect to HTTPS. Implement a bastion host or VPN gateway for secure administrative access. |
| **ISO 27001** | A.13 (Network Security) |
| **NIST** | SC (System and Communications Protection) |

---

### R4 — Lack of MFA for IAM Users

| Field | Detail |
|-------|--------|
| **Risk ID** | R4 |
| **Risk Title** | Multi-factor authentication (MFA) not configured for IAM users |
| **Affected Resource** | All IAM users (`admin-user`, `dev-user`, `analyst-user`) |
| **Impact** | High |
| **Likelihood** | High |
| **Severity** | High |
| **Control Gap** | Absence of MFA as a compensating authentication control |
| **Description** | None of the three IAM users created in this environment have MFA enabled. Without MFA, a compromised username and password is sufficient to gain full account access. This risk is especially critical for `admin-user` and `dev-user`, which hold elevated privileges. This is one of the most common and impactful misconfigurations in cloud environments. |
| **Potential Impact** | Credential theft via phishing, password spraying, or leaked credentials would result in full account access. Combined with the over-permissive `dev-user` policy (R2), this creates a critical attack chain. |
| **Recommendation** | Enforce MFA for all IAM users, especially those with elevated privileges. Use AWS IAM policy conditions to deny access unless MFA is authenticated (`aws:MultiFactorAuthPresent: true`). Consider using virtual MFA devices (e.g., Google Authenticator) or hardware tokens. |
| **ISO 27001** | A.9 (Access Control) |
| **NIST** | IA (Identification and Authentication) |

---

### R5 — Weak CloudTrail Configuration

| Field | Detail |
|-------|--------|
| **Risk ID** | R5 |
| **Risk Title** | Incomplete logging and monitoring configuration in CloudTrail |
| **Affected Resource** | AWS CloudTrail — `SecurityTrail` |
| **Impact** | Medium |
| **Likelihood** | Medium |
| **Severity** | Medium |
| **Control Gap** | Log file validation disabled; no CloudWatch integration; no Insights (anomaly detection) |
| **Description** | CloudTrail was enabled but configured with minimal features. Log file validation was disabled — meaning log tampering could go undetected. CloudWatch Logs integration was not configured, preventing real-time alerting on suspicious activity. CloudTrail Insights (automated anomaly detection) was also disabled. Together, these gaps significantly reduce the environment's ability to detect and respond to incidents. |
| **Potential Impact** | Delayed detection of malicious activity; inability to verify audit trail integrity; failure to alert on unusual API calls or access patterns; non-compliance with audit and logging requirements. |
| **Recommendation** | Enable CloudTrail log file validation to detect tampering. Integrate CloudTrail with CloudWatch Logs and configure metric filters and alarms for suspicious activity (e.g., root login, policy changes, failed authentication). Enable CloudTrail Insights for automated anomaly detection. Enable data event logging for S3 and Lambda if applicable. |
| **ISO 27001** | A.12 (Logging and Monitoring) |
| **NIST** | AU (Audit and Accountability) |

---

## Risk Summary Table

| Risk ID | Risk Title | Severity | Likelihood | Status |
|---------|-----------|----------|------------|--------|
| R1 | Public S3 bucket exposure | Critical | High | Open |
| R2 | Over-permissive IAM policy (`*:*`) | Critical | High | Open |
| R3 | EC2 instance exposed to internet | Medium | Medium | Open |
| R4 | No MFA for IAM users | High | High | Open |
| R5 | Weak CloudTrail configuration | Medium | Medium | Open |

---

*This risk register was produced as part of a simulated GRC assessment. All risks relate to an intentionally misconfigured lab environment.*
