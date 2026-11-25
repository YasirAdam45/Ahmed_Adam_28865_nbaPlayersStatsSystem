# Dataset Card: NBA Player and Team Statistics (1980–2026)

> **Author:** Ahmed Adam | **Student ID:** 28865 | **Group:** [your group] | **Project:** NBA Players Stats System (INSY 8311, Phase II)

## 1. Source

| Item | Value |
|---|---|
| Title | NBA Player and Team Statistics (1980–2026) |
| Author | Meruva, K. S. (2026) |
| Platform | Kaggle |
| Link | https://www.kaggle.com/datasets/meruvakodandasuraj/nba-player-and-team-statistics-19802026 |
| Licence | CC0: Public Domain (as stated in the Phase 0 proposal; confirm on the Kaggle page) |
| Date collected | **[YYYY-MM-DD: write the day you downloaded the files]** |
| Format | CSV, UTF-8, comma separated, one header line |
| Location in repo | `data/raw/` (files are kept exactly as downloaded and never edited) |

## 2. Files used

| File | Rows | Columns | Size | One record represents |
|---|---|---|---|---|
| `player_season_stats.csv` | 50,785 | 28 | 6,324,842 bytes | One player's statistical line for one team in one season |
| `team_season_records.csv` | 1,410 | 22 | 181,818 bytes | One team's record, ratings, payroll and attendance for one season |

Both files exceed the 500-record minimum. Coverage: seasons 1980–2026 (47 seasons), 30 team codes, 5 positions, 7 eras.

**Files in the project folder that are NOT stored in `data/raw/`:** `career_profiles.csv` (39 rows), `era_summary.csv` (7 rows) and `mvp_winners.csv` (0 rows). They are aggregates derived from the two files above, not source data. In Phase IV they are rebuilt as views (`vw_career_profiles`, `vw_era_summary`, `vw_mvp_winners`).

## 3. Privacy and anonymisation

- The files contain no phone numbers, addresses, birth dates, national IDs or any other personal contact data. They hold public sports performance figures only.
- Player identifiers are replaced by anonymised codes (`PLR_00001` format, see section 5) when the data is loaded in Phase IV. The raw file is never changed.
- `player_name` is kept as a display name. It is a public sports record (or a generated name) and has no link to a private person.

## 4. Data quality report (counts only; nothing is fixed in Phase II)

Produced by `data/quality_check.py` on the raw files. Cleaning happens in Phase IV.

### 4.1 player_season_stats.csv

| # | Check | Result |
|---|---|---|
| 1 | Missing values (all 28 columns) | **0** |
| 2 | Numeric columns stored as text | **0** |
| 3 | Fully duplicated rows | **0** |
| 4 | Distinct `player_id` values / distinct `player_name` values | **8,000** / **644** |
| 5 | **One `player_id` (P00001) used by 46 different players** | **627 rows** (Magic Johnson, Larry Bird, LeBron James, …) |
| 6 | Real distinct players, counted as (`player_id`, `player_name`) pairs | **8,045** (not 8,000) |
| 7 | Excess duplicate rows on (`player_id`, `season`, `team`) | **143** (123 key groups, 266 rows). All caused by the shared ID in check 5 |
| 8 | Excess duplicate rows on (`player_id`, `season`) | **580** (same cause) |
| 9 | Duplicates on (`player_id`, `player_name`, `season`, `team`) | **0** (the repaired key is unique) |
| 10 | Names that belong to more than one `player_id` | **600** names (e.g. 36 different "Marcus Anderson"). `player_name` can never be a key |
| 11 | P00001 rows with `is_legend` = 0 | **6** (player: Jamal Smith; this name also exists under another ID) |
| 12 | Players with more than one team in one season (trades) | **0** |
| 13 | Players with more than one team in their whole career | **0** |
| 14 | Summary rows such as TOT / 2TM / 3TM | **0** |
| 15 | Players with more than one position | **0** |
| 16 | `games_played` > 82 or < 1 | **0** / **0** |
| 17 | `minutes_per_game` < 0 or > 48 | **0** / **0** |
| 18 | `fg_pct`, `fg3_pct`, `ft_pct` outside 0–1 | **0** |
| 19 | `salary_million_usd` ≤ 0 (range 0.4–20.6) | **0** |
| 20 | `age` range and players whose age does not rise by 1 per season | 22–42; **1** player (LeBron James: age stays 42 in 2025 and 2026) |
| 21 | Rows with `mvp_award` = 1 | **0** (the column is empty of information; `mvp_winners.csv` has 0 rows for the same reason) |
| 22 | Rows with `all_star` = 1 / `all_nba` = 1 / `dpoy_award` = 1 | **170** / **64** / **2** (only the named stars carry awards) |
| 23 | Flag columns holding a value other than 0 or 1 | **0** |
| 24 | Rows with `all_nba` = 1 but `all_star` = 0 | **0** |

### 4.2 team_season_records.csv

| # | Check | Result |
|---|---|---|
| 1 | Missing values (all 22 columns) | **0** |
| 2 | Fully duplicated rows / duplicates on (`season`, `team_id`) | **0** / **0** |
| 3 | Distinct team codes in this file / in the player file | **30** / **30**; codes missing from either file: **0** |
| 4 | Team codes with more than one `team_name`, or more than one `division` | **0** / **0** (no spelling variants and no historical code changes) |
| 5 | Divisions that appear under more than one conference | **0** |
| 6 | Teams per season | 30 in every one of the 47 seasons (1,410 = 30 × 47) |
| 7 | Eras / seasons that belong to more than one era | **7** / **1** (era depends on the season only) |
| 8 | Rows where `wins` + `losses` ≠ 82 | **0** (every season has exactly 82 games) |
| 9 | **Rows where `win_pct` differs from wins ÷ (wins + losses)** | **1,227 of 1,410** (largest gap 0.013) |
| 10 | Rows where `net_rating` ≠ `offensive_rating` − `defensive_rating` | **0** (the column is fully derivable) |
| 11 | Rows with `playoff_wins` > 0 but `made_playoffs` = 0 | **0** |
| 12 | Seasons with 0 / 1 / more than 1 champion flagged | **43** / **4** (1980, 1999, 2003, 2022) / **0** |
| 13 | Rows with `home_attendance_avg` exactly 20,000 (looks capped) | **519** |
| 14 | `payroll_million_usd` range | 5.0–191.0 |

### 4.3 What these counts mean for the design

1. **The raw `player_id` cannot identify a player.** A player is identified by (`player_id`, `player_name`). Phase IV assigns each of the 8,045 pairs its own `player_code`.
2. **No trades exist in the data**, but the design keeps `team_id` inside the unique key of PLAYER_SEASON_STATS so the planned `trade_player` procedure still works.
3. **`win_pct` is unreliable (1,227 rows) and `net_rating` is derivable**, so neither is stored. Both are computed (function `team_win_pct`, view column `net_rating`).
4. **Award columns carry little information** (no MVP at all, 2 DPOY). They are kept, but analysis queries should not depend on them.
5. The Phase 0 problem statement mentions summary rows and franchise-code variants. The counts above show **0** of each, so the statement should be reworded around the real problems (shared ID, duplicate keys, inconsistent `win_pct`, no integrity rules).

## 5. Anonymised player code rule (applied in Phase IV)

| Raw situation | Rule | Example |
|---|---|---|
| `player_id` used by one name (7,999 IDs) | `PLR_` + the 5 digits of the raw ID | P00002 → `PLR_00002` |
| P00001, first name in file order | `PLR_00001` | Magic Johnson → `PLR_00001` |
| P00001, the other 45 names | `PLR_08001` to `PLR_08045`, in order of first appearance | next name in file order → `PLR_08001` |

## 6. Limitations

- Several patterns suggest the data may be partly simulated rather than an official export: all 30 current franchises exist in 1980, every season has exactly 82 games, there are 8,000 players with only 644 distinct names, and awards exist only for about 46 star names. Read the Kaggle description and state in your defence how the data was produced.
- Salaries (0.4–20.6 million USD) and payrolls do not match real NBA figures, so use them for database work, not for real-world claims.
