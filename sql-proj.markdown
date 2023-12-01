---
layout: page
title: SQL basketball stats project
permalink: /sql-project/
---

## Dataset

Recently I wanted to learn how to run SQL queries.  In deciding which data to use, I wanted a dataset where I'd have some level of intuition for what the data points mean.  An obvious choice is sports statistics, and basketball is the sport with which I'm most familiar.  

Instead of scraping data from another website or using an API, I decided this would be a good opportunity to further practice my python skills.  I wrote a program that would simulate a basketball season in an imaginary league with 30 teams, each playing every other team twice.  It would store the box score for each game in a csv file, which I could then store in an SQL database and run queries against.

The python libraries used are as follows:

```python
import numpy as np
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
            'height': (190, 5, 167, 226)
        }, seed=seed)
```

The tuples associated with each dictionary entry have the following meaning: (value, standard deviation, min, max).  I chose the values and standard deviations for all the attributes by hand, picking numbers that seemed appropriate given what I know about the basketball positions.  

There is then a ```Player``` class that, when called, generates a player of a particular position and gives them fixed attributes.

```python
class Player:
    def __init__(self, attribute_means_and_stds, seed=None):
        self.attributes = {}
        for key, (mean, std, MIN, MAX) in attribute_means_and_stds.items():
            self.attributes[key] = truncated_norm(mean, std, MIN, MAX, seed)
        
        # Define overall to be the average of the top 3 attributes, except for height
        # first create new dictionary that removes height
        self.attributes_without_height = {key: value for key, value in self.attributes.items() if key != 'height'}
        self.overall = np.mean(sorted(self.attributes_without_height.values())[-3:])
        
        # Define expected values of a player's stats
        self.pts = 0.1 * (self.attributes['outside_scoring'] + self.attributes['inside_scoring'])
        self.reb = 0.05 * self.attributes['rebounding']
        self.ast = 0.05 * self.attributes['playmaking']
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