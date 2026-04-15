# Setting Up EventBridge S3 Notifications (Console Method)

For those who prefer clicking through the AWS console instead of typing commands, this guide provides a visual approach. The console method makes it easier to see what's happening when learning something new.

## What We're Building

Same thing as the CLI version - upload a file to S3, get email notifications and CloudWatch logs automatically. The console just makes it easier to see what's happening.

## Before Starting

- AWS account with console access
- Email address for notifications
- About 20 minutes

---

## Step 1: Create S3 Bucket

1. Go to **S3 Console**: https://s3.console.aws.amazon.com
2. Click **"Create bucket"**
3. **Bucket name**: `my-eventbridge-test-bucket-[add-some-numbers]` (needs to be globally unique)
4. **Region**: `US East (N. Virginia) us-east-1`
5. Leave everything else default
6. Click **"Create bucket"**

Write down your bucket name - you'll need it later.

---

## Step 2: Set Up Email Notifications

1. Go to **SNS Console**: https://console.aws.amazon.com/sns
2. Click **"Topics"** in the sidebar
3. Click **"Create topic"**
4. **Type**: Standard
5. **Name**: `s3-upload-notifications`
6. Click **"Create topic"**

Now subscribe your email:
1. In the topic you just created, click **"Create subscription"**
2. **Protocol**: Email
3. **Endpoint**: Your email address
4. Click **"Create subscription"**

**Important**: Check your email right now and click "Confirm subscription". Don't skip this!

---

## Step 3: Create CloudWatch Log Group

1. Go to **CloudWatch Console**: https://console.aws.amazon.com/cloudwatch
2. Click **"Logs"** → **"Log groups"** in sidebar
3. Click **"Create log group"**
4. **Log group name**: `/s3-uploads/eventbridge`
5. Click **"Create"**

---

## Step 4: Enable EventBridge on S3 (The Important Part)

Here's the key distinction. S3 has two notification systems, and we need the right one.

1. Go back to **S3 Console**
2. Click your bucket name
3. Click **"Properties"** tab
4. Scroll down to **"Amazon EventBridge"** section (NOT "Event notifications")
5. Click **"Edit"**
6. Select **"On"** to send notifications to EventBridge
7. Click **"Save changes"**

The key is using "Amazon EventBridge" section, not "Event notifications". Event notifications go directly to SNS/SQS/Lambda. EventBridge integration goes to EventBridge first, then we can route it anywhere.

---

## Step 5: Create EventBridge Rule

1. Go to **EventBridge Console**: https://console.aws.amazon.com/events
2. Click **"Rules"** in sidebar
3. Click **"Create rule"**

**Rule details:**
- **Name**: `s3-eventbridge-rule`
- **Description**: `Route S3 upload events to SNS and CloudWatch`
- **Event bus**: `default`
- **Rule type**: `Rule with an event pattern`
4. Click **"Next"**

**Event pattern:**
1. **Event source**: Select "AWS services"
2. **AWS service**: Select "Simple Storage Service (S3)"
3. **Event type**: Select "Object Created"
4. **Specific bucket(s)**: Enter your bucket name
5. Click **"Next"**

---

## Step 6: Add Email Target

1. **Target types**: Select "AWS service"
2. **Select a target**: Choose "SNS topic"
3. **Topic**: Select `s3-upload-notifications`
4. Click **"Add another target"**

---

## Step 7: Add CloudWatch Logs Target

1. **Target types**: Select "AWS service"
2. **Select a target**: Choose "CloudWatch log group"
3. **Log group**: Enter `/s3-uploads/eventbridge`
4. Click **"Next"**

---

## Step 8: Review and Create

1. Skip tags (click **"Next"**)
2. Review everything:
   - Rule matches S3 Object Created events from your bucket
   - Two targets: SNS topic + CloudWatch log group
3. Click **"Create rule"**

---

## Step 9: Test It Out

1. Go back to **S3 Console**
2. Click your bucket
3. Click **"Upload"**
4. Add a few test files (any files work)
5. Click **"Upload"**

Within 2-3 minutes you should get email notifications for each file.

---

## Step 10: Check Results

**Email**: Look for AWS notification emails with JSON containing file details

**CloudWatch Logs**:
1. Go to **CloudWatch Console**
2. **Logs** → **Log groups**
3. Click `/s3-uploads/eventbridge`
4. Click the latest log stream
5. You should see S3 event data in JSON format

---

## Step 11: View EventBridge Metrics

1. Go to **EventBridge Console**
2. Click **"Rules"**
3. Click your rule name: `s3-eventbridge-rule`
4. Click **"Metrics"** tab
5. You should see "Matched events" and "Successful invocations"

This shows EventBridge is working and routing events properly.

---

## Troubleshooting

**No email notifications?**
- Did you confirm the email subscription in SNS?
- Check your spam folder
- Wait 5-10 minutes, there can be a brief delay

**No CloudWatch logs?**
- Make sure you enabled "Amazon EventBridge" on the S3 bucket (not "Event notifications")
- Check the EventBridge rule is enabled
- Verify the log group name is exactly `/s3-uploads/eventbridge`

**Nothing working?**
- Double-check you're using the "Amazon EventBridge" section in S3 Properties
- Try uploading another file and wait a few minutes
- Make sure all resources are in the same region (us-east-1)

---

## Cleaning Up

When you're done experimenting:

**EventBridge**:
1. Go to EventBridge Console → Rules
2. Select your rule → Delete

**SNS**:
1. Go to SNS Console → Topics
2. Select your topic → Delete

**CloudWatch**:
1. Go to CloudWatch Console → Log groups
2. Select `/s3-uploads/eventbridge` → Delete

**S3**:
1. Go to S3 Console
2. Empty your bucket first (select all objects → Delete)
3. Then delete the bucket

---

## What You Built

This is a real event-driven system that shows up in production applications everywhere:

- **File processing pipelines** - automatically scan, convert, or analyze uploaded files
- **Backup monitoring** - get notified when backup jobs complete
- **Content workflows** - alert teams when new content arrives
- **Compliance logging** - maintain audit trails for all file operations

The pattern is: one event triggers multiple independent actions. EventBridge makes this easy to set up and maintain.

## Why EventBridge vs Direct S3 Notifications

**Direct S3 notifications** (the "Event notifications" section):
- S3 → SNS/SQS/Lambda directly
- One event type can only go to one target type
- Simpler but limited

**EventBridge integration** (what we used):
- S3 → EventBridge → Multiple targets of different types
- Can filter, transform, and route events
- More powerful for complex workflows

Once you understand this difference, you can build much more sophisticated systems. EventBridge is the backbone of modern serverless architectures.