# secrets-leak-response-runbook

Eight runbooks for responding to specific secret leaks. Generic guidance ("rotate the secret") is the easy part; the hard parts are the rotation order, the dependent systems, the attacker-accessed window, and the audit trail. Each runbook covers all four.

## Runbook structure

Every runbook has these sections:

1. **Trigger**, how you found out about the leak
2. **Containment**, what to do before rotation, to limit further damage
3. **Rotation**, exact steps in the right order, including dependencies
4. **Investigation**, what the attacker may have done with the credential
5. **Audit logging**, where to look in vendor logs to scope the incident
6. **Recovery**, restoring service and access for legitimate users
7. **Post-rotation hardening**, controls to add so this leaks less easily next time

## Contents

| # | Runbook | When to use |
|---|---------|-------------|
| 01 | [AWS access keys](runbooks/01-aws-keys.md) | IAM user access key ID and secret in a leak |
| 02 | [GCP service account key](runbooks/02-gcp-service-account.md) | JSON key file leaked or detected in code |
| 03 | [GitHub PAT](runbooks/03-github-pat.md) | Personal access token in a public repo, log, or paste |
| 04 | [Slack webhook](runbooks/04-slack-webhook.md) | Incoming webhook URL exposed |
| 05 | [Database credentials](runbooks/05-database-credentials.md) | DB user/pass, connection string, or DSN |
| 06 | [JWT signing key](runbooks/06-jwt-signing-key.md) | Symmetric or private signing key for application JWTs |
| 07 | [TLS private key](runbooks/07-tls-private-key.md) | Server certificate private key exposed |
| 08 | [OAuth client secret](runbooks/08-oauth-client-secret.md) | Client secret for an OAuth/OIDC application |

## Intended use

When a secret is leaked, the panic instinct is to rotate immediately. That's often correct, but not always, for some credentials (TLS keys with active sessions, JWT signing keys with issued tokens still in flight), the order of operations matters. Read the runbook for the specific credential type before you start typing.

These runbooks assume you have the access needed to rotate and audit. If you don't, escalate first.

## Contributing

If you have a secret type not covered here that you've responded to in production, open a PR with a runbook in the same skeleton. Avoid runbooks for secrets where rotation is trivially "delete and recreate, nothing depends on it", those don't need a document.

## Related repositories

Part of a 10-repo security audit set.

Browser-based audit tools:
- [iam-policy-analyzer](https://github.com/0xelitesystem/iam-policy-analyzer)
- [terraform-security-linter](https://github.com/0xelitesystem/terraform-security-linter)
- [kubernetes-manifest-security-scanner](https://github.com/0xelitesystem/kubernetes-manifest-security-scanner)
- [session-cookie-auditor](https://github.com/0xelitesystem/session-cookie-auditor)
- [regex-redos-checker](https://github.com/0xelitesystem/regex-redos-checker)

Reference collections:
- [incident-response-runbooks](https://github.com/0xelitesystem/incident-response-runbooks)
- [ai-llm-security-audit](https://github.com/0xelitesystem/ai-llm-security-audit)
- [api-security-audit-checklist](https://github.com/0xelitesystem/api-security-audit-checklist)
- [threat-modeling-worksheets](https://github.com/0xelitesystem/threat-modeling-worksheets)

## License

MIT. See [LICENSE](LICENSE).
