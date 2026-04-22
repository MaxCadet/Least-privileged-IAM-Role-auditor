# 🔐 Least Privilege IAM Auditor

> Automated AWS security auditing tool that scans IAM roles and users for overly permissive policies and delivers scheduled alert reports.

![AWS](https://img.shields.io/badge/AWS-Lambda%20%7C%20SNS%20%7C%20S3%20%7C%20EventBridge%20%7C%20IAM-orange?logo=amazonaws)
![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 📌 Overview

The **Least Privilege IAM Auditor** is a serverless security tool that automatically scans an AWS account for IAM roles and users with overly permissive policies — such as wildcard (`*`) actions or resources. It runs on a configurable schedule via EventBridge, generates a structured report, stores it in S3, and sends an SNS alert notification to subscribed recipients.

This project reflects a real-world cloud security use case aligned with the **principle of least privilege**, a foundational AWS security best practice.

---

## 🏗️ Architecture

```
EventBridge (Scheduled Rule)
        │
        ▼
   Lambda Function
   ┌─────────────────────────────┐
   │  1. List IAM Roles & Users  │
   │  2. Analyze attached policies│
   │  3. Flag wildcard violations │
   │  4. Generate audit report    │
   └─────────────────────────────┘
        │               │
        ▼               ▼
   S3 Bucket        SNS Topic
 (report stored)  (alert sent to
                  subscribed emails)
```

---
## ✨ Features

- **Wildcard detection** — Flags IAM policies containing `"Action": "*"` or `"Resource": "*"`
- **Scheduled audits** — Runs automatically on a configurable daily or weekly EventBridge schedule
- **S3 report storage** — Saves timestamped JSON audit reports to an S3 bucket for historical tracking
- **SNS notifications** — Sends an email alert summarizing flagged roles and users to subscribed recipients
- **Serverless architecture** — Fully event-driven with no persistent infrastructure to manage 

---

## 🛠️ Tech Stack

| Service | Purpose |
|---|---|
| AWS Lambda | Core audit logic (Python + boto3) |
| AWS IAM | Target of audits; also provides Lambda execution role |
| AWS EventBridge | Schedules Lambda on a daily/weekly cron |
| AWS SNS | Sends email alert reports to subscribers |
| AWS S3 | Stores timestamped JSON audit reports |
| Python (boto3) | IAM policy analysis logic |

---

## 🚀 Setup & Deployment

### Prerequisites

- AWS account with appropriate permissions
- AWS CLI configured (`aws configure`)
- Python 3.11+

### 1. Clone the repository

```bash
git clone https://github.com/MaxCadet/least-privilege-iam-auditor.git
cd least-privilege-iam-auditor
```

### 2. Create the S3 bucket for reports

```bash
aws s3 mb s3://iam-audit-reports-max
```

### 3. Create the SNS topic and subscribe your email

```bash
aws sns create-topic --iam-audit-reports-max
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-2:166665178530:IAM-audit-alerts \
  --protocol email \
  --notification-endpoint Cadetmax68@gmail.com
```

### 4. Deploy the Lambda function

```bash
zip function.zip lambda_function.py
aws lambda create-function \
  --function-name MyLambdaSecRole \
  --runtime python3.12 \
  --role arn:aws:iam::166665178530:role/MyLambdaSecRole \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

### 5. Set environment variables

```bash
aws lambda update-function-configuration \
  --function-name MyLambdaSecRole \
  --environment "Variables={iam-audit-reports-max,SNS_TOPIC_ARN=arn:aws:sns:us-east-1:166665178530:iam-audit-alerts}"
```

### 6. Schedule with EventBridge (daily at 9am UTC)

```bash
aws events put-rule \
  --name DailyIAMAudit \
  --schedule-expression "cron(0 9 * * ? *)"
```
```bash
aws events put-targets \
  --rule DailyIAMAduit \
  --targets "Id"="1","Arn"="arn:aws:lambda:us-east-2:166665178530:function:LambdaSecRole"
---

## 📂 Project Structure

```
least-privilege-iam-auditor/
├── lambda_function.py       # Core Lambda handler and policy analysis logic
├── requirements.txt         # Python dependencies
├── iam_policy/
│   └── lambda_execution_role.json  # IAM policy for Lambda execution role
└── README.md
```

---

## 🔒 IAM Permissions Required

The Lambda execution role needs the following permissions:

```json
{
  "Effect": "Allow",
  "Action": [
    "iam:ListRoles",
    "iam:ListUsers",
    "iam:ListAttachedRolePolicies",
    "iam:ListAttachedUserPolicies",
    "iam:GetPolicyVersion",
    "s3:PutObject",
    "sns:Publish"
  ],
  "Resource": "*"
}
```

---

## 📊 Sample Audit Report (JSON)

```json
{
  "audit_timestamp": "2026-04-03T08:00:00Z",
  "total_roles_scanned": 8,
  "total_users_scanned": 1,
  "violations": [
    {
      "type": "role",
      "name": "DevOpsAdminRole",
      "policy": "AdminAccess",
      "issue": "Action: * detected"
    },
    {
      "type": "user",
      "name": "ci-deploy-user",
      "policy": "InlineDeployPolicy",
      "issue": "Resource: * detected"
    }
  ]
}
```

---

## 💡 What I Learned

- How to programmatically analyze IAM policy documents using **boto3**
- Structuring **least privilege** execution roles for Lambda functions
- Using **EventBridge cron expressions** to schedule serverless workflows
- Decoupling notification logic from compute using **SNS**
- Storing structured audit data in **S3** for compliance and historical tracking

---

## 🔭 Future Improvement

- [ ] Add support for scanning inline policies (not just managed policies)
- [ ] Integrate with AWS Security Hub for centralized findings
- [ ] Build a simple dashboard using QuickSight or a static S3 site
- [ ] Add Terraform or CDK for infrastructure-as-code deployment

---

## 👤 Author

**Max** — [@MaxCadet](https://github.com/MaxCadet)
