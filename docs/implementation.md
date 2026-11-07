# Implementation

All values below are read from console screenshots taken on 4 May 2026 in us-east-2 (Ohio). No Lambda code, IAM policy JSON, CLI output, or infrastructure-as-code was supplied, and none has been recreated. Account-specific identifiers are redacted.

## Order of work (screenshot times, PDT)

| Time | Action | Source |
|---|---|---|
| 17:31 | Cognito user pool created (pool created 17:30) | `cognito-user-pool-overview.png` |
| 18:05 | RDS database `banking-db` created | `rds-banking-db-mysql-available.png` |
| 18:20 | IAM role `LambdaBankingRole` created | `iam-lambdabankingrole-created.png` |
| 18:23 | Lambda `BankingProcessor` updated | `lambda-bankingprocessor-function-overview.png` |
| 18:30 | REST API `BankingAPI` and resource `/transactions` created | (originals 6.30.15 PM, 6.32.08 PM; not published) |
| 18:36 | `POST /transactions` method with Lambda integration | `api-gateway-transactions-post-lambda-integration.png` |
| 18:39 | `GET /transactions` method with Lambda integration | `api-gateway-transactions-get-lambda-integration.png` |
| 18:40 | Deployment to stage `prod` | `api-gateway-prod-stage-deployment.png` |
| 18:59 | CloudTrail trail `banking-audit-trail` created | `cloudtrail-banking-audit-trail-logging.png` |
| 19:09 | DynamoDB tables listed (both Active) | `dynamodb-customerprofiles-and-transactions-tables.png` |
| 19:35 | `POST /prod/transactions` → 200 (one item returned) | `api-test-post-transactions-200-first-response.png` |
| 19:46 | `POST /prod/transactions` → 200 (empty list) | `api-test-post-transactions-200-empty-body.png` |
| 19:50 | Lambda test event `BankingProcessorTest` saved and run | `lambda-test-event-execution-summary.png` |
| 19:53 | Lambda test with fraud-check output | `lambda-test-success-fraud-check-output.png` |
| 20:06 | CloudTrail digest objects in S3 | `s3-cloudtrail-digest-objects.png` |

The DynamoDB tables were already Active at 19:09; their creation was not captured.

## API Gateway

| Setting | Value |
|---|---|
| API | `BankingAPI` (REST) |
| Resources | `/`, `/transactions` |
| Methods | `GET`, `POST` on `/transactions`, both **Lambda integration** |
| Stage | `prod`, active deployment created 4 May 2026 18:40 |
| Throttling | Rate 10,000, burst 5,000 |
| Caching | Cache cluster inactive; method-level caching inactive |
| Web ACL | none ("–") |
| Client certificate | none ("–") |
| Authorization | not visible in any capture |

## Lambda

| Setting | Value |
|---|---|
| Function | `BankingProcessor` |
| Layers | 0 |
| Triggers shown in overview | none at 18:23 (captured before the API was created) |
| Memory | 128 MB configured; 93 MB max used |
| Test durations | 24.40 ms (billed 25 ms); 29.13 ms (billed 30 ms) |
| Version | `$LATEST` |
| Runtime / handler / code | not shown |

Lambda test output (19:53), **transcribed from the screenshot**:

```json
{
  "statusCode": 200,
  "headers": { "Content-Type": "application/json" },
  "body": "[{\"fraudFlag\": false, \"timestamp\": \"2026-05-05T02:49:51.022017\", \"amount\": \"500.0\", \"transactionId\": \"TXN-TEST\", \"reviewRequired\": false, \"customerId\": \"CUST-001\", \"type\": \"transfer\"}]"
}
```

The timestamp is UTC (02:49 UTC = 19:49 PDT, shortly before the screenshot). `amount` is returned as a string.

## API tests (Postman)

Request body used in both tests (transcribed):

```json
{ "transactionId": "TXN-001", "customerId": "CUST-001", "amount": 500, "type": "transfer" }
```

| Time | Status | Latency | Size | Response `body` field |
|---|---|---|---|---|
| 19:35 | 200 OK | 223 ms | 457 B | `[{"transactionId": "9292929239394"}]` |
| 19:46 | 200 OK | 307 ms | 418 B | `[]` |

In both responses the HTTP body itself contains `statusCode`, `headers`, and `body` keys. API Gateway passed the function's return object through as the payload instead of mapping it to the HTTP response. Neither response contains `TXN-001`.

## DynamoDB

| Table | Partition key | Sort key | Indexes | Replication | Deletion protection |
|---|---|---|---|---|---|
| `CustomerProfiles` | `customerId` (S) | – | 0 | 0 | Off |
| `Transactions` | `transactionId` (S) | – | 0 | 0 | Off |

Capacity mode, encryption type, and point-in-time recovery are not shown.

## RDS

`banking-db`: Status Available, Role Instance, Engine "MySQL Co…" (truncated, read as MySQL Community), upgrade rollout order SECOND, Region/AZ us-east-2b. Instance class, storage encryption, Multi-AZ, backup retention, and public accessibility are not shown.

## Cognito

User pool named "User pool - 6zvg8v" (auto-generated name), feature plan Essentials, estimated users 0, created 4 May 2026 17:30 PDT. App clients, MFA settings, and sign-in options are not shown.

## IAM

Role `LambdaBankingRole` created at 18:20. The roles list also shows unrelated service-linked and service roles from other work in the same account. Trust policy and permissions for `LambdaBankingRole` are not shown.

## CloudTrail and S3

| Setting | Value |
|---|---|
| Trail | `banking-audit-trail` |
| Home region | US East (Ohio) |
| Multi-region | Yes |
| Insights | Disabled |
| Organization trail | No |
| S3 bucket | `banking-cloudtrail-logs-<redacted>` |
| CloudWatch Logs | log group `banking-cloudtrail-logs-<redacted>` |
| Status | Logging |

S3 path `AWSLogs/<account>/CloudTrail-Digest/us-east-2/2026/05/05/` holds two `.json.gz` digest files (351 B each visible; Standard storage class). Bucket encryption, versioning, and access policy are not shown.
