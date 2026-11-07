# Validation

Status key:

- **Demonstrated**: visible in a supplied screenshot.
- **Partial**: some of the expected behaviour shown.
- **Not demonstrated**: a test exists, but its result does not show the expected outcome.
- **Not shown**: required or proposed; no evidence.
- **Not configured**: evidence shows the control is absent.

No written report was supplied, so no claims are "reported only".

## Functional tests

| Test | Expected outcome | Observed outcome | Evidence | Status |
|---|---|---|---|---|
| API reachable over HTTPS | 2xx from `prod` endpoint | `POST /prod/transactions` → 200 OK (223 ms; 307 ms) | `api-test-post-*.png` | Demonstrated |
| API invokes Lambda | Lambda response returned | Response contains Lambda-style `statusCode/headers/body` | same | Demonstrated |
| POST stores submitted transaction | Record `TXN-001` written / echoed | First response lists `9292929239394`; second lists none; no DynamoDB item view | same | Not demonstrated |
| GET returns transactions | List of transactions | GET method exists; no GET request captured | `api-gateway-transactions-get-lambda-integration.png` | Not shown |
| Lambda fraud-check logic | Output with fraud/review decision | `fraudFlag: false`, `reviewRequired: false` for a 500.0 transfer | `lambda-test-success-fraud-check-output.png` | Partial (one non-flagged case; rules unknown) |
| Fraud flag raised for a suspicious transaction | `fraudFlag: true` for a test case | Not tested in captures | — | Not shown |
| Data stores provisioned | DynamoDB and relational DB available | Two DynamoDB tables Active; RDS `banking-db` Available | DynamoDB and RDS captures | Demonstrated |
| Lambda uses RDS | Loan/account data from RDS | No connection shown | — | Not shown |

## Security and compliance requirements

| Test | Expected outcome | Observed outcome | Evidence | Status |
|---|---|---|---|---|
| Encryption at rest with KMS (S3, RDS, DynamoDB) | KMS keys applied | No KMS or encryption settings captured | — | Not shown |
| TLS 1.2+ in transit | HTTPS only, TLS policy | HTTPS endpoint used; TLS version/policy not shown | Postman captures | Partial |
| User authentication with Cognito MFA | Requests require a valid token | User pool exists with 0 users; no authorizer, no `Authorization` header in requests that succeeded | Cognito and Postman captures | Not shown |
| Least-privilege IAM | Narrow role policies | Role created; policies not shown | `iam-lambdabankingrole-created.png` | Not shown |
| WAF at the edge | Web ACL attached | Web ACL "–" on `prod` | `api-gateway-prod-stage-deployment.png` | Not configured |
| Audit all API calls (CloudTrail) | Trail logging, all regions | Multi-region trail, status Logging | `cloudtrail-banking-audit-trail-logging.png` | Demonstrated |
| Log integrity | Tamper-evident logs | Digest files present in S3 (generated only with log file validation on) | `s3-cloudtrail-digest-objects.png` | Demonstrated |
| Monitoring with CloudWatch | Metrics/alarms | Log groups only; no alarms or dashboards | — | Partial |
| Tokenization of card data (PCI DSS) | Tokenization in place | Not shown; no card data in tests | — | Not shown |
| PCI DSS / GLBA / SOX compliance | Compliance assessed | Out of scope of the evidence | — | Not shown |

## Availability and scale

| Test | Expected outcome | Observed outcome | Evidence | Status |
|---|---|---|---|---|
| RDS Multi-AZ + automated backups | Multi-AZ, backups on | Single instance listed in us-east-2b; settings not shown | `rds-banking-db-mysql-available.png` | Not shown |
| Peak-load scaling | Load test | No load test | — | Not shown |
| Zero downtime / disaster recovery | DR design tested | Not shown | — | Not shown |
| S3 versioning and lifecycle for statements | Versioned statement bucket | No statements bucket shown | — | Not shown |
| Throttling | Limits on API | Stage shows rate 10,000 / burst 5,000 (default values) | `api-gateway-prod-stage-deployment.png` | Partial |
