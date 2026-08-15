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
emotion scoring, topic summaries; falls back to a regex-based heuristic when
the LLM is unavailable

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

## Analysis Flow

![Analysis flow](../assets/flowchart.png)

Sources are fetched concurrently and normalized into a common model
(`Mention` / `TrendPoint` / `SourceResult`), deduplicated by a SHA-256
content hash. If the LLM is unavailable, sentiment/emotion scoring falls
back to a regex-based heuristic instead of failing the run. Each source
result is tracked independently, so a run can finish as `success`,
`partial`, or `failed` depending on which sources succeeded.

---

See each repository's README for local setup and CI/CD details.
