# JWT Signing Key Leaked

The secret used to sign JWTs is exposed. Anyone with the key can issue valid tokens for any user, including admins.

## Severity

This is the highest-severity credential type. With the signing key, the attacker can authenticate as anyone in the system. Treat as full account-takeover-of-everyone.

## Immediate

- Rotate the key immediately. All existing tokens become invalid.
- This will sign out every user. Accept the impact; the alternative is leaving the door open.
- For systems with a refresh + access token model, rotating the access-signing key invalidates access tokens. Refresh tokens may use a separate key, rotate that too if uncertain.

## Multi-key rotation (if your system supports it)

Better systems support a key set with current + previous keys (`kid` in the JWT header):

1. Add a new key to the keyset. Make it the active signing key.
2. Old key remains in the keyset for verification only.
3. Active sessions naturally migrate to new tokens as they refresh.
4. Once refresh window passes (e.g., 1 hour for short-lived access tokens), remove the old key.

If your system signs and verifies with the same single key, you don't have this option, rotation is hard sign-out.

## Investigate

- Forge detection is hard. A token signed by the legitimate key with the leaked key looks identical to a legitimate token.
- Audit logs of activity during the leak window. Look for unusual patterns: admin actions from new IPs, sessions for users who weren't actively using the system, account modifications.
- If your tokens include `iat` (issued-at) and you can correlate with login events, mismatched `iat` (token has iat=X but no login event at time X) indicates forgery.
- Sessions / tokens with unusual claim combinations (e.g., admin role for a user who shouldn't have it).

## Containment

- Force re-authentication for all users (achieved by key rotation).
- Re-enroll MFA where appropriate.
- Audit and revoke any account modifications made during the leak window.
- If the key was a shared-secret HMAC key, no further action, new key replaces it. If it was an asymmetric key (RSA/ECDSA private key), revoke the public key reference and rotate the keypair.

## Adjacent considerations

- Algorithm: was the leaked key used with HS256 (symmetric, the secret is everything) or RS256 (asymmetric, the leaked private key alone is enough)? Both are equally bad but matter for understanding scope.
- Key material in: env vars, secret manager, config files, KMS, wherever it lived, audit access logs to see who/what could have read it.
- Cookie-based sessions issued by the same auth system may also need rotation.

## Prevention

- Never commit signing keys to source control. Audit history; if it was ever there, treat any key from any commit as leaked.
- Store keys in KMS / HSM / Secret Manager. Sign by API call to the KMS rather than handling the key in app memory where possible.
- Keep keys in memory only; don't write to disk or log them.
- Periodic rotation as a routine practice (annually at minimum), so the procedure works when you need it for an incident.
- Asymmetric (RS256, ES256, EdDSA) preferred over symmetric (HS256), the verifying service doesn't need the signing key.
- Short token lifetimes (15 minutes for access tokens) reduce the value of historical leaked keys.
