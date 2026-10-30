# AWS Cloud Security & Networking Assessment

**Date:** October 30, 2026  
**Type:** Read-only AWS CLI investigation (existing personal lab environment)  
**Region:** US East (Ohio), `us-east-2`  
**Focus:** S3 data protection, VPC DNS, subnet routing, and security architecture  
**Changes made:** None. No resources provisioned or deleted.

## Objective

Use AWS CloudShell and AWS CLI to inspect existing cloud configurations, interpret security implications, and recommend an appropriate network design. This is an **assessment**, not a claim that a new VPC, private subnet, database, or Object Lock policy was deployed.

## Lab 1 — AWS CLI and S3 data protection

Validated AWS CLI availability and authenticated access with `aws --version` and `aws sts get-caller-identity`. Confirmed the active Region and inspected existing S3 buckets. The commands below use a placeholder bucket name; the original identifier is intentionally omitted.

```bash
aws --version
aws sts get-caller-identity
echo "$AWS_REGION"
aws s3 ls
aws s3api list-buckets --query 'Buckets[].Name' --output table
aws s3api get-bucket-versioning --bucket YOUR_AUTHORIZED_BUCKET
aws s3api get-object-lock-configuration --bucket YOUR_AUTHORIZED_BUCKET
```

**Observed:** The inspected bucket returned no versioning status (consistent with versioning never having been enabled). The Object Lock configuration lookup reported that no Object Lock configuration existed.

**Security interpretation:** Versioning supports recovery from overwrites/deletes, while Object Lock provides version-level write-once-read-many retention. Compliance mode protects retained object versions against deletion even by the account root user until retention expires. An Object Lock configuration lookup does not establish retention or legal-hold status on every object version.

**Recommendation:** For regulated records requiring immutable retention, evaluate S3 Versioning plus Object Lock compliance mode, appropriate retention periods, least-privilege IAM, encryption, logging, and lifecycle/cost requirements. Do not enable irreversible compliance retention on an existing production bucket without formal approval.

## Lab 2 — VPC networking and DNS

Inspected the existing default VPC, DNS attributes, subnets, route tables, and their associations.

```bash
aws ec2 describe-vpcs --region us-east-2 --query 'Vpcs[].{VpcId:VpcId,IsDefault:IsDefault,State:State,CIDR:CidrBlock}' --output table
aws ec2 describe-vpc-attribute --vpc-id YOUR_VPC_ID --attribute enableDnsSupport --region us-east-2
aws ec2 describe-vpc-attribute --vpc-id YOUR_VPC_ID --attribute enableDnsHostnames --region us-east-2
aws ec2 describe-subnets --region us-east-2 --query 'Subnets[].{AZ:AvailabilityZone,CIDR:CidrBlock,PublicIP:MapPublicIpOnLaunch}' --output table
aws ec2 describe-route-tables --region us-east-2 --query 'RouteTables[].Routes[].{Destination:DestinationCidrBlock,Gateway:GatewayId,State:State}' --output table
aws ec2 describe-route-tables --region us-east-2 --query 'RouteTables[].{Main:Associations[?Main==\`true\`].Main,SubnetAssociations:Associations[].SubnetId,Routes:Routes[].DestinationCidrBlock}' --output json
aws ec2 describe-route-tables --region us-east-2 --query 'RouteTables[].Associations[?SubnetId!=\`null\`].SubnetId' --output json
```

**Observed (from CLI outputs reviewed during the lab):**
- One default VPC in `us-east-2`.
- `enableDnsSupport` and `enableDnsHostnames` both `true`.
- Three subnets distributed across `us-east-2a`, `us-east-2b`, and `us-east-2c`; each had `MapPublicIpOnLaunch=true`.
- A `0.0.0.0/0` route targeting an Internet Gateway was reported.
- The main route table was identified and no explicit subnet associations were reported; the three subnets therefore inherit the main table.

**Interpretation and evidence boundary:** If the Internet Gateway default route belongs to that main route table, all three subnets are public. The commands established the individual findings; the precise route-to-main-table mapping was inferred from the lab's reported outputs rather than preserved as a full route table dump. Auto-assigned public IP addresses alone do not determine whether a subnet is public.

## Recommended two-tier architecture (not deployed)

```text
                     Internet
                        |
               Internet Gateway
                        |
            Public subnets across AZs
               Web / load balancer
                        |
              Security group rules
                        |
           Private subnets across AZs
                 Database tier
       (no direct route to Internet Gateway)
```

- Put internet-facing application entry points in public subnets.
- Put the database in private subnets without direct Internet Gateway routing.
- Restrict database ingress to the application security group on the required port.
- Use multiple Availability Zones for resilience.
- Evaluate private VPC endpoints, outbound access design, encryption, monitoring, and cost before deployment.

## Validation and learning

- Executed read-only CLI inspection; no resources created or modified.
- Answered two guided SAA-C03 scenarios correctly: S3 Object Lock compliance retention and public application/private database network segmentation.
- Practiced distinguishing observed configuration from inferred architecture and recommended future design.
- **Limitations:** No penetration test, vulnerability scan, deployed remediation, private subnet creation, or object-level retention inspection was performed. No unredacted account, VPC, or bucket identifiers are published.

## Next steps

1. Capture sanitized, non-sensitive screenshots in a future session if useful.
2. Revisit Object Lock retention versus legal hold in a later unprompted knowledge check.
3. Build a separate custom VPC with public/private subnets as a future deployment lab, using cost controls and cleanup verification.
