# AWS-CLOUD-SECURITY-GRC-SIMULATION
Simulated AWS cloud security assessment project identifying misconfigurations across IAM, S3, EC2, and logging, with risk analysis and ISO 27001 mapping.

## Project Overview
 
This project simulates a **cloud security risk assessment** of an intentionally misconfigured AWS environment. It replicates real-world enterprise misconfigurations across identity, storage, compute, and logging layers, then assesses and documents those risks using industry standard GRC methodology.
 
The goal is to demonstrate practical skills in:
- Cloud security configuration review
- Risk identification, assessment, and documentation
- Compliance mapping (ISO 27001 / NIST)
- Audit-style reporting and stakeholder communication
 
> **Note:** All misconfigurations were intentionally introduced in a controlled, isolated AWS environment for assessment purposes only.
 
---

## Repository Structure
 
```
aws-cloud-security-grc-simulation/
│
├── README.md                          ← Project overview and summary (this file)
│
├── docs/
│   ├── risk-register.md               ← Full risk register with 5 identified risks
│   ├── security-findings-report.md    ← Audit-style findings report
│   └── compliance-mapping.md          ← ISO 27001 & NIST control mapping
│
├── diagrams/
│   └── environment-architecture.png   ← AWS environment architecture diagram
│
└── screenshots/
    ├── iam/                           ← IAM user, group, and policy screenshots
    ├── cloudtrail/                    ← CloudTrail weak config evidence
    ├── s3/                            ← S3 public access and bucket policy screenshots
    ├── ec2/                           ← EC2 instance and security group screenshots
    └── security-hub/                  ← Security Hub findings dashboard screenshots
```
 
## Objectives
 
- Assess an AWS environment for security misconfigurations.
- Identify risks across IAM, storage, compute, and logging.
- Evaluate gaps in monitoring and governance controls.
- Simulate a GRC style audit end-to-end.
- Produce structured, professional security outputs.
 
---
 
## Environment Setup
 
The AWS environment was configured with a mix of intentionally secure and insecure settings to replicate a realistic enterprise scenario.

### Identity & Access Management (IAM)
 
| User | Permissions | Risk |
|------|-------------|------|
| `admin-user` | Full Administrative Privileges | Privileged account - monitored |
| `dev-user` | Custom policy with `*:*` (all actions, all resources) | Violates Least Privilege |
| `analyst-user` | ReadOnlyAccess (via group) | Appropriate scope |
 
- Group-based access control partially implemented (`LimitedAccess` group with `ReadOnlyAccess`)
- `dev-user` bypasses group policy via direct policy attachment (`FullAccessCustomPolicy`)
- MFA **not configured** for any IAM user
 
 ### Logging & Monitoring (CloudTrail)
 
| Feature | Status |
|---------|--------|
| CloudTrail Logging | Enabled |
| Log File Validation | Disabled |
| CloudWatch Integration | Disabled |
| Insights / Anomaly Detection | Disabled |
 
> AWS Console enforces multi region trails. Weak configuration was achieved by disabling critical features rather than limiting scope.

### Storage (S3)
 
| Configuration | Status |
|---------------|--------|
| Block All Public Access | Disabled |
| Bucket Policy | Public read access to all objects |
| Sample Data | `customer_data.txt` uploaded |
| Public Access Verified | Object URL accessible without authentication |
 
### Compute (EC2)
 
| Configuration | Status |
|---------------|--------|
| SSH (Port 22) | Open to `0.0.0.0/0` |
| HTTP (Port 80) | Open to `0.0.0.0/0` |
| Public IP Assigned | Yes |
| Security Group | Overly permissive - exposes instance to internet |
 
---
## Assessment Methodology
 
```
Configure AWS environment with controlled misconfigurations
          ↓
Enable limited logging and visibility
          ↓
Identify risks (manual review + AWS Security Hub)
          ↓
Evaluate risks (Impact × Likelihood × Exploitability)
          ↓
Map findings to ISO 27001 and NIST frameworks
          ↓
Produce Risk Register, Findings Report, and Executive Summary
```
 
---

## Risk Summary
 
| Risk ID | Risk | Severity | Likelihood |
|---------|------|----------|------------|
| R1 | Public S3 bucket - unauthorised data access | Critical | High |
| R2 | Over-permissive IAM policy (`*:*`) on dev-user | Critical | High |
| R3 | EC2 instance exposed via open SSH (port 22) | Medium | Medium |
| R4 | No MFA configured for IAM users | High | High |
| R5 | Weak CloudTrail configuration - limited visibility | Medium | Medium |
 
> **Overall Risk Posture: HIGH**
 
Full details → [`docs/risk-register.md`](docs/risk-register.md)
 
---

## Compliance Mapping Summary
 
| Risk | ISO 27001 Controls | NIST Controls |
|------|-------------------|---------------|
| R1 – Public S3 Exposure | A.9 (Access Control), A.13 (Network Security) | AC, SC |
| R2 – IAM Over Permission | A.9 (Access Control) | AC |
| R3 – EC2 Open to Internet | A.13 (Network Security) | SC |
| R4 – No MFA | A.9 (Access Control) | IA |
| R5 – Weak CloudTrail | A.12 (Logging & Monitoring) | AU |
 
Full mapping → [`docs/compliance-mapping.md`](docs/compliance-mapping.md)
 
---

## Security Hub Findings
 
AWS Security Hub was enabled to aggregate findings across configured resources. The following issues were automatically identified:
 
- EC2 instance with **SSH (port 22)** exposed to the internet (`0.0.0.0/0`)
- EC2 instance with **HTTP (port 80)** exposed to the internet (`0.0.0.0/0`)
 
The following **additional risks were identified through manual review**, beyond what Security Hub detected:
 
- Over-permissive IAM policy (`*:*`) on `dev-user`
- Public S3 bucket exposing `customer_data.txt`
- Absence of MFA for all IAM users
- Weak CloudTrail configuration (missing validation, monitoring, and anomaly detection)
 
> This highlights the importance of combining automated tooling with manual governance review.
 
---

## Executive Summary
 
This assessment evaluated the security posture of an AWS environment configured to simulate real-world misconfigurations across identity, storage, compute, and logging services.
 
**The overall risk posture is assessed as HIGH** due to the presence of critical security gaps:
 
- **Public S3 bucket** exposing sensitive data without authentication
- **IAM user with full `*:*` permissions**, violating least privilege principles
- **EC2 instance** accessible from the internet via open SSH and HTTP ports
- **No MFA** configured for any IAM user
- **Incomplete CloudTrail** configuration limiting audit trail integrity
 
These issues significantly increase the likelihood of unauthorised access, data breaches, privilege escalation, and delayed detection of malicious activity.
 
**Recommended immediate actions:**
1. Enable S3 Block Public Access and restrict bucket policies
2. Replace `*:*` policy with least privilege role-based access
3. Restrict SSH/HTTP to trusted IP ranges or use a bastion host
4. Enforce MFA for all IAM users, especially privileged accounts
5. Enable CloudTrail log validation, data event logging, and CloudWatch integration
 
Full findings → [`docs/security-findings-report.md`](docs/security-findings-report.md)
 
---

## IAM & GRC Concepts Demonstrated
 
| Concept | Applied In |
|---------|-----------|
| Principle of Least Privilege | IAM policy design and violation simulation |
| Identity-Based Access Control | IAM user and group configuration |
| Privilege Escalation Risk | Direct policy bypass of group controls |
| Audit Trail Integrity | CloudTrail configuration weaknesses |
| Detection & Visibility Gaps | CloudWatch and Insights disabled |
| Data Security & Classification | S3 public exposure with sensitive data |
| Network Security Controls | EC2 security group misconfiguration |
| Continuous Security Monitoring | AWS Security Hub integration |
| Risk Assessment Methodology | Impact × Likelihood evaluation |
| Compliance Mapping | ISO 27001 / NIST control alignment |
| Audit Reporting | Structured findings documentation |
| Risk Communication | Executive summary for stakeholders |
 
---
 
## Tools & Services Used
 
- **AWS IAM** — User, group, and policy management
- **AWS CloudTrail** — Logging and audit trail
- **AWS S3** — Storage and access control simulation
- **AWS EC2** — Compute and network exposure simulation
- **AWS Security Hub** — Automated findings aggregation
- **AWS VPC** — Network configuration
 
---
 
## Evidence Screenshots
 
Screenshots are organised in the `/screenshots` directory:
 
| Folder | Contents |
|--------|----------|
| `screenshots/iam/` | User creation, user list, group config, policy JSON, direct policy attachment |
| `screenshots/cloudtrail/` | Trail creation, log validation disabled, CloudWatch disabled |
| `screenshots/s3/` | Bucket list, public access unblocked, bucket policy JSON, file upload, public URL access |
| `screenshots/ec2/` | Instance state and IP, security group rules (SSH/HTTP open) |
| `screenshots/security-hub/` | Dashboard overview, open port 22 finding, open port 80 finding, risk register notes |
 
---
 
## Disclaimer
 
This project was conducted in a controlled, personal AWS environment created solely for educational and portfolio purposes. All misconfigurations are intentional simulations. No real production data or systems were involved.
 
---

## Author
 
**Tushar Sharma**  
Information Security Professional | IAM & GRC Specialist
