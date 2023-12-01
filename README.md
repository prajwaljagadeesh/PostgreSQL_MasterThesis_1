# PostgreSQL_MasterThesis_1
End to end Player and team performance analysis Sql queries in postgres


Create materialized view bowling_averages as
SELECT cricket_scores.bowler,
       matches.season,
       SUM(cricket_scores.total_run - cricket_scores.extras_run) AS total_runs,
       COUNT(CASE WHEN cricket_scores.is_wicket_delivery THEN 1 END) AS wickets,
	   CASE WHEN COUNT(CASE WHEN cricket_scores.is_wicket_delivery THEN 1 END) > 5
	   THEN ROUND(SUM(cricket_scores.total_run - cricket_scores.extras_run) / COUNT(CASE WHEN cricket_scores.is_wicket_delivery THEN 1 END), 2)
	   ELSE 0 END AS bowling_average
FROM cricket_scores
INNER JOIN matches ON cricket_scores.id = matches.id
GROUP BY cricket_scores.bowler, matches.season
HAVING COUNT(DISTINCT matches.id) >= 10 and
    COUNT(cricket_scores.is_wicket_delivery) > 10
order by bowling_average;

CREATE MATERIALIZED VIEW batting_average_by_season AS
SELECT 
    cs.batter AS batsman,
    cm3.Season,
    COUNT(DISTINCT cm3.ID) AS total_innings,
    SUM(cs.batsman_run) AS total_runs,
    CAST(ROUND(SUM(cs.batsman_run) * 1.0 / COUNT(DISTINCT cm3.ID)) AS INT) AS batting_average
FROM 
    cricket_stats AS cs
JOIN 
    cricket_matches_3 AS cm3 ON cs.ID = cm3.ID
GROUP BY 
    cs.batter, cm3.Season;

CREATE UNIQUE INDEX idx_batting_average_season
ON batting_average_by_season (batsman, Season);

CREATE MATERIALIZED VIEW Bowling_economy_rate as
SELECT
    cm.season AS Season,
    cs.bowler AS Bowler,
    count(cs.overs) AS TotalBalls,
    SUM(cs.total_run) AS TotalRunsConceded,
    SUM(cs.total_run) / (count(cs.overs) / 6) AS EconomyRate
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm ON cs.ID = cm.ID
WHERE
    cs.overs IS NOT NULL
GROUP BY
    cm.season, cs.bowler
HAVING
    COUNT(DISTINCT cm.id) > 3
ORDER BY
    Season ASC, EconomyRate;
   
   SELECT
    cm.season AS Season,
    cs.bowler AS Bowler,
    count(cs.overs) AS TotalBalls,
    SUM(cs.total_run) AS TotalRunsConceded,
    CASE WHEN count(cs.overs) = 0 THEN 0
         ELSE ROUND(SUM(cs.total_run) * 6 / count(cs.overs), 2)
    END AS EconomyRate
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm ON cs.ID = cm.ID
WHERE
    cs.overs IS NOT NULL
    AND cs.overs <= 6 -- Consider only powerplay overs (first 6 overs)
    cs.overs > 6 AND cs.overs <= 15  -- Consider middle overs (overs 7 to 15)
    and cs.overs > 15  -- Consider death overs (overs 16 onwards)
GROUP BY
    cm.season, cs.bowler
HAVING
    COUNT(DISTINCT cm.id) > 3
ORDER BY
    Season ASC, EconomyRate desc;
CREATE MATERIALIZED VIEW Bowling_middle_economy_rate AS
SELECT
    cm.season AS Season,
    cs.bowler AS Bowler,
    count(cs.overs) AS TotalBalls,
    SUM(cs.total_run) AS TotalRunsConceded,
    CASE WHEN count(cs.overs) = 0 THEN 0
         ELSE ROUND(SUM(cs.total_run) * 6 / count(cs.overs), 2)
    END AS MiddleEconomyRate  -- Rename the column here
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm ON cs.ID = cm.ID
WHERE
    cs.overs IS NOT NULL
    AND 
GROUP BY
    cm.season, cs.bowler
HAVING
    COUNT(DISTINCT cm.id) > 3
ORDER BY
    Season ASC, MiddleEconomyRate;  -- Adjust column name in ORDER BY clause
CREATE MATERIALIZED VIEW Bowling_death_economy_rate AS
SELECT
    cm.season AS Season,
    cs.bowler AS Bowler,
    count(cs.overs) AS TotalBalls,
    SUM(cs.total_run) AS TotalRunsConceded,
    CASE WHEN count(cs.overs) = 0 THEN 0
         ELSE ROUND(SUM(cs.total_run) * 6 / count(cs.overs), 2)
    END AS DeathEconomyRate  -- Rename the column here
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm ON cs.ID = cm.ID
WHERE
    cs.overs IS NOT NULL
    AND cs.overs > 15 
GROUP BY
    cm.season, cs.bowler
HAVING
    COUNT(DISTINCT cm.id) > 3
ORDER BY
    Season ASC, DeathEconomyRate;  -- Adjust column name in ORDER BY clause
CREATE MATERIALIZED VIEW Wickets_Per_Match AS
CREATE MATERIALIZED VIEW Wickets_Per_Match AS
SELECT
    cm.season AS Season,
    cs.bowler AS Bowler,
    COUNT(DISTINCT cm.id) AS MatchesPlayed,
    SUM(CASE WHEN cs.isWicketDelivery THEN 1 ELSE 0 END) AS TotalWickets,
    ROUND(SUM(CASE WHEN cs.isWicketDelivery THEN 1 ELSE 0 END) / COUNT(DISTINCT cm.id), 2) AS WicketsPerMatch
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm ON cs.ID = cm.ID
GROUP BY
    cm.season, cs.bowler
HAVING
    COUNT(DISTINCT cm.id) > 3  -- Minimum matches threshold
ORDER BY
    Season ASC, WicketsPerMatch DESC, MatchesPlayed ASC;

select * from cricket_scores;
select * from matches;

create materialized view Awards as
SELECT
    player_of_match,
    season,
    COUNT(*) AS total_awards
FROM
    cricket_matches_3 cm 
WHERE
    player_of_match IS NOT NULL
GROUP BY
    player_of_match, season
ORDER BY
    total_awards DESC
LIMIT 25;

select 
sum(cricket_scores.batsman_run) as total_runs_by_batsmen,
cricket_scores.batter,
matches.season
from cricket_scores 
inner join matches on matches.id = cricket_scores.id 
group by cricket_scores.batter, matches.season
order by total_runs_by_batsmen desc


select 
		matches.id, 	
		matches.season,
		(matches.team1, matches.team2) as both_teams
		from 
		matches
		
	SELECT
    m.id,
    m.season,
    (m.team1, m.team2) AS both_teams,
    cs.batting_team AS batting_team,
    CASE
        WHEN m.team1 = cs.batting_team THEN m.team2
        ELSE m.team1
    END AS bowling_team
FROM
    matches m
    JOIN cricket_scores cs ON m.id = cs.id;


	
select
	bowler, 
	is_wicket_delivery, 
	count(id)/count(distinct id) as aveage
from cricket_scores
where is_wicket_delivery is TRUE 
group by 1, 2
order by 3 desc


 WITH BattingAverages AS (
    SELECT 
        cs.batter AS batsman,
        t.team_name AS team,
        cm3.Season,
        COUNT(DISTINCT cm3.ID) AS total_innings,
        SUM(cs.batsman_run) AS total_runs,
        CAST(ROUND(SUM(cs.batsman_run) * 1.0 / COUNT(DISTINCT cm3.ID)) AS INT) AS batting_average
    FROM 
        cricket_stats AS cs
    JOIN 
        cricket_matches_3 AS cm3 ON cs.ID = cm3.ID
    JOIN
        teams AS t ON cs.BattingTeam = t.team_name
    GROUP BY 
        cs.batter, t.team_name, cm3.Season
),
Wins AS (
    SELECT 
        cs.batter AS batsman,
        t.team_name AS team,
        cm3.Season,
        COUNT(DISTINCT cm3.ID) AS wins
    FROM 
        cricket_stats AS cs
    JOIN 
        cricket_matches_3 AS cm3 ON cs.ID = cm3.ID
    JOIN
        teams AS t ON cs.BattingTeam = t.team_name
    WHERE
        cm3.WinningTeam = t.team_name
    GROUP BY 
        cs.batter, t.team_name, cm3.Season
),
Losses AS (
    SELECT 
        cs.batter AS batsman,
        t.team_name AS team,
        cm3.Season,
        COUNT(DISTINCT cm3.ID) AS losses
    FROM 
        cricket_stats AS cs
    JOIN 
        cricket_matches_3 AS cm3 ON cs.ID = cm3.ID
    JOIN
        teams AS t ON cs.BattingTeam = t.team_name
    WHERE
        cm3.WinningTeam <> t.team_name AND cm3.WinningTeam IS NOT NULL
    GROUP BY 
        cs.batter, t.team_name, cm3.Season
)
SELECT
    ba.batsman,
    ba.Season,
    ba.team,
    ba.batting_average,
    ba.total_innings AS total_matches,
    w.wins,
    l.losses
FROM
    BattingAverages ba
JOIN
    Wins w ON ba.batsman = w.batsman AND ba.team = w.team AND ba.Season = w.Season
JOIN
    Losses l ON ba.batsman = l.batsman AND ba.team = l.team AND ba.Season = l.Season;


SELECT
    cm3.Season,
    cm3.WinningTeam AS team,
    COUNT(DISTINCT cm3.ID) AS won_matches
FROM
    cricket_matches_3 cm3
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    cm3.Season, cm3.WinningTeam
ORDER BY
    cm3.Season, team;

   SELECT
    teams.team_name AS team,
    cm3.Season,
    COUNT(DISTINCT cm3.ID) AS total_matches
FROM
    (SELECT DISTINCT team1 AS team_name FROM cricket_matches_3
     UNION
     SELECT DISTINCT team2 AS team_name FROM cricket_matches_3) AS teams
JOIN
    cricket_matches_3 cm3 ON teams.team_name IN (cm3.team1, cm3.team2)
GROUP BY
    teams.team_name, cm3.Season
ORDER BY
    team, Season;

   
   SELECT
    teams.team_name AS team,
    cm3.Season,
    COUNT(DISTINCT cm3.ID) AS total_matches,
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS matches_won
FROM
    (SELECT DISTINCT team1 AS team_name FROM cricket_matches_3
     UNION
     SELECT DISTINCT team2 AS team_name FROM cricket_matches_3) AS teams
JOIN
    cricket_matches_3 cm3 ON teams.team_name IN (cm3.team1, cm3.team2)
GROUP BY
    teams.team_name, cm3.Season
ORDER BY
    team, Season;
   
   SELECT
    teams.team_name AS team,
    cm3.Season,
    COUNT(DISTINCT cm3.ID) AS total_matches,
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS matches_won,
    COUNT(DISTINCT cm3.ID) - COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS matches_lost
FROM
    (SELECT DISTINCT team1 AS team_name FROM cricket_matches_3
     UNION
     SELECT DISTINCT team2 AS team_name FROM cricket_matches_3) AS teams
JOIN
    cricket_matches_3 cm3 ON teams.team_name IN (cm3.team1, cm3.team2)
GROUP BY
    teams.team_name, cm3.Season
ORDER BY
    team, Season;

   SELECT
    teams.team_name AS team,
    cm3.Season,
    COUNT(DISTINCT cm3.ID) AS total_matches,
    COUNT(DISTINCT cm3.ID) - COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS matches_lost
FROM
    (SELECT DISTINCT team1 AS team_name FROM cricket_matches_3
     UNION
     SELECT DISTINCT team2 AS team_name FROM cricket_matches_3) AS teams
JOIN
    cricket_matches_3 cm3 ON teams.team_name IN (cm3.team1, cm3.team2)
GROUP BY
    teams.team_name, cm3.Season
ORDER BY
    team, Season;

SELECT
    cs.batter AS batsman,
    cm3.Season,
    cm3.WinningTeam AS team,
    SUM(cs.batsman_run) AS total_runs,
    COUNT(DISTINCT cm3.ID) AS won_matches,
    SUM(cs.batsman_run) * 1.0 / COUNT(DISTINCT cm3.ID) AS batting_average
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    cs.batter, cm3.Season, cm3.WinningTeam
ORDER BY
    batsman, Season, team;

  
SELECT
    teams.team_name AS team,
    cm3.Season,
    cs.batter AS batsman,
    COUNT(DISTINCT cm3.ID) AS total_matches,
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS matches_won,
    COUNT(DISTINCT cm3.ID) - COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS matches_lost,
    -- Batting Average for Matches Won
    CASE
        WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) = 0 THEN 0
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) * 1.0 / COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END), 2)
    END AS batting_average_won,
    -- Batting Average for Matches Lost
    CASE
        WHEN (COUNT(DISTINCT cm3.ID) - COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END)) = 0 THEN 0
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) * 1.0 / (COUNT(DISTINCT cm3.ID) - COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END)), 2)
    END AS batting_average_lost
FROM
    (SELECT DISTINCT team1 AS team_name FROM cricket_matches_3
     UNION
     SELECT DISTINCT team2 AS team_name FROM cricket_matches_3) AS teams
JOIN
    cricket_matches_3 cm3 ON teams.team_name IN (cm3.team1, cm3.team2)
JOIN
    cricket_stats cs ON cs.ID = cm3.ID
GROUP BY
    teams.team_name, cm3.Season, cs.batter
ORDER BY
    team, Season, batsman;
   
   
CREATE MATERIALIZED view batting_average_for_team AS 
SELECT
    cs.batter AS batsman,
    cm3.Season,
    teams.team_name AS team,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_won_matches,
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS won_matches,
    CASE
        WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) = 0 THEN 0
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) * 1.0 / COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END), 2)
    END AS batting_average_won_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_lost_matches,
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) AS lost_matches,
    CASE
        WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) = 0 THEN 0
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) * 1.0 / COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END), 2)
    END AS batting_average_lost_matches
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
JOIN
    teams ON (cs.BattingTeam = teams.team_name)
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    cs.batter, cm3.Season, teams.team_name
HAVING
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
ORDER BY
    batsman, Season, team;
   
   CREATE MATERIALIZED view Sum_batting_average_for_team AS 
   WITH BattingData AS (
    SELECT
        cs.batter AS batsman,
        cm3.Season,
        teams.team_name AS team,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_won_matches,
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS won_matches,
        CASE
            WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) = 0 THEN 0
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) * 1.0 / COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END), 2)
        END AS batting_average_won_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_lost_matches,
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) AS lost_matches,
        CASE
            WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) = 0 THEN 0
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) * 1.0 / COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END), 2)
        END AS batting_average_lost_matches
    FROM
        cricket_stats cs
    JOIN
        cricket_matches_3 cm3 ON cs.ID = cm3.ID
    JOIN
        teams ON (cs.BattingTeam = teams.team_name)
    WHERE
        cm3.WinningTeam IS NOT NULL
        AND cm3.Season IN ('2019', '2020/21') -- Filter for specific years
    GROUP BY
        cs.batter, cm3.Season, teams.team_name
    HAVING
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
)

SELECT
    Season,
    team,
    SUM(batting_average_won_matches) AS sum_batting_average_won_matches,
    SUM(batting_average_lost_matches) AS sum_batting_average_lost_matches,
    SUM(won_matches + lost_matches) AS total_matches_played,
    SUM(won_matches) AS matches_won,
    SUM(lost_matches) AS matches_lost
FROM
    BattingData
GROUP BY
    Season, team
ORDER BY
    Season, team;

   
  CREATE MATERIALIZED view Batting_strikerate_for_team AS 
SELECT
    cm3.Season,
    teams.team_name AS team,
    cs.batter AS batsman,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_won_matches,
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS won_matches,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS balls_faced_won_matches,
    CASE
        WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) = 0 THEN 0
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END), 2)
    END AS strike_rate_won_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_lost_matches,
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) AS lost_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS balls_faced_lost_matches,
    CASE
        WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) = 0 THEN 0
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END), 2)
    END AS strike_rate_lost_matches
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
JOIN
    teams ON (CASE WHEN cs.BattingTeam = teams.team_name THEN cs.BattingTeam ELSE cs.battingteam END = teams.team_name)
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    cs.batter, cm3.Season, teams.team_name
HAVING
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
    OR COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) > 2
ORDER BY
    batsman, Season, team;
   
WITH MatchCounts AS (
    SELECT
        cm3.Season AS season,
        teams.team_name AS team,
        COUNT(DISTINCT cm3.ID) AS total_matches_played
    FROM
        cricket_matches_3 cm3
    JOIN
        teams ON teams.team_name IN (cm3.Team1, cm3.Team2)
    WHERE
        cm3.Season IN ('2019', '2020/21')
    GROUP BY
        cm3.Season, teams.team_name
    HAVING
        COUNT(DISTINCT cm3.ID) > 2
)

SELECT
    mc.season,
    mc.team,
    mc.total_matches_played,
    SUM(CASE WHEN cm3.WinningTeam = mc.team THEN 1 ELSE 0 END) AS matches_won,
    SUM(CASE WHEN cm3.WinningTeam != mc.team THEN 1 ELSE 0 END) AS matches_lost,
    SUM(CASE WHEN cm3.WinningTeam = mc.team THEN cs.batsman_run * 100.0 / mc.total_matches_played ELSE 0 END) AS sum_strike_rate_won_matches,
    SUM(CASE WHEN cm3.WinningTeam != mc.team THEN cs.batsman_run * 100.0 / mc.total_matches_played ELSE 0 END) AS sum_strike_rate_lost_matches
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
JOIN
    teams ON (cs.BattingTeam = teams.team_name)
JOIN
    MatchCounts mc ON mc.season = cm3.Season AND mc.team = teams.team_name
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    mc.season, mc.team, mc.total_matches_played
ORDER BY
    mc.season, mc.team;

















   
  CREATE MATERIALIZED view Boundary_percentage_for_team AS 
  SELECT
    cm3.Season,
    teams.team_name AS team,
    cs.batter AS batsman,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND (cs.batsman_run = 4 OR cs.batsman_run = 6) THEN 1 ELSE 0 END) AS boundary_balls_won_matches,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS balls_faced_won_matches,
    CASE
        WHEN SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) = 0 THEN NULL
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) * 100.0 / NULLIF(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END), 0), 2)
    END AS boundary_percentage_won_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND (cs.batsman_run = 4 OR cs.batsman_run = 6) THEN 1 ELSE 0 END) AS boundary_balls_lost_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS balls_faced_lost_matches,
    CASE
        WHEN SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) = 0 THEN NULL
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND (cs.batsman_run = 4 OR cs.batsman_run = 6) THEN 1 ELSE 0 END) * 100.0 / NULLIF(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END), 0), 2)
    END AS boundary_percentage_lost_matches
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
JOIN
    teams ON (cs.BattingTeam = teams.team_name)
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    cs.batter, cm3.Season, teams.team_name
HAVING
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
    OR COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) > 2
ORDER BY
    batsman, Season, team;
   
   CREATE MATERIALIZED view Batting_strikerate_for_team AS 
   SELECT
    cm3.Season,
    teams.team_name AS team,
    cs.batter AS batsman,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_won_matches,
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS won_matches,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS balls_faced_won_matches,
    CASE
        WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) = 0 THEN 0
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END), 2)
    END AS strike_rate_won_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_lost_matches,
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) AS lost_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS balls_faced_lost_matches,
    CASE
        WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) = 0 THEN 0
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END), 2)
    END AS strike_rate_lost_matches
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
JOIN
    teams ON (CASE WHEN cs.BattingTeam = teams.team_name THEN cs.BattingTeam ELSE cs.battingteam END = teams.team_name)
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    cs.batter, cm3.Season, teams.team_name
HAVING
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
    AND SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) > 100
ORDER BY
    batsman, Season, team;
     
  CREATE MATERIALIZED view sum_Batting_strikerate_for_team AS
  WITH BattingData AS (
    SELECT
        cm3.Season,
        teams.team_name AS team,
        cs.batter AS batsman,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_won_matches,
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS won_matches,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS balls_faced_won_matches,
        CASE
            WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) = 0 THEN 0
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END), 2)
        END AS strike_rate_won_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_lost_matches,
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) AS lost_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS balls_faced_lost_matches,
        CASE
            WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) = 0 THEN 0
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END), 2)
        END AS strike_rate_lost_matches
    FROM
        cricket_stats cs
    JOIN
        cricket_matches_3 cm3 ON cs.ID = cm3.ID
    JOIN
        teams ON (CASE WHEN cs.BattingTeam = teams.team_name THEN cs.BattingTeam ELSE cs.battingteam END = teams.team_name)
    WHERE
        cm3.WinningTeam IS NOT NULL
        AND cm3.Season IN ('2019', '2020/21') -- Filter for specific years
    GROUP BY
        cm3.Season, teams.team_name, cs.batter
    HAVING
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
        AND SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) > 100
)

SELECT
    Season,
    team,
    SUM(strike_rate_won_matches) AS sum_strike_rate_won_matches,
    SUM(strike_rate_lost_matches) AS sum_strike_rate_lost_matches
FROM
    BattingData
GROUP BY
    Season, team
ORDER BY
    Season, team;


  CREATE MATERIALIZED view Boundary_percentage_for_team AS 
  SELECT
    cm3.Season,
    teams.team_name AS team,
    cs.batter AS batsman,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_won_matches,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND (cs.batsman_run = 4 OR cs.batsman_run = 6) THEN cs.batsman_run ELSE 0 END) AS boundary_runs_won_matches,
    CASE
        WHEN SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) = 0 THEN NULL
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND (cs.batsman_run = 4 OR cs.batsman_run = 6) THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END), 2)
    END AS boundary_percentage_won_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_lost_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND (cs.batsman_run = 4 OR cs.batsman_run = 6) THEN cs.batsman_run ELSE 0 END) AS boundary_runs_lost_matches,
    CASE
        WHEN SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) = 0 THEN NULL
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND (cs.batsman_run = 4 OR cs.batsman_run = 6) THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END), 2)
    END AS boundary_percentage_lost_matches
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
JOIN
    teams ON (cs.BattingTeam = teams.team_name)
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    cs.batter, cm3.Season, teams.team_name
HAVING
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
    OR COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) > 2
    AND SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) > 200
ORDER BY
    batsman, Season, team;
   
   
CREATE MATERIALIZED view Sum_Boundary_percentage_for_team AS 
WITH BattingData AS (
    SELECT
        cm3.Season,
        teams.team_name AS team,
        cs.batter AS batsman,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_won_matches,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND (cs.batsman_run = 4 OR cs.batsman_run = 6) THEN cs.batsman_run ELSE 0 END) AS boundary_runs_won_matches,
        CASE
            WHEN SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) = 0 THEN NULL
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND (cs.batsman_run = 4 OR cs.batsman_run = 6) THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END), 2)
        END AS boundary_percentage_won_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_lost_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND (cs.batsman_run = 4 OR cs.batsman_run = 6) THEN cs.batsman_run ELSE 0 END) AS boundary_runs_lost_matches,
        CASE
            WHEN SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) = 0 THEN NULL
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND (cs.batsman_run = 4 OR cs.batsman_run = 6) THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END), 2)
        END AS boundary_percentage_lost_matches
    FROM
        cricket_stats cs
    JOIN
        cricket_matches_3 cm3 ON cs.ID = cm3.ID
    JOIN
        teams ON (cs.BattingTeam = teams.team_name)
    WHERE
        cm3.WinningTeam IS NOT NULL
        AND cm3.Season IN ('2019', '2020/21') -- Filter for specific years
    GROUP BY
        cm3.Season, teams.team_name, cs.batter
    HAVING
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
        OR COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) > 2
        AND SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) > 200
)

SELECT
    Season,
    team,
    SUM(boundary_percentage_won_matches) AS sum_boundary_percentage_won_matches,
    SUM(boundary_percentage_lost_matches) AS sum_boundary_percentage_lost_matches
FROM
    BattingData
GROUP BY
    Season, team
ORDER BY
    Season, team;





WITH MatchCounts AS (
    SELECT
        cm3.Season AS season,
        teams.team_name AS team,
        COUNT(DISTINCT cm3.ID) AS total_matches_played
    FROM
        cricket_matches_3 cm3
    JOIN
        teams ON teams.team_name IN (cm3.Team1, cm3.Team2)
    WHERE
        cm3.Season IN ('2019', '2020/21')
    GROUP BY
        cm3.Season, teams.team_name
    HAVING
        COUNT(DISTINCT cm3.ID) > 2
)

SELECT
    mc.season,
    mc.team,
    mc.total_matches_played,
    SUM(CASE WHEN cm3.Season = mc.season AND cm3.WinningTeam = mc.team THEN 1 ELSE 0 END) AS matches_won,
    SUM(CASE WHEN cm3.Season = mc.season AND cm3.WinningTeam != mc.team THEN 1 ELSE 0 END) AS matches_lost,
    SUM(CASE WHEN cm3.WinningTeam = mc.team THEN cs.batsman_run ELSE 0 END) AS sum_strike_rate_won_matches,
    SUM(CASE WHEN cm3.WinningTeam != mc.team THEN cs.batsman_run ELSE 0 END) AS sum_strike_rate_lost_matches
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
JOIN
    teams ON (cs.BattingTeam = teams.team_name)
JOIN
    MatchCounts mc ON mc.season = cm3.Season AND mc.team = teams.team_name
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    mc.season, mc.team, mc.total_matches_played
ORDER BY
    mc.season, mc.team;

   
   WITH BattingData AS (
    SELECT
        cm3.Season,
        teams.team_name AS team,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_won_matches,
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) AS won_matches,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS balls_faced_won_matches,
        CASE
            WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) = 0 THEN 0
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END), 2)
        END AS strike_rate_won_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) AS total_runs_lost_matches,
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) AS lost_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS balls_faced_lost_matches,
        CASE
            WHEN COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) = 0 THEN 0
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.batsman_run ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END), 2)
        END AS strike_rate_lost_matches
    FROM
        cricket_stats cs
    JOIN
        cricket_matches_3 cm3 ON cs.ID = cm3.ID
    JOIN
        teams ON (cs.BattingTeam = teams.team_name)
    WHERE
        cm3.WinningTeam IS NOT NULL
        AND cm3.Season IN ('2019', '2020/21') -- Filter for specific years
    GROUP BY
        cm3.Season, teams.team_name
    HAVING
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
)
SELECT
    Season,
    team,
    SUM(won_matches + lost_matches) AS total_matches_played,
    SUM(won_matches) AS matches_won,
    SUM(lost_matches) AS matches_lost,
    SUM(strike_rate_won_matches) AS sum_strike_rate_won_matches,
    SUM(strike_rate_lost_matches) AS sum_strike_rate_lost_matches
FROM
    BattingData
GROUP BY
    Season, team
ORDER BY
    Season, team;

CREATE MATERIALIZED view Bowling_average_for_team AS 
SELECT
    cm3.Season,
    teams.team_name AS team,
    cs.bowler AS bowler,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.total_run ELSE 0 END) AS runs_conceded_won_matches,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND cs.isWicketDelivery = TRUE THEN 1 ELSE 0 END) AS wickets_won_matches,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_won_matches,
    CASE
        WHEN SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND cs.isWicketDelivery = TRUE THEN 1 ELSE 0 END) = 0 THEN NULL
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.total_run ELSE 0 END) * 1.0 / (SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) / 6), 2)
    END AS bowling_average_won_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.total_run ELSE 0 END) AS runs_conceded_lost_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND cs.isWicketDelivery = TRUE THEN 1 ELSE 0 END) AS wickets_lost_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_lost_matches,
    CASE
        WHEN SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND cs.isWicketDelivery = TRUE THEN 1 ELSE 0 END) = 0 THEN NULL
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.total_run ELSE 0 END) * 1.0 / (SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) / 6), 2)
    END AS bowling_average_lost_matches
FROM
    cricket_stats cs
INNER JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
INNER JOIN
    teams ON (cm3.team1_id = teams.team_id OR cm3.team2_id = teams.team_id)
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    cm3.Season, teams.team_name, cs.bowler
HAVING
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND cs.isWicketDelivery = TRUE THEN 1 ELSE 0 END) > 3
    AND (COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
    OR COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) > 2)
ORDER BY
    bowler, Season, team;

   CREATE MATERIALIZED view Sum_Bowling_average_for_team AS 
   WITH BowlingData AS (
    SELECT
        cm3.Season,
        teams.team_name AS team,
        cs.bowler AS bowler,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.total_run ELSE 0 END) AS runs_conceded_won_matches,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND cs.isWicketDelivery = TRUE THEN 1 ELSE 0 END) AS wickets_won_matches,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_won_matches,
        CASE
            WHEN SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND cs.isWicketDelivery = TRUE THEN 1 ELSE 0 END) = 0 THEN NULL
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.total_run ELSE 0 END) * 1.0 / (SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) / 6), 2)
        END AS bowling_average_won_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.total_run ELSE 0 END) AS runs_conceded_lost_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND cs.isWicketDelivery = TRUE THEN 1 ELSE 0 END) AS wickets_lost_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_lost_matches,
        CASE
            WHEN SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND cs.isWicketDelivery = TRUE THEN 1 ELSE 0 END) = 0 THEN NULL
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.total_run ELSE 0 END) * 1.0 / (SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) / 6), 2)
        END AS bowling_average_lost_matches
    FROM
        cricket_stats cs
    INNER JOIN
        cricket_matches_3 cm3 ON cs.ID = cm3.ID
    INNER JOIN
        teams ON (cm3.team1_id = teams.team_id OR cm3.team2_id = teams.team_id)
    WHERE
        cm3.WinningTeam IS NOT NULL
    GROUP BY
        cm3.Season, teams.team_name, cs.bowler
    HAVING
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND cs.isWicketDelivery = TRUE THEN 1 ELSE 0 END) > 3
        AND (COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
        OR COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) > 2)
),

MatchCounts AS (
    SELECT
        cm3.Season,
        teams.team_name AS team,
        COUNT(DISTINCT cm3.ID) AS total_matches_played,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS matches_won,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS matches_lost
    FROM
        cricket_matches_3 cm3
    JOIN
        teams ON teams.team_name IN (cm3.Team1, cm3.Team2)
    WHERE
        cm3.Season IN ('2019', '2020/21')
    GROUP BY
        cm3.Season, teams.team_name
    HAVING
        COUNT(DISTINCT cm3.ID) > 2
)

SELECT
    bd.Season,
    bd.team,
    SUM(total_matches_played) AS total_matches_played,
    SUM(matches_won) AS matches_won,
    SUM(matches_lost) AS matches_lost,
    SUM(bowling_average_won_matches) AS sum_bowling_average_won_matches,
    SUM(bowling_average_lost_matches) AS sum_bowling_average_lost_matches
FROM
    BowlingData bd
JOIN
    MatchCounts mc ON bd.Season = mc.Season AND bd.team = mc.team
GROUP BY
    bd.Season, bd.team
ORDER BY
    bd.Season, bd.team;

   









CREATE MATERIALIZED view Bowling_economy_rate_for_team AS 
SELECT
    cm3.Season,
    teams.team_name AS team,
    cs.bowler AS bowler,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.total_run ELSE 0 END) AS runs_conceded_won_matches,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_won_matches,
    CASE
        WHEN SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) = 0 THEN NULL
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.total_run ELSE 0 END) * 6.0 / SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END), 2)
    END AS economy_rate_won_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.total_run ELSE 0 END) AS runs_conceded_lost_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_lost_matches,
    CASE
        WHEN SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) = 0 THEN NULL
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.total_run ELSE 0 END) * 6.0 / SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END), 2)
    END AS economy_rate_lost_matches
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
JOIN
    (
        SELECT
            team_id,
            team_name
        FROM
            teams
    ) teams ON (cm3.team1_id = teams.team_id OR cm3.team2_id = teams.team_id)
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    cs.bowler, cm3.Season, teams.team_name
HAVING
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
    OR COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) > 2
ORDER BY
    bowler, Season, team;


CREATE MATERIALIZED view Sum_Bowling_economy_rate_for_team AS 
WITH BowlingData AS (
    SELECT
        cm3.Season,
        teams.team_name AS team,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.total_run ELSE 0 END) AS runs_conceded_won_matches,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_won_matches,
        CASE
            WHEN SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) = 0 THEN NULL
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN cs.total_run ELSE 0 END) * 6.0 / SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END), 2)
        END AS economy_rate_won_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.total_run ELSE 0 END) AS runs_conceded_lost_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_lost_matches,
        CASE
            WHEN SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) = 0 THEN NULL
            ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN cs.total_run ELSE 0 END) * 6.0 / SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END), 2)
        END AS economy_rate_lost_matches
    FROM
        cricket_stats cs
    JOIN
        cricket_matches_3 cm3 ON cs.ID = cm3.ID
    JOIN
        (
            SELECT
                team_id,
                team_name
            FROM
                teams
        ) teams ON (cm3.team1_id = teams.team_id OR cm3.team2_id = teams.team_id)
    WHERE
        cm3.WinningTeam IS NOT NULL
        AND cm3.Season IN ('2019', '2020/21') -- Filter for specific years
    GROUP BY
        cs.bowler, cm3.Season, teams.team_name
    HAVING
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
        OR COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) > 2
)

SELECT
    bd.Season,
    bd.team,
    SUM(economy_rate_won_matches) AS sum_economy_rate_won_matches,
    SUM(economy_rate_lost_matches) AS sum_economy_rate_lost_matches
FROM
    BowlingData bd
GROUP BY
    bd.Season, bd.team
ORDER BY
    bd.Season, bd.team;







   

CREATE MATERIALIZED view Bowling_dotball_percentage_for_team AS 
SELECT
    cm3.Season,
    teams.team_name AS team,
    cs.bowler AS bowler,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND cs.total_run = 0 THEN 1 ELSE 0 END) AS dot_balls_won_matches,
    SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_won_matches,
    CASE
        WHEN SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) = 0 THEN NULL
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND cs.total_run = 0 THEN 1 ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END), 2)
    END AS dot_ball_percentage_won_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND cs.total_run = 0 THEN 1 ELSE 0 END) AS dot_balls_lost_matches,
    SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_lost_matches,
    CASE
        WHEN SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) = 0 THEN NULL
        ELSE ROUND(SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND cs.total_run = 0 THEN 1 ELSE 0 END) * 100.0 / SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END), 2)
    END AS dot_ball_percentage_lost_matches
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
JOIN
    teams ON (cm3.team1_id = teams.team_id OR cm3.team2_id = teams.team_id)
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    cs.bowler, cm3.Season, teams.team_name
HAVING
    COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
    OR COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) > 2
ORDER BY
    bowler, Season, team;

   CREATE MATERIALIZED view Sum_Bowling_dotball_percentage_for_team AS 
   WITH BowlingData AS (
    SELECT
        cm3.Season,
        teams.team_name AS team,
        cs.bowler AS bowler,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name AND cs.total_run = 0 THEN 1 ELSE 0 END) AS dot_balls_won_matches,
        SUM(CASE WHEN cm3.WinningTeam = teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_won_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name AND cs.total_run = 0 THEN 1 ELSE 0 END) AS dot_balls_lost_matches,
        SUM(CASE WHEN cm3.WinningTeam != teams.team_name THEN 1 ELSE 0 END) AS balls_bowled_lost_matches
    FROM
        cricket_stats cs
    JOIN
        cricket_matches_3 cm3 ON cs.ID = cm3.ID
    JOIN
        (
            SELECT
                team_id,
                team_name
            FROM
                teams
        ) teams ON (cm3.team1_id = teams.team_id OR cm3.team2_id = teams.team_id)
    WHERE
        cm3.WinningTeam IS NOT NULL
        AND cm3.Season IN ('2019', '2020/21') -- Filter for specific years
    GROUP BY
        cs.bowler, cm3.Season, teams.team_name
    HAVING
        COUNT(DISTINCT CASE WHEN cm3.WinningTeam = teams.team_name THEN cm3.ID END) > 2
        OR COUNT(DISTINCT CASE WHEN cm3.WinningTeam != teams.team_name THEN cm3.ID END) > 2
)

SELECT
    bd.Season,
    bd.team,
    SUM(dot_balls_won_matches) AS sum_dot_balls_won_matches,
    SUM(dot_balls_lost_matches) AS sum_dot_balls_lost_matches
FROM
    BowlingData bd
GROUP BY
    bd.Season, bd.team
ORDER BY
    bd.Season, bd.team;


    CREATE MATERIALIZED VIEW cricket_economy_summary AS
SELECT
    cm.season AS Season,
    cs.bowler AS Bowler,
    COUNT(cs.overs) AS TotalBalls,
    SUM(cs.total_run) AS TotalRunsConceded,
    CASE
        WHEN cs.overs <= 6.0 THEN 'Powerplay'
        WHEN cs.overs > 6.0 AND cs.overs <= 15.0 THEN 'Middle Overs'
        ELSE 'Death Overs'
    END AS OverType,
    ROUND(
        CASE
            WHEN cs.overs <= 6.0 THEN SUM(cs.total_run) / (COUNT(cs.overs) / 6.0)
            WHEN cs.overs > 6.0 AND cs.overs <= 15.0 THEN SUM(cs.total_run) / (COUNT(cs.overs) / 9.0)
            ELSE SUM(cs.total_run) / (COUNT(cs.overs) / 4.0)
        END, 2
    ) AS EconomyRate
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm ON cs.ID = cm.ID
WHERE
    cs.overs IS NOT NULL
GROUP BY
    cm.season, cs.bowler, cs.overs,
    CASE
        WHEN cs.overs <= 6.0 THEN 'Powerplay'
        WHEN cs.overs > 6.0 AND cs.overs <= 15.0 THEN 'Middle Overs'
        ELSE 'Death Overs'
    END
HAVING
    COUNT(DISTINCT cm.id) > 3
ORDER BY
    Season ASC, EconomyRate;
   
WITH MatchCounts AS (
    SELECT
        cm3.Season AS season,
        teams.team_name AS team,
        COUNT(DISTINCT cm3.ID) AS total_matches_played
    FROM
        cricket_matches_3 cm3
    JOIN
        teams ON teams.team_name IN (cm3.Team1, cm3.Team2)
    WHERE
        cm3.Season IN ('2019', '2020/21')
    GROUP BY
        cm3.Season, teams.team_name
    HAVING
        COUNT(DISTINCT cm3.ID) > 2
)

SELECT
    mc.season,
    mc.team,
    mc.total_matches_played,
    SUM(CASE WHEN cm3.WinningTeam = mc.team THEN 1 ELSE 0 END) AS matches_won,
    SUM(CASE WHEN cm3.WinningTeam != mc.team THEN 1 ELSE 0 END) AS matches_lost,
    SUM(CASE WHEN cm3.WinningTeam = mc.team THEN cs.batsman_run * 100.0 / mc.total_matches_played ELSE 0 END) AS sum_strike_rate_won_matches,
    SUM(CASE WHEN cm3.WinningTeam != mc.team THEN cs.batsman_run * 100.0 / mc.total_matches_played ELSE 0 END) AS sum_strike_rate_lost_matches
FROM
    cricket_stats cs
JOIN
    cricket_matches_3 cm3 ON cs.ID = cm3.ID
JOIN
    teams ON (cs.BattingTeam = teams.team_name)
JOIN
    MatchCounts mc ON mc.season = cm3.Season AND mc.team = teams.team_name
WHERE
    cm3.WinningTeam IS NOT NULL
GROUP BY
    mc.season, mc.team, mc.total_matches_played
ORDER BY
    mc.season, mc.team;









  

