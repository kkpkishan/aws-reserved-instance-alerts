# Reserved Instance Expiry Monitor

This repository contains an AWS CloudFormation template and Lambda function to monitor expiring Reserved Instances across all AWS regions and send alerts via Amazon SNS. The solution is designed to help you proactively manage your Reserved Instances and avoid service interruptions or unexpected costs due to expirations.

---

## How It Works

1. **Cross-Region Monitoring**:
   - The Lambda function fetches Reserved Instance (RI) details from all AWS regions.
   - It identifies RIs that will expire within the next 3 days.

2. **Alerts**:
   - When expiring RIs are detected, a detailed alert is sent to the Amazon SNS topic specified in the `ParentAlertStack` parameter.
   - The alert includes:
     - Region
     - Reserved Instance ID
     - Expiry date
     - Instance type
     - Availability zone
     - Number of instances

3. **Scheduled Execution**:
   - The function is triggered daily by an Amazon EventBridge rule at 2:30 AM UTC.

---

## Prerequisites

- An existing SNS topic for notifications, deployed via the [Parent Alert Stack](https://github.com/kkpkishan/AWS-SNS/blob/master/alert.yaml).
- The `ParentAlertStack` must export the SNS topic ARN as `${ParentAlertStack}-TopicARN`.

---

## Deployment

1. Clone the repository:

   ```bash
   git clone https://github.com/kkpkishan/aws-reserved-instance-alerts.git
   cd aws-reserved-instance-alerts
   ```

2. Deploy the CloudFormation stack:

   ```bash
   aws cloudformation deploy \
       --template-file ReservedInstance.yaml \
       --stack-name ReservedInstanceExpiryMonitor \
       --parameter-overrides ParentAlertStack=<ParentAlertStackName>
   ```

3. Verify that the stack is created successfully.

---

## Example Alert

If Reserved Instances are found expiring within the next 3 days, you will receive an alert email like this:

**Subject**: Reserved Instance Expiry Alert

**Message**:
```
Expiring Reserved Instances across regions:

Region: us-east-1, ID: ri-12345abc, Expiry: 2024-12-13 23:59:59, Type: m5.large, AZ: us-east-1a, Count: 3
Region: us-west-2, ID: ri-67890xyz, Expiry: 2024-12-14 11:00:00, Type: c5.xlarge, AZ: us-west-2b, Count: 1
```

---

## Parameters

| Name              | Description                                                                                   | Required | Default   |
|-------------------|-----------------------------------------------------------------------------------------------|----------|-----------|
| ParentAlertStack  | Stack name of the parent alert stack exporting the SNS topic ARN.                            | Yes      | N/A       |

---

## Architecture

- **Lambda Function**:
  - Written in Python, runs daily, and fetches RI details from all regions.
  - Sends notifications to the specified SNS topic if expiring RIs are found.

- **EventBridge Rule**:
  - Triggers the Lambda function at a scheduled time (2:30 AM UTC).

- **IAM Role**:
  - Grants the necessary permissions for EC2, SNS, and Lambda operations.

---

## Customization

- Modify the `warning_date` threshold in the Lambda function (`index.lambda_handler`) if you want a different lead time for alerts.
- Adjust the `ScheduleExpression` in the EventBridge rule to change the frequency of monitoring.



## Repository Link

[GitHub Repository](https://github.com/kkpkishan/aws-reserved-instance-alerts) 
