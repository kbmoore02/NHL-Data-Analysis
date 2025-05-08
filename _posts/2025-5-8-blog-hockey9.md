---
layout: post
title: "NHL Playoff History"
author: Kelsey Moore
description: Analyze historical Stanley Cup Playoffs
image: /assets/images/hockey9-header.png
---

# Introduction

Let's look at some trends in past NHL Stanley Cup Playoffs, from 1968 to the present. 

I used [this](https://www.hockey-reference.com/playoffs/NHL_2025.html) website for my analysis.

## Code

##### Load/Create Data

```python
shots = pd.read_excel({insert filepath here})
shots = shots.loc[shots['isPlayoffGame']==1,]
goals = shots.loc[shots['event']=='GOAL',].reset_index(drop=True)
goals = goals.loc[:,['game_id','awayTeamCode','awayTeamGoals','event','goalieNameForShot','homeTeamCode','homeTeamGoals','isHomeTeam','period','shooterName','teamCode']]
```

```python
scores = pd.DataFrame(columns=['game_id','homeTeamCode','awayTeamCode','h_period_1','h_period_2','h_period_3','h_ot_1','h_ot_2','a_period_1','a_period_2','a_period_3','a_ot_1','a_ot_2','h_goals','a_goals'])
scores['game_id'] = goals['game_id'].unique().tolist()
scores = scores.fillna(0)
```

```python
for i in range(0,len(goals)):
    game_id = int(goals.loc[i,'game_id'])
    ind = int(scores.loc[scores['game_id']==game_id,].index[0])

    scores.loc[ind,'awayTeamCode'] = goals.loc[i,'awayTeamCode']
    scores.loc[ind,'homeTeamCode'] = goals.loc[i,'homeTeamCode']

    home_base_col = 2
    away_base_col = 7
    
    if goals.loc[i,'teamCode']==scores.loc[ind,'awayTeamCode']:
        # away team scored
        col = int(away_base_col + goals.loc[i,'period'])
        scores.iloc[ind,col] += 1
    else:
        # home team scored
        col = int(home_base_col + goals.loc[i,'period'])
        scores.iloc[ind,col] += 1
```

```python
for i in range(0,len(scores)):
    home_goals = scores.loc[i,'h_period_1'] + scores.loc[i,'h_period_2'] + scores.loc[i,'h_period_3'] + scores.loc[i,'h_ot_1'] + scores.loc[i,'h_ot_2']
    away_goals = scores.loc[i,'a_period_1'] + scores.loc[i,'a_period_2'] + scores.loc[i,'a_period_3'] + scores.loc[i,'a_ot_1'] + scores.loc[i,'a_ot_2']
    scores.loc[i,'h_goals'] = home_goals
    scores.loc[i,'a_goals'] = away_goals

for i in range(0,len(scores)):
    if scores.loc[i,'h_goals'] > scores.loc[i,'a_goals']:
        # home team won
        scores.loc[i,'winningTeamCode'] = scores.loc[i,'homeTeamCode']
        scores.loc[i,'winningTeamLoc'] = 'Home'
    else:
        # away team won
        scores.loc[i,'winningTeamCode'] = scores.loc[i,'awayTeamCode']
        scores.loc[i,'winningTeamLoc'] = 'Away'
```

```python
winning_after_1 = []
for i in range(0,len(scores)):
    if scores.loc[i,'h_period_1'] > scores.loc[i,'a_period_1']:
        winning_after_1.append(scores.loc[i,'homeTeamCode'])
    elif scores.loc[i,'h_period_1'] < scores.loc[i,'a_period_1']:
        winning_after_1.append(scores.loc[i,'awayTeamCode'])
    else:
        winning_after_1.append('Tied')

winning_after_1_check = []
for i in range(0,len(scores)):
    if winning_after_1[i] == 'Tied':
        winning_after_1_check.append('Tied')
    elif winning_after_1[i] == scores.loc[i,'winningTeamCode']:
        winning_after_1_check.append(True)
    else:
        winning_after_1_check.append(False)
```

```python
winning_after_2 = []
for i in range(0,len(scores)):
    if (scores.loc[i,'h_period_1'] + scores.loc[i,'h_period_2']) > (scores.loc[i,'a_period_1'] + scores.loc[i,'a_period_2']):
        winning_after_2.append(scores.loc[i,'homeTeamCode'])
    elif (scores.loc[i,'h_period_1'] + scores.loc[i,'h_period_2']) < (scores.loc[i,'a_period_1'] + scores.loc[i,'a_period_2']):
        winning_after_2.append(scores.loc[i,'awayTeamCode'])
    else:
        winning_after_2.append('Tied')

winning_after_2_check = []
for i in range(0,len(scores)):
    if winning_after_2[i] == 'Tied':
        winning_after_2_check.append('Tied')
    elif winning_after_2[i] == scores.loc[i,'winningTeamCode']:
        winning_after_2_check.append(True)
    else:
        winning_after_2_check.append(False)
```

```python
scored_first = []
for i in range(0,len(scores)):
    sf = goals.loc[goals['game_id']==goals['game_id'].unique().tolist()[i],].reset_index(drop=True).loc[0,'teamCode']
    scored_first.append(sf)

scored_first_check = []
for i in range(0,len(scores)):
    if scored_first[i] == scores.loc[i,'winningTeamCode']:
        scored_first_check.append(True)
    else:
        scored_first_check.append(False)
```

##### Create Table

```python
stats = pd.DataFrame([
    ['Home Ice', (int(scores['winningTeamLoc'].value_counts()['Home'])/len(scores))*100],
    ['Leading After 1st', (int(pd.Series(winning_after_1_check).value_counts()[True])/(int(pd.Series(winning_after_1_check).value_counts()[True]) + int(pd.Series(winning_after_1_check).value_counts()[False])))*100],
    ['Leading After 1st (Ties Inc.)', (int(pd.Series(winning_after_1_check).value_counts()[True])/int(len(scores)))*100],
    ['Leading After 2nd', (int(pd.Series(winning_after_2_check).value_counts()[True])/(int(pd.Series(winning_after_2_check).value_counts()[True]) + int(pd.Series(winning_after_2_check).value_counts()[False])))*100],
    ['Leading After 2nd (Ties Inc.)', (int(pd.Series(winning_after_2_check).value_counts()[True])/int(len(scores)))*100],
    ['Scored First', (int(pd.Series(scored_first_check).value_counts()[True])/int(len(scored_first_check)))*100]
],columns=['Situation','Percentage Won'])
```

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/playoff_stats_period.png" alt="" style="width:900px;">
