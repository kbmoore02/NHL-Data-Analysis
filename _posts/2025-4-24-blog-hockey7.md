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

for i in range(0,len(links)):
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
# repeat for assists
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
# repeat for whole game
```

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/canes_per_goals.png" style="width:600px; vertical-align: top;" />

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/canes_game_goals.png" style="width:600px; vertical-align: top;" /> 

## Shots Taken by the Whole Team

##### Import/Create Data
```python
data = pd.read_csv({insert filepath here})
location = data[['arenaAdjustedXCord','arenaAdjustedXCordABS','arenaAdjustedYCord','event','teamCode','shooterName','isHomeTeam','homeSkatersOnIce','awaySkatersOnIce','goalieNameForShot']].reset_index(drop=True)
for i in range(0,len(location)):
    if location.loc[i,'arenaAdjustedXCord']<0:
        location.loc[i,'reflectedYCord'] = location.loc[i,'arenaAdjustedYCord']*-1
    else:
        location.loc[i,'reflectedYCord'] = location.loc[i,'arenaAdjustedYCord']

canes = location.loc[location['teamCode']=='CAR',].reset_index(drop=True)

miss = canes.loc[canes['event']=='MISS',]
shots = canes.loc[canes['event']=='SHOT',]
goals = canes.loc[canes['event']=='GOAL',]
```

##### Create Plot

```python
plt.figure(figsize=(7.5,6.375))
plt.scatter(miss['arenaAdjustedXCordABS'], miss['reflectedYCord'], color='orchid', label='Miss')
plt.scatter(shots['arenaAdjustedXCordABS'], shots['reflectedYCord'], color='lightblue', label='Shot')
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
player = canes.loc[canes['shooterName']=='Seth Jarvis',].reset_index(drop=True)
p_miss = player.loc[player['event']=='MISS']
p_shots = player.loc[player['event']=='SHOT',]
p_goals = player.loc[player['event']=='GOAL',]
```

##### Create Plots

```python
plt.figure(figsize=(7.5,6.375))
plt.scatter(p_miss['arenaAdjustedXCordABS'], p_miss['reflectedYCord'], color='orchid', label='Miss')
plt.scatter(p_shots['arenaAdjustedXCordABS'], p_shots['reflectedYCord'], color='lightblue', label='Shot')
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

## Goals Scored Against Each Goalie

##### Create Data

```python
goalie = location.loc[location['goalieNameForShot']=='Frederik Andersen',].reset_index(drop=True)
for i in range(0,len(goalie)):
    if goalie.loc[i,'arenaAdjustedXCord']<0:
        goalie.loc[i,'reflectedYCord'] = goalie.loc[i,'arenaAdjustedYCord']*-1
    else:
        goalie.loc[i,'reflectedYCord'] = goalie.loc[i,'arenaAdjustedYCord']
g_shots = goalie.loc[goalie['event']=='SHOT',].reset_index(drop=True)
g_goals = goalie.loc[goalie['event']=='GOAL',].reset_index(drop=True)
```

##### Create Plots

```python
plt.figure(figsize=(10,8.5))
plt.scatter(g_shots['arenaAdjustedXCordABS'], g_shots['reflectedYCord'], color='lightblue', label='Shot')
plt.scatter(g_goals['arenaAdjustedXCordABS'], g_goals['reflectedYCord'], color='red', label='Goal')
plt.axvline(x=0, color='black', linestyle='--', linewidth=2)
plt.axvline(x=25, color='blue', linestyle='--', linewidth=2)
plt.axvline(x=89, color='red', linestyle='--', linewidth=2)
rect = patches.Rectangle((89, -3), 3.33, 6, linewidth=2, edgecolor='black', facecolor='black')
plt.gca().add_patch(rect)
plt.legend(loc='center left', bbox_to_anchor=(1, 0.5))
plt.title('Frederik Andersen Shots Against 2024-2025')
plt.show()
```

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/freddie_goals.png" style="width:400px; vertical-align: top;" />
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/kooch_goals.png" style="width:400px; vertical-align: top;" /> 
</p>

## Shots Taken in Man Advantage Situations

##### Create Data

```python
skatersOnIce = []
oppSkatersOnIce = []
for i in range(0,len(canes)):
    if canes.loc[i,'isHomeTeam'] == 1:
        skatersOnIce.append(canes.loc[i,'homeSkatersOnIce'])
        oppSkatersOnIce.append(canes.loc[i,'awaySkatersOnIce'])
    else:
        skatersOnIce.append(canes.loc[i,'awaySkatersOnIce'])
        oppSkatersOnIce.append(canes.loc[i,'homeSkatersOnIce'])
canes['skatersOnIce'] = skatersOnIce
canes['oppSkatersOnIce'] = oppSkatersOnIce
```

```python
less_men = canes.loc[canes['skatersOnIce']<canes['oppSkatersOnIce'],]

empty_net = less_men.loc[less_men['shotOnEmptyNet']==1,]
shortie = less_men.loc[less_men['shotOnEmptyNet']==0,]

en_shots = empty_net.loc[empty_net['event']=='SHOT',]
en_goals = empty_net.loc[empty_net['event']=='GOAL',]
en_miss = empty_net.loc[empty_net['event']=='MISS',]

shortie_shots = shortie.loc[shortie['event']=='SHOT',]
shortie_goals = shortie.loc[shortie['event']=='GOAL',]
shortie_miss = shortie.loc[shortie['event']=='MISS',]
```

```python
more_men = canes.loc[canes['skatersOnIce']>canes['oppSkatersOnIce'],]

pg = more_men.loc[more_men['skatersOnIce']==6,]
pp = more_men.loc[more_men['skatersOnIce']!=6,]

pg_shots = pg.loc[pg['event']=='SHOT',]
pg_goals = pg.loc[pg['event']=='GOAL',]
pg_miss = pg.loc[pg['event']=='MISS',]

pp_shots = pp.loc[pp['event']=='SHOT',]
pp_goals = pp.loc[pp['event']=='GOAL',]
pp_miss = pp.loc[pp['event']=='MISS',]
```

##### Create Plots

```python
plt.figure(figsize=(7.5, 6.375))
plt.scatter(shortie_miss['arenaAdjustedXCordABS'], shortie_miss['reflectedYCord'], color='orchid', label='Miss')
plt.scatter(shortie_shots['arenaAdjustedXCordABS'], shortie_shots['reflectedYCord'], color='lightblue', label='Shot')
plt.scatter(shortie_goals['arenaAdjustedXCordABS'], shortie_goals['reflectedYCord'], color='red', label='Goal')
plt.axvline(x=0, color='black', linestyle='--', linewidth=2)
plt.axvline(x=25, color='blue', linestyle='--', linewidth=2)
plt.axvline(x=89, color='red', linestyle='--', linewidth=2)
rect = patches.Rectangle((89, -3), 3.33, 6, linewidth=2, edgecolor='black', facecolor='black')
plt.gca().add_patch(rect)
plt.legend(loc='center left', bbox_to_anchor=(1, 0.5))
plt.title('Carolina Hurricanes Short-Handed Attempted Goals 2024-25')
plt.show()
# repeat for Empty Net, Power Play, and Pulled Goalie
```

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/shortie_goals.png" style="width:400px; vertical-align: top;" />
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/empty_net_goals.png" style="width:400px; vertical-align: top;" /> 
</p>

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/pulled_goalie_goals.png" style="width:400px; vertical-align: top;" />
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/powerplay_goals.png" style="width:400px; vertical-align: top;" /> 
</p>

##### Create Tables

```python
ev = canes.loc[canes['skatersOnIce']==canes['oppSkatersOnIce'],]
ev_goals = ev.loc[ev['event']=='GOAL',]
```

```python
xGoalTable = pd.DataFrame([
    ['Short Handed', float(np.mean(shortie['xGoal']))],
    ['Empty Net', float(np.mean(empty_net['xGoal']))],
    ['Power Play', float(np.mean(pp['xGoal']))],
    ['Pulled Goalie', float(np.mean(pg['xGoal']))],
    ['Even Strength', float(np.mean(ev['xGoal']))],
    ['Total', float(np.mean(ev['xGoal']))]
], columns = ['Man Advantage','Average xGoal'])
```

```python
percGoalTable = pd.DataFrame([
    ['Short Handed', len(shortie_goals)/len(shortie)],
    ['Empty Net', len(en_goals)/len(empty_net)],
    ['Power Play', len(pp_goals)/len(pp)],
    ['Pulled Goalie', len(pg_goals)/len(pg)],
    ['Even Strength', len(ev_goals)/len(ev)],
    ['Total', len(goals)/len(canes)]
], columns = ['Man Advantage','Percentage Goals'])
```

```python
all_goals = location.loc[location['event']=='GOAL'].reset_index()

skatersOnIce = []
oppSkatersOnIce = []
for i in range(0,len(all_goals)):
    if all_goals.loc[i,'isHomeTeam'] == 1:
        skatersOnIce.append(all_goals.loc[i,'homeSkatersOnIce'])
        oppSkatersOnIce.append(all_goals.loc[i,'awaySkatersOnIce'])
    else:
        skatersOnIce.append(all_goals.loc[i,'awaySkatersOnIce'])
        oppSkatersOnIce.append(all_goals.loc[i,'homeSkatersOnIce'])
all_goals['skatersOnIce'] = skatersOnIce
all_goals['oppSkatersOnIce'] = oppSkatersOnIce
```

```python
for i in range(0,len(all_goals)):
    if all_goals.loc[i,'skatersOnIce']==all_goals.loc[i,'oppSkatersOnIce']:
        all_goals.loc[i,'men'] = 'ev'
    if all_goals.loc[i,'skatersOnIce']<all_goals.loc[i,'oppSkatersOnIce']: # short handed
        if all_goals.loc[i,'shotOnEmptyNet']==1:
            all_goals.loc[i,'men'] = 'en' # empty net
        else:
            all_goals.loc[i,'men'] = 'sh' # true short handed
    if all_goals.loc[i,'skatersOnIce']>all_goals.loc[i,'oppSkatersOnIce']: # power play
        if all_goals.loc[i,'skatersOnIce']==6:
            all_goals.loc[i,'men'] = 'pg' # pulled goalie
        else:
            all_goals.loc[i,'men'] = 'pp' # true power play

ash = all_goals.loc[all_goals['men']=='sh',]
aen = all_goals.loc[all_goals['men']=='en',]
app = all_goals.loc[all_goals['men']=='pp',]
apg = all_goals.loc[all_goals['men']=='pg',]
aev = all_goals.loc[all_goals['men']=='ev',]

ash = pd.DataFrame(ash['teamCode'].value_counts()).reset_index()
aen = pd.DataFrame(aen['teamCode'].value_counts()).reset_index()
app = pd.DataFrame(app['teamCode'].value_counts()).reset_index()
apg = pd.DataFrame(apg['teamCode'].value_counts()).reset_index()
aev = pd.DataFrame(aev['teamCode'].value_counts()).reset_index()
tot = pd.DataFrame(all_goals['teamCode'].value_counts()).reset_index()
```

```python
goalRankTable = pd.DataFrame([
    ['Short Handed', int(ash.loc[ash['teamCode']=='CAR','count'].reset_index(drop=True)[0])],
    ['Empty Net', int(aen.loc[aen['teamCode']=='CAR','count'].reset_index(drop=True)[0])],
    ['Power Play', int(app.loc[app['teamCode']=='CAR','count'].reset_index(drop=True)[0])],
    ['Pulled Goalie', int(apg.loc[apg['teamCode']=='CAR','count'].reset_index(drop=True)[0])],
    ['Even Strength', int(aev.loc[aev['teamCode']=='CAR','count'].reset_index(drop=True)[0])],
    ['Total', int(tot.loc[tot['teamCode']=='CAR','count'].reset_index(drop=True)[0])]
], columns=['Man Advantage', 'Count'])
```

```python
tab1 = pd.merge(xGoalTable, percGoalTable, on='Man Advantage')
tab2 = pd.merge(tab1, goalRankTable, on='Man Advantage')
```

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/canes_shots_table.png" style="width:400px; vertical-align: top;" />
