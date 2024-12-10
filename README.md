# Reserved Instance Expiry Monitor

This repository contains an AWS CloudFormation template and Lambda function to monitor expiring Reserved Instances across all AWS regions and send alerts via Amazon SNS. The solution helps you proactively manage your Reserved Instances and avoid unexpected costs or service interruptions.

---

## How It Works

1. **Cross-Region Monitoring**:
   - The Lambda function retrieves Reserved Instance (RI) details from all AWS regions.
   - It identifies RIs that will expire within a configurable number of days.

2. **Alerts**:
   - When expiring RIs are detected, a detailed alert is sent to the Amazon SNS topic specified in the `ParentAlertStack` parameter.
   - The alert includes:
     - AWS Region
     - Reserved Instance ID
     - Expiry date
     - Instance type
     - Availability zone
     - Number of instances

3. **Scheduled Execution**:
   - The function is triggered daily by an Amazon EventBridge rule at 2:30 AM UTC.

4. **Configurable Warning Period**:
   - The number of days before expiration to trigger alerts can be specified during deployment via the `WarningDays` parameter.

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
       --parameter-overrides ParentAlertStack=<ParentAlertStackName> WarningDays=5
   ```

   Replace `<ParentAlertStackName>` with the name of your parent alert stack, and adjust `WarningDays` to your desired threshold (default is 3 days).

3. Verify that the stack is created successfully.

---

## Example Alert

If Reserved Instances are found expiring within the warning period, you will receive an alert email like this:

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
| WarningDays       | Number of days before expiration to trigger alerts.                                           | Yes       | 3         |

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

- Modify the `WarningDays` parameter during deployment to adjust the lead time for alerts.
- Adjust the `ScheduleExpression` in the EventBridge rule to change the frequency of monitoring.

---



## Repository Link

[GitHub Repository](https://github.com/kkpkishan/aws-reserved-instance-alerts.git)

---

For questions or support, please open an issue in the GitHub repository. Happy monitoring! 🚀
