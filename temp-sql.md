add a column for fantasy score
```sql
ALTER TABLE game_stats 
ADD fantasy_score INT;
```

calculate fantasy score
```sql
UPDATE game_stats
SET fantasy_score = PTS + FG3M - FG2A - FG3A + 2*FG2M + 2*FG3M + OREB + DREB + 2*AST + 4*STL + 4*BLK - 2*`TO`;
```

calculate rolling average
```sql
WITH FantasyScores AS
(
	SELECT GAME_ID, GAME_DATE, PLAYER_ID, fantasy_score 
	FROM game_stats
	WHERE `POSITION` != 'Team'
)
SELECT GAME_DATE, PLAYER_ID,
	AVG(fantasy_score) OVER (
		PARTITION BY PLAYER_ID
		ORDER BY GAME_DATE
		ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW 
	) AS rolling_avg_fantasy_score
FROM FantasyScores;
```

calculate rolling average and store it in a CTE instead, then rank players by rolling average on each day 
```sql
WITH 
FantasyScores AS (
	SELECT GAME_ID, GAME_DATE, PLAYER_ID, fantasy_score 
	FROM game_stats
	WHERE `POSITION` != 'Team'
),
RollingAverages AS (
	SELECT GAME_DATE, PLAYER_ID,
		AVG(fantasy_score) OVER (
			PARTITION BY PLAYER_ID
			ORDER BY GAME_DATE
			ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW 
		) AS rolling_avg_fantasy_score
	FROM FantasyScores
)
SELECT GAME_DATE, PLAYER_ID, rolling_avg_fantasy_score, RANK() OVER (
		PARTITION BY GAME_DATE
		ORDER BY rolling_avg_fantasy_score DESC
	) AS daily_rank
FROM RollingAverages
ORDER BY GAME_DATE, daily_rank;
```

store that in another CTE and then only show the top 8 players on each day, create a new table for this
```sql
CREATE TABLE top_fantasy_scores_over_time AS
WITH 
FantasyScores AS (
	SELECT GAME_ID, GAME_DATE, PLAYER_ID, fantasy_score 
	FROM game_stats
	WHERE `POSITION` != 'Team'
),
RollingAverages AS (
	SELECT GAME_DATE, PLAYER_ID,
		AVG(fantasy_score) OVER (
			PARTITION BY PLAYER_ID
			ORDER BY GAME_DATE
			ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW 
		) AS rolling_avg_fantasy_score
	FROM FantasyScores
),
RankedScores AS (
	SELECT GAME_DATE, PLAYER_ID, rolling_avg_fantasy_score, RANK() OVER (
			PARTITION BY GAME_DATE
			ORDER BY rolling_avg_fantasy_score DESC
		) AS daily_rank
	FROM RollingAverages
)
SELECT GAME_DATE, PLAYER_ID, rolling_avg_fantasy_score, daily_rank
FROM RankedScores
WHERE daily_rank <= 8
ORDER BY GAME_DATE, daily_rank;
```

track the fantasy_score rank over the course of the season of top 5 players at the end of the season
```sql
CREATE TABLE track_mvps_over_season AS
WITH 
FantasyScores AS (
	SELECT GAME_ID, GAME_DATE, PLAYER_ID, fantasy_score 
	FROM game_stats
	WHERE `POSITION` != 'Team'
),
RollingAverages AS (
	SELECT GAME_DATE, PLAYER_ID,
		AVG(fantasy_score) OVER (
			PARTITION BY PLAYER_ID
			ORDER BY GAME_DATE
			ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW 
		) AS rolling_avg_fantasy_score
	FROM FantasyScores
),
RankedScores AS (
	SELECT GAME_DATE, PLAYER_ID, rolling_avg_fantasy_score, RANK() OVER (
			PARTITION BY GAME_DATE
			ORDER BY rolling_avg_fantasy_score DESC
		) AS daily_rank
	FROM RollingAverages
),
TopFiveScoresEndOfSeason AS (
	SELECT GAME_DATE, PLAYER_ID, rolling_avg_fantasy_score, daily_rank
	FROM RankedScores
	WHERE daily_rank <= 5 AND GAME_DATE = (SELECT MAX(GAME_DATE) FROM RankedScores)
	ORDER BY GAME_DATE, daily_rank
)
SELECT GAME_DATE, PLAYER_ID, rolling_avg_fantasy_score, daily_rank
FROM RankedScores
WHERE PLAYER_ID IN (SELECT PLAYER_ID FROM TopFiveScoresEndOfSeason);
```