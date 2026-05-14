# OAuth Client Secret Leaked

The shared secret for an OAuth client (used in authorization code or client credentials flows) exposed.

## Impact

- For confidential clients in authorization code flow: attacker can complete the flow, obtaining tokens — but only if they also have a valid authorization code (which requires user interaction or a redirect_uri attack).
- For client credentials flow: attacker can directly obtain tokens with the client's permissions. This is high-impact.
- For public clients (mobile, SPA): client secrets shouldn't be used at all (use PKCE). If your "secret" is in a mobile binary, it was already public.

## Immediate

- Rotate the secret in the OAuth provider's console.
- Update the secret in the application using it.
- Confirm the application is working with the new secret.
- Audit recent token issuance for the client ID — pull provider logs for the leak window.

## Investigate

In the OAuth provider's logs:
- Token issuance for the client ID: source IPs, scopes requested, grant types, success/failure.
- Authorization code redemptions from unfamiliar IPs.
- Refresh token usage.

If the client had access to user data via tokens it obtained:
- Audit whose tokens were issued to the client during the leak window.
- For each such user, audit access patterns to their data.
- If any tokens were used from the attacker's infrastructure, treat as account-takeover for those users.

## Containment

- Revoke all tokens issued to the client during the leak window if possible. Many providers support this.
- For refresh tokens: revoke them. Users will need to re-authorize.
- For long-lived access tokens (anti-pattern, but exists): revoke them.

## Token type considerations

- **Bearer access tokens with offline scope:** attacker has long-lived access. Revoke aggressively.
- **JWT access tokens:** typically not centrally revocable. The validity ends when they expire. Decide whether to invalidate the signing key (impacts all users) or wait it out (impact bounded by token TTL).
- **DPoP / mTLS-bound tokens:** tied to the original holder's key, less impact from secret leak alone.

## Replacement

- Multiple secrets for rolling rotation: many providers support 2 active secrets at once. Add new, deploy, then remove old.
- Confidential clients only — never assume a "secret" in a public client is meaningful security.
- Prefer mTLS or private_key_jwt client authentication over shared client_secret.

## Prevention

- Per-environment client_id+secret: dev, staging, prod each have their own.
- Secrets in vault / secret manager, not env files committed to repos.
- Rotation cadence: at minimum annually, ideally quarterly for high-value clients.
- For new integrations, start with the highest-trust auth method the provider supports.
- For your own OAuth provider: detect and alert on unusual token issuance patterns (geo, volume, timing).

## Adjacent

- Client secret often co-located with other secrets in a config file. Audit other secrets in the same file/commit/place — assume those leaked too.
- For OAuth client_credentials grants used for service-to-service: the "service identity" is now potentially impersonable. If that service has high privileges in downstream systems, the blast radius extends there.
