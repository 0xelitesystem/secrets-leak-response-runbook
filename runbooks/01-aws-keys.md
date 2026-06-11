# AWS Access Keys Leaked

For full incident handling beyond credential rotation, see incident-response-runbooks: `01-leaked-aws-keys.md`. This file focuses specifically on the rotation procedure.

## Identify the key

```
AKIA*  → IAM user long-lived access key
ASIA*  → Temporary STS credentials (session token expires automatically)
```

`aws sts get-access-key-info --access-key-id AKIA...` tells you which account owns it.

## Disable, don't delete

```bash
aws iam update-access-key \
  --user-name <username> \
  --access-key-id AKIA... \
  --status Inactive
```

Disable first, investigate, delete after CloudTrail extraction is complete. Deleting immediately removes your trail.

## Identify usage

```bash
# CloudTrail lookup for the past 90 days
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=AKIA...
```

For longer history, query the CloudTrail S3 bucket directly with Athena.

## Replacement workflow

If a human user:
- Decide whether they need long-lived keys at all. Most don't, IAM Identity Center (formerly SSO) is the right answer.
- If they do, generate a new key, deliver out-of-band, confirm working, then deactivate the old one.

If a workload:
- The right answer is almost never a new long-lived key.
- For EC2: instance profile.
- For Lambda: execution role.
- For EKS / ECS: IRSA / task role.
- For CI: OIDC federation (GitHub Actions, GitLab CI, CircleCI all support this).
- For on-prem: IAM Roles Anywhere with X.509 cert.

## Once replaced

```bash
aws iam delete-access-key \
  --user-name <username> \
  --access-key-id AKIA...
```

## Verification

- Confirm no application is still using the old key (CloudTrail shows zero usage for 24h before deletion).
- Verify the new credentials are working in production (smoke test, not just deployment success).

## Common mistakes

- Rotating before identifying impact. You lose the ability to investigate.
- Generating a new key with the same scope as the old, when the leak revealed that scope was too broad.
- Forgetting that the same key may be in multiple deployment artifacts, secrets managers, developer machines, CI configs.
- Not searching for the key in your git history. Use `gitleaks` or `git log -S AKIA...` across all repos.
