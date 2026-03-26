# Architecture Diagram

This folder contains the architecture diagram for the simulated AWS environment used in the security assessment.

## `environment-architecture.svg'

The diagram illustrates the intentionally misconfigured AWS environment, including:

- **IAM layer** — Three users (`admin-user`, `dev-user`, `analyst-user`), the `LimitedAccess` group, and the `FullAccessCustomPolicy` attached directly to `dev-user`
- **Logging layer** — CloudTrail with disabled log validation, CloudWatch, and Insights
- **Storage layer** — S3 bucket (`tushar-risk-bucket`) with public access enabled and `customer_data.txt` exposed
- **Compute layer** — EC2 instance (`Risk-Instance`) in a default VPC with security group open to `0.0.0.0/0` on ports 22 and 80
- **Monitoring layer** — AWS Security Hub aggregating findings

Risk indicators are overlaid on misconfigured components to visually communicate the assessment findings.

---

## Suggested Tool for Recreation

The diagram was designed using [draw.io](https://app.diagrams.net/) with AWS architecture icon shapes.  
You can recreate or extend it using:
- [AWS Architecture Icons](https://aws.amazon.com/architecture/icons/)
- [draw.io AWS shape library](https://app.diagrams.net/)
- [Lucidchart](https://www.lucidchart.com/)

---

> See [README.md](../README.md) for a high-level summary of the environment configuration.
