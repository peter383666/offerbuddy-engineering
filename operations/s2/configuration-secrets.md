# Sprint 2 Configuration and Secrets

## Purpose

Records Sprint 2 configuration concerns without publishing secret values.

## Existing Baseline

Production secrets remain outside git:

- EC2-local `.env` for Compose/backend
- GitHub Environment secrets for deploy workflows

Never commit `.env`, tokens, dumps, or private keys.

## Sprint 2 Additions / Attention Points

| Concern | Notes |
| --- | --- |
| `GOOGLE_API_KEY` | Required for Gemini-backed Job Intelligence / AI URL parsing in environments where those features should run |
| Extension origins | Production Extension build targets offerbuddy.io; local build uses local origins |
| Analytics no-response threshold | Configurable application setting (default 14 days in implementation); not a secret |
| Extension credentials | Stored as hashes server-side; raw credentials exist only in the Extension privileged store |
| Redis | Compose may still define Redis host vars; application does not use Redis for S2 features |

## Related

- [Deployment](deployment.md)
- [Observability](observability.md)
- [Production Runbook](../production-runbook.md)
