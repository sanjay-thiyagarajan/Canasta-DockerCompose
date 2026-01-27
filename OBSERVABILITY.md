# Observable Stack Deployment

This stack includes an optional setup for centralized logging and observability using the **ELK Stack** (OpenSearch, Logstash, OpenSearch Dashboards).

## Quick Start

The observability services (`opensearch`, `logstash`) are part of the `observable` Docker Compose profile. They will not start by default.

### 1. Start the Stack
To start the standard stack *plus* the observability services:

```bash
docker compose --profile observable up -d
```

### 2. Access OpenSearch Dashboards
Navigate to:
**https://localhost/opensearch**

### 3. Login
You will be prompted for Basic Auth credentials:

- **Username**: `admin`
- **Password**: `password`

*(Note: Credentials are managed in `.env` via `OS_USER` and `OS_PASSWORD_HASH`. Use `caddy hash-password` to generate a new hash if needed. Highly recommended to change the password in production)*

### 4. Create Index Pattern
Before you can view logs, you need to tell OpenSearch Dashboards which indices to explore.

1.  Open the main menu (hamburger icon) and go to **Dashboards Management**.
2.  In the left sidebar, look under **OpenSearch Dashboards** and click **Index Patterns**.
2.  Click **Create index pattern**.
3.  **Index pattern name**: Enter `*-logs-*` (this matches `mediawiki-logs-*`, `caddy-logs-*`, etc.).
4.  **Time field**: Select `@timestamp`.
5.  Click **Create index pattern**.

### 5. View Logs
1.  Open the main menu and go to **Discover**.
2.  Select your `*-logs-*` index pattern in the top left.
3.  You can now see and search your logs (e.g., search for `log_type: "caddy_access"`).

## Architecture

### Log Flow
1.  **Services** (Caddy, MySQL, MediaWiki) write logs to shared Docker volumes (`caddy-logs`, `mysql-logs`, `mediawiki-logs`).
2.  **Logstash** mounts these volumes and tails the log files.
3.  **Logstash** parses the logs (JSON decoding, Grok patterns, Date parsing) and indexes them into **OpenSearch**.
4.  **OpenSearch Dashboards** queries OpenSearch to display the logs.

### Configuration
- **Logstash Pipelines**: Located in `config/logstash/pipeline/`.
    - `caddy.conf`: Parses Caddy JSON access logs.
    - `mysql.conf`: Parses MySQL error and general logs.
    - `mediawiki.conf`: Parses MediaWiki logs.
- **Access Control**:
    - **OpenSearch**: Open internally (port 9200) to allow easy communication for services (Logstash, MediaWiki).
    - **OpenSearch Dashboards**: Reverse-proxied by Caddy at `/opensearch` with **Basic Authentication**.

## Troubleshooting

**Logs aren't showing up?**
Check logstash logs to ensure parsing is working:
```bash
docker compose logs logstash --tail 50
```

** "OpenSearch Dashboards server is not ready yet"?**
OpenSearch Dashboards takes a minute to initialize. Wait 60 seconds and refresh.
