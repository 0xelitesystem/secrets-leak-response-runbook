# TLS Private Key Leaked

Private key for a TLS certificate exposed. Attacker can impersonate the server to anyone who connects, and decrypt traffic if they captured it (depending on cipher suite and forward secrecy).

## Immediate

- Revoke the certificate via the issuing CA's revocation process.
- Issue a new keypair and certificate.
- Deploy the new cert to all servers using it.
- Replace it in load balancers, CDN, API gateways, custom services, make a list before starting.

## Revocation mechanics

- Public CAs: use the CA's revocation procedure. Specify reason `keyCompromise`.
- For ACME / Let's Encrypt: `certbot revoke --cert-path <path> --reason keycompromise`.
- The CA publishes the revocation via CRL and OCSP.
- Modern browsers use OCSP stapling; revocation propagation varies. For high-stakes services, expect the leaked cert to remain accepted by some clients for hours to weeks.

## Forward secrecy considerations

- If the cert/key was used with a non-PFS cipher suite (RSA key exchange), past captured TLS sessions can be decrypted with the private key. Audit ciphers and assume worst case if any were non-PFS.
- Modern TLS 1.3 mandates PFS. TLS 1.2 with ECDHE/DHE cipher suites also provides PFS. RSA key exchange does not.
- If you have any reason to believe traffic was captured (network compromise, MITM-capable adversary), assume the captures are now decryptable for any non-PFS sessions.

## Investigate

- Where was the key stored? File system, secret manager, HSM, KMS?
- Who/what had read access? Check audit logs.
- Was the key ever copied to: backup files, deployment artifacts, container images, AMIs, developer machines?
- Was the key embedded in code or config that was committed to git? Search history.

## Replacement

- Generate new keypair with current best practices (ECDSA P-256 or Ed25519, 2048+ RSA).
- New certificate with `keyCompromise` flagged in the CSR if appropriate.
- Deploy and verify with `openssl s_client -showcerts -connect host:443` and TLS scanning tools.

## Adjacent

- If this was a wildcard cert and used in many places, the rotation effort is large. Plan phased rollout but prioritize internet-facing first.
- HPKP is deprecated but if you're still pinning, update pins.
- Certificate transparency logs: the new cert will appear in CT logs publicly. Anomaly detection on CT logs can spot future unauthorized issuances.
- Code-signing certs: a leaked code-signing cert is severely impactful, attackers can sign malware as you. Revoke immediately and notify customers / OS vendors who may need to flag your software.

## Prevention

- Store private keys in HSM / cloud KMS where possible. Servers use the key by API call rather than holding it.
- Short-lived certificates (Let's Encrypt 90 days, automated renewal). Limits the value of any historical key leak.
- Per-service certificates rather than wildcard. Lateral compromise contained.
- File system permissions: key file readable only by the service account that needs it.
- Include certificate inventory in your asset management, you can't rotate what you don't know exists.
