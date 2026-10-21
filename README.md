# AWS Cloud Operations & Incident Management System

## Overview

A hands-on AWS operations project demonstrating EC2 web-server administration, centralized Apache log collection, custom CloudWatch metrics, alarms, and SNS email notifications.

The lab uses an Amazon Linux 2023 EC2 instance running Apache HTTP Server in the Sydney AWS Region (`ap-southeast-2`).

## Architecture

<img width="1536" height="1024" alt="AWS Cloud Operations Monitoring Architecture" src="https://github.com/user-attachments/assets/e5262654-bdfc-4c06-b130-b51dc1777264" />


**Event flow:** HTTP request → Apache on EC2 → CloudWatch Logs → metric filter → `HTTP5xxCount` → CloudWatch alarm → SNS email notification.

A separate CloudWatch alarm monitors EC2 CPU utilization.

## AWS services and resources

| Service | Purpose | Lab resource |
|---|---|---|
| Amazon EC2 | Hosts the Apache web server | `aws-ops-lab` |
| Amazon CloudWatch Logs | Centralizes Apache access and error logs | `/aws/ec2/aws-ops-lab/apache-access`, `/aws/ec2/aws-ops-lab/apache-error` |
| CloudWatch Metrics | Tracks CPU and custom HTTP 5xx counts | `AWSOpsLab/Apache` → `HTTP5xxCount` |
| CloudWatch Alarms | Evaluates monitoring thresholds | `aws-ops-lab-high-cpu`, `aws-ops-lab-http-5xx-errors` |
| Amazon SNS | Delivers alarm email notifications | `aws-ops-lab-alerts` |
| IAM | Grants the EC2 CloudWatch Agent permission to publish telemetry | EC2 instance role with `CloudWatchAgentServerPolicy` |

## Monitoring configuration

- **CPU alarm:** `aws-ops-lab-high-cpu`; configured to detect high EC2 CPU utilization.
- **HTTP 5xx alarm:** `aws-ops-lab-http-5xx-errors`; monitors `HTTP5xxCount` using Sum over a 1-minute period and a threshold of at least 1.
- **Custom metric filter:** `Apache-HTTP-5xx-Errors`, on `/aws/ec2/aws-ops-lab/apache-access`.
- **SNS topic:** `aws-ops-lab-alerts`.

Confirm current alarm settings in the AWS Console before reproducing this lab; configuration may change over time.

## Validation performed

- Verified Apache was active on EC2 and served the site.
- Confirmed Apache access logs were ingested into CloudWatch Logs.
- Used CloudWatch Logs Insights to count HTTP status codes. During one observed query window, results included `200`, `404`, and `500`; counts vary with the selected time range and incoming requests.
- Generated a real HTTP 500 response from a temporary Apache test rule and observed the matching access-log entry.
- Observed a datapoint for the `HTTP5xxCount` custom metric.
- Confirmed CloudWatch alarm history recorded a transition to `In alarm` and successful execution of the SNS action.
- Received an SNS email during alarm testing. A manual `set-alarm-state` test was also performed, so correlate the real HTTP 500 datapoint with alarm history before claiming the real request alone caused a particular transition.
- Removed the temporary Apache test configuration after validation.

## Incident investigation workflow

1. Review the CloudWatch alarm and its state history.
2. Inspect the relevant Apache access/error log streams.
3. Use Logs Insights to identify the request path and HTTP status.
4. Correlate the log timestamp with the `HTTP5xxCount` datapoint and alarm evaluation window.
5. Check the SNS action history and email delivery.
6. Remove test configuration and verify normal service operation.
7. Record findings and follow-up actions in `incident-report.md`.

## Security and cost notes

- Restrict inbound SSH (TCP 22) to a trusted public IP; do not expose SSH to `0.0.0.0/0`.
- Keep only required inbound ports open. HTTP (TCP 80) is public in this lab.
- Use an EC2 IAM role instead of storing AWS credentials on the instance.
- Remove temporary test configuration after testing.
- Stop EC2 when the lab is not in use. EBS storage and public IPv4 address charges may still apply while the instance is stopped; review current AWS pricing and account billing.

## Repository contents

- `README.md` 
- `architecture.png` 
- `incident-report.md` 
- `screenshots/` 

## Suggested improvements

- Provision the infrastructure with AWS CloudFormation or Terraform.
- Add an alarm for HTTP 5xx rate, not only count.
- Add a runbook for common incidents and automated remediation.
- Add dashboard widgets for both alarm states and log-derived metrics.

