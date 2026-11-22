# Research Log – NBA Players System

> **Before submitting:** read each source and edit the "Key finding" and "Gap it shows" columns in your own words. Entries marked *(verify)* were drafted from general knowledge and must be checked against the source. Cite only sources you have actually read.

| # | Source (APA) | Type | Link | Key finding | Gap it shows |
|---|--------------|------|------|-------------|--------------|
| 1 | Kubatko, J., Oliver, D., Pelton, K., & Rosenbaum, D. T. (2007). A starting point for analyzing basketball statistics. *Journal of Quantitative Analysis in Sports, 3*(3). | Research paper | https://doi.org/10.2202/1559-0410.1070 | Counting totals mislead; possession-based and pace-adjusted measures allow fair comparison of players and teams across games and seasons *(verify)* | Defines the formulas but offers no database that stores the data and enforces rules on it |
| 2 | Sarlis, V., & Tjortjis, C. (2020). Sports analytics — Evaluation of basketball players and team performance. *Information Systems, 93*, 101562. | Research paper | https://doi.org/10.1016/j.is.2020.101562 | Reviews analytical methods for evaluating basketball players and team performance *(verify details)* | Concentrates on analysis methods, not on how the underlying data is structured, validated or audited |
| 3 | National Basketball Association. (2025). *NBA advanced stats official documentation and glossary*. NBA Media Central. | Report / statistics | https://www.nba.com/stats/help/glossary | Official definitions of advanced metrics such as true shooting % and net rating | Definitions only; users must compute and store the metrics themselves |
| 4 | Sports Reference LLC. (2026). *Basketball-Reference database architecture & historical logs*. | Existing system | https://www.basketball-reference.com | Large, searchable historical archive of player and team statistics | Read-only website: no user data entry, custom business rules, audit trail or Oracle integration |
| 5 | Meruva, K. S. (2026). *NBA Player and Team Statistics (1980–2026)* [Data set]. Kaggle. | Dataset | https://www.kaggle.com/datasets/meruvakodandasuraj/nba-player-and-team-statistics-19802026 | `player_season_stats.csv` (50,785 rows) and `team_season_records.csv` (1,410 rows) cover 1980–2026 | Flat files: repeated team and player values, text team names, no constraints or keys |
| 6 | Oracle Corporation. (2024). *Oracle Database PL/SQL language reference (21c/23ai)*. Oracle Help Center. | Technical source | https://docs.oracle.com/en/database/oracle/oracle-database/ | Specifies packages, triggers, cursors and exception handling in PL/SQL | Documents features generally; does not show them applied to sports statistics |

## Dataset check (to complete)
- Licence on Kaggle page: ______
- Date collected: ______
- Confirm `team` values in the player file match `team_id` in the team file: ______
