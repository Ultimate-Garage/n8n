# n8n Docker Compose

This repository contains a production-oriented Docker Compose setup for running n8n with PostgreSQL as the n8n application database, Redis-backed queue mode, a dedicated worker, persistent volumes, and an explicit n8n encryption key.

## Files

- `docker-compose.yml` defines PostgreSQL, Redis, the n8n web/main container, and an n8n worker.
- `.env.example` documents every required environment variable. Copy it to `.env` before starting.

## Quick start

1. Copy the example environment file:

   ```bash
   cp .env.example .env
   ```

2. Generate a strong n8n encryption key and put it in `.env`:

   ```bash
   openssl rand -hex 32
   ```

   Keep `N8N_ENCRYPTION_KEY` stable forever for this instance. n8n uses it to encrypt saved credentials; losing or changing it can make existing credentials unreadable.

3. Replace every `CHANGE_ME` value in `.env`.

4. Set the public URL values for your deployment:

   ```dotenv
   N8N_HOST=n8n.example.com
   N8N_PROTOCOL=https
   WEBHOOK_URL=https://n8n.example.com/
   ```

5. Start n8n:

   ```bash
   docker compose up -d
   ```

## Environment variables

### n8n public URL and runtime

| Variable | Purpose |
| --- | --- |
| `N8N_VERSION` | n8n image tag. Pin a tested version for production instead of leaving `latest`. |
| `N8N_HOST` | Public hostname used by n8n. |
| `N8N_PORT` | Host port exposed by Docker Compose. Defaults to `5678`. |
| `N8N_PROTOCOL` | External protocol, normally `https` behind a reverse proxy. |
| `WEBHOOK_URL` | Full externally reachable URL for webhook registration and callbacks. |
| `N8N_SECURE_COOKIE` | Keeps browser cookies HTTPS-only when serving n8n over TLS. |
| `GENERIC_TIMEZONE` | Default timezone for schedules and display. |
| `N8N_ENCRYPTION_KEY` | Required stable secret used to encrypt n8n credentials. Generate with `openssl rand -hex 32`. |
| `N8N_DIAGNOSTICS_ENABLED` | Set to `false` to disable diagnostics telemetry. |
| `N8N_PERSONALIZATION_ENABLED` | Set to `false` to disable personalization prompts/tracking. |
| `EXECUTIONS_DATA_MAX_AGE` | Execution history retention in hours. Defaults to 14 days (`336`). |
| `N8N_WORKER_CONCURRENCY` | Number of jobs the worker processes concurrently. |

### PostgreSQL for n8n

n8n stores its own workflows, credentials metadata, and execution data in PostgreSQL.

| Variable | Purpose |
| --- | --- |
| `POSTGRES_DB` | PostgreSQL database name for n8n. |
| `POSTGRES_USER` | PostgreSQL user for n8n. |
| `POSTGRES_PASSWORD` | Strong password for the n8n PostgreSQL user. |

### Redis queue mode

Redis powers n8n queue mode so the main container can hand executions to the worker.

| Variable | Purpose |
| --- | --- |
| `REDIS_PASSWORD` | Strong Redis password shared by the n8n main and worker containers. |

### Azure SQL / Microsoft SQL native connection

The Compose stack intentionally keeps n8n's own database on PostgreSQL. To connect workflows to Azure SQL Database, use n8n's built-in **Microsoft SQL** credential and node, then enter the values documented in `.env`:

| Variable | Enter in n8n Microsoft SQL credential |
| --- | --- |
| `AZURE_SQL_SERVER` | Server, for example `your-server.database.windows.net`. |
| `AZURE_SQL_PORT` | Port, normally `1433`. |
| `AZURE_SQL_DATABASE` | Azure SQL database name. |
| `AZURE_SQL_USER` | SQL authentication username. |
| `AZURE_SQL_PASSWORD` | SQL authentication password. |
| `AZURE_SQL_ENCRYPT` | Enable encryption/TLS. Use `true` for Azure SQL. |
| `AZURE_SQL_TRUST_SERVER_CERTIFICATE` | Trust server certificate. Use `false` for Azure SQL production. |
| `AZURE_SQL_CONNECTION_TIMEOUT` | Connection timeout in milliseconds. |

> Note: n8n's Microsoft SQL credential currently uses a SQL database connection. Keep these Azure SQL values in your password manager or secret store and enter them in n8n credentials; they are documented in `.env.example` so operators know exactly what is required.

## Operational recommendations

- Put n8n behind a TLS-terminating reverse proxy and keep `N8N_PROTOCOL=https`, `WEBHOOK_URL=https://...`, and `N8N_SECURE_COOKIE=true`.
- Back up the `postgres_data` volume and the `.env` file, especially `N8N_ENCRYPTION_KEY`.
- Pin `N8N_VERSION` to a tested version before production upgrades.
- Scale workers by increasing replicas for `n8n-worker` if your host has enough CPU and memory.
- Restrict access to `.env`; it contains database passwords and the credential encryption key.
