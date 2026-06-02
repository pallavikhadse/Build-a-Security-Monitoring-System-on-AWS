# AWS Security Monitoring System & Real-Time Alerts

A zero-trust AWS security infrastructure designed to monitor unauthorized or critical access to sensitive credentials. This project demonstrates how to track secret access events using AWS CloudTrail, centralize logging with Amazon CloudWatch, and instantly broadcast real-time email notifications using AWS SNS (Simple Notification Service) when a threshold is breached.

---

##  Project Overview

The objective of this project is to implement an automated security boundary around AWS Secrets Manager. By establishing an immediate alert pipeline, the system minimizes the detection-to-remediation window from hours to seconds.

### Key Learning Objectives
* Implementing identity access management and tracking API activity.
* Auditing data events and resource-level actions via **AWS CloudTrail**.
* Building custom **CloudWatch Metric Filters** and **Alarms** using aggregate metrics (`Sum`).
* Structuring decoupled pub/sub alert architectures with **AWS SNS**.
* Interfacing with AWS resources utilizing both the **AWS Management Console** and **AWS CLI**.

---

##  Architecture & Services Used

* **AWS Secrets Manager**: Secure storage of database credentials, API keys, and OAuth tokens.
* **AWS CloudTrail**: Management and Data event tracking to log operational history.
* **AWS CloudWatch Logs**: Centralized storage and real-time interactive pattern filtering.
* **AWS CloudWatch Alarms**: Metric monitoring set to evaluate breaches using zero-trust logic.
* **AWS SNS (Simple Notification Service)**: Decoupled messaging topic delivering cross-channel alerts.
* **AWS CLI / CloudShell**: Command-line verification of API access events.

---

##  Implementation Steps

### Step 1: Create a Secret in AWS Secrets Manager
* Deployed a secret named `MySecretInfo` containing sensitive key-value pairs.
* Configured standard encryption utilizing the default `aws/secretsmanager` KMS key.

### Step 2: Set Up AWS CloudTrail Monitoring
* Configured a dedicated trail to capture **Data Events** and tracking resource-level actions.
* Enabled logging for API activities, ensuring both *Read-only* (e.g., fetching details) and *Write* (e.g., modifying configurations) activities are audited.

### Step 3: Configure CloudWatch Logs & Custom Metric Filters
* Configured CloudTrail to stream log events directly into a CloudWatch Log Group (`mysecretsmanager-loggroup`).
* Created a metric filter named `GetSecretsValue` to parse raw log strings for targeted pattern matches.
* Set the **Metric Value** to `1` (incrementing the counter per access attempt) and the **Default Value** to `0` to maintain a contiguous baseline on dashboard graphs.

### Step 4: Establish a Zero-Trust CloudWatch Alarm
* Configured an alarm based on the custom metric filter.
* **Logic Rules Applied**: 
  * **Statistic**: `Sum` (Evaluated over a `1-minute` period for near real-time detection).
  * **Condition**: Greater than or equal to ($\ge$) `1`.
* **Security Outcome**: 
  * `0` Accesses = Normal State (`OK`)
  * `1+` Accesses = Security Breach State (`ALARM`)

### Step 5: Configure the AWS SNS Notification Pipeline
* Created a standard SNS topic named `SecurityAlarms`.
* Subscribed an administrative email address to the topic and verified the subscription link to grant delivery permissions.
* Linked the CloudWatch Alarm action state directly to the SNS topic ARN.

---

##  Verification & Testing

To test the resilience of the pipeline, the secret was deliberately retrieved via the **AWS CLI / CloudShell**:

```bash
aws secretsmanager get-secret-value --secret-id "MySecretInfo" --region us-east-1
