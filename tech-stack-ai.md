# Tech Stack (AI)

## 1. Run Questions

### 1a. Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|
| .env | learn-ops-api/.env | `LEARN_OPS_HOST` | Hostname/address of the Postgres database server | Read via `os.getenv()` into `settings.py`'s `DATABASES` dict so Django's ORM knows where to connect; `entrypoint.sh` also uses it to wait for Postgres to be reachable before starting the app |
| .env | learn-ops-api/.env | `LEARN_OPS_DB` | Name of the Postgres database the API should use | Read into `DATABASES['default']['NAME']` in `settings.py`, so every query/migration targets the correct database |
| .env | learn-ops-api/.env | `DEBUG` | Toggles Django's debug mode (verbose error pages, debug toolbar) | Read in `settings.py` to set Django's `DEBUG` flag; `entrypoint.sh` also checks it to decide whether to launch the server under `debugpy` for remote debugging |
| settings.py | learn-ops-api/LearningPlatform/settings.py | `DATABASES` | Defines the DB engine and connection details Django's ORM uses | Django reads this dict at startup to open a connection to Postgres for every query and migration |
| settings.py | learn-ops-api/LearningPlatform/settings.py | `CORS_ORIGIN_WHITELIST` | List of origins allowed to make cross-origin requests to the API | The `django-cors-headers` middleware checks each request's `Origin` header against this list to decide whether to allow it, which is what lets the React client (running on a different port) call the API |
| settings.py | learn-ops-api/LearningPlatform/settings.py | `ALLOWED_HOSTS` | List of hostnames Django will accept requests for | Django checks each request's `Host` header against this list and rejects requests that don't match, protecting against Host header attacks |
| docker-compose.yml | learn-ops-infrastructure/docker-compose.yml | `services.api` | Defines how to build/run the Django API container (image, env file, ports, dependencies) | `docker compose up` uses this block to build the API image, inject the `.env` variables, map its port to the host, and wait on the database's healthcheck before starting it |
| docker-compose.yml | learn-ops-infrastructure/docker-compose.yml | `services.client` | Defines how to build/run the React client container | `docker compose up` builds/runs the client from `learn-ops-client` and exposes its dev server port so the browser can load the frontend |
| docker-compose.yml | learn-ops-infrastructure/docker-compose.yml | `networks` | Declares the shared Docker network (named under the top-level `networks` key) that all services join | Lets containers resolve each other by service name (e.g. the API reaching the database via the hostname `database`) instead of hardcoded IPs |
| package.json | learn-ops-client/package.json | `dependencies` | Lists the packages (and versions) the React app needs to run | `npm install` reads this list to download and install the correct packages into `node_modules` |
| package.json | learn-ops-client/package.json | `browserslist` | Specifies which browsers/versions the client needs to support | Tools like Autoprefixer and Babel (used internally by `react-scripts`) read this to decide what CSS prefixes/JS transforms to add during the build |
| package.json | learn-ops-client/package.json | `eslintConfig` | Defines the linting rules for the client codebase | ESLint reads this to flag code-style and error issues when linting/testing runs |

### 1b. How to Start It

The `learn-ops-infrastructure/Makefile` wraps Docker Compose. The targets that actually **start** something differ mainly in scope (how many services) and whether they force a clean restart:

| Target | What it starts | Notes |
|---|---|---|
| `up` | Pulls (`pull`) then `docker compose up --build -d` with no service named — builds/starts everything (`database`, `api`, `client`, `prometheus`, `grafana`, `postgres_exporter`), then tails all logs | The "start everything" command; doesn't stop existing containers first, so it just reconciles changes |
| `up-api` | `docker compose up --build -d api` — only the `api` service (plus `database`, since `api` depends on it) | Skips `client` and the monitoring stack; useful when you only need the backend, e.g. hitting the API directly |
| `up-client-api` | `docker compose up --build -d api client` — `api` + `client` (and their dependencies) | The typical full-stack dev loop — frontend + backend, no monitoring stack |
| `restart` | Pulls, then explicit `docker compose down` followed by `up --build -d` (everything) | Unlike `up`, this forces a full stop-and-recreate instead of just reconciling — use it when you want a guaranteed clean state |

Two other targets are related but aren't really "starting the system":
- **`setup`** — runs `scripts/setup.sh`, a one-time onboarding script that installs prerequisites and creates the `.env` files needed before any `up` target will work.
- **`doctor`** — runs that same script in `--doctor` mode, which diagnoses your environment instead of starting anything.

### 1c. Where to Access It

| Service | Port | URL |
|---|---|---|
| database (Postgres) | `5433` (host) → `5432` (container) | N/A — DB connection, not HTTP; connect via `localhost:5433` with a Postgres client |
| api (Django) | `8000` (app), `5678` (debugpy attach) | http://localhost:8000 |
| client (React) | `3000` | http://localhost:3000 |
| prometheus | `9090` | http://localhost:9090 |
| grafana | `3001` (host) → `3000` (container) | http://localhost:3001 |
| postgres_exporter | `9187` | http://localhost:9187/metrics |
| valkey (cache) | `6379` | N/A — Redis-protocol connection, not HTTP; connect via `localhost:6379` |
| valkey-monitor | none exposed | N/A — attaches to `valkey` internally to stream commands, nothing to browse to |
| monarch | `8080` (Prometheus metrics), `8081` (log web UI) | http://localhost:8080/metrics and http://localhost:8081 |

### 1d. Service Dependencies

| Service | Depends On | Why |
|---|---|---|
| api | database | Django's ORM (`DATABASES` in `settings.py`) uses Postgres as its primary datastore for every query and migration; compose's `depends_on: database: condition: service_healthy` blocks the API from starting until Postgres is actually ready |
| api | valkey | `LearningAPI/views/popular_query.py` caches search results in Valkey (`valkey_client.get('search_results')`), and `LearningAPI/views/team_maker_view.py` publishes to the `channel_migrate_issue_tickets` channel — Valkey is used both as a cache and as a pub/sub message bus |
| client | api | The client's `REACT_APP_API_URI` env var points at the API's base URL; the React app holds no data of its own, so every page fetches its data from the API over HTTP |
| monarch | valkey | `service/core/monarch.py` subscribes to the same `channel_migrate_issue_tickets` channel the API publishes to, so Monarch reacts to issue-ticket migration events triggered from `team_maker_view.py` |
| monarch | GitHub API | `service/integrations/github.py` and `github_request.py` call `GITHUB_API_URL` (using `GITHUB_TOKEN`) to read and create issues in repos it manages |
| monarch | Slack | `service/notifications/slack.py` posts to Slack using `SLACK_BOT_TOKEN` to notify a channel about migration status |
| prometheus | api | `prometheus.yml`'s `django` scrape job polls the API's metrics endpoint (exposed by the `django-prometheus` middleware) to collect application metrics |
| prometheus | postgres_exporter | `prometheus.yml`'s `postgresql` scrape job polls `postgres_exporter` for database-level metrics |
| postgres_exporter | database | It connects to Postgres using `DATA_SOURCE_NAME` (built from the `POSTGRES_*` env vars) and translates Postgres's internal stats into a Prometheus-readable format |
| grafana | prometheus | Grafana's dashboards query Prometheus as their data source (declared via `depends_on: prometheus` in `docker-compose.yml`) — without it there's nothing for the dashboards to visualize |

### 1e. Main Entry Points

| Service | Startup File | Routes / URL Config File |
|---|---|---|
| api | `learn-ops-api/entrypoint.sh` / `learn-ops-api/manage.py` | `learn-ops-api/LearningPlatform/urls.py` |
| client | `learn-ops-client/src/index.js` | `learn-ops-client/src/components/LearnOps.js` |
| monarch | `service-monarch/service/main.py` | `service-monarch/service/custom_logging/web_interface.py` |

## 2. Services

| Service Name | Tech Stack (including version) | Purpose |
|---|---|---|
| database | Postgres 16 (`postgres:16` image) | Primary relational datastore for the API's application data |
| api | Python 3.11.11, Django 5.2.17, Django REST Framework 3.18.0 | Backend REST API serving the Learning Platform's business logic, auth, and data |
| client | Node 22.13.0, React 16.13.1, react-scripts (CRA) 5.0.1 | Frontend single-page app used to interact with the platform |
| prometheus | Prometheus (`prom/prometheus:latest`, unpinned) | Scrapes and stores metrics from `api` and `postgres_exporter` |
| grafana | Grafana (`grafana/grafana:latest`, unpinned) | Visualizes Prometheus metrics via dashboards |
| postgres_exporter | postgres-exporter (`quay.io/prometheuscommunity/postgres-exporter:latest`, unpinned) | Translates Postgres's internal stats into Prometheus-scrapeable metrics |
| valkey | Valkey (`valkey/valkey:latest`, unpinned) | In-memory cache and pub/sub message broker used by `api` and `monarch` |
| valkey-monitor | Valkey (`valkey/valkey:latest`, unpinned) | Sidecar that streams live commands hitting `valkey`, for debugging/observability |
| monarch | Python 3.11-slim (Docker image; Pipfile targets 3.10, but the actual container runs on 3.11), Flask 3.0.3, Pydantic 2.10.4 | Background service that manages GitHub issue/repo migrations, notifies Slack, and reacts to events the API publishes over Valkey pub/sub |

## 3. System Overview

This is a learning management system purpose-built for running a cohort-based coding bootcamp, rather than a general-purpose LMS like Canvas or Moodle. It solves the operational problem of coordinating a group of students moving together through a shared technical curriculum: tracking each student's mastery of specific skills and learning objectives, organizing them into cohorts with schedules, forming project teams, and handling the GitHub-specific busywork (creating repos, migrating issues) that a bootcamp's project-based curriculum generates. Identity is tied directly to GitHub via OAuth, reflecting that students and instructors are already living in GitHub for coursework.

From a user's perspective, the platform centers on tracking progress through a curriculum: courses are broken into learning objectives and learning weights, and students build up a record of what they've completed and assessed against. On top of that sits a set of assessment tools — book assessments, core-skill assessments, and personality assessments — that instructors use to evaluate students, along with notes and tags instructors can attach to a student's profile over time. The platform also automates team formation for group projects: when teams are created, it can automatically generate the corresponding GitHub repositories and notify the group over Slack, with a background service migrating any related GitHub issues into place. Beyond coursework, students can track capstone project proposals through defined stages and browse job opportunities posted through the platform.

There are two clearly distinct user roles, reflected directly in the client app's split between `StaffViews.js` and `StudentViews.js`. Students interact with the system largely as consumers of their own data: they log in with GitHub, view their own learning records and cohort schedule, complete assessments, get placed into teams, and browse opportunities. Staff/instructors have a much broader view and write access: they manage cohort and course content, record assessments and notes against any student, initiate team formation for the whole cohort, and review capstone proposals. A further superuser/admin layer exists at the infrastructure level (the seeded `LEARN_OPS_SUPERUSER_NAME` account and the `INSTRUCTOR_USERNAME`/`INSTRUCTOR_COHORT` elevation logic in `entrypoint.sh`) for platform administration rather than day-to-day teaching or learning.