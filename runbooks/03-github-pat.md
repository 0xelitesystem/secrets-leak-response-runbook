# GitHub Personal Access Token Leaked

## Token types

- **Classic PAT (`ghp_*`):** broad scopes (e.g., `repo` covers all the user's repos).
- **Fine-grained PAT (`github_pat_*`):** scoped to specific repos and resources.
- **OAuth token (`gho_*`):** issued via OAuth app authorization.
- **App installation token (`ghs_*`):** GitHub App tokens, expire in 1 hour.

## Revoke immediately

GitHub auto-revokes PATs they detect as exposed in public commits, but only on github.com. Don't rely on this.

- Classic / fine-grained PAT: github.com → Settings → Developer settings → Personal access tokens → Revoke.
- OAuth: github.com → Settings → Applications → Authorized OAuth Apps → Revoke.
- GitHub App: revoke the installation if the App itself is compromised; otherwise tokens expire.

## Investigate

GitHub's audit log (org-level, requires Enterprise for full retention) shows token activity:
- Recently created repos (private repo exfil staging).
- Repos accessed.
- Webhooks created (often used for persistent exfil).
- Workflows triggered.
- New SSH keys / GPG keys / deploy keys / new PATs added under the account.

For personal accounts, the security log under Settings → Security log is the equivalent.

## Containment

- If the PAT had `admin:org` or `delete_repo`, treat as severe — assume worst case.
- Audit any repos the user or org owns for changes during the leak window:
  - Force-pushed branches (`git reflog` on cloned copies of important branches).
  - New collaborators / new outside collaborators.
  - Modified GitHub Actions workflows (especially `.github/workflows/`).
  - Added secrets in repo or org settings (!).
  - New deploy keys.
- If branch protections rely on the user as a reviewer, attacker may have approved their own PRs.

## Replace

- Generate new PAT with minimum scope and shortest practical lifetime.
- Prefer fine-grained PATs over classic.
- For automation: use GitHub Apps or OIDC instead of long-lived PATs.

## Specific cleanup

- Rotate any secrets that were in the same place the PAT was leaked (one leak rarely happens in isolation).
- If the PAT had `workflow` scope, audit recent workflow changes.
- If the PAT could push to a repo whose workflow runs with sensitive secrets, treat those secrets as also leaked.

## Prevention

- Push protection: enable in repo / org settings to block commits containing detectable secrets.
- Secret scanning alerts: enable.
- Org policy: require fine-grained PATs.
- For CI: use OIDC federation to AWS/GCP/Azure rather than storing cloud keys as GitHub Actions secrets.
- For developer use: shortest practical TTL (7-30 days).
