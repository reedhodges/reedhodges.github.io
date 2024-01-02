---
layout: page
title: SQL basketball stats project
permalink: /sql-project/
---

## <ins>Dataset</ins>

Recently I wanted to learn how to run SQL queries.  In deciding which data to use, I wanted a dataset where I'd have some level of intuition for what the data points mean.  An obvious choice is sports statistics, and basketball is the sport with which I'm most familiar.  

I decided this also would be a good opportunity to further practice my python skills, so I wrote a program that would simulate a basketball season in an imaginary league with 30 teams, each playing every other team twice.  It would store the box score for each game in a csv file, which I could then store in an SQL database and run queries against.

I want the simulated basketball games to be as close to reality as possible, so first let's do some exploratory data analysis using a Kaggle dataset of real NBA games.  The dataset I start with has 5 csv files: teams, players, games, games_details, and ranking.  They all contain multiple columns of data, for example:

- teams.csv: founding year, mascot, arena name, owner name, etc.
- players.csv: player name, player ID, etc.
- games.csv: game date, team IDs, stats for each team, etc.
- games_details.csv: box scores for each game
- ranking.csv: team records for each season

My simulated league will be randomly generating the points scored by the players on a team, along with rebounds and assists.  Let's investigate the distributions of points, rebounds, and assists in a game by starting guards, forwards, and centers in the NBA dataset.  I only want the starters because the simulated league will not have players coming off the bench.  I can filter out the bench players in the NBA dataset by noting that only the starters are given a non-empty value for `START_POSITION`.  The appropriate SQL query is below.

```sql
SELECT START_POSITION, PTS, REB, AST
FROM games_details
WHERE START_POSITION IS NOT NULL AND START_POSITION != '';
```

This selects the points, rebounds, and assists columns for the rows of the table where `START_POSITION` is not null and not an empty string.  This script ran in about 25ms on my MySQL server running on an AWS RDP instance, while doing the same filtering on a pandas DataFrame on my machine took about 1s.  We can then export the SQL output to a csv file and plot the results using the `plotly` library.  For example, to create a histogram of the points scored in a single game by players, grouped by position, we have:

```python
import pandas as pd
import plotly.express as px

# read in data from nba_data/starter_stats.csv
starter_stats = pd.read_csv('nba_data/starter_stats.csv')

fig_pts = px.histogram(starter_stats, x='PTS', 
                   color='START_POSITION',  # Column to group by
                   nbins=50,  # Number of bins
                   title='Points scored in a game by NBA starters',
                   labels={'PTS': 'Points', 'START_POSITION': 'Position'},  # Rename axis
                   barmode='group')  # Color of the histogram
```

The histograms for the three stats are below.


![Pts plot](https://raw.githubusercontent.com/reedhodges/bball_league/main/images/fig_pts_by_pos.png)

![Reb plot](https://raw.githubusercontent.com/reedhodges/bball_league/main/images/fig_reb_by_pos.png)

![Ast plot](https://raw.githubusercontent.com/reedhodges/bball_league/main/images/fig_ast_by_pos.png)

We can see that the data likely follows a Poisson distribution, which makes sense, because it is reasonable to approximate that these events (i.e. getting a point, rebound or assist) occur at a constant mean rate and independently of the time since the last event.  

The graphs are consistent with our intuition about the means.  For example, centers tend get more rebounds, and guards tend to get more assists.  The means are:

- Guards: 15.1 pts, 3.8 reb, 4.6 ast
- Forwards: 14.0 pts, 6.2 reb, 2.3 ast
- Centers: 11.6 pts, 8.1 reb, 1.6 ast

We are now ready to write the code for our simulated league.  The python libraries used are as follows:

```python
import numpy as np
import pandas as pd
from scipy.stats import truncnorm
from itertools import combinations
from math import comb, trunc
```

There are five traditional positions on a basketball team.  Each position typically has certain skills they specialize in, e.g. point guards often run the offense and distribute the ball, and centers are usually tall players who score close to the basket and secure rebounds.  I created classes for each position, which contain a dictionary with attributes.  Here is the class for the point guard position:

```python
class pg(Player):
    def __init__(self, seed=None):
        super().__init__({
            'outside_scoring': (80, 15, 0, 100),
            'inside_scoring': (60, 10, 0, 100),
            'defending': (70, 20, 0, 100),
            'athleticism': (80, 10, 0, 100),
            'playmaking': (85, 10, 0, 100),
            'rebounding': (60, 15, 0, 100),
            'intangibles': (70, 5, 0, 100),
            'height': (190, 5, 167, 226),
            'poisson_means': (15.1, 3.8, 4.6) # pts, reb, ast
        }, seed=seed)
```

The tuples associated with all but the last dictionary key have the following meaning: (value, standard deviation, min, max).  I chose the values and standard deviations for all the attributes by hand, picking numbers that seemed appropriate given what I know about the basketball positions.  The last dictionary key has the mean values for the statistics we will use in the Poisson distributions later.

There is then a ```Player``` class that, when called, generates a player of a particular position and gives them fixed attributes.

```python
class Player:
    def __init__(self, attribute_means_and_stds, seed=None):
        self.attributes = {}
        for key, values in attribute_means_and_stds.items():
            # exclude poisson_means from the truncated normal distribution
            if key == 'poisson_means':
                continue
            else:
                mean, std, MIN, MAX = values
                self.attributes[key] = truncated_norm(mean, std, MIN, MAX, seed=seed)

        # Define overall to be the average of the top 3 attributes, except for height and poisson_means
        # first create new dictionary that removes height and poisson_means
        self.attributes_without_height = {key: value for key, value in self.attributes.items() if key != 'height' and key != 'poisson_means'}
        self.overall = np.mean(sorted(self.attributes_without_height.values())[-3:])
        
        # Define expected values of a player's stats.
        # These adjust the Poisson mean according to the player's attributes,
        # i.e. they give a buff or a nerf to the Poisson mean
        self.pts = (attribute_means_and_stds['poisson_means'][0] * self.attributes['outside_scoring'] * self.attributes['inside_scoring']) / (attribute_means_and_stds['outside_scoring'][0] * attribute_means_and_stds['inside_scoring'][0])
        self.reb = (attribute_means_and_stds['poisson_means'][1] * self.attributes['rebounding']) / attribute_means_and_stds['rebounding'][0]
        self.ast = (attribute_means_and_stds['poisson_means'][2] * self.attributes['playmaking']) / attribute_means_and_stds['playmaking'][0]

```

The attributes are fixed by picking a random number according to a truncated normal distribution.  That way the players of the same position on different teams will have some uniqueness to them.  

```python
def truncated_norm(mean, std_dev, MIN, MAX, seed=None):
    # make sure the seed is set so that the same player is generated
    if seed is not None:
        np.random.seed(seed)
    a = (MIN - mean) / std_dev
    b = (MAX - mean) / std_dev
    return truncnorm.rvs(a, b, loc=mean, scale=std_dev)
```

The `Player` class also sets the expected values for the points, rebounds, and assists that player will have for each game; these are calculated by taking the means of corresponding statistics from the real NBA dataset, and adjusting based on the player's attributes.

There is then a ```team``` class that initializes a team with a particular name and five players on the roster.

```python
class team:
    def __init__(self, name, seed=None):
        self.name = name
        self.seed = seed
        # roster
        self.pg = pg(seed=self.seed)
        self.sg = sg(seed=self.seed)
        self.sf = sf(seed=self.seed)
        self.pf = pf(seed=self.seed)
        self.c = c(seed=self.seed)
```

The ```seed``` will be set so that each team has its own seed, and therefore players are generated with the same attributes for a given team.  

The ```game``` class simulates a matchup between two teams.  The points, rebounds, and assists for each player are discrete random variables, described by a Poisson distribution.  The means of the distributions are the means defined in the `Player` class, compounded with either a penalty or a buff to their expected points/rebounds/assists based on the relative difference between their ```overall``` attribute and that of the opponent's player at the same position.  This penalty/buff adds an interesting wrinkle to the stats in the case that a player is in an uneven matchup.  

Each player's stats are then recorded in a NumPy array. 

```python
class game:
    def __init__(self, team1, team2, game_number, stats, team_stats):
        np.random.seed(None)
    
        self.game_number = game_number
        self.team1 = team1
        self.team2 = team2
        self.stats = stats
        self.team_stats = team_stats
        self.positions = ['pg', 'sg', 'sf', 'pf', 'c']

        # Calculate penalties for each position
        self.penalties = {pos: (getattr(team1, pos).overall - getattr(team2, pos).overall) / 100. 
                          for pos in self.positions}
        
        # Calculate box scores for both teams
        self.team1_box = self.calculate_box_score(team1, 1)
        self.team2_box = self.calculate_box_score(team2, -1)

    # Function to calculate box score for a team, and store the stats for each player
    def calculate_box_score(self, team, penalty_multiplier):
        box_scores = []
        for pos in self.positions:
            player = getattr(team, pos)
            penalty = self.penalties[pos] * penalty_multiplier
            # pick a random number from a Poisson distribution with mean = player.pts * (1 + penalty), etc.
            pts = np.ceil(np.random.poisson(player.pts * (1 + penalty)))
            reb = np.ceil(np.random.poisson(player.reb * (1 + penalty)))
            ast = np.ceil(np.random.poisson(player.ast * (1 + penalty)))
            box_scores.append([pts, reb, ast])
            self.stats[team.seed, self.game_number, np.where(np.array(self.positions) == pos)[0][0], :] = [pts, reb, ast]
        return np.array(box_scores)
```

The ```game``` class also keeps track of each team's win-loss-draw record.

```python
class game:
    ...
    def win_loss(self):
            team1_pts = np.sum(self.team1_box[:, 0])
            team2_pts = np.sum(self.team2_box[:, 0])
            if team1_pts > team2_pts:
                self.team_stats[self.team1.seed, 0] += 1
                self.team_stats[self.team2.seed, 1] += 1
            elif team2_pts > team1_pts:
                self.team_stats[self.team1.seed, 1] += 1
                self.team_stats[self.team2.seed, 0] += 1
            else:
                self.team_stats[self.team1.seed, 2] += 1
                self.team_stats[self.team2.seed, 2] += 1
```

Finally, the ```season``` class enumerates all the matchups in the league's season, and simulates all the games, storing every box score in an array.

```python
class season:
    def __init__(self, num_teams):
        self.num_teams = num_teams
        self.num_games = 2*comb(self.num_teams, 2)
        
        # Create teams names, position names, stat names
        self.teams = [team(name=team_nicknames[i],seed=i) for i in range(self.num_teams)]
        self.positions = ['pg', 'sg', 'sf', 'pf', 'c']
        self.stat_names = ['PTS', 'REB', 'AST']

        # Create numpy array with all combinations of two teams
        self.matchups = np.array(list(combinations(self.teams, 2)))
        # redefine it with two of each matchup
        self.matchups = np.concatenate((self.matchups, np.flip(self.matchups, axis=1)))

        # initialize stats for the league
        self.stats = np.zeros((self.num_teams,self.num_games,5,3))

        # initialize win-loss record for each team
        self.team_stats = np.zeros((self.num_teams,3))

        # iterate over each matchup
        for i in range(self.num_games):
            # create a game instance
            g = game(self.matchups[i,0], self.matchups[i,1], i, self.stats, self.team_stats)
            # calculate box scores for both teams
            g.calculate_box_score(self.matchups[i,0], 1)
            g.calculate_box_score(self.matchups[i,1], -1)
            g.win_loss()
```

All that is left is to simulate the full season.  We store the box scores in a Pandas DataFrame for ease of manipulation and conversion to a csv file.

```python
s = season(30)
game_data = s.stats

# create flattened array with team ID, game ID, position ID, PTS, REB, AST columns
entries = []
for team in range(game_data.shape[0]):
    for game in range(game_data.shape[1]):
        for position in range(game_data.shape[2]):
            entries.append((team, game, position, game_data[team, game, position, 0], game_data[team, game, position, 1], game_data[team, game, position, 2]))

# convert to dataframe
df_stats = pd.DataFrame(entries, columns=['team_id', 'game_id', 'pos_id', 'PTS', 'REB', 'AST'])
```

I then make a few changes to this DataFrame.  The for loops stored stats for players whose team was not even playing, so we need to remove those rows.  (I'm sure there is a way to do this without creating unnecessary rows... that is an area for improvement in the code.)  I also add columns for team names, position names, and a player ID.

```python
# remove rows whos stats are all zero
df_stats = df_stats[(df_stats.PTS != 0.0) | (df_stats.REB != 0.0) | (df_stats.AST != 0.0)]
# add a new column for team name
df_stats['team_name'] = df_stats.team_id.apply(lambda x: s.teams[x].name)
# add a new column for position name
df_stats['pos_name'] = df_stats.pos_id.apply(lambda x: s.positions[x])
# add a new column for player ID that is a combination of team ID and position ID, converted to int
df_stats['player_id'] = df_stats.apply(lambda x: int(str(x.team_id) + str(x.pos_id)), axis=1)
```

I also save a csv file that contains the attributes for each player, since these contain useful information that we might be interested in when running SQL queries.  We use the same player ID, so we can make connections between the two datasets once we upload them as tables in SQL.

```python
# now make a dataframe with player attributes from dictionary
# columns: player ID, team ID, team name, position ID, position name, all attributes
entries = []
for team in range(s.num_teams):
    for position in range(len(s.positions)):
        entries.append((int(str(team) + str(position)), team, s.teams[team].name, position, s.positions[position], *s.teams[team].__dict__[s.positions[position]].attributes.values(),s.teams[team].__dict__[s.positions[position]].overall))
# convert to dataframe
df_attributes = pd.DataFrame(entries, columns=['player_id', 'team_id', 'team_name', 'pos_id', 'pos_name', 'outside_scoring', 'inside_scoring', 'defending', 'athleticism', 'playmaking', 'rebounding', 'intangibles', 'height', 'overall'])
# save as csv
df_attributes.to_csv('player_attributes.csv', index=False)
```

All of the data is then ready to upload to a SQL database.

## <ins>Setting up the SQL database</ins>

I used Amazon Web Services' RDS to host the database, and the free software DBeaver as the SQL client.  Uploading the data from the csv files and setting up the tables was straightforward using these.

## <ins>SQL queries</ins>

As an example query, we can look for the instances where a player under 190.5 cm in height scored more than 30 points, and list those players by the number of times they achieved this feat.

```sql
WITH short_thirty_bombs AS
(
	SELECT g.player_id AS Player_ID, g.team_name AS Team_Name, g.pos_name AS Pos_Name, g.PTS AS Points
	FROM `game_stats`  AS g
	INNER JOIN `player_attributes` AS p 
	ON g.player_id = p.player_id
	WHERE g.PTS > 29.0 AND p.height < 190.5
)
SELECT COUNT(Player_ID) AS Num_Thirty_Bombs, Team_Name, Pos_Name
FROM short_thirty_bombs
GROUP BY Team_Name, pos_name 
ORDER BY Num_Thirty_Bombs DESC
```

The output from the query is:

```
6	Raccoons	pg
6	Cyclones	pg
5	Cobras	pg
4	Lions	pg
4	Dragons	pg
4	Knights	pg
4	Cheetahs	pg
3	Giants	pg
3	Tornadoes	pg
3	Wolves	sg
2	Zebras	pg
2	Panthers	sg
2	Bison	pg
2	Kangaroos	sg
2	Sharks	sg
2	Wildcats	sg
1	Aviators	pg
1	Panthers	pg
1	Falcons	pg
1	Tortoises	sg
1	Rhinos	pg
```

The Raccoons and Cyclones point guards were the leaders, with 6 instances each of scoring more than 30 points while being shorter than 190.5 cm.