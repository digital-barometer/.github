<div align="center">

<img src="../assets/logo.svg" width="128" height="128" alt="Digital Barometer">

# Digital Barometer

### Media monitoring with an LLM sentiment barometer

Collects mentions of a topic from across the web, rates their sentiment and
emotions, and turns them into trends and reports.

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

| Collect | Rate | Aggregate | Show |
| :---: | :---: | :---: | :---: |
| Pulls GDELT, NewsAPI, RSS, and Google Trends in parallel for a topic and period | An LLM rates each mention's sentiment and emotion; keywords take over if it's down | Mentions add up to a 0–100 barometer, an emotion breakdown, and short insights | A dashboard with the gauge, charts, trends, and the mentions themselves |

<div align="center">
<a href="../assets/architecture.png"><img src="../assets/architecture.png" width="680" alt="Architecture"></a><br>
<sub>Backend layers: <code>api → services → repositories → db</code>; models and migrations live in the <code>digital-barometer-db</code> package</sub>
</div>

## Repositories

<table>
<tr><th>Product</th></tr>
<tr>
<td><a href="https://github.com/digital-barometer/backend"><b>backend</b></a> — REST API, data collection, LLM analysis<br><sub>Python · FastAPI · dishka · SQLAlchemy · Alembic · PostgreSQL · LangChain</sub></td>
</tr>
<tr>
<td><a href="https://github.com/digital-barometer/frontend"><b>frontend</b></a> — the dashboard<br><sub>React 18 · TypeScript · Vite · Tailwind CSS · Recharts</sub></td>
</tr>
<tr><th>Platform</th></tr>
<tr>
<td><a href="https://github.com/digital-barometer/infra"><b>infra</b></a> — Traefik with automatic TLS and a shared PostgreSQL<br><sub>Traefik · Let's Encrypt · PostgreSQL 16 · Docker Compose</sub></td>
</tr>
</table>

## Analysis flow

<div align="center">
<a href="../assets/flowchart.png"><img src="../assets/flowchart.png" width="300" alt="Analysis flow"></a>
</div>

Sources are queried in parallel, mapped to one model (`Mention` / `TrendPoint` /
`SourceResult`), and deduplicated by content hash. Each source is tracked on its
own, so a run ends as `success`, `partial`, or `failed`. API keys are scrubbed
from error messages before they're saved.

## Getting started

| I want to… | Go to |
| --- | --- |
| **deploy it** | [infra](https://github.com/digital-barometer/infra#quick-start) → [backend](https://github.com/digital-barometer/backend#quick-start) → [frontend](https://github.com/digital-barometer/frontend#quick-start) |
| **use the API** | [backend contracts](https://github.com/digital-barometer/backend#contracts), Swagger at `/docs` |
| **add a data source** | [backend features](https://github.com/digital-barometer/backend#features) |

## Data model

<div align="center">
<a href="../assets/db.png"><img src="../assets/db.png" width="420" alt="ERD"></a>
</div>

| Table | What's in it |
| --- | --- |
| `topics` | topics and their keywords |
| `sources` | data sources (GDELT, NewsAPI, RSS, Trends) |
| `analysis_runs` | one analysis of a topic over a period |
| `source_results` | how each source did in a run |
| `mentions` | mentions with sentiment and emotion |
| `trend_points` | time series, e.g. Google Trends |
| `analysis_metrics` | totals and the barometer index |
| `reports` | generated report files |
