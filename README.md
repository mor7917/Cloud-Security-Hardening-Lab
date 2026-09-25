# AWS Cloud Security Hardening Lab

An end-to-end security assessment and hardening lab conducted on a live AWS environment. This project details the deployment of the Prowler security framework, an initial multi-framework compliance assessment, an analysis of security gaps, and immediate remediation actions taken on exposed infrastructure.

## 📌 Project Overview
As cloud infrastructure scales, maintaining continuous compliance and visibility across IAM, network boundaries, and storage layer configurations becomes a critical engineering challenge. 

This lab documents a structured approach to identifying security misconfigurations using the **Prowler CLI (v5.44.0)** security assessment tool across **655 distinct checks**. The environment was benchmarked against industry standards including **CIS AWS Foundations, PCI-DSS v4.0, and ISO 27001**. 

### Key Milestones Achieved:
* **Infrastructure Discovery & Scans:** Successfully deployed Prowler via Docker containerization to bypass python-dependency limits.
* **Storage Layer Remediation:** Fully hardened exposed S3 bucket permissions (`ak-0110`), closing potential public data-leaks.
* **Risk Mapping:** Aggregated and categorized 117 security failures across 15 services to create a high-priority remediation roadmap.

---

## 🛠️ Step-by-Step Lab Execution & Commands

Follow this sequence to replicate the lab architecture, execute the security scans, and conduct manual remediation.

### Step 1: Environment Setup
Initialize the project architecture and create dedicated domains for scans, configurations, and document persistence:
```bash
# Create project root directory
mkdir -p ~/cloud-security-hardening-lab
cd ~/cloud-security-hardening-lab

# Form standard workspace subfolders
mkdir -p prowler-results screenshots remediation terraform documentation

# Confirm folder structure
ls
```

### Step 2: Run the Security Assessment (Prowler)
*Note: A local python environment run was initially attempted but hit a Pydantic dependency error. The environment was successfully stabilized by switching to containerized execution.*

Run the comprehensive AWS scan using the official Docker image. Ensure your native local AWS CLI credentials (`~/.aws`) are mapped correctly:
```bash
sudo docker run --rm -it -v ~/.aws:/home/prowler/.aws:ro toniblyx/prowler:latest aws
```
*Scan Metrics: 655 checks executed over 16 minutes and 19 seconds.*

### Step 3: Audit & Remediate S3 Public Access
Inspect S3 properties and close configurations allowing public lookups:
```bash
# 1. List active buckets
aws s3 ls

# 2. Inspect target bucket public exposure settings (Unset/False flags found)
aws s3api get-public-access-block --bucket ak-0110

# 3. Remediate: Apply strict public access block policy configuration
aws s3api put-public-access-block --bucket ak-0110 \
  --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# 4. Verify remediation changes are live (All 4 flags must now evaluate to True)
aws s3api get-public-access-block --bucket ak-0110
```

### Step 4: Identity & Core Service Analysis
Manually audit current configuration maps for Identity, Network, and Audit trails to complement the automated scan findings:
```bash
# Inspect active identity configurations
aws iam list-users
aws iam list-access-keys --user-name ankit-01
aws iam list-attached-user-policies --user-name ankit-01

# Audit networking and perimeter settings
aws ec2 describe-security-groups

# Confirm active logging states
aws cloudtrail describe-trails
```

---

## 📊 Security Scan Analytics

### 1. Findings by AWS Service
The initial baseline assessment surfaced **117 Failed checks (23.88%)** and **292 Passed checks (59.59%)**.

| Service | Status | Critical | High | Medium | Low |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **iam** | 🔴 FAIL (34) | 1 | 7 | 7 | 19 |
| **ec2** | 🔴 FAIL (15) | 0 | 8 | 6 | 1 |
| **vpc** | 🔴 FAIL (6) | 0 | 3 | 3 | 0 |
| **s3** | 🔴 FAIL (28) | 0 | 1 | 15 | 12 |
| **secretsmanager**| 🔴 FAIL (4) | 0 | 2 | 2 | 0 |
| **cloudtrail** | 🔴 FAIL (7) | 0 | 0 | 4 | 3 |
| **config** | 🔴 FAIL (1) | 0 | 1 | 0 | 0 |
| **kms** | 🔴 FAIL (1) | 0 | 1 | 0 | 0 |
| **athena** | 🔴 FAIL (2) | 0 | 0 | 2 | 0 |
| **networkfirewall**| 🔴 FAIL (1) | 0 | 0 | 1 | 0 |
| **sagemaker** | 🔴 FAIL (17) | 0 | 0 | 0 | 17 |
| **cloudwatch** | 🔴 FAIL (1) | 0 | 0 | 1 | 0 |
| **account** | 🟢 PASS (0) | 0 | 0 | 0 | 0 |
| **bedrock** | 🟢 PASS (0) | 0 | 0 | 0 | 0 |
| **eventbridge** | 🟢 PASS (34) | 0 | 0 | 0 | 0 |

### 2. Compliance Framework Drift %
The environment was cross-referenced against global security standards, identifying severe non-compliance across enterprise profiles:

| Compliance Framework | Fail % |
| :--- | :---: |
| PCI_4.0_AWS | 53.78% |
| ISO27001_2013_AWS | 51.72% |
| NIS2_AWS | 48.44% |
| PCI_3.2.1_AWS | 47.22% |
| AWS_AUDIT_MANAGER_CONTROL_TOWER_GUARDRAILS_AWS | 45.45% |
| AWS_FOUNDATIONAL_TECHNICAL_REVIEW_AWS | 41.44% |
| CIS_2.0_AWS | 38.94% |
| CSA_CCM_4.0 | 38.31% |
| CIS_1.4_AWS / CIS_1.5_AWS | 37.50% |
| AWS_FOUNDATIONAL_SECURITY_BEST_PRACTICES_AWS | 35.04% |
| NIST_800_53_REVISION_5_AWS | 32.81% |

---

## 🛠️ Findings & Remediation Progress Tracker

* **[REMEDIATED]** **S3 Public Access (`ak-0110`):** Standardized bucket rules to explicit block constraints. 
* **[OPEN]** **IAM Over-Privilege Risk:** The user account (`ankit-01`) is highly over-privileged, referencing 10 broad managed policies directly with zero group architecture restrictions and an active Critical vulnerability.
* **[OPEN]** **EC2/VPC Vulnerabilities (11 High):** Inbound security rules passed manual validation; failures are likely driven by absent EBS Volume Encryption or legacy IMDSv1 availability.
* **[OPEN]** **CloudTrail Security Gaps:** Core trail lacks log-file validation configurations and standard KMS customer-managed key encryption.
* **[OPEN]** **GuardDuty Disconnected:** Scan configurations were blocked (`AccessDenied`). Elevated administrative execution is required to enable the global detector.

---

## 🚀 Next Steps Roadmap

### Phase 1: High Severity Mitigation (Immediate)
1. **Remount & Capture Reports:** Re-run the Prowler Docker container using explicit file volume mapping (`-v $(pwd)/prowler-results:/workspace`) to output the full CSV, HTML, and OCSF-JSON logs locally instead of allowing the `--rm` flag to discard them.
2. **Refactor Identity Access:** Deprecate direct policy assignment on `ankit-01`. Group users under a Least-Privilege IAM structure, strip excessive wildcard configurations, and mandate Multi-Factor Authentication (MFA).
3. **Execute Core Infrastructure Enforcements:** Encrypt active CloudTrail logs using a Customer Managed Key (CMK) via KMS and enable log-integrity validation checks.

### Phase 2: Medium & Low Priority Hardening
1. Leverage a break-glass administrative role to enable **AWS GuardDuty** across all active zones.
2. Systematically patch EC2 IMDS deployment models to enforce IMDSv2 and audit the remaining 28 lower-priority S3 alerts.
