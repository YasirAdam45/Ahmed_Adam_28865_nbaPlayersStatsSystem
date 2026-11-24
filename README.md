# NBA Players System – Player and Team Performance Database

**Student:** Ahmed Adam | **ID:** 28865
**Course:** INSY 8311 – Database Development with PL/SQL | **Instructor:** Eric Maniraguha
**Repository:** `ahmed_28865_nbaPlayersSystem`

## Problem Statement
Public NBA statistics are distributed as wide, flat files in which player, team and season details repeat on every row. Traded players appear on several rows in the same season, team names and abbreviations vary across eras, and nothing stops invalid stat lines (for example, field goals made greater than field goals attempted) from entering the data. Analysts, coaches and scouts therefore cannot reliably compare players and teams across seasons. NBA Players System organises 1980–2026 NBA data into normalised Oracle tables, enforces data rules in PL/SQL and records every change in an audit log.

## Objectives
**Main objective:** Build a Third Normal Form (3NF) Oracle database with PL/SQL that makes NBA player and team performance data reliable to store, query and compare.

**Specific objectives**
1. Normalise the raw player and team files into at least 5 related tables with enforced constraints.
2. Compute efficiency metrics (e.g. true shooting %) and league leaders with PL/SQL functions and cursors.
3. Block invalid stat lines with triggers and record all data changes in an audit log.
4. Provide an interactive interface for browsing, comparing and entering data (innovation).

## Data Source
Meruva, K. S. (2026). *NBA Player and Team Statistics (1980–2026)* [Data set]. Kaggle. https://www.kaggle.com/datasets/meruvakodandasuraj/nba-player-and-team-statistics-19802026

| File | Rows | Used for |
|---|---|---|
| `player_season_stats.csv` | 50,785 | Players, positions, seasons, player season statistics, awards, salary |
| `team_season_records.csv` | 1,410 | Teams, conferences, divisions, eras, team season records |
| `career_profiles.csv`, `era_summary.csv` | 39 / 7 | Not loaded; they are aggregates, rebuilt later as views |
| `mvp_winners.csv` | 0 | Empty; MVPs come from `mvp_award` in the player file |

Licence:CC0: Public Domain (Permits educational and academic research use without restriction). The data contains player names of public figures and no private personal information.

## Innovation
**Option:** User interface. **Tool:** Streamlit with python-oracledb. Users browse and compare players and enter stat lines; the app calls the PL/SQL procedures, and all business rules stay in the database.

## Planned Tables
`TEAM`, `ERA`, `PLAYER`, `POSITION`, `SEASON`, `PLAYER_SEASON_STAT`, `TEAM_SEASON_STAT`, `AUDIT_LOG`

## Planned PL/SQL Features
| Feature | Where it is used |
|---|---|
| Procedure `add_player_season_stat` | Inserts a validated stat line for a player and season |
| Procedure `trade_player` | Moves a player to another team in the middle of a season |
| Function `true_shooting_pct` | Calculates PTS / (2 × (FGA + 0.44 × FTA)) |
| Function `team_win_pct` | Returns a team's win percentage for a season |
| Cursor | Loops through seasons to list the league leaders in each |
| Package `nba_players_pkg` | Groups the analytics procedures and functions |
| Trigger | Rejects any stat line where made shots exceed attempts |
| Trigger | Writes every INSERT, UPDATE and DELETE to `AUDIT_LOG` |
| Restriction rule | Blocks data changes outside allowed days |
| Custom exceptions | Handle unknown players and duplicate player-season entries |

## Project Phases
- Phase 0 – Research and proposal: `docs/research/`, `docs/proposal/`, `docs/presentation/`
- Later phases are added as they are completed.

## References
- Kubatko, J., Oliver, D., Pelton, K., & Rosenbaum, D. T. (2007). A starting point for analyzing basketball statistics. *Journal of Quantitative Analysis in Sports, 3*(3). https://doi.org/10.2202/1559-0410.1070
- Meruva, K. S. (2026). *NBA Player and Team Statistics (1980–2026)* [Data set]. Kaggle. https://www.kaggle.com/datasets/meruvakodandasuraj/nba-player-and-team-statistics-19802026
- National Basketball Association. (2025). *NBA advanced stats official documentation and glossary*. NBA Media Central. https://www.nba.com/stats/help/glossary
- Oracle Corporation. (2024). *Oracle Database PL/SQL language reference (21c/23ai)*. Oracle Help Center. https://docs.oracle.com/en/database/oracle/oracle-database/
- Sarlis, V., & Tjortjis, C. (2020). Sports analytics — Evaluation of basketball players and team performance. *Information Systems, 93*, 101562. https://doi.org/10.1016/j.is.2020.101562
- Sports Reference LLC. (2026). *Basketball-Reference database architecture & historical logs*. https://www.basketball-reference.com

## Phase I - Business Process Modeling

**Process:** Recording and validating a player's season stat line. A clerk enters
a stat line through the web form, the Oracle database validates it, stores it and
logs the change, and analysts and auditors retrieve the data.

**Modeling tool:** draw.io (diagrams.net)

**Published diagram link:** <[paste your link here](https://drive.google.com/file/d/1byglr146xGvUidh-VqZeRB7JG4TV_62q/view?usp=sharing)>

**Supporting documents:**
- [Business Process Diagram](docs/business_process/business_process_diagram.pdf)
- [Explanation (PDF)](docs/business_process/business_process_explaination.pdf)
