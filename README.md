# NBA Players Stats System: Player and Team Performance Database

**Student:** Ahmed Adam | **ID:** 28865 | **Group:** B  
**Course:** INSY 8311 – Database Development with PL/SQL | **Instructor:** Eric Maniraguha
**Repository:** `Ahmed_Adam_28865_nbaPlayersStatsSystem`

## Problem Statement

Public NBA statistics are published as wide flat files in which player, team and season details repeat on every row and nothing enforces integrity. The dataset chosen for this project (50,785 player rows and 1,410 team rows, 1980–2026) shows the consequences: one player identifier (`P00001`) is shared by 46 different players, which creates 143 duplicate keys; the stored win percentage disagrees with wins ÷ (wins + losses) in 1,227 of 1,410 team seasons; and no rule prevents impossible stat lines, such as more than 82 games or a shooting percentage above 1. Analysts, coaches and scouts therefore struggle to compare players and teams reliably across seasons. Existing portals such as Basketball-Reference are read-only and offer no custom integrity rules, controlled data entry or audit trail.

## Objectives

**Main objective:** Build a Third Normal Form (3NF) Oracle database with PL/SQL that makes multi-decade NBA player and team performance data reliable to store, query and compare.

**Specific objectives**

1. Design and build eight related 3NF tables with enforced constraints.
2. Calculate team winning percentage and player scoring efficiency with stored functions, and rank seasonal league leaders with a parameterized cursor.
3. Block invalid stat lines with triggers and record every data change in an audit log.
4. Provide a live Tableau dashboard connected to the Oracle database (innovation).

## Data Source

Meruva, K. S. (2026). *NBA Player and Team Statistics (1980–2026)* [Data set]. Kaggle. https://www.kaggle.com/datasets/meruvakodandasuraj/nba-player-and-team-statistics-19802026

| File | Rows | Used for |
|---|---|---|
| `player_season_stats.csv` | 50,785 | Players, positions, player season statistics, awards, salary |
| `team_season_records.csv` | 1,410 | Teams, conferences, divisions, eras, team season records |
| `career_profiles.csv`, `era_summary.csv` | 39 / 7 | Not loaded. They are aggregates, rebuilt later as views |
| `mvp_winners.csv` | 0 | Not loaded. It is empty, and `mvp_award` is 0 in every row of the player file |

Licence: CC0: Public Domain (permits educational and academic use). The data holds public sports statistics and no private personal information. Player identifiers are replaced by anonymized `PLR_` codes when the data is loaded. See [`data/dataset_card.md`](data/dataset_card.md).

## Innovation

**Option:** Analytics dashboard. **Tool:** Tableau Desktop, connected live to the Oracle pluggable database.

The dashboard reads Oracle views (created in Phase IV) through a read-only database user. It is documented in the [`innovation/`](innovation/) folder.

| Dashboard sheet | Oracle view | What it shows | Filters |
|---|---|---|---|
| Team performance | `vw_team_season_summary` | Wins, net rating, playoff and championship seasons per team | Season, era, conference, division |
| Payroll vs wins | `vw_team_season_summary` | Whether higher payroll produces more wins | Season, era, team |
| Season league leaders | `vw_season_leaders` | Top 10 scorers of the selected season | Season, position |
| Player comparison | `vw_player_season_detail` | Points per 36 minutes, PER and win shares across seasons for chosen players | Player, team, season |

**Business rules stay in PL/SQL.** Tableau only reads data. New stat lines enter the database through `add_player_season_stat`, where the triggers validate every value and write the audit log. The live demo adds a stat line with the procedure, refreshes the Tableau data source and shows the new row on the dashboard.

**Documentation (`innovation/` folder):** what the dashboard does, how to connect and run it, the Tableau workbook, and screenshots.

## Planned Tables

Seven tables are built from the data, and the eighth (the audit log) is created in Phase VI.

| Table | Purpose |
|---|---|
| `DIVISIONS` | Division and conference |
| `TEAMS` | The 30 franchises, each in one division |
| `SEASONS` | Seasons 1980–2026 with their era |
| `POSITIONS` | The five court positions |
| `PLAYERS` | One row per real player, with an anonymized code and primary position |
| `PLAYER_SEASON_STATS` | One stat line per player, season and team (junction table) |
| `TEAM_SEASON_STATS` | One record per team and season (junction table) |
| `STAT_AUDIT_LOG` | Every INSERT, UPDATE and DELETE (Phase VI) |

## Planned PL/SQL Features

| Feature | Where it is used |
|---|---|
| Procedure `add_player_season_stat` | Checks the player exists, rejects a duplicate, validates ranges and inserts a stat line |
| Procedure `trade_player` | Moves a player to another team |
| Function `team_win_pct` | Calculates wins ÷ (wins + losses), because the stored `win_pct` is inconsistent |
| Function `points_per_36` | Scoring efficiency from points and minutes per game |
| Cursor `cur_season_leaders` | Parameterized by season; ranks the qualified players of that season |
| Package `nba_players_pkg` | Groups all procedures, functions and the cursor |
| Trigger `trg_validate_stat_line` | Blocks games outside 1–82, minutes above 48 and percentages outside 0–1 |
| Trigger `trg_audit_stats` | Writes every INSERT, UPDATE and DELETE to `STAT_AUDIT_LOG` |
| Restriction trigger | Blocks data changes on non-working days |
| Custom exceptions | Unknown player and duplicate stat line |

## Project Phases

- **Phase 0:** Research and proposal: `docs/research/`, `docs/proposal/`, `docs/presentation/`
- **Phase I:** Business process modeling: `docs/business_process/`
- **Phase II:** Data sourcing and normalization: `data/`, `docs/design/`
- **Innovation:** Tableau dashboard: `innovation/` (added in Phase VII)
- Later phases are added as they are completed.

## Phase I - Business Process Modeling

**Process:** Player Season Stat Recording. A clerk enters a stat line, which is submitted to `add_player_season_stat`. The Oracle database checks that the player exists, rejects duplicates, validates the statistics, applies the non-working-day rule, inserts the record and writes an audit log entry. The clerk corrects any rejected entry. Analysts, coaches and scouts retrieve the data, and the auditor reviews the audit log.

**Modeling tool:** draw.io (diagrams.net)

**Published diagram link:** [the diagram link](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=business%20process%20diagram.drawio&dark=auto#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1byglr146xGvUidh-VqZeRB7JG4TV_62q%26export%3Ddownload)

**Supporting documents:**

- [Business process diagram (PDF)](docs/business_process/business_process_diagram.pdf)
- [Business process explanation (PDF)](docs/business_process/business_process_explaination.pdf)

## Phase II - Data Sourcing, Normalization and Logical Design

| Deliverable | File |
|---|---|
| Raw data, unedited | [`data/raw/`](data/raw/) |
| Dataset card and data quality report | [`data/dataset_card.md`](data/dataset_card.md) |
| Quality check script (reproduces every count) | [`data/quality_check.py`](data/quality_check.py) |
| Normalization UNF → 1NF → 2NF → 3NF, field mapping, class demo | [`docs/design/normalization.md`](docs/design/normalization.md) |
| Logical ERD (crow's-foot) | [`docs/design/erd.png`](docs/design/erd.png) |
| Data dictionary | [`docs/design/data_dictionary.md`](docs/design/data_dictionary.md) |

**Key findings from the data quality checks**

- One raw `player_id` (`P00001`) is shared by 46 players, so a player is identified by the pair (`player_id`, `player_name`): **8,045 players**, not 8,000.
- No player has two teams in one season, and there are no summary rows. The schema still allows trades because `team_id` is part of the unique key.
- `win_pct` is inconsistent in 1,227 of 1,410 rows and `net_rating` is derivable, so neither is stored.
- No MVP is recorded anywhere in the data (`mvp_award` is 0 in all rows).

Run the checks from the repository root: `python data/quality_check.py`

## References

- Kubatko, J., Oliver, D., Pelton, K., & Rosenbaum, D. T. (2007). A starting point for analyzing basketball statistics. *Journal of Quantitative Analysis in Sports, 3*(3). https://doi.org/10.2202/1559-0410.1070
- Meruva, K. S. (2026). *NBA player and team statistics (1980–2026)* [Data set]. Kaggle. https://www.kaggle.com/datasets/meruvakodandasuraj/nba-player-and-team-statistics-19802026
- National Basketball Association. (2025). *NBA advanced stats official documentation and glossary*. NBA Media Central. https://www.nba.com/stats/help/glossary
- Oracle Corporation. (2024). *Oracle Database PL/SQL language reference (21c/23ai)*. Oracle Help Center. https://docs.oracle.com/en/database/oracle/oracle-database/
- Sarlis, V., & Tjortjis, C. (2020). Sports analytics: Evaluation of basketball players and team performance. *Information Systems, 93*, Article 101562. https://doi.org/10.1016/j.is.2020.101562
- Sports Reference LLC. (2026). *Basketball-Reference database architecture and historical logs*. https://www.basketball-reference.com
