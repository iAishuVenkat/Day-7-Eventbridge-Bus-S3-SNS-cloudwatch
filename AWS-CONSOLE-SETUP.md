# AWS Console Setup for EventBridge S3 Notifications

**This is the RECOMMENDED approach** - it's visual, easier to understand, and handles permissions automatically.

## Prerequisites
- AWS account with access to S3, EventBridge, SNS, and CloudWatch
- Email address to receive notifications

---

## Step 1: Create S3 Bucket

1. **Go to S3 Console:** https://s3.console.aws.amazon.com
2. **Click "Create bucket"**
3. **Bucket name:** `my-eventbridge-test-bucket-[random-number]` (must be globally unique)
4. **Region:** `US East (N. Virginia) us-east-1`
5. **Leave other settings as default**
6. **Click "Create bucket"**

---

## Step 2: Create SNS Topic

1. **Go to SNS Console:** https://console.aws.amazon.com/sns
2. **Click "Topics" in left sidebar**
3. **Click "Create topic"**
4. **Type:** Standard
5. **Name:** `s3-upload-notifications`
6. **Click "Create topic"**
7. **Copy the Topic ARN** - you'll need it later

---

## Step 3: Subscribe Your Email to SNS Topic

1. **In the SNS topic you just created**
2. **Click "Create subscription"**
3. **Protocol:** Email
4. **Endpoint:** Enter your email address
5. **Click "Create subscription"**
6. **🚨 CRITICAL:** Check your email and click "Confirm subscription"

---

## Step 4: Create CloudWatch Log Group

1. **Go to CloudWatch Console:** https://console.aws.amazon.com/cloudwatch
2. **Click "Logs" → "Log groups" in left sidebar**
3. **Click "Create log group"**
4. **Log group name:** `/s3-uploads/eventbridge`
5. **Click "Create"**

---

## Step 5: Enable EventBridge on S3 Bucket (CRITICAL STEP)

1. **Go back to S3 Console**
2. **Click your bucket name**
3. **Click "Properties" tab**
4. **Scroll down to "Amazon EventBridge" section** (NOT "Event notifications")
5. **Click "Edit"**
6. **Select "On" to send notifications to EventBridge**
7. **Click "Save changes"**

### ⚠️ Important Note
**Do NOT use the "Event notifications" section** - that's for direct integrations (SNS, SQS, Lambda). 
**Use the "Amazon EventBridge" section** - that's what sends events to EventBridge.

---

## Step 6: Create EventBridge Rule

1. **Go to EventBridge Console:** https://console.aws.amazon.com/events
2. **Click "Rules" in left sidebar**
3. **Click "Create rule"**

### Rule Details:
- **Name:** `s3-eventbridge-rule`
- **Description:** `Route S3 upload events to SNS and CloudWatch`
- **Event bus:** `default`
- **Rule type:** `Rule with an event pattern`
4. **Click "Next"**

---

## Step 7: Configure Event Pattern

1. **Event source:** Select "AWS services"
2. **AWS service:** Select "Simple Storage Service (S3)"
3. **Event type:** Select "Object Created"
4. **Specific bucket(s):** Enter your bucket name (e.g., `my-eventbridge-test-bucket-12345`)
5. **Click "Next"**

---

## Step 8: Add SNS Target

1. **Target types:** Select "AWS service"
2. **Select a target:** Choose "SNS topic"
3. **Topic:** Select `s3-upload-notifications`
4. **Message group ID:** Leave empty
5. **Message deduplication ID:** Leave empty
6. **Click "Add another target"**

---

## Step 9: Add CloudWatch Logs Target

1. **Target types:** Select "AWS service"
2. **Select a target:** Choose "CloudWatch log group"
3. **Log group:** Enter `/s3-uploads/eventbridge`
4. **Click "Next"**

---

## Step 10: Configure Tags and Review

1. **Tags:** Skip this (click "Next")
2. **Review all settings:**
   - Rule name: `s3-eventbridge-rule`
   - Event pattern: S3 Object Created from your specific bucket
   - Targets: SNS topic + CloudWatch log group
3. **Click "Create rule"**

---

## Step 11: Test the Setup

### Upload Test Files:
1. **Go back to S3 Console**
2. **Click your bucket**
3. **Click "Upload"**
4. **Add files** or drag and drop test files
5. **Click "Upload"**

### Expected Results (within 2-3 minutes):
- **Email notification** to your subscribed email address
- **CloudWatch logs** showing the S3 event details

---

## Step 12: Verify Results

### Check Email:
- Look for AWS notification emails
- Each file upload should trigger a separate email
- Email contains JSON with file details (bucket, key, size, timestamp)

### Check CloudWatch Logs:
1. **Go to CloudWatch Console**
2. **Logs → Log groups**
3. **Click `/s3-uploads/eventbridge`**
4. **Click the latest log stream**
5. **You should see S3 event data in JSON format**

---

## Step 13: View EventBridge Metrics

1. **Go to EventBridge Console**
2. **Click "Rules"**
3. **Click your rule name: `s3-eventbridge-rule`**
4. **Click "Metrics" tab**
5. **You should see "Matched events" and "Successful invocations"**

---

## Troubleshooting

### No Email Notifications:
- ✅ Confirm email subscription in SNS
- ✅ Check spam folder
- ✅ Verify SNS topic is selected as target
- ✅ Wait 5-10 minutes (sometimes delayed)

### No CloudWatch Logs:
- ✅ Verify log group name is correct: `/s3-uploads/eventbridge`
- ✅ Check EventBridge rule targets
- ✅ Ensure S3 EventBridge is enabled (not Event notifications)

### No Events at All:
- ✅ Verify "Amazon EventBridge" is enabled on S3 bucket
- ✅ Check EventBridge rule event pattern matches your bucket
- ✅ Ensure files are actually uploading to S3

---

## What You Built

🎉 **A complete event-driven system:**
- **S3 upload** → **EventBridge** → **Multiple targets**
- **Fan-out pattern:** One event triggers multiple independent actions
- **Real-time notifications:** Email + detailed logging
- **Serverless:** No servers to manage, scales automatically
- **Production-ready:** This pattern is used in real applications

## Next Steps

- Try uploading different file types
- Add more targets (Lambda functions, SQS queues)
- Modify event patterns to filter specific files
- Set up alerts based on CloudWatch logs

This demonstrates the power of EventBridge for building loosely coupled, event-driven architectures!