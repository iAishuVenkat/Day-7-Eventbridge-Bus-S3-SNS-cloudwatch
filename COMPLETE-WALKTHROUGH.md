# Setting Up S3 Upload Notifications with EventBridge

I spent way too much time figuring this out, so here's the step-by-step process that actually works.

## What We're Building

Upload a file to S3 → EventBridge catches it → sends email + logs to CloudWatch

The tricky part is S3 has two different ways to send notifications, and most tutorials don't explain which one to use.

## Before You Start

You'll need:
- AWS account with basic permissions
- Email address for notifications
- AWS CLI installed and configured

If you don't have AWS CLI set up, run `aws configure` and enter your access keys.

---

## Step 1: Create Your S3 Bucket

```bash
aws s3 mb s3://my-eventbridge-test-bucket-$(date +%s) --region us-east-1
```

The `$(date +%s)` adds a timestamp to make the bucket name unique. Write down whatever bucket name it creates - you'll need it later.

---

## Step 2: Set Up SNS for Email Notifications

```bash
aws sns create-topic --name s3-upload-notifications --region us-east-1
```

Copy the TopicArn from the output. It looks like `arn:aws:sns:us-east-1:123456789012:s3-upload-notifications`

Now subscribe your email:
```bash
aws sns subscribe \
  --topic-arn YOUR-TOPIC-ARN-HERE \
  --protocol email \
  --notification-endpoint your-email@example.com \
  --region us-east-1
```

**Important:** Check your email right now and confirm the subscription. The rest won't work if you skip this.

---

## Step 3: Create CloudWatch Log Group

```bash
aws logs create-log-group --log-group-name /s3-uploads/eventbridge --region us-east-1
```

This is where EventBridge will log the S3 events.

---

## Step 4: The Key Step - Enable EventBridge on S3

This is where I got confused initially. S3 has two notification systems:

1. **Event notifications** (the obvious one) - goes directly to SNS/SQS/Lambda
2. **EventBridge integration** (the one we want) - goes to EventBridge first

We want #2 because it lets us send one event to multiple places.

```bash
aws s3api put-bucket-notification-configuration \
  --bucket YOUR-BUCKET-NAME \
  --notification-configuration '{"EventBridgeConfiguration": {}}'
```

Replace `YOUR-BUCKET-NAME` with your actual bucket name from step 1.

---

## Step 5: Create EventBridge Rule

Now we tell EventBridge what events to watch for:

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

This rule matches any S3 object creation events.

---

## Step 6: Connect EventBridge to Your Targets

First, get your AWS account ID:
```bash
aws sts get-caller-identity --query Account --output text
```

Add SNS as the first target:
```bash
aws events put-targets \
  --rule s3-upload-rule \
  --targets "Id"="1","Arn"="YOUR-TOPIC-ARN" \
  --region us-east-1
```

Add CloudWatch Logs as the second target:
```bash
aws events put-targets \
  --rule s3-upload-rule \
  --targets "Id"="2","Arn"="arn:aws:logs:us-east-1:YOUR-ACCOUNT-ID:log-group:/s3-uploads/eventbridge" \
  --region us-east-1
```

Replace `YOUR-TOPIC-ARN` and `YOUR-ACCOUNT-ID` with your actual values.

---

## Step 7: Fix Permissions

EventBridge needs permission to publish to your SNS topic:

```bash
aws sns set-topic-attributes \
  --topic-arn YOUR-TOPIC-ARN \
  --attribute-name Policy \
  --attribute-value '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {"Service": "events.amazonaws.com"},
        "Action": "sns:Publish",
        "Resource": "YOUR-TOPIC-ARN"
      }
    ]
  }' \
  --region us-east-1
```

---

## Step 8: Test It

Create some test files and upload them:

```bash
echo "Test file 1" > test1.txt
echo "Test file 2" > test2.txt

aws s3 cp test1.txt s3://YOUR-BUCKET-NAME/ --region us-east-1
aws s3 cp test2.txt s3://YOUR-BUCKET-NAME/ --region us-east-1
```

Within a couple minutes, you should get email notifications for each upload.

---

## Step 9: Check CloudWatch Logs

```bash
aws logs describe-log-streams \
  --log-group-name /s3-uploads/eventbridge \
  --order-by LastEventTime \
  --descending \
  --region us-east-1
```

If you see log streams, grab the latest one and check its contents:

```bash
aws logs get-log-events \
  --log-group-name /s3-uploads/eventbridge \
  --log-stream-name "STREAM-NAME-FROM-ABOVE" \
  --region us-east-1
```

You should see detailed JSON with all the S3 event info.

---

## Troubleshooting

**No email notifications?**
- Double-check you confirmed the SNS subscription
- Make sure you're using the right TopicArn
- Check your spam folder

**No CloudWatch logs?**
- Verify the EventBridge rule exists: `aws events describe-rule --name s3-upload-rule --region us-east-1`
- Check targets are configured: `aws events list-targets-by-rule --rule s3-upload-rule --region us-east-1`

**Still not working?**
- Wait 5-10 minutes - sometimes there's a delay
- Try uploading another file
- Make sure you enabled EventBridge on the S3 bucket (step 4)

---

## Cleaning Up

When you're done experimenting:

```bash
# Remove EventBridge stuff
aws events remove-targets --rule s3-upload-rule --ids "1" "2" --region us-east-1
aws events delete-rule --name s3-upload-rule --region us-east-1

# Delete SNS topic
aws sns delete-topic --topic-arn YOUR-TOPIC-ARN --region us-east-1

# Delete CloudWatch log group
aws logs delete-log-group --log-group-name /s3-uploads/eventbridge --region us-east-1

# Delete S3 bucket
aws s3 rm s3://YOUR-BUCKET-NAME --recursive --region us-east-1
aws s3 rb s3://YOUR-BUCKET-NAME --region us-east-1
```

---

## What You Just Built

This is a classic event-driven pattern. One S3 upload triggers multiple independent actions:
- Email notification (so you know it happened)
- CloudWatch logging (for monitoring and debugging)

In real applications, you might also trigger:
- Lambda functions to process the file
- SQS queues for batch processing
- Step Functions for complex workflows

The beauty of EventBridge is you can easily add more targets without changing the S3 configuration. That's why it's more powerful than direct S3 notifications.

## Why This Matters

Most modern applications are built this way - loosely coupled services that react to events. Understanding this pattern helps you build scalable, maintainable systems.

Plus, once you get comfortable with EventBridge, you can use it for way more than just S3 events. It's the backbone of serverless architectures.