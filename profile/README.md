# Digital Barometer

A media-monitoring service that tracks mentions of a topic across the web,
scores their sentiment and emotions via an LLM, and turns the results into
trends and reports.

## Architecture

![Architecture](../assets/architecture.png)

| Repository | Stack | Role |
| --- | --- | --- |
| [**backend**](https://github.com/digital-barometer/backend) | FastAPI, dishka (DI), PostgreSQL, LangChain | REST API, data collection, LLM analysis |
| [**frontend**](https://github.com/digital-barometer/frontend) | React 18, TypeScript, Vite, Tailwind CSS, Recharts | Web UI: topics, sources, analysis charts |
| [**infra**](https://github.com/digital-barometer/infra) | Traefik, PostgreSQL, Docker Compose | Reverse proxy (TLS via Let's Encrypt) and database |

Backend is layered `api → services → repositories → db`, with a separate
`digital-barometer-db` package (SQLAlchemy models + Alembic migrations)
shared across services.

## Tech Stack

**Core:** FastAPI, dishka (DI), Pydantic Settings

**Data sources:** GDELT Doc API, NewsAPI, RSS feeds, Google Trends (via SerpApi) —
pluggable through a `ConnectorFactory`, fetched concurrently with a
configurable outbound proxy

**AI:** LangChain, OpenAI-compatible LLM endpoint — batched sentiment and
emotion scoring, topic summaries

**Database:** PostgreSQL + SQLAlchemy (async) + Alembic

**Infrastructure:** Docker Compose, Traefik (automatic TLS), GitLab CI
(`test → build → deploy`, staging + production)

Sensitive data (API keys, `Authorization` headers) is redacted from error
logs before they're persisted.

## Database Schema

![ERD](../assets/db.png)

| Table | Purpose |
| --- | --- |
| `topics` | Monitored topics and their keywords |
| `sources` | Configured data sources (GDELT, NewsAPI, RSS, Trends) per topic |
| `analysis_runs` | A single analysis execution for a topic over a date range |
| `source_results` | Per-source fetch outcome within a run (status, raw payload, item counts) |
| `mentions` | Individual mentions collected from sources, with sentiment/emotion scores |
| `trend_points` | Time-series metrics per source (e.g. Google Trends values) |
| `analysis_metrics` | Aggregated sentiment/emotion counts and the resulting "barometer" score |
| `reports` | Generated report files per analysis run |

## How analysis works

1. A topic is created with a set of sources.
2. Backend builds a search plan and fetches sources concurrently.
3. The LLM scores sentiment and emotion for each mention in batches.
4. Results are aggregated into trends and a barometer score, served to the
   frontend as charts.

---

See each repository's README for local setup and CI/CD details.
