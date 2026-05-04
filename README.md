# Clutchlytics Data Platform

> Sports analytics pipeline built on Databricks, Delta Lake, and the ESPN Public API. Powers the Clutchlytics analytics and media brand.

---

## Overview

Clutchlytics is a sports analytics and media platform built on a cloud-native data architecture. The platform ingests data from public sports APIs, processes it through a medallion pipeline on Databricks, and produces analytics outputs that power editorial content and interactive tools.

This repository contains all pipeline notebooks, schema definitions, and documentation. Raw data files are not tracked — they are managed separately via Databricks Volumes.

---

## Repository Structure

```
Clutchlytics_DataPlatform/
  notebooks/
    bronze/          # Raw ingestion notebooks
    silver/          # Transform and dimension notebooks
    gold/            # Analysis model notebooks
  docs/
    wiki/            # Individual wiki documents
  .gitignore
  README.md
```

---

## Architecture

Three-layer medallion architecture hosted on Databricks Free Edition with Unity Catalog.

| Layer | Purpose |
|-------|---------|
| **Bronze** | Raw JSON ingested from ESPN API. No transformation. One table per entity. |
| **Silver** | Normalized, typed, deduplicated. Dimension tables designed for multi-sport use. |
| **Gold** | Analysis models, aggregated outputs, CSV exports for content use. |

---

## Catalog Structure

| Catalog | Schema | Purpose |
|---------|--------|---------|
| clutchlytics | bronze | Raw ingested data from ESPN API |
| clutchlytics | silver | Cleaned dimensions and fact tables |
| clutchlytics | gold | Analysis models and content outputs |

---

## Naming Conventions

| Pattern | Example | Used For |
|---------|---------|---------|
| `dimXxx` | dimTeams, dimAthletes | Dimension tables in silver |
| `fctXxx` | fctGameLogs, fctStats | Fact tables in silver |
| `raw_xxx` | raw_nhl_teams | Bronze tables |
| `gold_xxx` | gold_player_delta | Gold analysis tables |
| `clutch_team_id` | Integer surrogate PK | Cross-sport unique team identifier |
| `clutch_athlete_id` | Integer surrogate PK | Cross-sport unique athlete identifier |

---

## Current Sports Coverage

| Sport | League | Status | Season |
|-------|--------|--------|--------|
| Hockey | NHL | Active — 2026 Playoffs | 2026 |
| Football | NFL | Planned — Fall 2026 | — |
| Basketball | NBA | Planned — Future | — |

---

## Wiki Documents

Detailed documentation for each component lives in `docs/wiki/`.

| Document | Covers |
|----------|--------|
| [01_Project_Overview.md](docs/wiki/01_Project_Overview.md) | Goals, architecture decisions, tech stack |
| [02_Infrastructure.md](docs/wiki/02_Infrastructure.md) | Databricks setup, Unity Catalog, Volumes, GitHub integration |
| [03_ESPN_API.md](docs/wiki/03_ESPN_API.md) | Endpoint map, pull strategy, local file structure |
| [04_Bronze_Layer.md](docs/wiki/04_Bronze_Layer.md) | Bronze tables, ingestion notebooks, file conventions |
| [05_Silver_dimTeams.md](docs/wiki/05_Silver_dimTeams.md) | dimTeams schema, transform logic, multi-sport design |
| [06_Silver_dimAthletes.md](docs/wiki/06_Silver_dimAthletes.md) | dimAthletes schema, SCD Type 2 logic, extensibility |

---

## Local Development Setup

1. Clone this repository and connect to Databricks via Git integration
2. Install Python dependencies: `pip install requests`
3. Run local pull scripts to land JSON files in `nhl_data/`
4. Upload files to the `clutchlytics.bronze.nhl_raw` Volume in Databricks
5. Trigger Bronze notebook → run Silver and Gold notebooks in sequence

---

## Contributing

Private repository. All notebooks maintained as `.py` files synced via Databricks Git integration. Do not commit raw data files or credentials.
