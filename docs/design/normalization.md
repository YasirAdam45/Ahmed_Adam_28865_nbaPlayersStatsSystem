# Normalization: UNF → 1NF → 2NF → 3NF

> **Student:** Ahmed Adam | **Student ID:** 28865 | **Group:** Group B | **Project:** NBA Players Stats System (INSY 8311, Phase II)

Two flat CSV files are normalized into 7 related tables. Every table below was built from the real data, and a **lossless-join test** (section 6) rebuilds all 50,785 player rows and all 1,410 team rows from the 3NF tables with no difference.

## 1. Starting point (UNF)

| Flat relation | Source | Rows | Columns | 
| ----- | ----- | ----- | ----- | 
| PLAYER_FLAT | `player_season_stats.csv` | 50,785 | 28 | 
| TEAM_FLAT | `team_season_records.csv` | 1,410 | 22 | 

Both files already hold single values in every cell (no lists, no nested records), so the main 1NF work is **choosing a real primary key**. All repeating information (player name, team name, division, era) is what the later steps remove.

## 2. PLAYER_FLAT

### 2.1 UNF: the file as downloaded

Five real rows (CSV line numbers 2, 42, 26, 33 and 629; the file header is line 1):

| player_id | player_name | season | team | position | is_legend | points_per_game | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | 
| P00001 | Magic Johnson | 1980 | LAL | PG | 1 | 18.4 | 
| P00001 | Kareem Abdul-Jabbar | 1980 | LAL | C | 1 | 15.4 | 
| P00001 | Moses Malone | 1980 | PHI | C | 1 | 17.0 | 
| P00001 | Julius Erving | 1980 | PHI | SF | 1 | 17.0 | 
| P00002 | Anthony Thompson | 2002 | GSW | SF | 0 | 18.5 | 

**Problem:** the obvious key (`player_id`, `season`, `team`) is **not unique**. Rows 1 and 2 are two different players (Magic Johnson, Kareem Abdul-Jabbar) with the same key (P00001, 1980, LAL). Rows 3 and 4 collide the same way (P00001, 1980, PHI). In the full file there are **143 excess duplicate keys** and one ID (P00001) shared by **46 players** (627 rows).

### 2.2 1NF: atomic values and a valid primary key

Dependency removed: **an invalid (non-unique) key**. A player is identified by the pair (`player_id`, `player_name`); each of the 8,045 pairs receives its own `player_code` (rule in `data/dataset_card.md` section 5). The key becomes (`player_code`, `season`, `team`), with **0 duplicates**. `team` stays in the key so that a traded player can have two rows in one season.

| raw_row | player_code | player_name | season | team | position | is_legend | points_per_game | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | 
| 2 | PLR_00001 | Magic Johnson | 1980 | LAL | PG | 1 | 18.4 | 
| 42 | PLR_08004 | Kareem Abdul-Jabbar | 1980 | LAL | C | 1 | 15.4 | 
| 26 | PLR_08002 | Moses Malone | 1980 | PHI | C | 1 | 17.0 | 
| 33 | PLR_08003 | Julius Erving | 1980 | PHI | SF | 1 | 17.0 | 
| 629 | PLR_00002 | Anthony Thompson | 2002 | GSW | SF | 0 | 18.5 | 

### 2.3 2NF: remove partial dependencies

In PLAYER_FLAT the key is (`player_code`, `season`, `team`). Four kinds of column depend on only **part** of it:

| Columns | Depends only on | Moves to | 
| ----- | ----- | ----- | 
| `player_name`, `position`, `is_legend` | `player_code` | **PLAYERS** | 
| `season` (as a value, one per year) | `season` | **SEASONS** | 
| `team` (as a value, one per franchise) | `team` | **TEAMS** | 
| `position` (5 repeating values) | `position` | **POSITIONS** (lookup) | 

All players have exactly one position (0 players with more than one), so position stays in PLAYERS, as promised in the Phase 0 proposal.

PLAYERS (sample):

| player_id | player_code | full_name | position_id | position_code | is_legend | 
| ----- | ----- | ----- | ----- | ----- | ----- | 
| 1 | PLR_00001 | Magic Johnson | 1 | PG | 1 | 
| 8004 | PLR_08004 | Kareem Abdul-Jabbar | 5 | C | 1 | 
| 8002 | PLR_08002 | Moses Malone | 5 | C | 1 | 
| 8003 | PLR_08003 | Julius Erving | 3 | SF | 1 | 
| 2 | PLR_00002 | Anthony Thompson | 3 | SF | 0 | 

PLAYER_SEASON_STATS (sample, same five rows; every statistic depends on the whole key):

| stat_id | player_id | season_id | team_id | games_played | points_per_game | salary_million_usd | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | 
| 1 | 1 | 1 | 14 | 67 | 18.4 | 0.5 | 
| 41 | 8004 | 1 | 14 | 69 | 15.4 | 4.56 | 
| 25 | 8002 | 1 | 23 | 66 | 17.0 | 3.67 | 
| 32 | 8003 | 1 | 23 | 72 | 17.0 | 3.17 | 
| 628 | 2 | 23 | 10 | 54 | 18.5 | 11.73 | 

A surrogate primary key `stat_id` is used, and (`player_id`, `season_id`, `team_id`) is a UNIQUE constraint. `age` and `salary_million_usd` describe the player in that season. They are kept on the stat line because a player's age and salary change every season.

### 2.4 3NF: remove transitive dependencies

* `team_name`, `division`, `conference` are not stored in PLAYER_FLAT (only the team code is), so they are handled in TEAM_FLAT (section 3.4).

* In PLAYERS, `position_id` → `position_code` → `position_name`: the position description depends on the position, not on the player. It lives in POSITIONS.

* Every remaining column of PLAYER_SEASON_STATS depends directly on `stat_id`. The award flags (`mvp_flag`, `all_star_flag`, `all_nba_flag`, `dpoy_flag`) describe that stat line, so they stay there.

## 3. TEAM_FLAT

### 3.1 UNF: the file as downloaded

| season | team_id | team_name | conference | division | era | wins | losses | win_pct | net_rating | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | 
| 1980 | ATL | Atlanta Hawks | East | Southeast | Showtime/Bird Era | 42 | 40 | 0.521 | 2.1 | 
| 1980 | BOS | Boston Celtics | East | Atlantic | Showtime/Bird Era | 47 | 35 | 0.585 | 3.4 | 
| 1980 | BKN | Brooklyn Nets | East | Atlantic | Showtime/Bird Era | 53 | 29 | 0.658 | 5.0 | 
| 1981 | MIN | Minnesota Timberwolves | West | Northwest | Showtime/Bird Era | 32 | 50 | 0.393 | \-3.1 | 
| 1981 | NOP | New Orleans Pelicans | West | Southwest | Showtime/Bird Era | 41 | 41 | 0.511 | 0.5 | 

`team_name`, `conference`, `division` and `era` are repeated on every row. `win_pct` can be calculated from `wins` and `losses`, and `net_rating` from the offensive and defensive ratings.

### 3.2 1NF: atomic values and a valid primary key

Dependency removed: **none needed, the key is already valid**. All cells are single values and (`season`, `team_id`) has **0 duplicates**, so it is the primary key (1,410 rows = 30 teams × 47 seasons).

### 3.3 2NF: remove partial dependencies

| Columns | Depends only on | Moves to | 
| ----- | ----- | ----- | 
| `team_name`, `conference`, `division` | `team_id` (part of the key) | **TEAMS** | 
| `era` | `season` (part of the key) | **SEASONS** | 

Everything else (wins, losses, ratings, payroll, attendance) depends on both `season` and `team_id` and stays in **TEAM_SEASON_STATS**.

### 3.4 3NF: remove transitive dependencies

| Transitive chain | Removed by | 
| ----- | ----- | 
| `team_id` → `division` → `conference` (0 divisions appear in two conferences) | `conference_name` moves to **DIVISIONS**; TEAMS keeps only `division_id` | 
| `wins`, `losses` → `win_pct` | `win_pct` dropped (calculated by function `team_win_pct`; the stored value disagrees with wins ÷ (wins + losses) in 1,227 rows) | 
| `offensive_rating`, `defensive_rating` → `net_rating` | `net_rating` dropped (equals offensive − defensive in all 1,410 rows; shown in a view) | 

DIVISIONS:

| division_id | division_name | conference_name | 
| ----- | ----- | ----- | 
| 1 | Atlantic | East | 
| 2 | Central | East | 
| 3 | Southeast | East | 
| 4 | Northwest | West | 
| 5 | Pacific | West | 
| 6 | Southwest | West | 

TEAMS (sample):

| team_id | team_code | team_name | division_id | 
| ----- | ----- | ----- | ----- | 
| 1 | ATL | Atlanta Hawks | 3 | 
| 2 | BKN | Brooklyn Nets | 1 | 
| 3 | BOS | Boston Celtics | 1 | 
| 4 | CHA | Charlotte Hornets | 3 | 
| 5 | CHI | Chicago Bulls | 2 | 

SEASONS (sample) and POSITIONS:

| season_id | season_year | era_name | 
| ----- | ----- | ----- | 
| 1 | 1980 | Showtime/Bird Era | 
| 2 | 1981 | Showtime/Bird Era | 
| 3 | 1982 | Showtime/Bird Era | 
| 4 | 1983 | Showtime/Bird Era | 

| position_id | position_code | position_name | 
| ----- | ----- | ----- | 
| 1 | PG | Point Guard | 
| 2 | SG | Shooting Guard | 
| 3 | SF | Small Forward | 
| 4 | PF | Power Forward | 
| 5 | C | Center | 

TEAM_SEASON_STATS (sample):

| team_stat_id | team_id | season_id | wins | losses | made_playoffs | playoff_wins | won_championship | payroll_million_usd | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | 
| 1 | 1 | 1 | 42 | 40 | 1 | 4 | 0 | 18.0 | 
| 2 | 3 | 1 | 47 | 35 | 1 | 6 | 0 | 5.0 | 
| 3 | 2 | 1 | 53 | 29 | 1 | 6 | 0 | 15.2 | 

*Design note:* `era_name` stays in SEASONS rather than in its own table because no other attribute depends on the era. Era is a label of the season, so SEASONS is already in 3NF.

## 4. Final tables and relationships

| \# | Table | Rows to load | Primary key | Foreign keys | Unique business key | 
| ----- | ----- | ----- | ----- | ----- | ----- | 
| 1 | DIVISIONS | 6 | division_id | none | division_name | 
| 2 | POSITIONS | 5 | position_id | none | position_code | 
| 3 | SEASONS | 47 | season_id | none | season_year | 
| 4 | TEAMS | 30 | team_id | division_id → DIVISIONS | team_code, team_name | 
| 5 | PLAYERS | 8,045 | player_id | position_id → POSITIONS | player_code | 
| 6 | PLAYER_SEASON_STATS | 50,785 | stat_id | player_id, season_id, team_id | (player_id, season_id, team_id) | 
| 7 | TEAM_SEASON_STATS | 1,410 | team_stat_id | team_id, season_id | (team_id, season_id) | 
| 8 | STAT_AUDIT_LOG | built in Phase VI | audit_id | none | none | 

**Many-to-many relationships and their junction tables**

| Many-to-many | Resolved by | Why | 
| ----- | ----- | ----- | 
| PLAYERS ↔ SEASONS ↔ TEAMS | **PLAYER_SEASON_STATS** | A player plays in many seasons, a team has many players, and a traded player can appear for two teams in one season. The junction table also holds the statistics. | 
| TEAMS ↔ SEASONS | **TEAM_SEASON_STATS** | A team plays many seasons and a season has 30 teams. | 

**Load order for Phase IV (parents first):** DIVISIONS, POSITIONS, SEASONS → TEAMS, PLAYERS → TEAM_SEASON_STATS, PLAYER_SEASON_STATS.

## 5. Field mapping and Phase IV cleaning checklist

### 5.1 player_season_stats.csv (28 fields)

| Raw field | Target table.column | Conversion / cleaning in Phase IV | 
| ----- | ----- | ----- | 
| player_id | PLAYERS.player_code | Not used as a key. Rebuild as `PLR_` code using the (player_id, player_name) pair rule | 
| player_name | PLAYERS.full_name | TRIM; keep as display name; never use as a key (600 names repeat) | 
| is_legend | PLAYERS.is_legend | 0/1 check; one value per player | 
| position | PLAYERS.position_id (via POSITIONS.position_code) | UPPER/TRIM; look up the id | 
| season | PLAYER_SEASON_STATS.season_id (via SEASONS.season_year) | Text → NUMBER; look up the id | 
| team | PLAYER_SEASON_STATS.team_id (via TEAMS.team_code) | UPPER/TRIM; look up the id | 
| age | PLAYER_SEASON_STATS.age | Text → NUMBER (LeBron James 2025–2026 stays 42) | 
| games_played | PLAYER_SEASON_STATS.games_played | Text → NUMBER; rule 1–82 | 
| minutes_per_game | PLAYER_SEASON_STATS.minutes_per_game | Text → NUMBER; rule 0–48 | 
| points_per_game, assists_per_game, rebounds_per_game, steals_per_game, blocks_per_game, turnovers_per_game | same names | Text → NUMBER; rule ≥ 0 | 
| fg_pct, fg3_pct, ft_pct | same names | Text → NUMBER; rule between 0 and 1 | 
| plus_minus, player_efficiency_rating, win_shares, box_plus_minus, vorp | same names | Text → NUMBER; negatives allowed except win_shares | 
| salary_million_usd | PLAYER_SEASON_STATS.salary_million_usd | Text → NUMBER; rule > 0 | 
| mvp_award | PLAYER_SEASON_STATS.mvp_flag | Renamed; 0/1 check (all 0 in the file) | 
| all_star | PLAYER_SEASON_STATS.all_star_flag | Renamed; 0/1 check | 
| all_nba | PLAYER_SEASON_STATS.all_nba_flag | Renamed; 0/1 check | 
| dpoy_award | PLAYER_SEASON_STATS.dpoy_flag | Renamed; 0/1 check | 

### 5.2 team_season_records.csv (22 fields)

| Raw field | Target table.column | Conversion / cleaning in Phase IV | 
| ----- | ----- | ----- | 
| season | TEAM_SEASON_STATS.season_id (via SEASONS.season_year) | Text → NUMBER; same SEASONS rows as the player file | 
| team_id | TEAM_SEASON_STATS.team_id (via TEAMS.team_code) | UPPER/TRIM; identical 30 codes as the player file | 
| team_name | TEAMS.team_name | TRIM | 
| conference | DIVISIONS.conference_name | TRIM; East / West | 
| division | DIVISIONS.division_name | TRIM; one conference per division | 
| era | SEASONS.era_name | TRIM; one era per season | 
| wins, losses, playoff_wins | TEAM_SEASON_STATS (same names) | Text → NUMBER; wins + losses ≤ 82 | 
| made_playoffs, won_championship | TEAM_SEASON_STATS (same names) | 0/1 check | 
| points_for, points_against, pace, offensive_rating, defensive_rating | TEAM_SEASON_STATS (same names) | Text → NUMBER | 
| three_point_attempt_pct, true_shooting_pct | TEAM_SEASON_STATS (same names) | Text → NUMBER; between 0 and 1 | 
| payroll_million_usd, home_attendance_avg | TEAM_SEASON_STATS (same names) | Text → NUMBER; payroll > 0 | 
| **win_pct** | **Dropped** | Derived; replaced by function `team_win_pct` (1,227 stored values are inconsistent) | 
| **net_rating** | **Dropped** | Derived (offensive − defensive); exposed in a Phase IV view | 

### 5.3 Files outside the 3NF tables

| File | Decision | 
| ----- | ----- | 
| career_profiles.csv | Not loaded. Rebuilt as a view over PLAYERS and PLAYER_SEASON_STATS | 
| era_summary.csv | Not loaded. Rebuilt as a GROUP BY view over SEASONS and TEAM_SEASON_STATS | 
| mvp_winners.csv | Not loaded (0 rows). The view will stay empty until MVP data exists | 

## 6. Proof that no information is lost

The 7 tables were built with Python and joined back together (`PLAYER_SEASON_STATS ⋈ PLAYERS ⋈ POSITIONS ⋈ SEASONS ⋈ TEAMS`, and `TEAM_SEASON_STATS ⋈ TEAMS ⋈ DIVISIONS ⋈ SEASONS`).

| Check | Result | 
| ----- | ----- | 
| Player rows rebuilt vs raw file | 50,785 = 50,785, every value identical | 
| Team rows rebuilt vs raw file | 1,410 = 1,410, every value identical (`net_rating` = offensive − defensive) | 
| Duplicates on (player_id, season_id, team_id) | 0 | 
| Duplicates on (team_id, season_id) | 0 | 

## 7. Demo for class: where one raw record ends up (5 minutes)

### 7.1 Raw record: line 2 of `player_season_stats.csv`

| Raw field | Raw value | Ends up in | Stored as | 
| ----- | ----- | ----- | ----- | 
| player_id + player_name | P00001 + Magic Johnson | PLAYERS.player_code, full_name | PLR_00001, Magic Johnson (PLAYERS.player_id = 1) | 
| is_legend | 1 | PLAYERS.is_legend | 1 | 
| position | PG | POSITIONS → PLAYERS.position_id | 1 (Point Guard) | 
| season | 1980 | SEASONS → PLAYER_SEASON_STATS.season_id | 1 (era: Showtime/Bird Era) | 
| team | LAL | TEAMS → PLAYER_SEASON_STATS.team_id | 14 (Los Angeles Lakers) | 
| age, games_played, minutes_per_game | 22, 67, 28.3 | PLAYER_SEASON_STATS | same values (stat_id = 1) | 
| points, assists, rebounds, steals, blocks, turnovers per game | 18.4, 6.4, 3.2, 1.1, 0.2, 1.6 | PLAYER_SEASON_STATS | same values | 
| fg_pct, fg3_pct, ft_pct | 0.463, 0.303, 0.807 | PLAYER_SEASON_STATS | same values | 
| plus_minus, PER, win_shares, BPM, VORP | 5.4, 16.4, 6.4, 1.13, 2.32 | PLAYER_SEASON_STATS | same values | 
| salary_million_usd | 0.5 | PLAYER_SEASON_STATS.salary_million_usd | 0.5 | 
| mvp_award, all_star, all_nba, dpoy_award | 0, 0, 0, 0 | PLAYER_SEASON_STATS flags | same values | 

### 7.2 Raw record: line 2 of `team_season_records.csv`

| Raw field | Raw value | Ends up in | Stored as | 
| ----- | ----- | ----- | ----- | 
| team_id | ATL | TEAMS.team_code | team_id = 1 | 
| team_name | Atlanta Hawks | TEAMS.team_name | Atlanta Hawks | 
| division, conference | Southeast, East | DIVISIONS | division_id = 3 (Southeast, East) | 
| season, era | 1980, Showtime/Bird Era | SEASONS | season_id = 1 | 
| wins, losses, playoffs, championship | 42, 40, 1/4/0 | TEAM_SEASON_STATS | team_stat_id = 1 | 
| win_pct, net_rating | 0.521, 2.1 | not stored | calculated: 42/(42+40) = 0.512; 107.7 − 105.6 = 2.1 | 

## 8. Consistency with the other phases

### 8.1 Phase 0 proposal: what stays and what must be updated

| Phase 0 statement | Phase II finding | Action | 
| ----- | ----- | ----- | 
| Eight tables: DIVISIONS, TEAMS, SEASONS, POSITIONS, PLAYERS, PLAYER_SEASON_STATS, TEAM_SEASON_STATS, STAT_AUDIT_LOG | Same eight names. Seven come from the data, STAT_AUDIT_LOG is created in Phase VI | **Matches** | 
| Each player has one primary position | 0 players have more than one position | **Matches** (position in PLAYERS) | 
| Player identifiers anonymised (e.g. PLR_00001) | Raw `player_id` is shared by 46 players | **Matches**, with the code rule in `dataset_card.md` section 5 | 
| Traded players appear on several rows in one season, with summary rows | **0** traded players and **0** summary rows | **Update the proposal wording.** The schema still supports trades through `team_id` in the unique key, so `trade_player` still works | 
| Franchise abbreviations vary across eras | **0** variants, 30 identical codes in both files | **Update the proposal wording** (see 4.3 in the dataset card) | 
| Trigger blocks FG made > FG attempted | The player file has **no** FGM/FGA columns | **Replace with** a trigger on `fg_pct`, `fg3_pct`, `ft_pct` between 0 and 1 and `minutes_per_game` ≤ 48, keeping `games_played` ≤ 82 | 
| Function `true_shooting_pct` on players | No FGA/FTA columns, so it cannot be computed per player; the team file already stores it | **Replace with** `points_per_36` (points × 36 ÷ minutes) or compute team efficiency; keep `team_win_pct`, now justified by 1,227 inconsistent rows | 
| Procedures `add_player_season_stat`, `trade_player`; cursor `cur_season_leaders`; package `nba_players_pkg` | Supported by the tables above | **Matches** | 
| Innovation: Tableau Desktop analytics dashboard reading Oracle views | Updated from initial UI note to match the analytics dashboard plan | **Matches `README.md`** | 

### 8.2 Phase I business process: "Player Season Stat Recording"

Source: your Phase I swimlane diagram (draw.io). Lanes: Clerk, Streamlit App, Oracle Database, Analyst/Coach/Scout, Auditor. The table maps each activity to the Phase II tables.

| Diagram activity | Lane | Operation | Tables and columns used | 
| ----- | ----- | ----- | ----- | 
| Enter stat line in form | Clerk | Input | The form collects `player_code`, `season_year`, `team_code` and the statistic columns of PLAYER_SEASON_STATS | 
| Submit to `add_player_season_stat` | Streamlit App | Calls the procedure | none | 
| Check player exists (gateway "Player exists?") | Oracle Database | **Retrieve** | PLAYERS (`player_code`); SEASONS (`season_year`) and TEAMS (`team_code`) are looked up to get `season_id` and `team_id` | 
| Duplicate for this season? | Oracle Database | **Retrieve** | PLAYER_SEASON_STATS, unique key (`player_id`, `season_id`, `team_id`) | 
| Stats valid? (games ≤ 82, ranges) | Oracle Database | Retrieve / reject | `games_played` 1–82, `minutes_per_game` 0–48, `fg_pct`, `fg3_pct`, `ft_pct` 0–1, `salary_million_usd` > 0, flags 0/1 | 
| Change allowed today? (non-working-day rule) | Oracle Database | Retrieve / reject | System date only (restriction trigger, Phase VI) | 
| Insert record | Oracle Database | **Create** | PLAYER_SEASON_STATS | 
| Write audit log entry | Oracle Database | **Create** | STAT_AUDIT_LOG (built in Phase VI) | 
| Confirm success to clerk / Show error to clerk / Correct entry | Streamlit App, Clerk | Display | none | 
| Auditor box | Auditor | **Retrieve** | STAT_AUDIT_LOG | 
| (no activity in the diagram) | Analyst/Coach/Scout | **Retrieve** | PLAYER_SEASON_STATS, TEAM_SEASON_STATS and all lookup tables | 

| End event in the diagram | Data effect | 
| ----- | ----- | 
| Stat line stored | One new row in PLAYER_SEASON_STATS and one in STAT_AUDIT_LOG | 
| Rejected: unknown player | No change (the `player_code` is not in PLAYERS) | 
| Rejected: duplicate | No change (the unique key would be violated) | 
| Blocked | No change (non-working-day rule) |
