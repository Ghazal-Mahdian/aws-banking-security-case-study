# Evidence Index

Source archive: `DevOps535-Final.zip` (17 screenshots: 16 from 4 May 2026, 1 from 9 May 2026). No report, recording, code, or exports were supplied.

## Redaction applied to all AWS console captures

- The AWS top navigation bar was cropped off. It showed the account name and 12-digit account ID.
- Browser chrome (tabs, bookmarks bar, profile photo) was cropped from the Cognito capture.
- Masked in page content: account ID, API ID (it appears in ARNs, breadcrumbs, and the invoke URL), Cognito user pool ID and ARN, token-signing URL, Lambda function ARN, and account-derived S3 bucket and CloudWatch log group suffixes.
- Left visible: resource names chosen by the author (`BankingAPI`, `BankingProcessor`, `banking-db`, `banking-audit-trail`, table names), Lambda request IDs, the code SHA-256 hash, and the API Gateway resource and deployment IDs. These are not usable without account access.

## Published screenshots

| Published file | Original (4 May 2026 unless noted) | Shows |
|---|---|---|
| `architecture-diagram-banking-aws.png` | Screenshot 2026-05-09 10.48.55 AM | Target architecture diagram (illustrative) |
| `cognito-user-pool-overview.png` | 5.31.08 PM | User pool overview |
| `rds-banking-db-mysql-available.png` | 6.05.26 PM | RDS `banking-db` created, Available |
| `iam-lambdabankingrole-created.png` | 6.20.25 PM | Role `LambdaBankingRole` created |
| `lambda-bankingprocessor-function-overview.png` | 6.23.38 PM | Lambda function overview |
| `api-gateway-transactions-post-lambda-integration.png` | 6.36.37 PM | POST method → Lambda |
| `api-gateway-transactions-get-lambda-integration.png` | 6.39.42 PM | GET method → Lambda |
| `api-gateway-prod-stage-deployment.png` | 6.40.54 PM | `prod` stage, throttling, no Web ACL |
| `cloudtrail-banking-audit-trail-logging.png` | 6.59.38 PM | Trail settings, Logging |
| `dynamodb-customerprofiles-and-transactions-tables.png` | 7.10.10 PM | Two tables |
| `api-test-post-transactions-200-first-response.png` | 7.35.43 PM | Postman POST → 200, one item |
| `api-test-post-transactions-200-empty-body.png` | 7.46.39 PM | Postman POST → 200, `[]` |
| `lambda-test-event-execution-summary.png` | 7.50.33 PM | Test event saved; execution summary and log |
| `lambda-test-success-fraud-check-output.png` | 7.53.22 PM | Test output with fraud flag |
| `s3-cloudtrail-digest-objects.png` | 8.06.41 PM | Digest files in log bucket |

## Originals not published

| Original | Reason |
|---|---|
| 6.30.15 PM: "Successfully created REST API 'BankingAPI'", no methods | Superseded by the POST/GET method captures |
| 6.32.08 PM: "Successfully created resource '/transactions'", no methods | Superseded by the POST/GET method captures |

## Claim → source

| Claim | Source |
|---|---|
| Region us-east-2 | Region selector in every console capture (cropped from published versions); ARNs; invoke URL |
| No WAF on `prod` | `api-gateway-prod-stage-deployment.png` (Web ACL "–") |
| Both methods use Lambda integration | `api-gateway-transactions-*-lambda-integration.png` |
| Cognito user pool has 0 users | `cognito-user-pool-overview.png` |
| RDS engine MySQL (Community) in us-east-2b | `rds-banking-db-mysql-available.png` |
| Fraud flag output | `lambda-test-success-fraud-check-output.png` |
| POST response does not echo submitted record | `api-test-post-transactions-200-*.png` |
| Multi-region trail, Logging, CloudWatch Logs | `cloudtrail-banking-audit-trail-logging.png` |
| Log file validation (digest files) | `s3-cloudtrail-digest-objects.png` |
| Diagram components not deployed (WAF, KMS, MFA) | Compare diagram with the captures above |
