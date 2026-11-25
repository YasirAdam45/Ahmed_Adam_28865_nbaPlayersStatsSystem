# Data Dictionary (logical design)

> **Author:** Ahmed Adam | **Student ID:** 28865 | **Group:** B | **Project:** NBA Players Stats System (INSY 8311, Phase II)

Oracle data types and sizes are left out on purpose; they are added in Phase IV. "Required" means the column can never be empty. "Source" shows the raw CSV field the column comes from (see the full mapping in `normalization.md` section 5). The ERD is in `erd.png`.

## Relationships

| Parent (one) | Child (many) | Meaning | Child side is |
|---|---|---|---|
| DIVISIONS | TEAMS | A division groups several teams; every team belongs to exactly one division | Mandatory |
| POSITIONS | PLAYERS | A position is the primary position of many players; every player has one | Mandatory |
| PLAYERS | PLAYER_SEASON_STATS | A player has one stat line per season and team | Mandatory |
| SEASONS | PLAYER_SEASON_STATS | A season contains many player stat lines | Mandatory |
| TEAMS | PLAYER_SEASON_STATS | A team has many player stat lines | Mandatory |
| TEAMS | TEAM_SEASON_STATS | A team has one record per season | Mandatory |
| SEASONS | TEAM_SEASON_STATS | A season has one record for each of the 30 teams | Mandatory |

Cardinality is one-to-many at every relationship. The parent end is "exactly one" and the child end is "zero or many" (a division, position, season or team may exist before it has any child rows). Every foreign key is required, so no child row can exist without its parent.

## 1. DIVISIONS (6 rows)
Purpose: removes the repeated conference text (transitive dependency division → conference).

| Column | Key | Required? | Unique? | Source (raw field) | Meaning |
|---|---|---|---|---|---|
| division_id | PK | Yes | Yes | generated | Internal identifier of the division |
| division_name | UQ | Yes | Yes | division | Division name (Atlantic, Central, Southeast, Northwest, Pacific, Southwest) |
| conference_name |  | Yes | No | conference | Conference of the division (East or West); one conference per division |

## 2. POSITIONS (5 rows)
Purpose: lookup table for the five court positions.

| Column | Key | Required? | Unique? | Source (raw field) | Meaning |
|---|---|---|---|---|---|
| position_id | PK | Yes | Yes | generated | Internal identifier of the position |
| position_code | UQ | Yes | Yes | position | Short code: PG, SG, SF, PF, C |
| position_name |  | Yes | No | added by designer | Full position name (reference data, not in the raw file) |

## 3. SEASONS (47 rows)
Purpose: one row per season; holds the era label shared by both files.

| Column | Key | Required? | Unique? | Source (raw field) | Meaning |
|---|---|---|---|---|---|
| season_id | PK | Yes | Yes | generated | Internal identifier of the season |
| season_year | UQ | Yes | Yes | season | Season year, 1980 to 2026 |
| era_name |  | Yes | No | era | Named era of the league that contains the season (7 eras) |

## 4. TEAMS (30 rows)
Purpose: one row per franchise; removes repeated team name, division and conference.

| Column | Key | Required? | Unique? | Source (raw field) | Meaning |
|---|---|---|---|---|---|
| team_id | PK | Yes | Yes | generated | Internal identifier of the team |
| team_code | UQ | Yes | Yes | team_id (team file), team (player file) | Three-letter franchise code, e.g. LAL |
| team_name | UQ | Yes | Yes | team_name | Full team name |
| division_id | FK → DIVISIONS | Yes | No | division | Division the team plays in |

## 5. PLAYERS (8,045 rows)
Purpose: one row per real player, with the anonymised code that replaces the unreliable raw `player_id`.

| Column | Key | Required? | Unique? | Source (raw field) | Meaning |
|---|---|---|---|---|---|
| player_id | PK | Yes | Yes | generated | Internal identifier of the player |
| player_code | UQ | Yes | Yes | player_id + player_name | Anonymised player code (PLR_00001 format) |
| full_name |  | Yes | No | player_name | Display name (not unique: 600 names are shared by several players) |
| position_id | FK → POSITIONS | Yes | No | position | Primary position of the player |
| is_legend |  | Yes | No | is_legend | 1 if the player is marked as a legend, otherwise 0 |

## 6. PLAYER_SEASON_STATS (50,785 rows)
Purpose: junction table between PLAYERS, SEASONS and TEAMS; stores one statistical line per player, season and team.

| Column | Key | Required? | Unique? | Source (raw field) | Meaning |
|---|---|---|---|---|---|
| stat_id | PK | Yes | Yes | generated | Internal identifier of the stat line |
| player_id | FK → PLAYERS | Yes | UQ (with season_id, team_id) | player_id + player_name | Player the stat line belongs to |
| season_id | FK → SEASONS | Yes | UQ (with player_id, team_id) | season | Season of the stat line |
| team_id | FK → TEAMS | Yes | UQ (with player_id, season_id) | team | Team the player played for in that season (part of the key to allow trades) |
| age |  | Yes | No | age | Age of the player in that season |
| games_played |  | Yes | No | games_played | Games played, 1 to 82 |
| minutes_per_game |  | Yes | No | minutes_per_game | Average minutes per game, 0 to 48 |
| points_per_game |  | Yes | No | points_per_game | Average points per game |
| assists_per_game |  | Yes | No | assists_per_game | Average assists per game |
| rebounds_per_game |  | Yes | No | rebounds_per_game | Average rebounds per game |
| steals_per_game |  | Yes | No | steals_per_game | Average steals per game |
| blocks_per_game |  | Yes | No | blocks_per_game | Average blocks per game |
| turnovers_per_game |  | Yes | No | turnovers_per_game | Average turnovers per game |
| fg_pct |  | Yes | No | fg_pct | Field goal percentage as a fraction, 0 to 1 |
| fg3_pct |  | Yes | No | fg3_pct | Three-point percentage as a fraction, 0 to 1 |
| ft_pct |  | Yes | No | ft_pct | Free throw percentage as a fraction, 0 to 1 |
| plus_minus |  | Yes | No | plus_minus | Team point difference while the player is on court |
| player_efficiency_rating |  | Yes | No | player_efficiency_rating | Player efficiency rating (PER) |
| win_shares |  | Yes | No | win_shares | Estimated wins contributed, 0 or more |
| box_plus_minus |  | Yes | No | box_plus_minus | Box plus/minus score |
| vorp |  | Yes | No | vorp | Value over replacement player |
| salary_million_usd |  | Yes | No | salary_million_usd | Salary in millions of US dollars, above 0 |
| mvp_flag |  | Yes | No | mvp_award | 1 if the player won MVP that season (0 in all rows of the file) |
| all_star_flag |  | Yes | No | all_star | 1 if selected for the All-Star game |
| all_nba_flag |  | Yes | No | all_nba | 1 if named to an All-NBA team |
| dpoy_flag |  | Yes | No | dpoy_award | 1 if Defensive Player of the Year |

**Table-level unique rule:** (`player_id`, `season_id`, `team_id`) must be unique.

## 7. TEAM_SEASON_STATS (1,410 rows)
Purpose: junction table between TEAMS and SEASONS; stores one record per team and season. `win_pct` and `net_rating` are not stored because they are derived.

| Column | Key | Required? | Unique? | Source (raw field) | Meaning |
|---|---|---|---|---|---|
| team_stat_id | PK | Yes | Yes | generated | Internal identifier of the team-season record |
| team_id | FK → TEAMS | Yes | UQ (with season_id) | team_id | Team of the record |
| season_id | FK → SEASONS | Yes | UQ (with team_id) | season | Season of the record |
| wins |  | Yes | No | wins | Regular-season wins |
| losses |  | Yes | No | losses | Regular-season losses (wins + losses ≤ 82) |
| made_playoffs |  | Yes | No | made_playoffs | 1 if the team reached the playoffs, otherwise 0 |
| playoff_wins |  | Yes | No | playoff_wins | Playoff games won (0 if no playoffs) |
| won_championship |  | Yes | No | won_championship | 1 if the team won the title that season |
| points_for |  | Yes | No | points_for | Average points scored per game |
| points_against |  | Yes | No | points_against | Average points allowed per game |
| pace |  | Yes | No | pace | Possessions per 48 minutes |
| offensive_rating |  | Yes | No | offensive_rating | Points scored per 100 possessions |
| defensive_rating |  | Yes | No | defensive_rating | Points allowed per 100 possessions |
| three_point_attempt_pct |  | Yes | No | three_point_attempt_pct | Share of shots that are three-point attempts, 0 to 1 |
| true_shooting_pct |  | Yes | No | true_shooting_pct | Team true shooting percentage, 0 to 1 |
| payroll_million_usd |  | Yes | No | payroll_million_usd | Team payroll in millions of US dollars, above 0 |
| home_attendance_avg |  | Yes | No | home_attendance_avg | Average home attendance per game |

**Table-level unique rule:** (`team_id`, `season_id`) must be unique.

## 8. STAT_AUDIT_LOG (planned, built in Phase VI)
Purpose: records every INSERT, UPDATE and DELETE on the statistics tables. It has no foreign keys because log entries must survive when the audited row is deleted.

| Column | Key | Required? | Unique? | Source (raw field) | Meaning |
|---|---|---|---|---|---|
| audit_id | PK | Yes | Yes | generated | Identifier of the audit entry |
| table_name |  | Yes | No | generated | Table that was changed |
| operation |  | Yes | No | generated | INSERT, UPDATE or DELETE |
| record_key |  | Yes | No | generated | Primary key value of the changed row |
| old_values / new_values |  | No | No | generated | Values before and after the change |
| changed_by |  | Yes | No | generated | Database user who made the change |
| changed_at |  | Yes | No | generated | Date and time of the change |

## Business rules found in the data (enforced later by constraints and triggers)

| Rule | Table.column | Observed in raw data | Enforced in |
|---|---|---|---|
| Games played between 1 and 82 | PLAYER_SEASON_STATS.games_played | min 1, max 82, 0 violations | CHECK + trigger (Phase VI) |
| Minutes between 0 and 48 | PLAYER_SEASON_STATS.minutes_per_game | 0 violations | CHECK |
| Percentages between 0 and 1 | fg_pct, fg3_pct, ft_pct, three_point_attempt_pct, true_shooting_pct | 0 violations | CHECK + trigger |
| Flags are 0 or 1 | all `*_flag`, `is_legend`, `made_playoffs`, `won_championship` | 0 violations | CHECK |
| Salary and payroll above 0 | salary_million_usd, payroll_million_usd | 0 violations | CHECK |
| Wins + losses ≤ 82 | TEAM_SEASON_STATS | always exactly 82 | CHECK |
| Playoff wins only if the team made the playoffs | TEAM_SEASON_STATS | 0 violations | CHECK |
| One stat line per player, season and team | PLAYER_SEASON_STATS | 0 duplicates after the player-code repair | UNIQUE |
| One record per team and season | TEAM_SEASON_STATS | 0 duplicates | UNIQUE |
| No changes on non-working days | all statistics tables | not in data (Phase 0 restriction rule) | Restriction trigger (Phase VI) |
