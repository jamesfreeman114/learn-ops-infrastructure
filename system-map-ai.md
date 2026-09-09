# System Map (AI)

## 1. System Diagram

```mermaid
graph LR
    Client["Client<br/>(React)"]
    API["API<br/>(Django + DRF)"]
    Database["Database<br/>(PostgreSQL 16)"]
    Valkey["Valkey<br/>(cache + pub/sub)"]
    Monarch["Monarch<br/>(Python service)"]
    GitHubAPI["GitHub API<br/>(external)"]
    SlackAPI["Slack API<br/>(external)"]
    Grafana["Grafana"]
    Prometheus["Prometheus"]
    PostgresExporter["Postgres Exporter"]

    Client -->|"HTTP REST/JSON :8000"| API
    API -->|"SQL :5432"| Database
    API -->|"Valkey GET/SET :6379 (cache/session)"| Valkey
    API -->|"Valkey PUBLISH :6379 (channel_migrate_issue_tickets)"| Valkey
    API -->|"HTTPS REST (OAuth2, repos, org)"| GitHubAPI
    API -->|"HTTPS REST (Slack Web API)"| SlackAPI
    Monarch -->|"Valkey SUBSCRIBE :6379 (state/log storage)"| Valkey
    Monarch -->|"HTTPS REST (issue migration)"| GitHubAPI
    Monarch -->|"HTTPS REST (Slack Web API)"| SlackAPI
    Grafana -->|"HTTP PromQL :9090"| Prometheus
    Prometheus -->|"HTTP scrape :8000 (/metrics/metrics)"| API
    Prometheus -->|"HTTP scrape :9187"| PostgresExporter
    PostgresExporter -->|"Postgres wire protocol :5432"| Database
```