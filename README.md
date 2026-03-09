# Automating-AWS-Cost-Management-Reports-with-Lambda
An automated AWS Lambda Python script that audits your account for running resources, unattached volumes, and monthly costs, then emails you a detailed report. Costs just $0.01/month to run!

# AWS Monthly Cost & Resource Auditor ☁️💰

An automated, serverless Python script that runs on AWS Lambda to audit your AWS account. It checks for running instances, flags money-wasting idle resources, fetches your exact monthly bill, and emails you a clean summary report. 

Perfect for developers and small teams who want to avoid the dreaded "surprise AWS bill" at the end of the month.

## ✨ Features
* **💰 Cost Tracking:** Fetches the exact unblended cost from the previous month via AWS Cost Explorer.
* **🟢 EC2 Instances:** Lists all actively running EC2 instances and their types.
* **💾 EBS Volumes:** Counts total volumes and specifically flags **Unattached** volumes that are costing you money.
* **📸 EBS Snapshots:** Counts your total backup snapshots.
* **🌐 Elastic IPs:** Counts total EIPs and flags **Unassociated** IPs that AWS is actively billing you for.
* **🪣 S3 Buckets:** Lists all S3 buckets and their creation dates.
* **✉️ Email Delivery:** Bypasses AWS SES and uses pure Python `smtplib` to send the report directly via Gmail (or any SMTP server).

## 💸 Cost to Run
**$0.01 per month.**
* **Lambda & EventBridge:** $0.00 (Easily fits within the AWS Free Tier).
* **EC2/S3 API Calls:** $0.00 (Standard read/list queries are free).
* **Cost Explorer API:** $0.01 per API call. Running this script once a month costs exactly one penny.

## 🚀 Setup Instructions

### 1. Create the Lambda Function
* Go to AWS Lambda and create a new function.
* Runtime: **Python 3.10** (or higher).
* Architecture: x86_64.
* Go to **Configuration > General configuration** and increase the **Timeout to 30 seconds** (to allow time for the API calls and SMTP connection).

### 2. Configure IAM Permissions
Your Lambda function needs permission to read your billing and resource data. Attach an inline policy to your Lambda's Execution Role with the following JSON:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ce:GetCostAndUsage",
                "ec2:DescribeInstances",
                "ec2:DescribeVolumes",
                "ec2:DescribeSnapshots",
                "ec2:DescribeAddresses",
                "s3:ListAllMyBuckets"
            ],
            "Resource": "*"
        }
    ]
}
```

### 3. Set Environment Variables
Go to your Lambda function's Configuration > Environment variables tab and add the following keys securely (do not hardcode these in the script):

SENDER_EMAIL: Your Gmail address (e.g., you@gmail.com)

APP_PASSWORD: Your 16-character Google App Password (requires 2FA enabled on your Google account)

RECIPIENT_EMAIL: The email address where you want to receive the report

### 4. Deploy the Code
Copy the lambda_function.py from this repository and paste it into your Lambda code editor. Click Deploy.

### 5. Schedule with EventBridge
To automate this, add an EventBridge (CloudWatch Events) trigger to your Lambda function.

Create a new rule.

Set the schedule using a cron expression. Example for the 1st of every month at 12:00 PM UTC:
cron(0 12 1 * ? *)

### 📝 Example Output Email: 
## AWS Monthly Cost & Resource Report
==================================

💰 Total AWS Cost (2026-02-01 to 2026-03-01): $14.32

🟢 Running EC2 Instances:
  - WebServer-Production (i-0abcd1234efgh5678) | Type: t3.micro

💾 EBS Volumes: 3 Total
  - ⚠️ UNATTACHED (Wasting Money): 1

📸 EBS Snapshots: 5 Total

🌐 Elastic IPs: 2 Total
  - ⚠️ UNASSOCIATED (Wasting Money): 0

🪣 S3 Buckets (2 Total):
  - my-app-backups (Created: 2026-01-15)
  - my-website-assets (Created: 2026-02-20)

----------------------------------
Automated by AWS Lambda & EventBridge
