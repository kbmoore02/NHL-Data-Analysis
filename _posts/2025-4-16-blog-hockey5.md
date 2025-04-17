---
layout: post
title: "Comparing Playoff Teams"
author: Kelsey Moore
description: Compare various metrics of the 16 teams that made the 2024-25 Stanley Cup playoffs
image: /assets/images/hockey5-header.jpg
---

# Introduction

Now that the Stanley Cup bracket has been finalized, I want to dive into the performance of the teams that made it.

I used the data from [this](https://www.hockey-reference.com/leagues/NHL_2025_games.html) website to perform my analysis.

# Analysis

##### Load Libraries
```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
from openpyxl import load_workbook
import matplotlib.pyplot as plt
```

##### Import/Create Data
```python
schedule = pd.read_excel(f'{insert filepath here}NHL Schedule.xlsx')

toronto_maple_leafs = pd.concat([schedule.loc[schedule['Visitor']=='Toronto Maple Leafs'],schedule.loc[schedule['Home']=='Toronto Maple Leafs']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
tampa_bay_lightning = pd.concat([schedule.loc[schedule['Visitor']=='Tampa Bay Lightning'],schedule.loc[schedule['Home']=='Tampa Bay Lightning']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
florida_panthers = pd.concat([schedule.loc[schedule['Visitor']=='Florida Panthers'],schedule.loc[schedule['Home']=='Florida Panthers']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
washington_capitals = pd.concat([schedule.loc[schedule['Visitor']=='Washington Capitals'],schedule.loc[schedule['Home']=='Washington Capitals']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
carolina_hurricanes = pd.concat([schedule.loc[schedule['Visitor']=='Carolina Hurricanes'],schedule.loc[schedule['Home']=='Carolina Hurricanes']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
new_jersey_devils = pd.concat([schedule.loc[schedule['Visitor']=='New Jersey Devils'],schedule.loc[schedule['Home']=='New Jersey Devils']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
ottawa_senators = pd.concat([schedule.loc[schedule['Visitor']=='Ottawa Senators'],schedule.loc[schedule['Home']=='Ottawa Senators']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
montreal_canadiens = pd.concat([schedule.loc[schedule['Visitor']=='Montreal Canadiens'],schedule.loc[schedule['Home']=='Montreal Canadiens']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
winnipeg_jets = pd.concat([schedule.loc[schedule['Visitor']=='Winnipeg Jets'],schedule.loc[schedule['Home']=='Winnipeg Jets']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
dallas_stars = pd.concat([schedule.loc[schedule['Visitor']=='Dallas Stars'],schedule.loc[schedule['Home']=='Dallas Stars']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
colorado_avalanche = pd.concat([schedule.loc[schedule['Visitor']=='Colorado Avalanche'],schedule.loc[schedule['Home']=='Colorado Avalanche']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
vegas_golden_knights = pd.concat([schedule.loc[schedule['Visitor']=='Vegas Golden Knights'],schedule.loc[schedule['Home']=='Vegas Golden Knights']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
los_angeles_kings = pd.concat([schedule.loc[schedule['Visitor']=='Los Angeles Kings'],schedule.loc[schedule['Home']=='Los Angeles Kings']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
edmonton_oilers = pd.concat([schedule.loc[schedule['Visitor']=='Edmonton Oilers'],schedule.loc[schedule['Home']=='Edmonton Oilers']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
minnesota_wild = pd.concat([schedule.loc[schedule['Visitor']=='Minnesota Wild'],schedule.loc[schedule['Home']=='Minnesota Wild']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
st_louis_blues = pd.concat([schedule.loc[schedule['Visitor']=='St. Louis Blues'],schedule.loc[schedule['Home']=='St. Louis Blues']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
```

```python
playoff_teams = [toronto_maple_leafs, tampa_bay_lightning, florida_panthers, washington_capitals, carolina_hurricanes, new_jersey_devils, 
                 ottawa_senators, montreal_canadiens, winnipeg_jets, dallas_stars, colorado_avalanche, vegas_golden_knights, los_angeles_kings,  
                 edmonton_oilers, minnesota_wild, st_louis_blues]
team_names = ['Toronto Maple Leafs', 'Tampa Bay Lightning', 'Florida Panthers', 'Washington Capitals', 'Carolina Hurricanes', 'New Jersey Devils',
        'Ottawa Senators', 'Montreal Canadiens', 'Winnipeg Jets', 'Dallas Stars', 'Colorado Avalanche', 'Vegas Golden Knights', 'Los Angeles Kings', 
        'Edmonton Oilers', 'Minnesota Wild', 'St. Louis Blues']
```

```python
for t in range(0,len(playoff_teams)):
    team = playoff_teams[t]

    # WLOL
    wlol = []
    for i in range(0,len(team)):
        if team.iloc[i,1]==team_names[t]: # visitor
            if team.iloc[i,2] > team.iloc[i,4]:
                wlol.append('W')
            if team.iloc[i,2] < team.iloc[i,4]:
                if team.iloc[i,5]=='R':
                    wlol.append('L')
                else:
                    wlol.append('OL')
        if team.iloc[i,3]==team_names[t]: # home
            if team.iloc[i,2] < team.iloc[i,4]:
                wlol.append('W')
            if team.iloc[i,2] > team.iloc[i,4]:
                if team.iloc[i,5]=='R':
                    wlol.append('L')
                else:
                    wlol.append('OL')
    team['WLOL'] = wlol

    # PTS
    team['PTS'] = 0
    for i in range(1,len(team)):
        wlol = team.iloc[i,7]
        pts = 0
        if wlol == 'W':
            pts = 2
        if wlol == 'OL':
            pts = 1
        team.iloc[i,8] = team.iloc[i-1,8] + pts
```

## Comparing Points
```python
for t in range(0,8):
    team = playoff_teams[t]
    plt.plot(team['Date'], team['PTS'], label=team_names[t])

plt.xlabel('Date')
plt.ylabel('PTS')
plt.title('Date vs. PTS - Eastern Conference')
plt.grid(True)
plt.tight_layout()
plt.legend(loc='center left', bbox_to_anchor=(1, 0.5))
plt.show()
```

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/pts_east.png" style="width:50%; vertical-align: top;" />
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/pts_west.png" style="width:50%; vertical-align: top;" /> 
</p>

## Win Streak
```python
for team in playoff_teams:
    team['STRK'] = 0
    if team.iloc[0,7] == 'W':
        team.iloc[0,9] = 1
    for i in range(1,len(team)):
        if team.iloc[i,7] == 'W':
            team.iloc[i,9] = 1 + team.iloc[i-1,9]
```

```python
max_streak = []
for team in playoff_teams:
    max_streak.append(max(team['STRK']))
```

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/max_win_streak.png" style="width:700px; vertical-align: top;" />

As we can see, the St. Louis Blues had the longest win streak out of any team. Let's look at their games throughout the season.

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/blues_streak.png" style="width:700px; vertical-align: top;" />

Interestingly, the Blues didn't have an outstanding performance in the first 2/3 of the season. We can see this by the fact that they couldn't win more than 2 games in a row until March. Also, it we look back at the graph of points of the course of the season, we can see that the Blues were trailing behind, but have caught up since they went on this heater.

The Winnipeg Jets had the second highest win streak. Let's look at their games.

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/jets_streak.png" style="width:700px; vertical-align: top;" />

They did a little better than the jets over the season, coming out hot at the beginning and doing well pretty consistently. Doing so well in the beginning and then having a good midseason as well helped propel them forward as the overall points leader this season.

## Lose Streak
```python
for team in playoff_teams:
    team['LOS-STRK'] = 0
    if team.iloc[0,7] != 'W':
        team.iloc[0,10] = 1
    for i in range(1,len(team)):
        if team.iloc[i,7] != 'W':
            team.iloc[i,10] = 1 + team.iloc[i-1,10]
```

```python
max_loss_streak = []
for team in playoff_teams:
    max_loss_streak.append(max(team['LOS-STRK']))
```

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/loss_streak.png" style="width:700px; vertical-align: top;" />

## Goals Scored
```python
max_goals = []
avg_goals = []
avg_home_goals = []
avg_away_goals = []
for t in range(0,len(playoff_teams)):
    team = playoff_teams[t]
    goals = []
    home = []
    away = []
    for i in range(0,len(team)):
        if team.iloc[i,1] == team_names[t]: # visitor
            goals.append(team.iloc[i,2])
            away.append(team.iloc[i,2])
        else: # home
            goals.append(team.iloc[i,4])
            home.append(team.iloc[i,4])
    max_goals.append(max(goals))
    avg_goals.append(np.mean(goals))
    avg_home_goals.append(np.mean(home))
    avg_away_goals.append(np.mean(away))
```

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/max_goals.png" style="width:50%; vertical-align: top;" />
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/avg_goals.png" style="width:50%; vertical-align: top;" /> 
</p>

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/avg_goals_home.png" style="width:50%; vertical-align: top;" />
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/avg_goals_away.png" style="width:50%; vertical-align: top;" /> 
</p>
