# CloudTrail Access Denied & IAM Authorization Troubleshooting — November 6, 2026

**Security Investigation #10 | Lab #7 | Read-only AWS Blue Team investigation**

## Objective
Investigate recorded AWS API authorization failures, correlate principal identity, compare actual outcomes with IAM Policy Simulator, and document the limits of available evidence without changing AWS permissions or billing settings.

## Method and evidence
Evidence comes from user-reported AWS CloudShell output, a Billing and Cost Management console screenshot, and guided read-only queries. No raw account identifiers, ARNs, access keys, or billing data are included.

### CloudTrail findings
- **us-east-2:** Recent `ec2:DescribeInstances` events were found; the inspected events had no `errorCode`, consistent with successful API responses. Principal types were not recorded.
- **us-east-2:** A broader sample of up to 200 recent events returned zero matches for selected authorization-error patterns; the number actually scanned was not recorded.
- **us-east-1:** Of 200 recent events examined, seven had `AccessDenied`: `ce:GetCostAndUsage` (four, IAMUser), `ce:GetCostForecast` (two, IAMUser), and `cloud9:ListEnvironments` (one, AssumedRole). These are sampled counts, not the total account history.
- **us-east-1, action-specific lookup:** A later query returned 150 `ce:GetCostAndUsage` `AccessDenied` events, all reported as matching the current IAM user. The different counts reflect different query selection windows.
- The sampled Cost Explorer error message was 29 characters long and mentioned IAM; its text did not match tested diagnostic phrases for explicit deny, missing identity-based Allow, billing restriction, or IAM access disabled. The raw message was deliberately not published.

### Billing console observations
A user-provided screenshot displayed authorization errors for:
- `billing:GetIAMAccessPreference` (unable to view IAM billing access preference)
- `billing:GetContractInformation`
- `account:GetAccountInformation`

The billing IAM access preference was **not visible**, so whether it was enabled or disabled remains **unknown**.

### Policy simulation
A read-only `iam:SimulatePrincipalPolicy` check reported **allowed** for all four actions:
- `ce:GetCostAndUsage`
- `billing:GetIAMAccessPreference`
- `billing:GetContractInformation`
- `account:GetAccountInformation`

Prior Lab #6 found a directly attached managed policy statement on the current IAM user with `Effect: Allow`, `Action: "*"`, and `Resource: "*"`; no user permissions boundary, no IAM groups, and AWS Organizations not in use. The simulation evaluates supported policies in a specified context and does not reproduce all account-level, resource-level, service-specific, or console request conditions.

## Assessment
**Confirmed:** There is a discrepancy between the simulator's `allowed` decisions and observed `AccessDenied` responses for Cost Explorer and Billing/Account console functionality.

**Root cause: UNRESOLVED.** An account-level IAM billing access setting or another service-specific access requirement is a hypothesis, not a verified explanation. The console errors do not prove which control denied Cost Explorer, and an IAM Allow does not guarantee success of every live API request. The separate Cloud9 assumed-role denial was not investigated further and must not be attributed to the IAM user.

**Incident status:** No evidence of malicious activity, compromise, or a proven configuration defect was established. Repeated denials can also be caused by ordinary console activity or automation. The 150 denied events are not a count of distinct actors or incidents.

## Recommended follow-up (not performed)
1. Have an appropriately authorized account administrator inspect the billing IAM access preference and applicable Cost Explorer prerequisites.
2. Correlate precise timestamps, caller/session context, and diagnostic details for representative denied events without publishing sensitive identifiers.
3. Validate the effect of any authorized setting or policy adjustment using a planned test and change-control process.
4. Investigate the `cloud9:ListEnvironments` assumed-role denial separately if relevant.
5. Retain the observed outcome and uncertainty in the security record until a cause is independently verified.

## SAA-C03 formative check
Final scenario: **C — correct (1/1, 100%)**. Have an authorized administrator verify account-level billing access and relevant requirements, then correlate the evidence before making changes. One guided question is not a certification-readiness estimate.

## Business lens
A prospective AWS Security Snapshot client may have a broad IAM Allow but still encounter a service authorization failure. An evidence-based assessment distinguishes IAM simulation, CloudTrail outcomes, console evidence, and unverified hypotheses before recommending changes.

## Safety and disclosure
Read-only commands and console inspection only. No IAM, billing, logging, or resource settings were modified; no cleanup required. This public write-up omits account identifiers, full ARNs, and raw error messages.
