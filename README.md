# EventBridge S3 Upload Notifications

I built this project to learn EventBridge by creating something actually useful - getting notified when files are uploaded to S3. Turns out it's a great way to understand event-driven architecture.

## What This Does

Upload a file to S3, and you'll automatically get:
- Email notification with file details
- Event logged to CloudWatch for monitoring
- Shows how one event can trigger multiple actions

## Why EventBridge?

I tried the direct S3 → SNS approach first, but EventBridge is way more powerful. You can route one event to multiple services, filter events, and build complex workflows. Plus it's what most companies use in production.

## What I Learned

The biggest "aha" moment was understanding S3 has two different notification systems:
- **Event notifications** - sends directly to SNS/SQS/Lambda (simple but limited)
- **EventBridge integration** - sends to EventBridge first, then you can route anywhere (powerful)

Most tutorials don't explain this difference clearly, which is why I got stuck initially.

## Setup Options

I've included two ways to set this up:

**AWS-CONSOLE-SETUP.md** - If you prefer clicking through the console (I recommend this for your first time)

**COMPLETE-WALKTHROUGH.md** - If you want to use CLI commands (better for automation later)

Both approaches work the same way, just different interfaces.

## Real Uses

This pattern shows up everywhere:
- File processing pipelines (scan uploaded files, convert formats, etc.)
- Backup monitoring (know when backups complete)
- Content workflows (notify teams when new content arrives)
- Compliance logging (audit trail for all file operations)

## Getting Started

Pick whichever setup guide matches how you like to work. The console approach is more visual, CLI is faster once you know what you're doing.

After setup, just upload any file to your S3 bucket and watch the magic happen!