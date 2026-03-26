# Security Findings Report

**Project:** AWS Cloud Security Risk Assessment & GRC Simulation  
**Author:** Tushar Sharma  
**Classification:** Internal – Portfolio / Educational  
**Assessment Type:** Manual Review + Automated (AWS Security Hub)  
**Overall Risk Posture:** HIGH  

---

## 1. Introduction

This report presents the findings of a cloud security assessment conducted on an AWS environment intentionally configured with a range of security misconfigurations. The assessment simulates the type of review an Information Security or GRC Analyst would conduct on an enterprise cloud environment.

The assessment focused on four core areas: identity and access management, object storage, compute networking, and audit logging. Both automated tooling (AWS Security Hub) and manual configuration review were used to identify and evaluate security gaps.

---

## 2. Scope

| Service | Scope |
|---------|-------|
| AWS IAM | Users, groups, policies, access control governance |
| AWS S3 | Storage configuration, access permissions, data exposure |
| AWS EC2 | Network exposure, security group configuration |
| AWS CloudTrail | Logging completeness, monitoring integration |
| AWS Security Hub | Automated findings aggregation |

**Excluded from scope:**
- Advanced threat simulation or penetration testing
- Third-party security tooling
- Production-grade or multi-account architecture

---

## 3. Methodology

The assessment was conducted using a structured, risk-based approach:

1. **Environment configuration** — AWS services were configured with controlled misconfigurations to simulate realistic enterprise scenarios
2. **Automated review** — AWS Security Hub was enabled to aggregate findings across resources
3. **Manual review** — Configuration of IAM, S3, EC2, and CloudTrail was reviewed manually for risks beyond what automated tools detected
4. **Risk evaluation** — Each finding was assessed based on Impact, Likelihood, and Exploitability
5. **Framework mapping** — Findings were mapped to ISO 27001 controls and NIST control families
6. **Documentation** — Structured outputs produced: Risk Register, Compliance Mapping, and this Findings Report

---

## 4. Findings

---

### Finding 1: Public S3 Bucket Exposing Sensitive Data

**Severity:** Critical  
**Detection Method:** Manual Review  
**Risk Reference:** R1  

**Description:**  
The S3 bucket `tushar-risk-bucket` was configured with Block Public Access disabled and a bucket policy explicitly granting `s3:GetObject` permissions to all principals (`*`). A file named `customer_data.txt`, simulating sensitive customer information, was uploaded to the bucket and confirmed to be publicly accessible via its object URL — without any authentication.

**Evidence:**
- Screenshot 10: S3 public access unblocked
- Screenshot 11: S3 bucket list
- Screenshot 12: Bucket showing uploaded `customer_data.txt`
- Screenshot 13: Bucket policy JSON (public read enabled)
- Screenshot 14: File properties showing object URL
- Screenshot 15: File content visible via public URL (no auth)

**Risk:**  
Exposure of sensitive or personally identifiable data to the public internet. In a production context, this could result in a data breach and regulatory penalties under frameworks such as GDPR or ISO 27001 A.18 (Compliance).

**Recommended Remediation:**
- Enable S3 Block Public Access at both bucket and account level
- Remove the bucket policy granting public read access
- Apply S3 Object Ownership controls and restrict ACLs
- Enable server-side encryption (SSE-S3 or SSE-KMS) for data at rest
- Implement S3 Access Logging to track data access events

---

### Finding 2: Over-Permissive IAM Policy (`*:*`) on dev-user

**Severity:** Critical  
**Detection Method:** Manual Review  
**Risk Reference:** R2  

**Description:**  
The IAM user `dev-user` was assigned a custom policy (`FullAccessCustomPolicy`) granting full access to all AWS actions on all resources (`Effect: Allow, Action: *:*, Resource: *`). This policy was attached directly to the user, bypassing the group-level `ReadOnlyAccess` policy applied via the `LimitedAccess` group. This constitutes a clear violation of the Principle of Least Privilege and a governance gap in access control.

**Evidence:**
- Screenshot 5: Policy JSON screen (showing `*:*` configuration)
- Screenshot 6: Policy created confirmation
- Screenshot 7: `dev-user` showing direct policy attachment (`FullAccessCustomPolicy`)
- Screenshot 4: Group page showing `LimitedAccess` group (bypassed)

**Risk:**  
`dev-user` has effective administrative control over the entire AWS account. If these credentials were compromised (especially without MFA — see Finding 4), an attacker could create new admin users, exfiltrate data, modify security controls, or destroy resources. Combined with the absence of MFA, this represents the highest-impact risk chain in this assessment.

**Recommended Remediation:**
- Replace `*:*` policy with a scoped, job-function-aligned IAM policy
- Remove direct policy attachment; manage all permissions through group membership
- Enable IAM Access Analyser to continuously monitor for overly permissive policies
- Implement Permission Boundaries to cap maximum permissions regardless of attached policies
- Review and audit all IAM policies on a regular cadence

---

### Finding 3: EC2 Instance Exposed to the Internet via Open Ports

**Severity:** Medium  
**Detection Method:** AWS Security Hub + Manual Review  
**Risk Reference:** R3  

**Description:**  
The EC2 instance `Risk-Instance` was launched with an associated security group configured to allow inbound SSH (port 22) and HTTP (port 80) traffic from `0.0.0.0/0` (any IP address on the internet). The instance was assigned a public IPv4 address. AWS Security Hub independently identified and flagged both the SSH and HTTP exposure as security findings.

**Evidence:**
- Screenshot 16: EC2 network settings showing SSH and HTTP rules
- Screenshot 17: EC2 instance state and public IP
- Screenshot 18: Security Hub dashboard
- Screenshot 19: Security Hub findings list
- Screenshot 20: Open port 22 finding detail
- Screenshot 21: Open port 80 finding detail

**Risk:**  
SSH exposure on port 22 from `0.0.0.0/0` exposes the instance to brute-force attacks, credential stuffing, and unauthorised remote access. HTTP exposure on port 80 increases the attack surface and may expose unencrypted communications. Both findings represent unnecessary public accessibility that violates network security best practices.

**Recommended Remediation:**
- Restrict SSH access to specific known IP ranges (e.g., corporate IPs only)
- Prefer AWS Systems Manager Session Manager over SSH for administrative access (eliminates the need for open port 22 entirely)
- Remove HTTP (port 80) if not serving web traffic; if required, redirect to HTTPS (port 443)
- Review all security group rules to apply least-privilege network access
- Consider deploying a bastion host or VPN for secure access

---

### Finding 4: No Multi-Factor Authentication (MFA) for IAM Users

**Severity:** High  
**Detection Method:** Manual Review  
**Risk Reference:** R4  

**Description:**  
None of the three IAM users — `admin-user`, `dev-user`, and `analyst-user` — had MFA configured. MFA is a fundamental compensating control that significantly reduces the risk of account compromise in the event of credential theft. Its absence is especially critical for privileged accounts.

**Evidence:**
- Screenshot 2: Users list showing `analyst-user`
- Screenshot 3: User page showing all 3 users
- (MFA status observable via IAM console — not separately captured)

**Risk:**  
Without MFA, a compromised set of IAM credentials is sufficient to gain full account access. Given that `dev-user` holds a `*:*` policy (Finding 2), the absence of MFA on that account creates a critical combined attack vector — credential compromise alone would yield complete AWS account control.

**Recommended Remediation:**
- Enforce MFA for all IAM users via an IAM policy condition: `aws:MultiFactorAuthPresent: true`
- Prioritise MFA enforcement for `admin-user` and `dev-user` immediately
- Consider virtual MFA (e.g., Google Authenticator, Authy) or hardware MFA tokens
- Create an IAM policy that denies all actions if MFA is not authenticated, and attach it to all users

---

### Finding 5: Weak CloudTrail Configuration — Incomplete Audit Trail

**Severity:** Medium  
**Detection Method:** Manual Review  
**Risk Reference:** R5  

**Description:**  
AWS CloudTrail was enabled (trail name: `SecurityTrail`) and actively logging API activity. However, three critical features were disabled: log file validation, CloudWatch Logs integration, and CloudTrail Insights (anomaly detection). This means the environment has basic logging but lacks the controls needed to ensure log integrity, enable real-time alerting, or detect unusual activity patterns automatically.

**Evidence:**
- Screenshot 8: SecurityTrail weak logging confirmation
- Screenshot 9: Log validation and CloudWatch disabled

> **Note:** AWS Console enforces multi-region trail creation via the UI. The weak configuration was achieved by disabling key features rather than restricting trail scope — this is a realistic simulation of an organisation that has enabled logging but not fully configured it.

**Risk:**  
Without log file validation, malicious actors who gain sufficient access could tamper with CloudTrail logs without detection. Without CloudWatch integration, there is no real-time alerting capability for suspicious events (e.g., root account login, IAM changes, failed authentication). Without Insights, automated anomaly detection is absent. Collectively, these gaps significantly reduce the organisation's ability to detect, investigate, and respond to security incidents in a timely manner.

**Recommended Remediation:**
- Enable **log file validation** to detect any tampering with CloudTrail log files
- Integrate CloudTrail with **CloudWatch Logs** and configure metric filters and alarms for high-priority events
- Enable **CloudTrail Insights** for automated detection of unusual API activity
- Enable **data event logging** for S3 (object-level access) and Lambda if applicable
- Store CloudTrail logs in a separate, access-restricted S3 bucket with SSE-KMS encryption

---

## 5. Findings Summary

| # | Finding | Severity | Detection Method |
|---|---------|----------|-----------------|
| 1 | Public S3 Bucket Exposure | Critical | Manual |
| 2 | Over-permissive IAM Policy (`*:*`) | Critical | Manual |
| 3 | EC2 Open to Internet (SSH/HTTP) | Medium | Security Hub + Manual |
| 4 | No MFA for IAM Users | High | Manual |
| 5 | Weak CloudTrail Configuration | Medium | Manual |

---

## 6. Observations

**Automated tooling limitations:**  
AWS Security Hub successfully identified the EC2 network exposure findings (open SSH and HTTP ports). However, it did not detect the over-permissive IAM `*:*` policy, the public S3 bucket, the absence of MFA, or the CloudTrail configuration weaknesses during the assessment window. This underscores the importance of combining automated tools with structured manual reviews and GRC-based assessments.

**Risk chaining:**  
Findings R2 (over-permissive IAM) and R4 (no MFA) together create a particularly high-impact attack chain. Credential compromise of `dev-user` — a realistic scenario given the absence of MFA — would yield full administrative control over the AWS account due to the `*:*` policy.

---

## 7. Conclusion

The assessment identified **five significant security findings** across identity, storage, compute, and logging layers. The environment's overall security posture is assessed as **HIGH RISK**.

Critical misconfigurations — public S3 data exposure and an unrestricted IAM policy — require **immediate remediation**. The absence of MFA compounds the IAM risk substantially. CloudTrail and EC2 findings, while medium severity in isolation, reduce the environment's ability to detect and respond to the critical risks identified.

A follow-up review should be conducted post-remediation to validate that all controls have been effectively implemented.

---

*This report was produced as part of a simulated GRC assessment for portfolio and educational purposes.*
