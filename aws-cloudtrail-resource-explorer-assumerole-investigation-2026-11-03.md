# CloudTrail AWS Service AssumeRole Investigation — November 3, 2026

**Focus:** Blue Team cloud audit analysis and evidence-based triage  
**Environment:** Existing AWS account, CloudTrail Event History, `us-east-2`  
**Method:** Read-only AWS CLI / CloudShell inspection; observations reported from command output  
**Disposition:** No confirmed security incident; expected-service explanation is plausible but not independently verified.

## Objective
Identify recent AWS management events, investigate an `AssumeRole` event, determine the initiating identity, and avoid declaring a compromise without supporting evidence.

## Read-only commands
```bash
aws cloudtrail lookup-events \
  --region us-east-2 \
  --max-results 10 \
  --query 'Events[].{Time:EventTime,Event:EventName,Source:EventSource}' \
  --output table

aws cloudtrail lookup-events \
  --region us-east-2 \
  --lookup-attributes AttributeKey=EventName,AttributeValue=AssumeRole \
  --max-results 5 \
  --query 'Events[].{Time:EventTime,Event:EventName,Source:EventSource}' \
  --output table
```
The analyst subsequently extracted selected fields from the most recent matching event using Python, avoiding display of full event JSON and sensitive identifiers.

## Reported evidence
| Field | Observation |
| --- | --- |
| Other recent event names | `ListClusters`, `GetDatabases` |
| Event | `AssumeRole` |
| Event source | `sts.amazonaws.com` |
| Observed event timestamp | `2026-11-03T18:26:25+00:00` |
| Most recent inspected event identity type | `AWSService` |
| Invoked by | `resource-explorer-2.amazonaws.com` |
| Role session name | `resource-explorer-2` |
| Error code | None recorded |
| MFA indicator | Not specified |
| Source IP category | Other / Not specified |

**Important evidence boundary:** The timestamp was reported from an initial five-event lookup, whereas the detailed identity and service metadata came from a later `--max-results 1` lookup. These may not represent the exact same event; correlation by event ID and timestamp was not performed. An absent `errorCode` is consistent with success but the response was not independently validated.

## Analysis
- `AWSService` and the `resource-explorer-2.amazonaws.com` service principal are consistent with AWS Resource Explorer invoking AWS STS to assume a service role.
- `resource-explorer-2` as the role session name reinforces the service-workflow interpretation.
- MFA not specified is not by itself a finding for a service identity.
- No evidence reviewed establishes an unauthorized role assumption, privilege escalation, or compromised identity.
- Service attribution is **not** proof that the role trust policy, attached permissions, or broader activity was authorized.

## Recommended follow-up if this were a production investigation
1. Correlate the exact CloudTrail `eventID`, `eventTime`, `requestID`, and assumed role across related records.
2. Verify the IAM role trust policy permits the intended service principal, with appropriate conditions where applicable.
3. Review the role's permissions and relevant Resource Explorer configuration for least privilege.
4. Check related CloudTrail events for unexpected role use, anomalous regions, changes, or failures.
5. Escalate only if contextual evidence indicates a policy violation or suspicious behavior.

## SAA-C03 learning
**1/1 correct** guided final scenario: review service identity, role trust policy, permissions, and related CloudTrail activity before deciding whether an `AssumeRole` event is unauthorized. This is a formative scenario, not an independent certification readiness measurement.

**Service distinctions:** CloudTrail records API activity; GuardDuty identifies potential threats from supported signals; AWS Config evaluates resource configurations and compliance.

## Business lens and limitations
This demonstrates a repeatable, low-risk audit-triage workflow suitable for a small-business cloud security assessment. Evidence and conclusions are separated; no unverified compromise claim or remediation is made.

No resources were provisioned, no AWS configuration was modified, and no cleanup was required. Public documentation excludes AWS account IDs, role ARNs, access keys, principal IDs, and full raw event JSON.
