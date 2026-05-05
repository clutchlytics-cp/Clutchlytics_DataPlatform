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
    bronze/          # Raw ingestion notebooks (00x_)
    silver/          # Transform and dimension notebooks (01x_, 02x_)
    gold/            # Analysis model notebooks (03x_)
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
| **Silver** | Normalized, typed, deduplicated. Enterprise dims + league-specific cleaned tables. |
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
| `dimXxx` | dimTeams, dimAthletes, dimGames | Enterprise dimension tables — all sports |
| `fctXxx` | fctGames | Enterprise fact tables — all sports |
| `nhl_xxx` | nhl_skater_game_logs | League-specific cleaned tables |
| `raw_xxx` | raw_nhl_teams | Bronze tables |
| `gold_xxx` | gold_player_delta | Gold analysis tables |
| `clutch_team_id` | Integer surrogate PK | Cross-sport unique team identifier |
| `clutch_athlete_id` | Integer surrogate PK | Cross-sport unique athlete identifier |
| `clutch_game_id` | Integer surrogate PK | Cross-sport unique game identifier |

---

## Silver Layer Design

Silver has two tiers:

**Enterprise-wide** — all sports in one table, joined via surrogate PKs.

**League-specific** — scoped to one sport/league, named with league prefix (e.g. `nhl_`). Carries sport-specific fields without polluting the enterprise layer.

---

## Current Table Inventory

### Bronze

| Table | Rows | Notes |
|-------|------|-------|
| `raw_nhl_teams` | 32 | Full refresh |
| `raw_nhl_rosters` | ~750 | Full refresh |
| `raw_nhl_games` | 45 | R1 — MERGE on event_id |
| `raw_nhl_scoreboard` | 45 | R1 — MERGE on event_id + comp_id |
| `raw_nhl_skater_logs` | 424 | MERGE on athlete_id |
| `raw_nhl_goalie_logs` | 42 | MERGE on athlete_id |
| `raw_nhl_game_summaries` | 45 | R1 — MERGE on event_id |

### Silver — Enterprise

| Table | Rows | Notes |
|-------|------|-------|
| `dimTeams` | 32 | NHL loaded. Multi-sport ready. |
| `dimAthletes` | ~750 | SCD Type 2. NHL loaded. |
| `dimGames` | 45 | R1 playoff schedule. Multi-sport ready. |
| `fctGames` | 47 | R1 complete + 2 R2 pending dimGames update |

### Silver — NHL Specific

| Table | Rows | Notes |
|-------|------|-------|
| `nhl_skater_game_logs` | 25,739 | R1 + 2026 regular season |
| `nhl_game_summaries` | 90 | Team stats tall format — 2 rows per game |
| `nhl_player_game_stats` | 1,713 | Additive player stats from boxscore |
| `nhl_goalie_game_logs` | 1,466 | R1 + 2026 regular season |
| `nhl_series` | 45 | Series state — one row per game per series |

### Gold

| Table | Status |
|-------|--------|
| `gold_series_momentum` | Planned |
| `gold_home_ice` | Planned |
| `gold_player_delta` | Planned |
| `gold_goalie_ratings` | Planned |
| `gold_role_expansion` | Planned |
| `gold_goalie_matchup` | Planned |

---

## Current Sports Coverage

| Sport | League | Status | Season |
|-------|--------|--------|--------|
| Hockey | NHL | Active — 2026 Playoffs | 2026 |
| Football | NFL | Planned — Fall 2026 | — |
| Basketball | NBA | Planned — Future | — |

---

## Wiki Documents

| Document | Covers |
|----------|--------|
| [01_Project_Overview.md](docs/wiki/01_Project_Overview.md) | Goals, architecture decisions, tech stack |
| [02_Infrastructure.md](docs/wiki/02_Infrastructure.md) | Databricks, Unity Catalog, Volumes, GitHub |
| [03_ESPN_API.md](docs/wiki/03_ESPN_API.md) | Endpoint map, pull strategy, file conventions |
| [04_Bronze_Layer.md](docs/wiki/04_Bronze_Layer.md) | All bronze tables, notebooks, conventions |
| [05_Silver_dimTeams.md](docs/wiki/05_Silver_dimTeams.md) | dimTeams schema, multi-sport design |
| [06_Silver_dimAthletes.md](docs/wiki/06_Silver_dimAthletes.md) | dimAthletes schema, SCD Type 2 |
| [07_Silver_dimGames.md](docs/wiki/07_Silver_dimGames.md) | dimGames schema, series key derivation |
| [08_Silver_fctGames.md](docs/wiki/08_Silver_fctGames.md) | fctGames schema, sources, derivations |
| [09_Silver_nhl_skater_game_logs.md](docs/wiki/09_Silver_nhl_skater_game_logs.md) | Skater game logs, label/stat parsing, TOI |
| [10_Silver_nhl_game_summaries.md](docs/wiki/10_Silver_nhl_game_summaries.md) | Team stats tall format, grain rationale |
| [11_Silver_nhl_player_game_stats.md](docs/wiki/11_Silver_nhl_player_game_stats.md) | Additive player stats, stat_group, TOI splits |
| [12_Silver_nhl_goalie_game_logs.md](docs/wiki/12_Silver_nhl_goalie_game_logs.md) | Goalie logs, save_pct parsing, two flags |
| [13_Silver_nhl_series.md](docs/wiki/13_Silver_nhl_series.md) | Series tracker, team_a/b ordering, momentum |

---

## Local Development Setup

1. Clone this repository and connect to Databricks via Git integration
2. Install Python dependencies: `pip install requests`
3. Run local pull scripts to land JSON files in sport-specific data folders
4. Upload files to the relevant Databricks Volume
5. Trigger Bronze → Silver → Gold notebooks in sequence

---

## .gitignore

```
*.json
nhl_data/
nfl_data/
__pycache__/
*.pyc
.env
.DS_Store
```
