# S3 Image Object Detection with AWS Rekognition 🖼️

An event-driven serverless pipeline that automatically detects and labels objects in images when they are uploaded to an S3 bucket — powered by AWS Lambda and Amazon Rekognition.

---

## Architecture

```
User Uploads Image
        ↓
S3 Bucket (Image Storage)
        ↓
S3 Event Trigger
        ↓
Lambda Function (Python)
        ↓
Amazon Rekognition (detect_labels)
        ↓
Returns Detected Labels
```

---

## AWS Services Used

| Service | Purpose |
|---|---|
| S3 | Stores uploaded images, triggers Lambda on upload |
| Lambda (Python) | Processes the event, calls Rekognition |
| Amazon Rekognition | AI-powered object and scene detection |
| IAM | Permissions for Lambda to access S3 and Rekognition |

---

## How It Works

1. User uploads an image to the S3 bucket
2. S3 automatically triggers the Lambda function
3. Lambda reads the image bytes from S3
4. Image is sent to Amazon Rekognition's `detect_labels` API
5. Rekognition returns a list of detected objects/scenes
6. Lambda logs and returns the labels

### Example Output

If you upload a photo of a dog in a park:
```json
{
  "statusCode": 200,
  "body": "Object detection completed successfully - ['Dog', 'Animal', 'Park', 'Grass', 'Outdoor', 'Nature']"
}
```

---

## Setup Instructions

### Step 1 — Create S3 Bucket
- Go to **S3 → Create Bucket**
- Give it a unique name e.g. `my-rekognition-images`
- Keep all defaults, click **Create**

### Step 2 — Create Lambda Function
- Go to **Lambda → Create Function**
- Runtime: `Python 3.x`
- Paste the code from `lambda_function.py`

### Step 3 — Add IAM Permissions to Lambda Role
Your Lambda execution role needs these permissions:
```json
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "rekognition:DetectLabels",
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ],
  "Resource": "*"
}
```

### Step 4 — Add S3 Trigger to Lambda
- Go to Lambda → **Add Trigger**
- Select **S3**
- Choose your bucket
- Event type: `PUT` (triggered on upload)
- Click **Add**

---

## Testing

1. Upload any image (`.jpg`, `.png`) to your S3 bucket
2. Go to **Lambda → Monitor → View CloudWatch Logs**
3. You will see the detected labels printed in the logs

Or test manually using a test event in Lambda console:
```json
{
  "Records": [
    {
      "s3": {
        "bucket": { "name": "my-rekognition-images" },
        "object": { "key": "test-image.jpg" }
      }
    }
  ]
}
```

---

## Lambda Function Summary

| Component | Detail |
|---|---|
| Trigger | S3 PUT event |
| Runtime | Python 3.x |
| Key API | `rekognition.detect_labels()` |
| Input | Image bytes from S3 |
| Output | List of detected label names |

---

## Possible Extensions

- Store detected labels in **DynamoDB** for search/history
- Send alert via **SNS** if specific object is detected (e.g. "Person", "Weapon")
- Build a **search engine** on top — find all images containing "Car"
- Add **confidence score filtering** — only keep labels above 90% confidence

---

## Region

Deployed in: `ap-south-1` (Mumbai)
