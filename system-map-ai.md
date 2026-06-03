# System Map (AI)

## 1. System Diagram

```mermaid
graph LR
    subgraph Application
        client["learn-ops-client<br/>React 16.13<br/>port: 3000"]
        api["learn-ops-api<br/>Django REST Framework · Python 3.11<br/>port: 8000 · debug: 5678"]
        monarch["service-monarch<br/>Flask · Python 3.11<br/>metrics: 8080 · logs: 8081"]
    end

    subgraph DataStores["Data Stores"]
        db[("database<br/>PostgreSQL 16<br/>port: 5432")]
        valkey[("valkey<br/>Redis-compatible<br/>port: 6379")]
    end

    subgraph Monitoring
        exporter["postgres_exporter<br/>port: 9187"]
        prometheus["prometheus<br/>port: 9090"]
        grafana["grafana<br/>port: 3001"]
        valkeymon["valkey-monitor<br/>debug sidecar"]
    end

    subgraph External["External Services"]
        github["GitHub API<br/>api.github.com · HTTPS<br/>Auth: PAT"]
        slack["Slack API<br/>slack.com/api · HTTPS<br/>Auth: Bot Token"]
        logstash["Logstash<br/>port: 5000 · TCP<br/>configured, not in compose"]
    end

    client      -->|"HTTP REST · :8000"| api
    api         -->|"DB query (psycopg2) · :5432"| db
    api         -->|"pub/sub publish + cache r/w · :6379"| valkey
    valkey      -->|"pub/sub subscribe · channel_migrate_issue_tickets"| monarch
    exporter    -->|"DB query · :5432"| db
    prometheus  -->|"HTTP scrape · :8000/metrics · 15s"| api
    prometheus  -->|"HTTP scrape · :9187"| exporter
    grafana     -->|"HTTP query · :9090"| prometheus
    valkeymon   -->|"MONITOR cmd · :6379"| valkey
    api         -->|"HTTP REST · HTTPS"| github
    monarch     -->|"HTTP REST · HTTPS"| github
    api         -->|"HTTP REST · HTTPS"| slack
    monarch     -->|"HTTP REST · HTTPS"| slack
    api         -->|"TCP logging · :5000"| logstash
```
