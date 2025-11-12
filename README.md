# ⚽ Manchester United Loss Analysis — European Soccer Database  

## 📘 Project Overview  
This project analyzes match data from the **European Soccer Database**, focusing specifically on **Manchester United’s performance during the 2014/2015 season**.  

The SQL query investigates the question:  
> 🧐 **How badly did Manchester United lose in each match?**  

By comparing home and away matches, the query identifies **all games Manchester United lost** and ranks those defeats by **goal difference**, revealing which matches were the worst defeats of the season.

---

## 🎯 Main Question  
> **How badly did Manchester United lose in each match?**  

The purpose of this analysis is to measure the **severity of Manchester United’s losses** by calculating the goal difference in every match they lost during the 2014/2015 season.

---

## 🗂️ Dataset: European Soccer Database  

📦 **Source:** [Kaggle – European Soccer Database](https://www.kaggle.com/datasets/hugomathien/soccer)  

**Description:**  
The dataset contains detailed data on over 25,000 European football matches, including:  
- Match scores, dates, and seasons  
- Team and player attributes  
- League and country information  
- Advanced team performance metrics  

**Tables Used:**  
- `match` – contains match-level data (date, goals, team IDs, season, etc.)  
- `team` – contains team names and IDs for joining  

---

## 🧩 SQL Query  

```sql
-- Set up the home team CTE
WITH home AS (
  SELECT m.id, t.team_long_name,
    CASE WHEN m.home_goal > m.away_goal THEN 'MU Win'
         WHEN m.home_goal < m.away_goal THEN 'MU Loss' 
         ELSE 'Tie' END AS outcome
  FROM match AS m
  LEFT JOIN team AS t ON m.hometeam_id = t.team_api_id
),

-- Set up the away team CTE
away AS (
  SELECT m.id, t.team_long_name,
    CASE WHEN m.home_goal > m.away_goal THEN 'MU Loss'
         WHEN m.home_goal < m.away_goal THEN 'MU Win' 
         ELSE 'Tie' END AS outcome
  FROM match AS m
  LEFT JOIN team AS t ON m.awayteam_id = t.team_api_id
)

-- Select columns and rank the matches by goal difference
SELECT DISTINCT
    date,
    home.team_long_name AS home_team,
    away.team_long_name AS away_team,
    m.home_goal, m.away_goal,
    RANK() OVER(ORDER BY ABS(home_goal - away_goal) DESC) as match_rank

-- Join the CTEs onto the match table
FROM match AS m
LEFT JOIN home ON m.id = home.id
LEFT JOIN away ON m.id = away.id
WHERE m.season = '2014/2015'
      AND ((home.team_long_name = 'Manchester United' AND home.outcome = 'MU Loss')
      OR (away.team_long_name = 'Manchester United' AND away.outcome = 'MU Loss'));
