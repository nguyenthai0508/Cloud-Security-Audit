# Cloud Security Audit — AWS

A hands-on cloud security audit project: deploy a deliberately misconfigured
AWS lab, audit it with [Prowler](https://github.com/prowler-cloud/prowler)
(open-source CSPM), fix every finding, and prove the improvement with a
before/after scan. Built as a personal portfolio project for a CV.

## What this proves

- You can run a real cloud audit end-to-end: scope → scan → triage → remediate → rescan → report.
- You understand the CIS AWS Foundations Benchmark and the most common
  AWS misconfigurations (public S3, open security groups, no MFA, no CloudTrail).
- You can write infrastructure as code (Terraform) and document findings like
  a consultant.

## Architecture

```
AWS account (dedicated lab account, free tier, us-east-1)
├── S3 bucket (public policy, no encryption, no versioning)   ← FINDING
├── EC2 t2.micro + security group open 0.0.0.0/0 on 22, 3389 ← FINDING
├── IAM user with AdministratorAccess, console login, no MFA ← FINDING
├── Weak account password policy (min length 6)              ← FINDING
└── No CloudTrail trail                                       ← FINDING

Prowler (read-only profile) ──► HTML/CSV/JSON reports in output/
```

## How to run

```powershell
# 1. AWS account: enable MFA on root, set a $5 billing alarm (see PLAN.md)
# 2. Install tools
pip install prowler==5.47.0
aws configure --profile prowler-readonly   # read-only keys (SecurityAudit policy)
terraform init && terraform apply          # terraform/  (creates the vulnerable lab)

# 3. Baseline scan (BEFORE fixes)
.\scripts\run_prowler.ps1 -Stage baseline

# 4. Fix every finding (see checks.md + report/TEMPLATE.md)

# 5. Rescan (AFTER fixes) and compare
.\scripts\run_prowler.ps1 -Stage after

# 6. Destroy the lab when done
terraform destroy
```

## Safety rules (read first)

- Use a **dedicated AWS account** (or a fresh free-tier account). Never run
  this in an account with real data.
- Enable **MFA on the root user** and a **billing alarm** before creating
  anything. Everything here fits the free tier, but an alarm is mandatory.
- Prowler runs with a **read-only** profile. Never give it admin keys.
- `terraform destroy` at the end. An open security group left running is a
  real risk, not a lab exercise.

## Repo layout

```
README.md                  this file
PLAN.md                    2-week plan (Vietnamese)
checks.md                  the 10 target findings (Prowler check IDs)
terraform/                 intentionally vulnerable lab (Terraform)
scripts/                   Prowler scan commands (PowerShell + bash)
report/TEMPLATE.md         audit report template (English)
output/                    scan results — baseline/ and after/ (gitignored)
```

## Results (fill in as you go)

| Scan | Date | FAIL | PASS | Notes |
|------|------|------|------|-------|
| Baseline | | | | |
| After fixes | | | | |

## Tools

- Prowler 5.47.0 (pinned) — CIS AWS Foundations Benchmark checks
- Terraform ~> 1.5, AWS provider ~> 5.0
- AWS CLI v2
