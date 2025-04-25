---
layout: post
title: "Carolina Hurricanes Analysis"
author: Kelsey Moore
description: Dive into shots taken by the Carolina Hurricanes in the 2024-25 season
image: /assets/images/hockey7-header.jpg
---

# Introduction

Let's dive into shots taken by the Carolina Hurricanes this season. I will first look at our top point scorers, and see when in the period we score most often. I then want to see if there is a particular area on the ice where we score from most often. I will also look at where shots get past our goalies.

I used the data from [this](https://www.moneypuck.com/data.htm) website to perform my analysis.

# Analysis

## Top Point Scorers

##### Load Libraries
```python
import requests
import pandas as pd
from bs4 import BeautifulSoup as bs
import numpy as np
from openpyxl import load_workbook
from datetime import datetime
from pathlib import Path
from io import StringIO
from datetime import datetime, timedelta
import matplotlib.pyplot as plt
```

##### Load/Create Data

```python
url = 'https://www.hockey-reference.com/teams/CAR/2025_gamelog.html?utm_source=chatgpt.com'
req = requests.get(url)
soup = bs(req.text, 'html.parser')

table = str(soup.find_all('table')[0])
games = pd.read_html(StringIO(table))[0]
games.columns = games.columns.droplevel(0)
games = games.iloc[np.r_[0:9,10:25,26:39,40:55,56:63,64:78,79:88],:].reset_index(drop=True)

dates = games['Date']
for i in range(0,len(dates)):
    dates[i] = dates[i].replace('-','')
```

```python
loc = []
for i in range(0,len(games)):
    if games.iloc[i,3]=='@':
        loc.append(games.iloc[i,4])
    else:
        loc.append('CAR')

links = []
for i in range(0,len(dates)):
    l = f'https://www.hockey-reference.com/boxscores/{dates[i]}0{loc[i]}.html'
    links.append(l)
```
```python
goals = pd.DataFrame(columns=['Time','Men','Goal','Assist','Period'])

for i in range(63,len(links)):
    url = links[i]
    req = requests.get(url)
    soup = bs(req.text, 'html.parser')
    table = str(soup.find_all('table')[0])
    score = pd.read_html(StringIO(table))[0]
    score.columns = ['Time','Team','Men','Goal','Assist']

    for i in range(0,len(score)):
        score.iloc[i,3] = score.iloc[i,3].split('(')[0].strip()

    curr_per = 1
    per = []
    for i in range(0,len(score)):
        if 'Period' in score.iloc[i,0]:
            curr_per += 1
        per.append(curr_per)
    score['Period'] = per

    score = score[~score['Time'].str.contains('Period', case=False, na=False)]
    score['Men'] = score['Men'].fillna('EV')
    score = score.loc[score['Team']=='CAR']
    score = score.iloc[:,np.r_[0,2:6]]

    goals = pd.concat([goals,score])
```

```python
name_counts = goals['Goal'].value_counts().to_dict()

assists = goals['Assist'].str.split(', ', expand=True).fillna('None')
assists = pd.concat([assists.iloc[:,0],assists.iloc[:,1]]).reset_index(drop=True)
assists = [x for x in assists if x != 'None']
name_counts_assists = pd.DataFrame(assists, columns=['Name'])['Name'].value_counts().to_dict()
```

##### Create Plots

```python
bars = plt.bar(name_counts.keys(), name_counts.values())
plt.xlabel('Player')
plt.ylabel('Goals Scored')
plt.title('Goals Scored by Each Player')

plt.xticks(rotation=45, ha='right', va='top')

for bar in bars:
    height = bar.get_height()
    plt.text(bar.get_x() + bar.get_width() / 2, height, f'{int(height)}', ha='center', va='bottom', fontsize=9)

plt.tight_layout()
plt.ylim(0,34)
plt.show()
```

```python
bars = plt.bar(name_counts_assists.keys(), name_counts_assists.values())
plt.xlabel('Player')
plt.ylabel('Assists')
plt.title('Assists by Each Player')

plt.xticks(rotation=45, ha='right', va='top')

for bar in bars:
    height = bar.get_height()
    plt.text(bar.get_x() + bar.get_width() / 2, height, f'{int(height)}', ha='center', va='bottom', fontsize=9)

plt.tight_layout()
plt.ylim(0,48)
# plt.savefig("C:\\Users\\kelse\\Pictures\\2008 (first camera)\\NHL Project\\NHL Graphs\\canes_player_assists.png", bbox_inches='tight')
plt.show()
```

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/canes_player_goals.png" style="width:600px; vertical-align: top;" />

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/canes_player_assists.png" style="width:600px; vertical-align: top;" /> 

## When We Score

##### Import/Create Data

```python
per_time = []
for g in range(0,len(goals)):
    time_str = goals.iloc[g,0]
    pt = [int(t) for t in time_str.split(':')][0] + [int(t) for t in time_str.split(':')][1]/60
    per_time.append(pt)
goals['PerTime'] = per_time

game_time = []
for g in range(0,len(goals)):
    per = goals.iloc[g,4]-1
    time = goals.iloc[g,5]
    gt = time + per*20
    game_time.append(gt)
goals['GameTime'] = game_time
```

##### Create Plots

```python
plt.hist(goals['PerTime'], bins=20, edgecolor='black')
plt.title('Goals Per Period')
plt.xlabel('Time')
plt.ylabel('Frequency')
plt.show()

plt.hist(goals['GameTime'], bins=65, edgecolor='black')
plt.title('Goals Per Game')
plt.xlabel('Time')
plt.ylabel('Frequency')
plt.show()
```

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/canes_per_goals.png" style="width:600px; vertical-align: top;" />

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/canes_game_goals.png" style="width:600px; vertical-align: top;" /> 

## Shots Taken by the Whole Team

##### Load Libraries
```python
import pandas as pd
import numpy as np
from openpyxl import load_workbook
import matplotlib.pyplot as plt
import matplotlib.patches as patches
```

##### Import/Create Data
```python
data = pd.read_csv({insert filepath here})
location = data[['arenaAdjustedXCord','arenaAdjustedXCordABS','arenaAdjustedYCord','event','teamCode','shooterName','isHomeTeam','homeSkatersOnIce','awaySkatersOnIce','goalieNameForShot']].reset_index(drop=True)
for i in range(0,len(location)):
    if location.loc[i,'arenaAdjustedXCord']<0:
        location.loc[i,'reflectedYCord'] = location.loc[i,'arenaAdjustedYCord']*-1
    else:
        location.loc[i,'reflectedYCord'] = location.loc[i,'arenaAdjustedYCord']

canes = location.loc[location['teamCode']=='CAR',]
canes = canes.reset_index(drop=True)
shots = canes.loc[canes['event']!='GOAL',]
goals = canes.loc[canes['event']=='GOAL',]
```

##### Create Plot

```python
plt.figure(figsize=(7.5,6.375))
plt.scatter(shots['arenaAdjustedXCordABS'], shots['reflectedYCord'], color='lightblue', label='Missed Shot')
plt.scatter(goals['arenaAdjustedXCordABS'], goals['reflectedYCord'], color='red', label='Goal')
plt.axvline(x=0, color='black', linestyle='--', linewidth=2)
plt.axvline(x=25, color='blue', linestyle='--', linewidth=2)
plt.axvline(x=89, color='red', linestyle='--', linewidth=2)
rect = patches.Rectangle((89, -3), 3.33, 6, linewidth=2, edgecolor='black', facecolor='black')
plt.gca().add_patch(rect)
plt.legend(loc='center left', bbox_to_anchor=(1, 0.5))
plt.title('Carolina Hurricanes Attempted Shots 2024-25')
plt.show()
```

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/canes_goals.png" style="width:600px; vertical-align: top;" /> 

## Shots Taken by Specific Players

##### Create Data

```python
player = canes.loc[canes['shooterName']=='Seth Jarvis',]
player = player.reset_index(drop=True)
p_shots = player.loc[player['event']!='GOAL',]
p_goals = player.loc[player['event']=='GOAL',]
```

##### Create Plots

```python
plt.figure(figsize=(7.5,6.375))
plt.scatter(p_shots['arenaAdjustedXCordABS'], p_shots['reflectedYCord'], color='lightblue', label='Missed Shot')
plt.scatter(p_goals['arenaAdjustedXCordABS'], p_goals['reflectedYCord'], color='red', label='Goal')
plt.axvline(x=0, color='black', linestyle='--', linewidth=2)
plt.axvline(x=25, color='blue', linestyle='--', linewidth=2)
plt.axvline(x=89, color='red', linestyle='--', linewidth=2)
rect = patches.Rectangle((89, -3), 3.33, 6, linewidth=2, edgecolor='black', facecolor='black')
plt.gca().add_patch(rect)
plt.legend(loc='center left', bbox_to_anchor=(1, 0.5))
plt.title('Seth Jarvis Attempted Shots 2024-25')
plt.show()
```

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/jarvis_goals.png" style="width:400px; vertical-align: top;" />
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/aho_goals.png" style="width:400px; vertical-align: top;" /> 
</p>
<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/rosie_goals.png" style="width:400px; vertical-align: top;" />
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/svech_goals.png" style="width:400px; vertical-align: top;" /> 
</p>
<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/blake_goals.png" style="width:400px; vertical-align: top;" />
