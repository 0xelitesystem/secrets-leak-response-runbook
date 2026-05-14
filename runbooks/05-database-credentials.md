# Database Credentials Leaked

DB connection strings or credentials exposed.

## Immediate

If the database is reachable from the internet:
- Block at the network layer immediately. Security group / firewall rule to drop all traffic.
- This will break your application — accept the outage, restore via the next steps.

If the database is internal-only:
- Lower urgency, but still rotate immediately.

## Identify exposure

- Was the DB internet-reachable during the leak window? Check security groups, network ACLs, public IP / DNS.
- Was the credential the only authentication, or also TLS / IP allowlist / VPN?
- What was the user's privilege level? (Read-only is much better than DBA.)

## Investigate

Database-side logs:
- PostgreSQL: `log_connections=on`, `log_disconnections=on`, `log_statement=ddl` minimum. Review pg_stat_activity for long-running suspicious queries.
- MySQL: general query log if enabled, audit plugin if available.
- SQL Server: SQL Server Audit, Extended Events.
- MongoDB: `auditLog.destination`.

Look for:
- Connections from unfamiliar IPs in the leak window.
- Large `SELECT *` or `mysqldump`-style queries (data exfil).
- DDL changes (table creation/modification, especially `CREATE USER`, privilege grants).
- Stored procedure / function creation.

## Rotate

- Create new credentials.
- Update application configuration / secret manager.
- Deploy the change.
- Verify the application is working with new credentials.
- Disable / drop the old user (`ALTER USER ... NOLOGIN` or `DROP USER`).

## Containment

- Audit DB users for any created during the leak window — attacker persistence.
- Check for new triggers, stored procedures, views — backdoors.
- If the leaked credential had write access, check for malicious data modifications. This may require comparing to a backup taken before the leak.
- If the leaked credential had DBA access, treat the database as fully compromised. Restore from clean backup or perform very thorough cleanup.

## Replace pattern (for the future)

- Use IAM database authentication where supported (RDS IAM auth, Cloud SQL IAM, Azure AD).
- Use a secret manager that rotates DB credentials automatically (AWS Secrets Manager, HashiCorp Vault dynamic secrets).
- Move the database off the public internet. Use VPC, PrivateLink, IP allowlists.
- Per-application credentials with least privilege; not "one DBA account everyone uses".
- Connection pooler (PgBouncer, RDS Proxy) so credential rotation doesn't require app restart in some configurations.

## Notification

- If customer data may have been accessed, follow data breach notification rules (see incident-response-runbooks: 08-data-exfiltration).
