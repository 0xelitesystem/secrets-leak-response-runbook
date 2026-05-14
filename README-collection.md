# About this collection

Per-credential-type runbooks for the most common secret leak scenarios. Each follows: identify, revoke, investigate, replace, prevent.

## Index

1. AWS Access Keys
2. GCP Service Account Keys
3. GitHub Personal Access Tokens
4. Slack Webhook URLs
5. Database Credentials
6. JWT Signing Keys
7. TLS Private Keys
8. OAuth Client Secrets

## How to use

Find the credential type, follow the procedure. The runbooks assume you've already detected the leak — for full incident handling (forensics, regulatory notification, etc.), see the incident-response-runbooks collection.

## General principles for any credential leak

- Rotate before deleting. Preserve the old credential identifier for log correlation.
- Investigate before assuming clean. Most leaks have evidence of attempted use even when not visibly exploited.
- Look for adjacent secrets. Where one was leaked, others often were.
- Update the prevention controls — leak detection, rotation cadence, scope reduction — for next time.
