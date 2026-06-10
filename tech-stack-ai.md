<<<<<<< HEAD
# Tech Stack (AI)

## 1. Run Questions

### 1a. Config Files

| Config File | Location | Config Value | What It's For | How It's Used |
|---|---|---|---|---|
| `.env.template` | `learn-ops-api/` | `LEARNING_GITHUB_CALLBACK` | URL GitHub redirects to after OAuth login | Read by django-allauth to complete the GitHub OAuth flow |
| `.env.template` | `learn-ops-api/` | `LEARN_OPS_DB` | Name of the PostgreSQL database | Passed to Django's `DATABASES` setting via `dj-database-url` |
| `.env.template` | `learn-ops-api/` | `LEARN_OPS_DJANGO_SECRET_KEY` | Django's cryptographic signing key | Used by Django to sign cookies, sessions, and CSRF tokens |
| `.env.template` | `learn-ops-api/` | `VALKEY_HOST` | Hostname of the Valkey container | Used by the API and Monarch to connect to the Valkey message broker |
| `.env.template` | `learn-ops-api/` | `DEBUG` | Toggles Django debug mode | When `True`, Django shows detailed error pages; must be `False` in production |
| `.env.template` | `learn-ops-api/` | `SLACK_BOT_TOKEN` | Auth token for the Slack bot | Used to post notifications to Slack channels from the API |
| `.env` | `learn-ops-client/` | `REACT_APP_API_URI` | Base URL of the Django API | Prepended to all fetch calls in the React app to reach backend endpoints |
| `.env` | `learn-ops-client/` | `REACT_APP_ENV` | Current environment name | Used in the app to conditionally enable development-only features |
| `.env` | `learn-ops-client/` | `CHOKIDAR_USEPOLLING` | Forces file-watching to use polling | Required inside Docker containers where inotify events don't fire reliably |
| `.env` | `learn-ops-client/` | `GENERATE_SOURCEMAP` | Controls source map output during build | Set to `false` to reduce build size and avoid exposing source code in production |
| `.env` | `learn-ops-infrastructure/` | `POSTGRES_DB` | Name of the PostgreSQL database | Passed to the `postgres:16` Docker container at startup to create the database |
| `.env` | `learn-ops-infrastructure/` | `POSTGRES_USER` | Database username | Used by both the PostgreSQL container and `postgres_exporter` to authenticate |
| `.env` | `learn-ops-infrastructure/` | `DATA_SOURCE_NAME` | Full PostgreSQL connection string | Used by `postgres_exporter` to connect to the database and expose metrics to Prometheus |
| `.env.template` | `service-monarch/` | `GH_PAT` | GitHub Personal Access Token | Authenticates all GitHub API calls made by the Monarch migration service |
| `.env.template` | `service-monarch/` | `VALKEY_HOST` | Hostname of the Valkey container | Monarch connects here to subscribe to the `channel_migrate_issue_tickets` pub/sub channel |
| `.env.template` | `service-monarch/` | `SLACK_WEBHOOK_URL` | Slack incoming webhook URL | Monarch posts migration completion notifications to Slack via this URL |
| `learn-ops-api.yaml` | `learn-ops-api/config/` | `region: nyc` | DigitalOcean deployment region | Tells the DigitalOcean App Platform where to host the API droplet |
| `learn-ops-api.yaml` | `learn-ops-api/config/` | `deploy_on_push: true` | Auto-deploy on git push | DigitalOcean watches the `main` branch and redeploys the API automatically on every push |
| `learn-ops-api.yaml` | `learn-ops-api/config/` | `DATABASE_URL` | Runtime database connection string | Injected into the running container by DigitalOcean so Django can connect to the managed database |
| `nginx.api.conf` | `learn-ops-api/config/` | `server_name learningapi.nss.team` | Public domain for the API | nginx uses this to route incoming requests to the correct server block |
| `nginx.api.conf` | `learn-ops-api/config/` | `proxy_pass http://127.0.0.1:8000` | Forwards requests to Gunicorn | nginx receives HTTPS traffic and proxies it to the Django/Gunicorn process on port 8000 |
| `nginx.api.conf` | `learn-ops-api/config/` | `ssl_certificate` | Path to the TLS certificate | Used by nginx to terminate SSL/TLS — provided and managed by Certbot (Let's Encrypt) |
| `nginx.client.conf` | `learn-ops-api/config/` | `server_name learning.nss.team` | Public domain for the React client | nginx routes requests for this domain to the built React app's static files |
| `nginx.client.conf` | `learn-ops-api/config/` | `root /home/.../build` | Path to the React build output | nginx serves static files directly from this directory without hitting the API |
| `nginx.client.conf` | `learn-ops-api/config/` | `try_files $uri $uri/ /index.html` | Client-side routing fallback | Ensures React Router can handle deep links by always returning `index.html` for unknown paths |
| `nginx.conf` | `learn-ops-api/config/nginx/` | `worker_processes auto` | Number of nginx worker processes | Set to `auto` so nginx spawns one worker per CPU core for optimal performance |
| `nginx.conf` | `learn-ops-api/config/nginx/` | `client_max_body_size 40m` | Max allowed request body size | Prevents nginx from rejecting large file uploads (e.g. images) to the API |
| `nginx.conf` | `learn-ops-api/config/nginx/` | `keepalive_timeout 65` | How long to keep idle connections open | Reduces connection overhead for clients making multiple requests |
| `api.conf` | `learn-ops-api/config/nginx/conf.d/` | `upstream learningapicontainer` | Defines the upstream API server group | nginx load-balances requests to `apihost:8000` using this named upstream block |
| `api.conf` | `learn-ops-api/config/nginx/conf.d/` | `Access-Control-Allow-Origin *` | CORS header for cross-origin requests | Allows the React frontend (on a different domain) to call the API |
| `api.conf` | `learn-ops-api/config/nginx/conf.d/` | `return 301 https://$host$request_uri` | HTTP to HTTPS redirect | Forces all plain HTTP traffic to redirect to the secure HTTPS version of the site |
| `prometheus.yml` | `learn-ops-infrastructure/` | `scrape_interval: 15s` | How often Prometheus collects metrics | Prometheus polls all targets every 15 seconds and stores the results in its time-series database |
| `prometheus.yml` | `learn-ops-infrastructure/` | `job_name: 'django'` | Label for the Django metrics job | Groups all Django metrics under the `django` job name in Grafana dashboards |
| `prometheus.yml` | `learn-ops-infrastructure/` | `targets: ['postgres_exporter:9187']` | Address of the PostgreSQL exporter | Tells Prometheus where to scrape PostgreSQL database metrics |
| `docker-compose.yml` (valkey) | `learn-ops-infrastructure/valkey/` | `image: valkey/valkey:latest` | Valkey container image | Pulls the official Valkey image to run the Redis-compatible message broker |
| `docker-compose.yml` (valkey) | `learn-ops-infrastructure/valkey/` | `--save 900 1` | Persistence configuration | Tells Valkey to write a snapshot to disk if at least 1 key changes within 900 seconds |
| `docker-compose.yml` (valkey) | `learn-ops-infrastructure/valkey/` | `valkey-monitor` sidecar | Real-time command monitor | Runs `valkey-cli monitor` in a second container to stream all Valkey commands for debugging |
| `pytest.ini` | `learn-ops-api/` | `DJANGO_SETTINGS_MODULE` | Points pytest to the Django settings | Required so pytest-django can set up the Django app before running any tests |
| `pytest.ini` | `learn-ops-api/` | `--reuse-db` | Reuses the test database between runs | Skips recreating the database on every run, making tests significantly faster |
| `pytest.ini` | `learn-ops-api/` | `testpaths = LearningAPI/tests` | Directory where pytest looks for tests | Scopes test discovery so pytest doesn't scan the entire project on every run |
| `.pylintrc` | `learn-ops-api/` | `good-names=i,j,ex,pk` | Allowed short variable names | Tells pylint not to flag `i`, `j`, `ex`, or `pk` as too-short names (common in Django for primary keys) |
| `.pylintrc` | `learn-ops-api/` | `disable=broad-except` | Suppresses the broad-except warning | Allows `except Exception` catches without pylint errors — used in resilience/retry code |
| `.pylintrc` | `learn-ops-api/` | `disable=missing-function-docstring` | Suppresses missing docstring warnings | Lets developers skip docstrings on functions without pylint failing the lint check |

### 1b. How to Start It

All targets are run from `learn-ops-infrastructure/` using `make <target>`.

| Target | Command It Runs | What It Does |
|---|---|---|
| `make setup` | `./scripts/setup.sh` | First-time setup — runs a setup script that likely creates the Docker network, copies `.env` files, and prepares the environment before any containers are started |
| `make doctor` | `./scripts/setup.sh --doctor` | Checks that your local environment is healthy — verifies prerequisites like Docker, required env vars, and network configuration without changing anything |
| `make up` | `docker compose up --build -d` | Builds and starts **all** services (database, API, client, Prometheus, Grafana, postgres_exporter) in the background |
| `make up-api` | `docker compose up --build -d api` | Builds and starts **only the API** container — useful when you're only working on backend changes and don't need the client running |
| `make up-client-api` | `docker compose up --build -d api client` | Builds and starts **the API and client together** — the most common dev workflow when working across both frontend and backend |
| `make down` | `docker compose down` | Stops and removes all running containers, but **preserves volumes** (database data is kept) |
| `make restart` | `docker compose down` then `up --build -d` | Stops everything and brings it all back up with a fresh build — useful after config changes |
| `make reset` | `docker compose down -v --remove-orphans` | Nuclear option: stops containers **and deletes all volumes**, wiping the database. Use when you need a completely clean slate |
| `make logs` | `docker compose logs -f` | Streams live logs from all running containers to your terminal |
| `make ps` | `docker compose ps` | Lists all containers and their current status (running, stopped, health state) |

**Key differences to know:**
- `down` vs `reset` — `down` keeps your database data; `reset` deletes it permanently via `-v`
- `up` vs `up-api` vs `up-client-api` — these are scoped shortcuts so you don't spin up the full stack when you only need part of it
- `restart` vs `reset` — `restart` is a safe reboot; `reset` is destructive and drops all data

### 1c. Where to Access It

| Service | Port | Local URL |
|---|---|---|
| React Client | 3000 | http://localhost:3000 |
| Django API | 8000 | http://localhost:8000 |
| Grafana | 3001 | http://localhost:3001 |
| Prometheus | 9090 | http://localhost:9090 |
| Valkey | 6379 | `valkey:6379` (internal only — no browser UI) |
| PostgreSQL | 5432 | `database:5432` (internal only — connect via a DB client) |
| postgres_exporter | 9187 | http://localhost:9187/metrics |
| Monarch metrics | 8080 | http://localhost:8080/metrics |
| Monarch log viewer | 8081 | http://localhost:8081 |
| Django debugpy | 5678 | `localhost:5678` (attach a remote debugger, not a browser) |

**Production URLs (from nginx config):**

| Service | URL |
|---|---|
| React Client | https://learning.nss.team |
| Django API | https://learningapi.nss.team |


### 1d. Service Dependencies

| Service | Depends On | Why |
|---|---|---|
| `api` | `database` (health check) | Django runs migrations and opens a connection pool on startup — if PostgreSQL isn't ready yet, the app crashes immediately. The health check forces Docker to wait until Postgres is actually accepting connections, not just started |
| `api` | `valkey` (implicit) | The API publishes messages to Valkey's pub/sub channel so Monarch knows when to start a migration. Without Valkey, that publish call fails at runtime |
| `client` | `api` (implicit) | The React app makes fetch calls to the Django API for all its data. The client container can start without it, but nothing in the UI will work until the API is up |
| `prometheus` | `api` | Prometheus is configured to scrape the `/metrics/metrics` endpoint on the API container. If the API isn't running, Prometheus has nothing to scrape — it would just log errors on every scrape interval |
| `grafana` | `prometheus` | Grafana reads all its time-series data from Prometheus as a data source. Without Prometheus running, every dashboard panel shows "no data" |
| `postgres_exporter` | `database` | Its entire job is to connect to PostgreSQL and translate database internals (connections, locks, query stats) into Prometheus metrics. It can't function at all without a live database to query |
| `valkey-monitor` | `valkey` | It runs `valkey-cli monitor` which streams every command processed by Valkey in real time. It has nothing to connect to if Valkey itself isn't running |
| `monarch` | `valkey` (implicit) | Monarch subscribes to a Valkey pub/sub channel to receive migration requests. Without Valkey, the service starts but never receives any work and can't store logs or heartbeats |
| `monarch` | GitHub API (external) | Every migration task calls the GitHub REST API to read issues from a source repo and create them in a target repo. The circuit breaker trips and migrations fail if GitHub is unreachable |

### 1e. Main Entry Points

| Service | Startup File | Routes / URL Config File |
|---|---|---|
| `learn-ops-api` | `entrypoint.sh` | `LearningPlatform/urls.py` |
| `learn-ops-client` | `src/index.js` | `src/components/ApplicationViews.js` |
| `service-monarch` (migration engine) | `service/main.py` | N/A — event-driven, no HTTP routes |
| `service-monarch` (log web UI) | `service/custom_logging/web_interface.py` | `service/custom_logging/web_interface.py` |

**What each file does:**

- `entrypoint.sh` — the shell script Docker runs first when the API container starts. It waits for PostgreSQL to be ready, runs `manage.py migrate` to apply any pending database changes, seeds initial fixture data if the database is empty, then hands off to Gunicorn (or `debugpy` if `DEBUG=True`)
- `LearningPlatform/urls.py` — the single source of truth for every URL the Django API responds to. It registers all REST resource routes (assessments, cohorts, students, etc.) via DRF's router, plus auth endpoints, the GitHub OAuth callback, the Prometheus `/metrics` endpoint, and the admin panel
- `src/index.js` — the React app's entry point. It mounts the root `<LearnOps />` component into the HTML page, wraps it in `BrowserRouter` (enabling client-side routing), and applies the Radix UI theme
- `src/components/ApplicationViews.js` — defines all the React Router `<Route>` paths for the app. This is where URLs map to page components (dashboards, cohort views, course forms, etc.)
- `service/main.py` — starts the Monarch migration service by creating a `TicketMigrator` instance and running its async event loop. It has no HTTP routes — it listens for messages on a Valkey pub/sub channel instead
- `service/custom_logging/web_interface.py` — a small Flask app that runs inside Monarch on port 8081. It defines the routes for the log viewer UI (`/`), the health check (`/health`), and the log query API (`/api/logs`, `/api/log-levels`, `/api/services`)

## 2. Services

| Service Name | Tech Stack | Purpose |
|---|---|---|
| `learn-ops-api` | Python 3.11, Django 4.x, Django REST Framework, PostgreSQL 16, Gunicorn, Valkey, structlog, django-prometheus, django-allauth, dj-rest-auth | The backend REST API for the LMS. Manages all data (students, cohorts, assessments, courses), handles GitHub OAuth login, exposes Prometheus metrics, and publishes migration events to Valkey for Monarch to pick up |
| `learn-ops-client` | React 16, Node 22.13.0, React Router v5, Radix UI v1, Tailwind CSS v3, Chart.js v4 | The instructor-facing frontend web app. Lets staff manage cohorts, track student progress, record assessments, and visualize learning data through charts |
| `service-monarch` | Python 3.x, asyncio, Flask 3.0.3, Pydantic v2, Valkey 6.0.2, prometheus-client 0.21.1, structlog 24.4.0, tenacity 9.0.0 | A standalone microservice that migrates GitHub issues from source repositories to target repositories. Listens for work on a Valkey pub/sub channel, calls the GitHub API to move issues, and exposes a Prometheus metrics endpoint and a browser-based log viewer |
| `learn-ops-infrastructure` | Docker Compose, PostgreSQL 16, Prometheus, Grafana, postgres_exporter, Valkey (valkey/valkey:latest), nginx | Orchestrates the entire stack locally using Docker Compose. Provides the database, message broker, reverse proxy, and the full observability stack (metrics collection, dashboards, and database monitoring) |

## 3. System Overview

This is a Learning Management System (LMS) built for a software development bootcamp. The problem it solves is the operational complexity of running cohort-based technical education at scale — tracking where each student is in a curriculum, recording assessment results, managing course content, and coordinating the people and teams involved. Rather than relying on spreadsheets or generic tools, it provides a purpose-built platform that keeps instructors, students, and curriculum data in one place.

From a user's perspective, the system centers on a dashboard that gives an at-a-glance view of student progress through the curriculum. Instructors can create and manage cohorts, assign students to teams, record assessment outcomes, and leave notes on individual students. The curriculum itself — books, projects, learning objectives, and timelines — is managed through dedicated views. Students get their own dashboard showing their personal goals, assessment status, a cohort calendar, and links to their GitHub repositories and capstone proposal forms. Charts powered by Chart.js give instructors visual summaries of assessment data across a cohort, and Slack notifications keep the team informed of key events without requiring them to stay in the app.

Two distinct roles interact with the system differently. Instructors (staff) have access to the full application — cohort management, student records, assessment scoring, course authoring, and team assignment. Students have a narrower, read-focused view: they can see their own dashboard, check their learning goals, view their cohort calendar, and submit capstone-related forms, but they cannot see or modify other students' data or manage curriculum content. Authentication for both roles is handled via GitHub OAuth, which means every user logs in with their existing GitHub account rather than creating a separate password.
=======
AI overview
>>>>>>> upstream/main
