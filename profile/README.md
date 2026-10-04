<div align="center">

![Digital Barometer](../assets/logo.svg){width=128 height=128}

# Digital Barometer

### Media monitoring with an LLM sentiment barometer

We collect mentions of a topic across the web, rate their sentiment and emotions,
and turn them into trends and reports.

![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![React](https://img.shields.io/badge/React_18-20232A?logo=react&logoColor=61DAFB)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-1C3C3C?logo=langchain&logoColor=white)
![Docker](https://img.shields.io/badge/Docker_Compose-2496ED?logo=docker&logoColor=white)
![Traefik](https://img.shields.io/badge/Traefik-24A1C1?logo=traefikproxy&logoColor=white)

</div>

---

## How it works

| 🔎 Collects | 🧠 Rates | 📈 Aggregates | 📊 Shows |
| :---: | :---: | :---: | :---: |
| GDELT, NewsAPI, RSS feeds and Google Trends are queried in parallel for a topic and period | An LLM rates the sentiment and emotion of every mention; a keyword heuristic takes over if it is unavailable | Mentions roll up into a 0–100 barometer index, emotion distribution and short insights | The dashboard shows the gauge, charts, trends and the mentions themselves |

<div align="center">
<a href="../assets/architecture.png"><img src="../assets/architecture.png" width="680" alt="Architecture"></a><br>
<sub>Backend layers: <code>api → services → repositories → db</code>; models and migrations in a separate <code>digital-barometer-db</code> package</sub>
</div>

## Repositories

<table>
<tr><th colspan="2">🖥 Product</th></tr>
<tr>
<td width="48" align="center">🧠</td>
<td><a href="https://github.com/digital-barometer/backend"><b>backend</b></a> — REST API, data collection, LLM sentiment and emotion analysis<br><sub>Python · FastAPI · dishka · SQLAlchemy · Alembic · PostgreSQL · LangChain</sub></td>
</tr>
<tr>
<td align="center">📊</td>
<td><a href="https://github.com/digital-barometer/frontend"><b>frontend</b></a> — dashboard: topics, sources, barometer and analysis charts<br><sub>React 18 · TypeScript · Vite · Tailwind CSS · Recharts</sub></td>
</tr>
<tr><th colspan="2">🏗️ Platform</th></tr>
<tr>
<td align="center">🛠️</td>
<td><a href="https://github.com/digital-barometer/infra"><b>infra</b></a> — Traefik reverse proxy with automatic TLS and the shared PostgreSQL<br><sub>Traefik · Let's Encrypt · PostgreSQL 16 · Docker Compose</sub></td>
</tr>
</table>

## Analysis flow

<div align="center">
<a href="../assets/flowchart.png"><img src="../assets/flowchart.png" width="300" alt="Analysis flow"></a>
</div>

Sources are queried in parallel, results are normalized to a common model (`Mention` / `TrendPoint` /
`SourceResult`) and deduplicated by a SHA-256 content hash. Each source is tracked separately, so a run ends
as `success`, `partial` or `failed` depending on which sources worked. API keys and `Authorization` headers
are stripped from error messages before they are stored.

## Getting started

| I want to… | Go to |
| --- | --- |
| **deploy it** | [infra](https://github.com/digital-barometer/infra#quick-start) → [backend](https://github.com/digital-barometer/backend#quick-start) → [frontend](https://github.com/digital-barometer/frontend#quick-start) |
| **integrate over REST** | [backend contracts](https://github.com/digital-barometer/backend#contracts) — Swagger at `/docs` |
| **add a data source** | [backend features](https://github.com/digital-barometer/backend#features) — connectors and `ConnectorFactory` |

## Data model

<div align="center">
<a href="../assets/db.png"><img src="../assets/db.png" width="420" alt="ERD"></a>
</div>

| Table | Purpose |
| --- | --- |
| `topics` | tracked topics and their keywords |
| `sources` | configured data sources (GDELT, NewsAPI, RSS, Trends) |
| `analysis_runs` | one analysis of a topic over a period |
| `source_results` | per-source result within a run: status, item counts, metrics |
| `mentions` | collected mentions with sentiment and emotion |
| `trend_points` | time series from sources, e.g. Google Trends values |
| `analysis_metrics` | aggregated sentiment and emotion figures and the barometer index |
| `reports` | generated report files for a run |
