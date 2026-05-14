# Slack Webhook URL Leaked

Slack incoming webhooks (`https://hooks.slack.com/services/T.../B.../...`) allow anyone with the URL to post messages to the configured channel as the configured app. Often leaked in repos and treated as low-severity, but they enable phishing within the workspace.

## Impact

- Attacker can post messages to the channel impersonating the app.
- Used for phishing: messages crafted to look like alerts or shared links containing malware/credential-harvesters.
- Cannot read messages, only post.
- Cannot post outside the configured channel (each webhook is channel-bound).

## Revoke

- Slack workspace admin → Apps → find the integration → revoke the specific webhook URL.
- Alternative: revoke the app entirely if the webhook was its only legitimate use.

## Investigate

- Check the channel for messages from the integration in the leak window.
- Did anyone click links in those messages? Likely candidates for follow-up phishing investigation.
- Audit log (Slack Enterprise / Business+): any other actions taken by the integration's bot user.

## Replace

- Create a new webhook URL.
- Store as a secret (not in code, not in environment files committed to git).
- Reference via secret manager / env var injected at deploy time.

## Adjacent risk

- Many Slack apps have other capabilities beyond webhooks: bot tokens, signing secrets. Audit the full app for any leaked components.
- If the leak was in a public repo, search for other secrets in the same files / commits.

## Prevention

- Pre-commit secret scanning that catches `hooks.slack.com` URLs.
- Internal allowlist of which channels can have webhooks; periodic review.
- Prefer Slack Apps with proper OAuth and bot tokens for any integration that needs to be more than fire-and-forget posts.
- Webhook rotation as part of regular secret rotation.
