# AWS IAM Policy-Change Alerting

**Project:** Secure AWS Environment & Monitoring Lab  
**Status:** Exercise completed; broader project in progress  
**Type:** Controlled hands-on test

## Objective
Practice detecting an IAM policy change and validating an operator notification.

## Hands-on work
Reviewed AWS audit logging and configured event routing, log-based metrics, an alarm, and an email notification. Created a temporary, unattached test policy and removed it after validation.

## Validation and result
Observed the initial alarm transition, received the notification, and observed recovery to OK. The cleanup action produced a second alert. Reviewed the corresponding audit event to connect the action with the notification.

## Skills demonstrated
CloudTrail audit review, EventBridge event routing, CloudWatch Logs and alarms, SNS notifications, controlled testing, and cleanup.

## Limitations
This demonstrated change monitoring, not malicious-activity classification or least-privilege enforcement. Final alarm recovery after the cleanup alert was not verified in the last evidence.

## Evidence
Screenshots and validation notes are retained privately. Operational identifiers and unredacted screenshots are excluded from this public write-up.

