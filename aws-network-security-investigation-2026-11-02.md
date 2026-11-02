# AWS Network Security Investigation — Security Groups and NACLs

**Date:** November 2, 2026  
**Environment:** Existing AWS lab, Region `us-east-2` (Ohio)  
**Method:** Read-only AWS CLI inspection via CloudShell; findings reported from command output  
**Changes made:** None

## Objective

Inspect Security Group inbound/outbound rules and Network ACL behavior; distinguish legitimate internet access from unnecessarily broad permissions. Apply AWS Certified Solutions Architect – Associate (SAA-C03) networking principles.

## Commands used

These read-only commands omit account, VPC, Security Group, and subnet identifiers.

```bash
aws ec2 describe-security-groups \
  --region us-east-2 \
  --query 'SecurityGroups[].{Name:GroupName,Description:Description,InboundRules:IpPermissions}' \
  --output json

aws ec2 describe-network-acls \
  --region us-east-2 \
  --query 'NetworkAcls[].{IsDefault:IsDefault,Entries:Entries}' \
  --output json

aws ec2 describe-security-groups \
  --region us-east-2 \
  --query 'SecurityGroups[].{Name:GroupName,OutboundRules:IpPermissionsEgress}' \
  --output json
```

## Reported observations

| Control | Observation | Interpretation |
| --- | --- | --- |
| Security Group inbound | A rule permitted `0.0.0.0/0` on TCP port 80 | Public HTTP may be intentional for a web service; evaluate TLS and redirect design |
| Security Group inbound | No `0.0.0.0/0` rule reported for SSH (22) or RDP (3389) | No unrestricted IPv4 access on those ports was observed in the reviewed output |
| Network ACL | `IsDefault=true`; both `allow` and `deny` entries reported | Consistent with a default NACL; rule numbers, directions, and CIDRs were not fully verified |
| Security Group outbound | `IpProtocol=-1` to `0.0.0.0/0` | Broad outbound IPv4 permission; actual connectivity also depends on routes and other controls |

**Scope limitation:** These are observations from the user's summaries of the CLI output, not a retained full ruleset or independent assessment of every Security Group/NACL association. The presence of one rule does not establish which workload it protects. IPv6 rules, effective NACL order, route reachability, actual exposure, and application TLS behavior were not verified. No vulnerability is asserted solely from these settings.

## Security Groups versus Network ACLs

- **Security Groups:** Stateful allow-only controls associated with network interfaces. Permitted connection response traffic is automatically allowed.
- **Network ACLs:** Stateless subnet-level controls with ordered allow/deny rules; inbound and outbound directions are evaluated separately.
- For an inbound HTTPS connection to an EC2 server on TCP 443, the NACL must also allow appropriate outbound response traffic to the client's ephemeral port. The exact client ephemeral range depends on operating systems and requirements; `1024–65535` is a broad example, not a universal least-privilege setting.

## Risk review and recommendations

1. **Public HTTP (port 80):** Determine whether it serves a deliberate HTTP-to-HTTPS redirect or an application endpoint. Use TLS on port 443 for sensitive traffic and consider an Application Load Balancer/ACM where appropriate.
2. **Outbound `0.0.0.0/0`:** Inventory required destinations and ports before tightening egress; restrict permissions to legitimate application needs and test for breakage.
3. **AWS service access:** Consider Gateway or Interface VPC endpoints for supported services when they meet routing, security, and cost requirements. Software updates may still require controlled internet access or internal mirrors.
4. **NACL verification:** If investigating a real connectivity issue, capture sanitized rule numbers, ingress/egress directions, port ranges, CIDRs, and subnet associations before diagnosing a block.
5. **Defense in depth:** Pair network controls with IAM least privilege, logging, patching, encryption, and monitoring.

## SAA-C03 learning validation

**2/2 correct** on two guided scenarios:
1. Stateless NACLs require outbound ephemeral-port response rules for client-initiated HTTPS connections.
2. Least-privilege outbound rules and appropriate VPC endpoints are preferable to blanket egress when business requirements permit.

This is formative practice, **not** a timed independent exam or proof of complete topic mastery.

## Business lens

A repeatable read-only review can help a small business identify potentially unnecessary network exposure before making risky production changes. Deliverables should separate **observed configuration**, **verified impact**, and **recommended remediation**, and any change requires authorization, a rollback plan, and validation.

## Evidence and cleanup

No new AWS infrastructure was provisioned, no settings were changed, and no cleanup was necessary. The write-up is sanitized; no credentials, AWS account IDs, or operational resource identifiers are included. Further validation would be required for a formal security finding.
