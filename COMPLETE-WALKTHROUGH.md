# Complete EventBridge S3 Notifications Walkthrough

**UPDATED WITH WORKING SOLUTION** - Follow these exact steps for guaranteed success.

## Prerequisites

✅ AWS CLI installed and configured  
✅ Valid AWS account with permissions for S3, EventBridge, SNS, CloudWatch  
✅ Email address to receive notifications  

---

## CRITICAL UNDERSTANDING

**The key to success:** S3 has TWO different notification systems:

1. **"Event notifications"** → Direct to SNS/SQS/Lambda (simpler, limited)
2. **"Amazon EventBridge"** → To EventBridge service (powerful, fan-out)

**We use #2 for this tutorial!**

---

## Step 1: Create S3 Bucket

Copy and paste this command (it creates a unique bucket name):

```bash
aws s3 mb s3://my-eventbridge-test-bucket-$(date +%s) --region us-east-1
```

**Expected Output:**
```
make_bucket: my-eventbridge-test-bucket-1642248645
```

**📝 IMPORTANT:** Write down your bucket name! You'll need it in later steps.  
**Your bucket name:** `_________________________`

---

## Step 2: Create SNS Topic

```bash
aws sns create-topic --name s3-upload-notifications --region us-east-1
```

**Expected Output:**
```json
{
    "TopicArn": "arn:aws:sns:us-east-1:123456789012:s3-upload-notifications"
}
```

**📝 IMPORTANT:** Copy the TopicArn! You'll need it multiple times.  
**Your TopicArn:** `_________________________`

---

## Step 3: Create CloudWatch Log Group

```bash
aws logs create-log-group --log-group-name /s3-uploads/eventbridge --region us-east-1
```

**Expected Output:** (No output means success)

---

## Step 4: Subscribe Your Email to SNS

Replace `YOUR-EMAIL@example.com` with your actual email and `TOPIC-ARN` with the ARN from Step 2:

```bash
aws sns subscribe \
  --topic-arn TOPIC-ARN \
  --protocol email \
  --notification-endpoint YOUR-EMAIL@example.com \
  --region us-east-1
```

**Example:**
```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:s3-upload-notifications \
  --protocol email \
  --notification-endpoint john@example.com \
  --region us-east-1
```

**Expected Output:**
```json
{
    "SubscriptionArn": "pending confirmation"
}
```

**🚨 CRITICAL:** Check your email NOW and click "Confirm subscription" before continuing!

---

## Step 5: Enable EventBridge on S3 Bucket (THE KEY STEP)

Replace `YOUR-BUCKET-NAME` with your bucket name from Step 1:

```bash
aws s3api put-bucket-notification-configuration \
  --bucket YOUR-BUCKET-NAME \
  --notification-configuration '{"EventBridgeConfiguration": {}}'
```

**Example:**
```bash
aws s3api put-bucket-notification-configuration \
  --bucket my-eventbridge-test-bucket-1642248645 \
  --notification-configuration '{"EventBridgeConfiguration": {}}'
```

**Expected Output:** (No output means success)

**⚠️ CRITICAL:** This is NOT the same as S3 Event Notifications. This enables S3 → EventBridge integration.

---

## Step 6: Create EventBridge Rule

```bash
aws events put-rule \
  --name s3-upload-rule \
  --event-pattern '{
    "source": ["aws.s3"],
    "detail-type": ["Object Created"],
    "detail": {
      "eventSource": ["s3.amazonaws.com"],
      "eventName": ["PutObject", "PostObject", "CopyObject", "CompleteMultipartUpload"]
    }
  }' \
  --region us-east-1
```

**Expected Output:**
```json
{
    "RuleArn": "arn:aws:events:us-east-1:123456789012:rule/s3-upload-rule"
}
```

---

## Step 7: Get Your AWS Account ID

```bash
aws sts get-caller-identity --query Account --output text
```

**Expected Output:**
```
123456789012
```

**📝 IMPORTANT:** Write down your Account ID!  
**Your Account ID:** `_________________________`

---

## Step 8: Add SNS as First Target

Replace `TOPIC-ARN` with your SNS topic ARN from Step 2:

```bash
aws events put-targets \
  --rule s3-upload-rule \
  --targets "Id"="1","Arn"="TOPIC-ARN" \
  --region us-east-1
```

**Example:**
```bash
aws events put-targets \
  --rule s3-upload-rule \
  --targets "Id"="1","Arn"="arn:aws:sns:us-east-1:123456789012:s3-upload-notifications" \
  --region us-east-1
```

**Expected Output:**
```json
{
    "FailedEntryCount": 0,
    "FailedEntries": []
}
```

---

## Step 9: Add CloudWatch Logs as Second Target

Replace `ACCOUNT-ID` with your account ID from Step 7:

```bash
aws events put-targets \
  --rule s3-upload-rule \
  --targets "Id"="2","Arn"="arn:aws:logs:us-east-1:ACCOUNT-ID:log-group:/s3-uploads/eventbridge" \
  --region us-east-1
```

**Example:**
```bash
aws events put-targets \
  --rule s3-upload-rule \
  --targets "Id"="2","Arn"="arn:aws:logs:us-east-1:123456789012:log-group:/s3-uploads/eventbridge" \
  --region us-east-1
```

**Expected Output:**
```json
{
    "FailedEntryCount": 0,
    "FailedEntries": []
}
```

---

## Step 10: Grant EventBridge Permission to Publish to SNS

Replace `TOPIC-ARN` with your SNS topic ARN from Step 2:

```bash
aws sns set-topic-attributes \
  --topic-arn TOPIC-ARN \
  --attribute-name Policy \
  --attribute-value '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {"Service": "events.amazonaws.com"},
        "Action": "sns:Publish",
        "Resource": "TOPIC-ARN"
      }
    ]
  }' \
  --region us-east-1
```

**Example:**
```bash
aws sns set-topic-attributes \
  --topic-arn arn:aws:sns:us-east-1:123456789012:s3-upload-notifications \
  --attribute-name Policy \
  --attribute-value '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {"Service": "events.amazonaws.com"},
        "Action": "sns:Publish",
        "Resource": "arn:aws:sns:us-east-1:123456789012:s3-upload-notifications"
      }
    ]
  }' \
  --region us-east-1
```

**Expected Output:** (No output means success)

---

## Step 11: Test by Uploading Files

Create test files and upload them. Replace `YOUR-BUCKET-NAME` with your bucket name from Step 1:

```bash
# Create test files
echo "Hello EventBridge!" > test-file.txt
echo "Second test file" > document.pdf
echo "Third test file" > image.jpg

# Upload files one by one
aws s3 cp test-file.txt s3://YOUR-BUCKET-NAME/ --region us-east-1
aws s3 cp document.pdf s3://YOUR-BUCKET-NAME/ --region us-east-1
aws s3 cp image.jpg s3://YOUR-BUCKET-NAME/ --region us-east-1
```

**Example:**
```bash
echo "Hello EventBridge!" > test-file.txt
echo "Second test file" > document.pdf
echo "Third test file" > image.jpg

aws s3 cp test-file.txt s3://my-eventbridge-test-bucket-1642248645/ --region us-east-1
aws s3 cp document.pdf s3://my-eventbridge-test-bucket-1642248645/ --region us-east-1
aws s3 cp image.jpg s3://my-eventbridge-test-bucket-1642248645/ --region us-east-1
```

**Expected Output for each upload:**
```
upload: ./test-file.txt to s3://my-eventbridge-test-bucket-1642248645/test-file.txt
```

---

## Step 12: Check Email Notifications

**Within 2-3 minutes**, you should receive email notifications for each file upload.

**Email Subject:** "AWS Notification Message"

**Email will contain:** JSON with file details including bucket name, file name, upload time, and event metadata.

---

## Step 13: Check CloudWatch Logs

```bash
aws logs describe-log-streams \
  --log-group-name /s3-uploads/eventbridge \
  --order-by LastEventTime \
  --descending \
  --region us-east-1
```

**Expected Output:**
```json
{
    "logStreams": [
        {
            "logStreamName": "2024/01/15/[$LATEST]abcd1234567890",
            "creationTime": 1642248645000,
            "lastEventTime": 1642248645000
        }
    ]
}
```

Copy the `logStreamName` and view the events:

```bash
aws logs get-log-events \
  --log-group-name /s3-uploads/eventbridge \
  --log-stream-name "LOG-STREAM-NAME-FROM-ABOVE" \
  --region us-east-1
```

**You should see:** Detailed S3 event data for each file upload.

---

## Step 14: Verify Everything Works

✅ **Email notifications received** for each upload  
✅ **CloudWatch logs show events** for each upload  
✅ **Both happen automatically** when you upload files  

**Test one more upload to confirm:**
```bash
echo "Final test" > final-test.txt
aws s3 cp final-test.txt s3://YOUR-BUCKET-NAME/ --region us-east-1
```

You should get both email notification AND see it in CloudWatch logs!

---

## Step 15: Clean Up (When Done Learning)

Replace `YOUR-BUCKET-NAME` and `TOPIC-ARN` with your actual values:

```bash
# Remove EventBridge targets
aws events remove-targets --rule s3-upload-rule --ids "1" "2" --region us-east-1

# Delete EventBridge rule
aws events delete-rule --name s3-upload-rule --region us-east-1

# Delete SNS topic
aws sns delete-topic --topic-arn TOPIC-ARN --region us-east-1

# Delete CloudWatch log group
aws logs delete-log-group --log-group-name /s3-uploads/eventbridge --region us-east-1

# Delete S3 bucket contents and bucket
aws s3 rm s3://YOUR-BUCKET-NAME --recursive --region us-east-1
aws s3 rb s3://YOUR-BUCKET-NAME --region us-east-1
```

---

## 🎉 Congratulations!

You've successfully built an event-driven system where:
- **One S3 upload** triggers **multiple actions**
- **Email notifications** keep you informed
- **CloudWatch logs** provide audit trail
- **No servers** or code required
- **Scales automatically** with your usage

## Key Learning Points

### S3 EventBridge Integration vs Event Notifications

**Event Notifications (Direct):**
- S3 Properties → Event notifications → SNS/SQS/Lambda
- One event type → One target type
- Simpler but limited

**EventBridge Integration (What we used):**
- S3 Properties → Amazon EventBridge → Enable
- EventBridge Rules → Multiple targets
- Fan-out pattern, filtering, transformation

### Fan-Out Pattern

One S3 upload event automatically triggers:
- Email notification (immediate awareness)
- CloudWatch logging (audit trail)
- Could easily add: Lambda processing, SQS queues, etc.

### Real-World Applications

This pattern is used for:
- **File processing workflows** - scan, transform, archive
- **Backup notifications** - alert when backups complete
- **Content management** - notify teams of new uploads
- **Compliance logging** - audit all file operations
- **Multi-step workflows** - trigger different processes

This is the foundation of modern serverless, event-driven architecture!