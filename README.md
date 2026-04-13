# EventBridge S3 Upload Notifications

Learn EventBridge with a real-world example: Get email notifications when files are uploaded to S3.

## What You'll Build

When you upload a file to S3, EventBridge automatically:
- Sends you an email notification
- Logs event details to CloudWatch
- Demonstrates the fan-out pattern (one event → multiple actions)

## What You'll Learn

- S3 EventBridge integration vs direct event notifications
- EventBridge rules and event pattern matching
- Fan-out pattern with multiple targets
- Real event-driven architecture

## Files in This Project

### 1. **AWS-CONSOLE-SETUP.md** (Recommended for Beginners)
Complete visual guide using AWS Console. Easier to understand and handles permissions automatically.

### 2. **COMPLETE-WALKTHROUGH.md** (For CLI Users)
Detailed step-by-step CLI commands with explanations, expected outputs, and troubleshooting.

## Key Learning

**The Critical Difference:**
- S3 "Event notifications" → Direct to SNS/SQS/Lambda (limited)
- S3 "Amazon EventBridge" → EventBridge service → Multiple targets (powerful)

**We use EventBridge integration for the fan-out pattern!**

## Quick Start

1. **Beginners:** Follow `AWS-CONSOLE-SETUP.md`
2. **CLI Users:** Follow `COMPLETE-WALKTHROUGH.md`
3. **Upload test files to S3**
4. **Get email notifications + CloudWatch logs automatically**

## Real-World Applications

This pattern is used for:
- File processing workflows
- Backup notifications  
- Content management systems
- Compliance logging
- Multi-step automation

Start with the setup guide that matches your preference!