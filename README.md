# EC2 Lifecycle Automation

## Overview
Serverless automation solution for managing EC2 instance lifecycles using AWS Lambda and EventBridge. This project reduces operational costs by up to 65% by automatically starting and stopping instances based on business hours, demonstrating proficiency in serverless architecture and cost optimization strategies.

## Architecture
```
EventBridge Rules → Lambda Functions → EC2 Instances
     (Cron)              (Python)         (Start/Stop)
                            ↓
                       CloudWatch Logs
```

<img width="13616" height="8005" alt="Image" src="https://github.com/user-attachments/assets/a7c7f0d8-e87b-4d5f-ada4-a606b4c22b29" />

## Technologies Used
- **AWS Services:** Lambda, EventBridge, IAM, EC2, CloudWatch
- **Language:** Python 3.x (Boto3 SDK)
- **Automation:** Event-driven serverless architecture
- **Monitoring:** CloudWatch Logs for execution tracking

## Features
- ⏰ Automated start/stop scheduling based on cron expressions
- 💰 Cost reduction through intelligent resource management
- 🔒 Secure IAM role-based access control
- 📊 CloudWatch logging for audit trails
- 🚀 Zero-maintenance serverless operation

## Business Value
**Cost Savings Example:**
- Instance Type: t3.medium ($0.0416/hour)
- Without automation: 24/7 operation = $30.34/month
- With automation: 12/5 operation = $10.83/month
- **Savings: $19.51/month (64.3%) per instance**

## Setup Instructions

### Prerequisites
- AWS Account with Lambda and EC2 permissions
- Python 3.x knowledge
- AWS CLI configured

### Step 1: Create IAM Roles
```bash
# Create role for Lambda execution
aws iam create-role --role-name EC2-Start-Lambda-Role \
  --assume-role-policy-document file://lambda-trust-policy.json

# Attach policy
aws iam attach-role-policy --role-name EC2-Start-Lambda-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
```

### Step 2: Deploy Lambda Functions

**Start Instance Function:**
```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    instance_id = 'i-xxxxxxxxxxxxxxxxx'  # Replace with your instance ID
    
    try:
        ec2.start_instances(InstanceIds=[instance_id])
        return {
            'statusCode': 200,
            'body': f'Successfully started instance {instance_id}'
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': f'Error starting instance: {str(e)}'
        }
```

**Stop Instance Function:**
```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    instance_id = 'i-xxxxxxxxxxxxxxxxx'  # Replace with your instance ID
    
    try:
        ec2.stop_instances(InstanceIds=[instance_id])
        return {
            'statusCode': 200,
            'body': f'Successfully stopped instance {instance_id}'
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': f'Error stopping instance: {str(e)}'
        }
```

### Step 3: Configure EventBridge Rules

**Start Schedule (Monday-Friday, 8 AM UTC):**
```
Cron Expression: 0 8 ? * MON-FRI *
Target: Lambda function (start-ec2-instance)
```

**Stop Schedule (Monday-Friday, 6 PM UTC):**
```
Cron Expression: 0 18 ? * MON-FRI *
Target: Lambda function (stop-ec2-instance)
```

### Step 4: Test the Automation
```bash
# Manually invoke Lambda function
aws lambda invoke --function-name start-ec2-instance output.json

# Check instance state
aws ec2 describe-instances --instance-ids i-xxxxxxxxxxxxxxxxx \
  --query 'Reservations[0].Instances[0].State.Name'
```

## IAM Policies

**Lambda Execution Policy:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:DescribeInstances"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}
```

## Monitoring & Verification
```bash
# View CloudWatch logs
aws logs tail /aws/lambda/start-ec2-instance --follow

# Check EventBridge rule status
aws events describe-rule --name start-ec2-daily

# View cost savings in Cost Explorer
# Navigate to AWS Console → Cost Explorer → Filter by EC2
```

## Error Handling
- Lambda functions include try-catch blocks for graceful failure handling
- CloudWatch alarms configured for failed executions
- SNS notifications for critical errors (optional)

## Scalability
This solution can be extended to:
- Manage multiple instances using tags
- Implement different schedules for dev/staging/prod environments
- Add approval workflows for production instances
- Integrate with Slack/Teams for notifications

## What I Learned
- Designing event-driven serverless architectures
- Implementing secure IAM policies with least privilege principle
- Calculating and demonstrating ROI for automation projects
- Using CloudWatch for operational monitoring and debugging

## Future Enhancements
- [ ] Tag-based instance management for bulk operations
- [ ] Add SNS notifications for execution status
- [ ] Implement Step Functions for complex orchestration
- [ ] Create CloudFormation template for one-click deployment
- [ ] Add cost tracking dashboard using QuickSight

---

**AWS Certified Solutions Architect**
