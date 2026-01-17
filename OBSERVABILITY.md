# Observable Stack Deployment

This stack includes an optional setup for centralized logging and observability using the **ELK Stack** (Elasticsearch, Logstash, Kibana).

## Quick Start

The observability services (`kibana`, `logstash`) are part of the `observable` Docker Compose profile. They will not start by default.

### 1. Start the Stack
To start the standard stack *plus* the observability services:

```bash
docker compose --profile observable up -d
```

### 2. Access Kibana
Navigate to:
**https://localhost/kibana**

### 3. Login
You will be prompted for Basic Auth credentials:

- **Username**: `admin`
- **Password**: `password`

*(Note: Credentials are managed in `.env` via `KIBANA_USER` and `KIBANA_PASSWORD_HASH`. Use `caddy hash-password` to generate a new hash if needed. Highly recommended to change the password in production)*

### 4. Create Index Pattern
Before you can view logs, you need to tell Kibana which Elasticsearch indices to explore.

1.  Open the main menu (hamburger icon) and go to **Stack Management** > **Data Views** (or **Index Patterns**).
2.  Click **Create data view**.
3.  **Name**: Enter `Logs`.
4.  **Index pattern**: Enter `logstash-*` (this matches the indices created by Logstash).
5.  **Timestamp field**: Select `@timestamp`.
6.  Click **Save data view to Kibana**.

### 5. View Logs
1.  Open the main menu and go to **Discover**.
2.  Select your `Logs` data view in the top left.
3.  You can now see and search your logs (e.g., search for `service.type: "caddy"`).

## Architecture

### Log Flow
1.  **Services** (Caddy, MySQL, MediaWiki) write logs to shared Docker volumes (`caddy-logs`, `mysql-logs`, `mediawiki-logs`).
2.  **Logstash** mounts these volumes and tails the log files.
3.  **Logstash** parses the logs (JSON decoding, Grok patterns, Date parsing) and indexes them into **Elasticsearch**.
4.  **Kibana** queries Elasticsearch to display the logs.

### Configuration
- **Logstash Pipelines**: Located in `config/logstash/pipeline/`.
    - `caddy.conf`: Parses Caddy JSON access logs.
    - `mysql.conf`: Parses MySQL error and general logs.
    - `mediawiki.conf`: Parses MediaWiki logs.
- **Access Control**:
    - **Elasticsearch**: Open internally (port 9200) to allow easy communication for services (Logstash, MediaWiki).
    - **Kibana**: Reverse-proxied by Caddy at `/kibana` with **Basic Authentication**.

## Troubleshooting

**Logs aren't showing up?**
Check logstash logs to ensure parsing is working:
```bash
docker compose logs logstash --tail 50
```

**Kibana "Kibana server is not ready yet"?**
Kibana takes a minute to initialize. Wait 60 seconds and refresh.
