# AWS IAM Security & Least-Privilege Investigation — November 4, 2026

**Investigation #8 | Blue Team / SAA-C03 | Read-only IAM review**

## Objective
Inventory IAM roles, identify role trust controls and permission sources, and evaluate least-privilege indicators without changing AWS configuration.

## Scope and methodology
- AWS CLI / CloudShell read-only IAM role inventory, trust-policy inspection, managed-policy and inline-policy enumeration, and selected statement review.
- Approximately 6–15 IAM roles observed; paths included `/aws-service-role/` and `/`.
- Focused on two existing DevOps Agent roles: `DevOpsAgentRole-AgentSpace` and `DevOpsAgentRole-WebappAdmin`.
- Evidence consists of user-reported command results and screenshots. This was a guided investigation, not a comprehensive IAM audit.

## Trust-policy findings
| Control | AgentSpace | WebappAdmin |
|---|---|---|
| Service principal | `aidevops.amazonaws.com` | `aidevops.amazonaws.com` |
| Source account restriction | `aws:SourceAccount` observed | `aws:SourceAccount` observed |
| Source resource restriction | `aws:SourceArn` observed | `aws:SourceArn` observed |

These condition keys can help mitigate confused-deputy scenarios when their values and matching operators are configured correctly. The exact values, operators for SourceArn, complete statements, and their effectiveness were not independently verified.

## Permission-source findings
| Property | AgentSpace | WebappAdmin |
|---|---|---|
| Attached managed policy | `AIDevOpsAgentAccessPolicy` | `AIDevOpsOperatorAppAccessPolicy` |
| Inline policy | `AIDevOpsAllowAwsSupportActionsPolicy` | None observed |
| Permissions boundary | None observed | None observed |
| Full-action wildcard (`Action: "*"`) | Not observed in reviewed managed-policy output | Not observed in reviewed managed-policy output |
| Wildcard resource (`Resource: "*"`) | Present in at least one managed-policy statement | Present in at least one managed-policy statement |
| Policy condition keys | Present | Present |

**AgentSpace inline policy:** A reviewed Allow statement included `iam:CreateServiceLinkedRole`. Its resource list was reported as scoped rather than exactly `"*"`; full ARN patterns and condition keys were not established. Other inline statements, if any, were not comprehensively evaluated.

**AgentSpace managed policy:** Statement 4 was reported as `Effect: Allow`, `Action: s3:ListBucket`, with a scoped resource and `s3:prefix` condition. This is a positive scoping indicator; exact prefix values and bucket scope were not independently checked.

**WebappAdmin managed policy:** No `Action: "*"` was reported; some statements use wildcard resources and some include conditions. Specific actions were not supplied, so this role's least-privilege suitability remains undetermined.

## Analysis and disposition
**No confirmed IAM misconfiguration. Further validation recommended.**

- Specific service principals and source restrictions suggest an intentional trust design, but do not prove full authorization.
- A `Resource: "*"` statement can be legitimate for actions lacking resource-level permission support; it is not inherently a vulnerability.
- Absence of a permissions boundary is not by itself a security finding.
- Managed and inline permissions must be considered together; policy names are not evidence of effective privilege.
- Effective permissions can also depend on SCPs, explicit denies, resource policies, and service context.

## Recommended next checks
1. Compare each wildcard-resource action with AWS Service Authorization Reference supported resource types and condition keys.
2. Verify complete trust-policy statements, exact `aws:SourceAccount` and `aws:SourceArn` values, and appropriate matching operators.
3. Examine all managed and inline policy statements, including any `NotAction`/`NotResource` patterns and deny statements.
4. Review actual role usage and CloudTrail activity to justify privileges; consider IAM Access Analyzer where applicable.
5. Test any proposed reductions in a controlled environment before modifying service-managed workflows.

## SAA-C03 formative scenario
**1/1 correct (100%).** Selected reviewing wildcard-resource statements against supported IAM actions, resource types, conditions, and service needs, then narrowing permissions where feasible. This guided single-question result is not a certification-readiness estimate.

## Business lens
This exercise demonstrates a repeatable read-only IAM assessment: inventory identities, inspect trust, enumerate permissions, identify evidence boundaries, and communicate risk without overstating findings.

## Evidence hygiene
No AWS resources were created, altered, or deleted. No cleanup required. Public report omits account IDs, full ARNs, condition values, access keys, and raw policy JSON. Findings are limited to the inspected evidence.
