# Architecture: Proposed vs. Deployed

## Assignment architecture (summarized)

The case study proposed API Gateway for requests, Cognito with MFA for authentication, Lambda for transaction logic and fraud checks, and Step Functions for workflows such as KYC. For data it listed DynamoDB (profiles, transaction logs), Aurora (relational data), and S3 (versioned statements). KMS would manage keys for all stores, TLS 1.2+ would protect data in transit, and CloudTrail and CloudWatch would provide audit and monitoring. Scaling would rely on DynamoDB/S3/Lambda elasticity and RDS Multi-AZ with automated backups. It mapped these controls to PCI DSS, GLBA, and SOX.

## Component status

| Component | Proposed role | Status | Evidence |
|---|---|---|---|
| API Gateway | Entry point | **Deployed**: REST API `BankingAPI`, `/transactions` GET/POST, stage `prod` | `api-gateway-*.png` |
| AWS WAF | Edge filtering (diagram) | **Not configured**: `prod` stage Web ACL "–" | `api-gateway-prod-stage-deployment.png` |
| Cognito | Staff/customer auth with MFA | **Created, not integrated**: user pool, 0 users; no authorizer or app client shown | `cognito-user-pool-overview.png` |
| Lambda | Validation, fraud checks | **Deployed and tested**: `BankingProcessor` | `lambda-*.png` |
| IAM | Least-privilege roles | **Role created**: `LambdaBankingRole`; policies not shown | `iam-lambdabankingrole-created.png` |
| Step Functions | Workflow orchestration | **Not shown** | — |
| DynamoDB | Profiles, transactions | **Deployed**: `CustomerProfiles`, `Transactions` | `dynamodb-customerprofiles-and-transactions-tables.png` |
| Aurora / RDS | Relational data, Multi-AZ | **RDS MySQL created** (`banking-db`, us-east-2b); Aurora not used; Multi-AZ not shown; no connection from Lambda shown | `rds-banking-db-mysql-available.png` |
| S3 | Statements, versioning, lifecycle | **Only CloudTrail log bucket shown** | `s3-cloudtrail-digest-objects.png` |
| KMS | Keys for all stores | **Not shown** | — |
| CloudTrail | Audit every API call | **Deployed**: multi-region trail, Logging, digest files | `cloudtrail-banking-audit-trail-logging.png`, `s3-cloudtrail-digest-objects.png` |
| CloudWatch | Monitoring | **Log groups only**: CloudTrail log group and Lambda log link; no alarms/dashboards | CloudTrail and Lambda captures |

## Deployed request path

```
HTTP client (Postman)
   │  HTTPS  POST/GET /prod/transactions
   ▼
API Gateway REST API "BankingAPI"  (stage prod; throttle 10,000 rps / burst 5,000; no WAF; authorizer not shown)
   │  Lambda integration
   ▼
Lambda "BankingProcessor"  (128 MB; execution role LambdaBankingRole)
   │  (reads/writes not directly evidenced)
   ▼
DynamoDB  Transactions (PK transactionId) · CustomerProfiles (PK customerId)

Account-level:  CloudTrail "banking-audit-trail" (all regions) ──► S3 log bucket (+ digest files)
                                                            └──► CloudWatch Logs log group
Standalone:     RDS MySQL "banking-db"   ·   Cognito user pool
```

The link between Lambda and DynamoDB is inferred. The table names match the fields in the Lambda output (`transactionId`, `customerId`), but no screenshot shows the function code, item counts, or IAM permissions.

## Diagram caveats

The architecture diagram (`architecture-diagram-banking-aws.png`) is the target design. It differs from the deployment in these ways:

- It shows WAF on API Gateway; none is attached.
- It shows Cognito MFA in the request path; no authorizer is evidenced.
- It shows KMS encrypting S3, RDS, and DynamoDB; no keys are evidenced.
- It shows Lambda connected to RDS and S3; no such connection is evidenced.
- It draws Cognito as a step after API Gateway. In AWS, Cognito usually authorizes requests at API Gateway (an authorizer) rather than sitting between API Gateway and Lambda.

## Throttling note

Rate 10,000 and burst 5,000 are the API Gateway account-level defaults in many regions. The screenshot does not show whether these were set deliberately.
