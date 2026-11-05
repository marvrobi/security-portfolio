# AWS IAM Policy Evaluation & Broad User Permissions Investigation — November 5, 2026

**Security Investigation #9 | Lab #6 | SAA-C03 / Blue Team | Read-only**

## Objective
Investigate an IAM user's identity-based policy sources, evaluate a broad permission grant, and validate a sample authorization decision without changing AWS resources or permissions.

## Scope and method
Used AWS CloudShell, AWS IAM read APIs, AWS Organizations discovery, and IAM Policy Simulator. Findings are based on reported CLI results from a guided investigation, not an independent comprehensive audit.

## Verified reported observations
| Check | Result |
|---|---|
| STS execution identity | IAM user (full ARN withheld) |
| Directly attached managed policies | One or more |
| Direct inline user policies | None |
| IAM group membership | None |
| User permissions boundary | None |
| Managed policy wildcard actions | Present |
| Managed policy wildcard resources | Present |
| Explicit Deny in reviewed managed policy statements | None reported |
| Conditions in reviewed managed policy statements | None reported |
| Combined broad Allow | At least one directly attached managed policy statement with `Effect: Allow`, `Action: "*"`, `Resource: "*"` |
| AWS Organizations describe-organization | `AWSOrganizationsNotInUseException` |
| IAM simulation | `ec2:DescribeInstances` returned `allowed`; no missing context reported |

The specific managed policy name and the statement number were not recorded. The Organizations exception indicates that AWS Organizations is not in use for this account at the time of the test, so Organizations SCPs are not an applicable restriction.

## Security assessment
**Finding:** A directly attached managed policy grants broad identity-based administrative permissions to an IAM user. This presents a potential least-privilege concern if the user's responsibilities do not require such breadth.

**Evidence limits:** A simulator decision of `allowed` for `ec2:DescribeInstances` is evidence about that simulated request, not proof that every real AWS request succeeds. Applicable resource-based policies, service rules, request context, and other controls can influence real outcomes. Absence of an explicit Deny in the reviewed managed policies does not prove that none exists anywhere in the environment. No unauthorized activity or compromise was established.

**Risk:** Broad user permissions can increase the impact of stolen credentials, unintended changes, or misuse. Severity cannot be assigned reliably without business justification, authentication posture, usage patterns, and compensating controls.

## Recommended remediation plan (not performed)
1. Confirm why this IAM user requires broad access and identify necessary administrative workflows.
2. Review usage evidence and AWS IAM Access Analyzer policy-generation options where suitable.
3. Design scoped policies and, where practical, use temporary assumed roles for elevated tasks instead of standing broad user permissions.
4. Validate intended access and denied actions in a test environment, including resource-policy and context constraints.
5. Roll out changes gradually with a rollback plan, monitor CloudTrail, and review MFA/credential hygiene separately.

## SAA-C03 assessment
Final guided scenario: **B — correct (1/1, 100%)**. Review actual responsibilities, develop least-privilege access, validate it, and transition carefully without operational disruption. This is one formative question, not an exam-readiness estimate.

## Business lens
A small-business AWS Security Snapshot could flag standing broad IAM permissions as an evidence-based risk, distinguish identity-policy grants from effective authorization, and provide a staged remediation recommendation without altering the customer's environment.

## Evidence and safety
No AWS configuration changes or chargeable resources were created. No cleanup required. Public documentation omits account identifiers, full ARNs, access keys, and raw policy documents.
