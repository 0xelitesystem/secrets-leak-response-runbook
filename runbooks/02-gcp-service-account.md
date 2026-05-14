# GCP Service Account Key Leaked

GCP service account keys are JSON files containing a private key. Highly sensitive.

## Identify the key

The leaked file has fields: `type: service_account`, `client_email`, `private_key_id`, `project_id`.

The `private_key_id` is the specific key identifier. A service account can have multiple.

```bash
# List keys for the service account
gcloud iam service-accounts keys list \
  --iam-account=<email>
```

## Disable / delete

GCP doesn't have a "disable" state for keys — you delete:

```bash
gcloud iam service-accounts keys delete <KEY_ID> \
  --iam-account=<email>
```

Once deleted, the key is unusable immediately. There's no grace period.

## Investigate

GCP Audit Logs show service account activity:
- Console → Logging → Logs Explorer
- Filter: `protoPayload.authenticationInfo.principalEmail="<service-account-email>"`
- Look at: BigQuery queries (data exfil), Compute creation (mining), IAM modifications, GCS bucket changes, Secret Manager reads.

## Replacement

If the workload runs on GCP:
- Use Workload Identity Federation (for GKE).
- Use service account attached to GCE / Cloud Run / App Engine / Cloud Functions.
- Avoid creating new keys when possible.

If keys are unavoidable (external systems):
- Generate new key, deploy, verify, delete old.
- Configure key rotation cadence (90 days max).

## Broader containment

- Check IAM policies the service account had. Anything sensitive (Owner, Editor, custom roles with secret access) means treat the leak as severe.
- Check what data it could read. If it had `roles/storage.objectViewer` on production data, assume that data is exposed.
- Look for created keys, modified IAM bindings, new service accounts — attacker persistence patterns.

## Org-level prevention

- Disable service account key creation at the org level: `iam.disableServiceAccountKeyCreation` constraint.
- Enforce key rotation: `iam.serviceAccountKeyExpiry` (Premium tier).
- Use Cloud KMS-managed keys for what would otherwise be embedded credentials.
- Detect leaked keys with Sensitive Data Protection (formerly DLP) on Cloud Storage and BigQuery.

## Don't forget

- Rotate any secrets the key could have read from Secret Manager.
- If `serviceAccountActAs` was granted to the key, identify and address downstream service accounts the attacker could have impersonated.
