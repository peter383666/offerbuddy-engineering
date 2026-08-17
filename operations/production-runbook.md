# Production Runbook

## Purpose

This runbook records the Sprint 1 checks used to deploy, inspect, restart, recover, and roll back the single-host production environment. It does not replace AWS account or host-access procedures.

Production layout:

```text
/var/www/offerbuddy            React files served by Nginx
/opt/offerbuddy                Compose file, .env, deployment files, backups
Docker Compose                 backend, postgres, reserved redis
host systemd                   Nginx
```

Never copy `.env`, private keys, tokens, database dumps, or secret values into issues or documentation.

## Routine Health Checks

From the EC2 host:

```bash
cd /opt/offerbuddy
docker compose ps
curl -fsS http://127.0.0.1:8080/actuator/health
sudo nginx -t
curl -fsS -o /dev/null https://offerbuddy.io
```

Expected results:

- PostgreSQL is healthy.
- The backend is running/healthy and returns `{"status":"UP"}`.
- Nginx configuration validation succeeds.
- The public HTTPS homepage returns success.

The Nginx configuration also proxies `/actuator/`; production health details are restricted with `show-details: never`. Loopback is the preferred operator health path.

## Post-Deployment Verification

After a frontend or backend deployment:

1. Record the selected full SHA.
2. Check Compose service state.
3. Check backend health on loopback.
4. Check the HTTPS homepage.
5. Verify Google sign-in when authentication/proxy/configuration changed.
6. Exercise the affected application path, such as list or create.
7. Inspect recent backend and Nginx errors.
8. Confirm persistence when the change affects data.

Do not interpret homepage reachability alone as proof that OAuth, the API, PostgreSQL, or Gemini is working.

## Logs

### Backend

```bash
cd /opt/offerbuddy
docker compose logs --tail=200 backend
docker compose logs -f backend
```

### PostgreSQL

```bash
cd /opt/offerbuddy
docker compose logs --tail=200 postgres
docker compose logs -f postgres
```

### Redis

Redis is inactive from the application's perspective, but its container logs are available for infrastructure checks:

```bash
cd /opt/offerbuddy
docker compose logs --tail=200 redis
```

### Nginx

```bash
sudo systemctl status nginx
sudo journalctl -u nginx --since "30 minutes ago"
sudo tail -n 200 /var/log/nginx/access.log
sudo tail -n 200 /var/log/nginx/error.log
```

Avoid publishing logs containing personal data, URLs, tokens, OAuth codes, or environment values.

## Service Restart

Before restarting, record the current backend image/SHA and inspect service state.

Restart only the affected application container where possible:

```bash
cd /opt/offerbuddy
docker compose restart backend
docker compose ps
curl -fsS http://127.0.0.1:8080/actuator/health
```

Nginx changes use validation followed by reload:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Do not remove volumes during a routine restart. Commands that delete Compose volumes would remove persistent PostgreSQL data.

## EC2 Reboot Recovery

Sprint 1 verified recovery after an EC2 reboot. The expected sequence is:

1. EC2 returns to a reachable state.
2. Nginx starts through systemd.
3. Docker starts the Compose services configured with `restart: unless-stopped`.
4. PostgreSQL reuses its named volume.
5. The backend waits for healthy PostgreSQL and becomes healthy.

Verify after reboot:

```bash
sudo systemctl status nginx
cd /opt/offerbuddy
docker compose ps
curl -fsS http://127.0.0.1:8080/actuator/health
curl -fsS -o /dev/null https://offerbuddy.io
```

Then run an authenticated read and confirm existing PostgreSQL-backed data is present.

## Rollback

### Application rollback

1. Identify the last known-good full commit SHA.
2. Confirm its backend GHCR image or frontend CI artifact still exists.
3. Manually dispatch the appropriate deployment workflow with `version=sha` and the full SHA.
4. Observe workflow health checks.
5. Perform the post-deployment verification above.

Frontend and backend may be rolled back independently only when their API contract remains compatible.

### Database considerations

Redeploying an earlier application artifact does not undo Flyway migrations or restore data. Before rolling back code across a schema change, determine whether the earlier application is compatible with the current schema.

Database recovery uses a verified backup or a separately planned corrective migration. Never edit an already-deployed Flyway migration to simulate rollback.

## Production Configuration

The current production configuration is an EC2-local `/opt/offerbuddy/.env` plus GitHub Environment secrets used by deployment workflows.

Required categories include:

- database name, user, URL, and password
- Google OAuth client ID and secret
- frontend base URL
- backend image identity
- optional Gemini API key

GitHub Environment secrets include EC2 host/user/key and, where required, a registry read token.

The `.env` file should be readable only by the appropriate host account. Real values must never be committed. AWS SSM Parameter Store, Secrets Manager, and IAM-role-based retrieval remain future hardening work.

## Escalation Triggers

Stop routine deployment and investigate when:

- PostgreSQL is unhealthy or the volume is missing
- Flyway fails
- the backend never reaches `UP`
- OAuth redirects use the wrong host or scheme
- Nginx strips the `/api` prefix
- an expected immutable artifact cannot be identified
- restored data cannot be verified
- rollback would cross an incompatible database migration

See [Deployment Strategy](deployment-strategy.md) and [PostgreSQL Backup and Restore](postgresql-backup-and-restore.md).
