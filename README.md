# :soccer:Football Across The Ages:soccer:
<br/><br/>

Football (soccer) isn't one of the most common team sports in the USA. Globally however, due to it's accesibilty and simplicity, the sport has mass appeal and is pervasive in the culture and even identity of many countres. With an estimated [250 million](https://en.wikipedia.org/wiki/Association_football) amateur players globally and a cumilative viewership of 5.4 billion people at the recent 2022 Qatar World Cup, football's popularity dwarves all other sports.

[This kaggle dataset](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017?select=goalscorers.csv) contains international football match results going back to 1872. It lists games played, along with goal scorers, across many different international tournaments. It's a great tool for finding global trends given football's prevailling international popularity.


In this project, my goal is to use this dataset to demonstrate some basic SQL skills by addressing the following:


* Conduct some basic exploratory analaysis of the data and vizualise the reults.
* What parts of the world are most succesful at football?
* When a team hosts a tournament, are they more succesful than when they compete in foreign-hosted tournaments?



<br/><br/>
## The data...


<img width="734" alt="Screen Shot 2023-02-07 at 2 18 37 PM" src="https://user-images.githubusercontent.com/121225842/217379634-39179c7d-761a-4a48-b445-2c557a145b1d.png">
<br/>

## Some interesting fact bites from the data

<br/>

The data was uploaded to Google's Big Query platform where queries were run. All queries for this section can be found in the drop down below the table.


| Total numbers of games played | 44,353 |
|-------------------------------|--------|
| Number of teams               | 316    |
| Number of tournaments         | 141    |
| Average goals per game        | 2.92   |
| Average number of goals scored by players across entire career | 3.07   |
|Percentage of games that ended in a penalty shootout| 23.02% |






<details>
  <summary>See my code</summary>
  
 ```sql
--Number of games played
SELECT 
  COUNT(date)
FROM `football-across-the-ages.football.results`
```

44,353


```sql
--Number of unique teams 
WITH home AS
(
    SELECT DISTINCT(home_team) AS home_teams
    FROM `football-across-the-ages.football.results`
),

away AS
(
    SELECT DISTINCT(away_team) AS away_teams
    FROM `football-across-the-ages.football.results`
),

homeaway AS
(
  SELECT * FROM home
  UNION DISTINCT
  SELECT * FROM away
)

SELECT COUNT(*)
FROM homeaway
```
316

```sql
--Number of unique tournaments
SELECT  
  COUNT(DISTINCT tournament)
FROM `football-across-the-ages.football.results`
```

141

```sql
--Average goals scored per game
SELECT 
  ROUND(AVG(home_score + away_score), 2) AS avg_goals_per_game
 FROM `football-across-the-ages.football.results`
 ```
 2.92
 
 
 ```sql
 --Average goals scored per scorer across all games
 SELECT 
  ROUND(AVG(goals_scored), 2) AS avg_goals_per_scorer
FROM
(SELECT
  scorer,
  COUNT(date) AS goals_scored
FROM `football-across-the-ages.football.goalscorers`
WHERE scorer IS NOT NULL
GROUP BY scorer
ORDER BY COUNT(date) DESC)
```
3.07


```sql
--Percentage of games that ended in a penalty shootout
WITH count_of_draws AS
(SELECT
    COUNT(date)
  FROM
    `football-across-the-ages.football.results`
  WHERE 
    home_score = away_score),

count_of_games AS
  (SELECT
    COUNT(date)
  FROM `football-across-the-ages.football.results`)


SELECT 
  ROUND((SELECT * FROM count_of_draws)/(SELECT * FROM count_of_games)
  * 100, 2) AS perc_games_shootout

FROM `football-across-the-ages.football.results`
```
23.02%
 
```sql
--Better code for percentage of games that ended in a penalty shootout
WITH counted_games AS
(SELECT
    COUNTIF(home_score = away_score) AS count_draw,
   COUNT(date) AS count_all
  FROM
    `football-across-the-ages.football.results`
)

SELECT 
  ROUND(count_draw / count_all
  * 100, 2) AS perc_games_shootout

FROM
  counted_games
```
 
 
```sql
--Countries with the most number of touraments
SELECT 
  country,
  COUNT(DISTINCT tournament) AS distinct_tournaments 
FROM `football-across-the-ages.football.results`
GROUP BY country
ORDER BY distinct_tournaments DESC
LIMIT 10
```
```sql
--Distribution of goals scored by minute.
SELECT 
  CAST(minute AS numeric) AS min_of_game,
  COUNT(CAST(minute AS numeric)) AS num_of_goals

FROM `football-across-the-ages.football.goalscorers`

/* The minute column is a string Type in the table. CASTing the column as numeric fixes this except for one value whose result is the letters 'NA'. I am assuming that when this value is present it is an error and possibly stands for 'N/A'. The below line of code filters this rvalueow out. Once grouped, there is also a suspicious value for the 90th minute as it is much higher than surrounding values. I assumed that values have been rounded somehow as the 90th minute is the last minute of regular full time in football. I also filtered this value out. */
WHERE
  minute <> 'NA'
  AND minute <> '90'
GROUP BY minute
ORDER BY min_of_game DESC
```


</details>







(Note that although there are only 195 countries in the world currently, in this dataset there are 316 unique teams. This is because in many instances across the history of when the data is collected from, countries have been split down into regions (eg. Asturias- autonomous region in Spain, or Brittany- region in France).

|   Countries that have hosted the most tournaments|   |
| --- | --- |
| __Country__       | __distinct_tournaments__ |
| Argentina     | 16                   |
| United States | 14                   |
| Uruguay       | 13                   |
| England       | 12                   |
| France        | 12                   |
| South Africa  | 12                   |
| Brazil        | 12                   |
| Mexico        | 10                   |
| India         | 10                   |
| Chile         | 10                   |


<br/><br/>

> __Distribution of goals scored across 90 minutes of football__


<img width="785" alt="Screen Shot 2023-02-08 at 2 39 54 PM" src="https://user-images.githubusercontent.com/121225842/217667576-70fd6f68-854a-4794-9919-620fd186569c.png">

Goals scored are distributed fairly evenly across the 90 minutes of football with a slight upwards trend towards the latter half. This would be good information to know if you were coordinating the teams defense!


# What parts of the world are most succesful...
  
  


<br /><br />
Scoring goals is an important part of being a succesful team! Here are the all time top scorers:
  <br />
  
<details>
<summary>code</summary>
 
```sql
--top goal scorers
  SELECT
  scorer AS goal_scorer,
  COUNT(date) AS num_goals_scored
FROM `football-across-the-ages.football.goalscorers`
GROUP BY goal_scorer
ORDER BY num_goals_scored DESC
LIMIT 10
```
  
</details>
  
<img width="888" alt="Screen Shot 2023-02-07 at 5 10 53 PM" src="https://user-images.githubusercontent.com/121225842/217403029-13367f06-7c83-4410-9568-b146d5efddcc.png">
<br />
But a better measure of success for a team is __WINNING__ :medal_sports: :partying_face: :confetti_ball: Let's get a list of how many times each team has won.
<br />
<details>
<summary>code</summary>
  
```sql
--CTE identifying whether the home or away team won.
WITH winning_team AS
(
  SELECT
    home_team,
    away_team,
    CASE
      WHEN home_score > away_score
      THEN 'home win'
      WHEN away_score > home_score
      THEN 'away win'
      ELSE 'draw' END AS match_winner
  FROM `football-across-the-ages.football.results`
),

--Counting the number of wins at home for each team.
home_wins AS
(
  SELECT
    home_team AS team,
    COUNTIF(match_winner = 'home win') AS num_of_home_games
  FROM winning_team
  GROUP BY home_team
),

--Counting the number of wins not at home for each team.
away_wins AS
(
  SELECT
    away_team AS team,
    COUNTIF(match_winner = 'away win') AS num_of_away_games
  FROM winning_team
  GROUP BY away_team
)

--Joining the two CTEs together and totalling home and away wins for overall wins.
SELECT
  h.team,
  h.num_of_home_games + a.num_of_away_games AS total_games_won
FROM home_wins AS h
INNER JOIN away_wins AS a
ON h.team = a.team
ORDER BY total_games_won DESC
LIMIT 10
```
</details>
  

<br />
  
  

<img width="650" alt="Screen Shot 2023-02-08 at 4 22 02 PM" src="https://user-images.githubusercontent.com/121225842/217682185-b7ca1fbd-9542-47a0-a66a-09173ebc2e2a.png">
<br />
  
Here's the teams with the top 10 number of wins With Brazil topping the list at 654 wins with England in second place and Germany in third. You can see that these teams are centered around Europe and South America. It should be said that this is a list of wins across ALL tournaments. But not all tournaments are equal; some are higher profile than others. The World Cup for example is the most prestigious of all tournaments. Let compare total wins across all tournaments and total wins in a World Cup.

<details>
<summary>code</summary>

```sql
--CTE identifying whether the home or away team won.
WITH winning_team AS
(
  SELECT
    home_team,
    away_team,
    CASE
      WHEN home_score > away_score
      THEN 'home win'
      WHEN away_score > home_score
      THEN 'away win'
      ELSE 'draw' END AS match_winner
  FROM `football-across-the-ages.football.results`
  WHERE tournament = 'FIFA World Cup'
),

--Counting the number of wins at home for each team.
home_wins AS
(
  SELECT
    home_team AS team,
    COUNTIF(match_winner = 'home win') AS num_of_home_games
  FROM winning_team
  GROUP BY home_team
),

--Counting the number of wins not at home for each team.
away_wins AS
(
  SELECT
    away_team AS team,
    COUNTIF(match_winner = 'away win') AS num_of_away_games
  FROM winning_team
  GROUP BY away_team
)

--Joining the two CTEs together and totalling home and away wins for overall wins.
SELECT
  h.team,
  h.num_of_home_games + a.num_of_away_games AS total_wc_games_won
  RANK() OVER (ORDER BY (h.num_of_home_games + a.num_of_away_games)DESC) AS world_cup_rank
FROM home_wins AS h
INNER JOIN away_wins AS a
ON h.team = a.team
ORDER BY total_wc_games_won DESC
LIMIT 10
```

</details>

|     Team    | World Cup Games Won | Total Games Won |
|:-----------:|:-------------------:|:---------------:|
|    Brazil   |          76         |       654       |
|   Germany   |          68         |       574       |
|  Argentina  |          47         |       551       |
|    Italy    |          45         |       445       |
|    France   |          39         |       442       |
|   England   |          32         |       597       |
|    Spain    |          31         |       426       |
| Netherlands |          30         |       423       |
|   Uruguay   |          25         |       400       |
|   Belgium   |          21         |       355       |
<br />
It looks like generally teams who have won the most games tend to well in the FIFA World Cup tournament. There are however some exceptions: England for example are 2nd in the list of total wins but are only 6th in World Cup wins. Maybe this is an indication that England don't perform as well in high pressure tournaments, like the World up. Performance in a World Cup could be influenced by a number of things, but one key variable is the location the tournament is heald- especially if the tournament takes places in the teams home country. Let's look at this next!

<br />
<br />

# Home Advantage...

To asses a teams performance at home vs. away, it's useful to create a metric. For this first metric, let compare goal scoring in home vs away matches:



<details>
<summary>code</summary>

```sql
WITH home_goals AS
(
  SELECT
    home_team,
    ROUND(AVG(home_score), 2) AS avg_home_goals
  FROM `football-across-the-ages.football.results`
  GROUP BY home_team
),

away_goals AS
(
  SELECT
    away_team,
    ROUND(AVG(away_score), 2) AS avg_away_goals
  FROM `football-across-the-ages.football.results`
  GROUP BY away_team
)

SELECT
  h.home_team AS team,
  ROUND(h.avg_home_goals - a.avg_away_goals, 2) AS home_V_away_goals,
  RANK() OVER (ORDER BY (ROUND(h.avg_home_goals - a.avg_away_goals, 2))DESC) AS rank
FROM
  home_goals AS h
  INNER JOIN away_goals AS a
  ON h.home_team = a.away_team
ORDER BY rank
  ```
</details>

Here we have the metric, __*home_V_away_goals*__. This is a number that represents goal scoring performance in home vs. away games. If the metric is a positive value, it means the team performs better in goal scoring at home games than it does away. If it is a negative value, then the reciprocal is true. Let's take our top ten performing teams from earlier and see what this metric looks like:

| Country     | home_V_away_goals | Overall Rank of Metric |
|-------------|-------------------|------------------------|
| Spain       | 0.72              | 70                     |
| Germany     | 0.49              | 160                    |
| Argentina   | 0.74              | 64                     |
| Belgium     | 0.56              | 123                    |
| Italy       | 0.66              | 91                     |
| France      | 0.55              | 129                    |
| England     | 0.22              | 243                    |
| Uruguay     | 0.35              | 216                    |
| Brazil      | 0.67              | 82                     |
| Netherlands | 0.62              | 101                    |

Notice the rank of these teams. We know that this is a list of the most succesful teams but their rankings on the metric aren't that high. I included the rank to demonstrate that the metric doesn't indicate success, but rather it indicates the consistency of performance across home and away games. Interestingly, __*THE MOST SUCCESFUL TEAMS*__ all __*perform better at home*__ rather than away, as their values for home_V_away_goals are all positive. 

So now we know that teams perform better at home than away, let's see if this has an affect on tournament success.
  
  
percentage of wins at home tournaments vs away.
