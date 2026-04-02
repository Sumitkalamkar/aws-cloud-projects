# AI-Ready Observability System 🔍

A fully automated AWS observability pipeline that detects EC2 CPU alerts and stores structured alarm data into DynamoDB — deployed entirely via CloudFormation.

---

## Architecture

```
EC2 Instance (CPU > 70%)
        ↓
CloudWatch Alarm
        ↓
SNS Topic (ObservabilityAlerts)
        ↓
SQS Queue (ObservabilityQueue)
        ↓
Lambda Function (ObservabilityProcessor)
        ↓
DynamoDB Table (ObservabilityAlertData)
```

---

## AWS Services Used

| Service | Purpose |
|---|---|
| CloudWatch | Monitors EC2 CPU utilization, triggers alarm |
| SNS | Receives alarm notification and fans out |
| SQS | Buffers messages reliably for Lambda |
| Lambda (Python 3.9) | Processes messages and extracts alert data |
| DynamoDB | Stores structured alert records |
| IAM | Least-privilege role for Lambda execution |

---

## What Gets Stored in DynamoDB

Each alarm event creates a record with:

| Field | Description |
|---|---|
| `alert_id` | Unique UUID for each alert |
| `alarm_name` | Name of the triggered CloudWatch alarm |
| `metric_name` | e.g. `CPUUtilization` |
| `instance_id` | EC2 instance that triggered the alarm |
| `threshold` | Alarm threshold value (e.g. `70`) |
| `actual_value` | Actual CPU % at time of alarm |
| `state` | `ALARM` or `OK` |
| `region` | AWS region |
| `timestamp` | When the alarm triggered |
| `ingested_at` | When Lambda processed the message |

---

## Deployment

### Prerequisites
- AWS account with sufficient IAM permissions
- An EC2 instance already running

### Steps

1. Open **AWS CloudFormation → Create Stack**
2. Upload `template.yaml`
3. Before deploying, replace the EC2 Instance ID in the template:
   ```yaml
   Value: i-xxxxxxxxxxxxxxxxx   # Replace with your EC2 Instance ID
   ```
4. Acknowledge IAM resource creation
5. Click **Submit**

---

## Testing

After deployment, manually trigger the alarm using AWS CloudShell:

```bash
# First, find your exact alarm name
aws cloudwatch describe-alarms \
  --region ap-south-1 \
  --query 'MetricAlarms[*].AlarmName'

# Then trigger it
aws cloudwatch set-alarm-state \
  --alarm-name "YOUR-ALARM-NAME" \
  --state-value ALARM \
  --state-reason "Manual test trigger"
```

Then verify in DynamoDB:
> DynamoDB → Tables → `ObservabilityAlertData` → Explore table items

You should see a new record appear within seconds ✅

---

## CloudFormation Resources Created

- `ObservabilityAlertData` — DynamoDB Table
- `ObservabilityAlerts` — SNS Topic
- `ObservabilityQueue` — SQS Queue
- `ObservabilityProcessor` — Lambda Function
- `LambdaSQSExecutionRole` — IAM Role
- `CPUAlarm` — CloudWatch Alarm

---

## Key Design Decisions

- **SQS between SNS and Lambda** — provides message buffering and retry on failure
- **RawMessageDelivery: true** — Lambda receives clean JSON without SNS envelope wrapper
- **SQS Queue Policy** — restricts `sqs:SendMessage` only to the SNS topic (secure)
- **PAY_PER_REQUEST billing** on DynamoDB — no capacity planning needed

---

## Region

Deployed in: `ap-south-1` (Mumbai)
