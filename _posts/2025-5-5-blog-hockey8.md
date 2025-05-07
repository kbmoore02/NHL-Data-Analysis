---
layout: post
title: "Playoffs Round 1"
author: Kelsey Moore
description: Analyze the 1st round of the 2025 Stanley Cup Playoffs
image: /assets/images/hockey8-header.png
---

# Introduction

Now that the first round of the playoffs are over, lets look at some statistics for these games.

I used [this](https://www.hockey-reference.com/playoffs/NHL_2025.html) website for my analysis.

## Goals Scored

##### Create/Load Data

```python
url = f"https://www.hockey-reference.com/playoffs/NHL_2025.html"
req = requests.get(url)
soup = bs(req.text, 'html.parser')

html_table = str(soup.find_all('table')[12])
stats = pd.read_html(StringIO(html_table))[0]
```

```python
link = [
    'minnesota-wild-vs-vegas-golden-knights-western',
    'colorado-avalanche-vs-dallas-stars-western',
    'st-louis-blues-vs-winnipeg-jets-western',
    'edmonton-oilers-vs-los-angeles-kings-western',
    'carolina-hurricanes-vs-new-jersey-devils-eastern',
    'montreal-canadiens-vs-washington-capitals-eastern',
    'florida-panthers-vs-tampa-bay-lightning-eastern',
    'ottawa-senators-vs-toronto-maple-leafs-eastern'
]

players = pd.DataFrame(columns=['No.', 'Player', 'Pos', 'Age', 'GP', 'G', 'A', 'PTS', '+/-', 'PIM',
       'EV', 'PP', 'SH', 'GW', 'EV', 'PP', 'SH', 'S', 'S%', 'SHFT', 'TOI',
       'ATOI', 'Team'])
goalies = pd.DataFrame(columns=['No.', 'Player', 'Pos', 'Age', 'GP', 'DEC', 'GA', 'SA', 'SV', 'SV%',
       'SO', 'PIM', 'TOI', 'ATOI', 'Team'])
```

```python
for l in range(0,len(link)):
    base_url = f"https://www.hockey-reference.com/playoffs/2025-{link[l]}-first-round.html"
    req = requests.get(base_url)
    soup = bs(req.text, 'html.parser')
    teams = [text.title().strip() for text in link[l].replace('-',' ').replace('western','').replace('eastern','').split('vs')]
    gp = int(team_stats.loc[team_stats['Team']==teams[1],'GP'].reset_index(drop=True)[0])

    # Team 1 Players
    html_table = str(soup.find_all('table')[gp])
    df = pd.read_html(StringIO(html_table))[0]
    df.columns = df.columns.droplevel(0)
    df['Team'] = teams[0]
    df = df.iloc[:-1]
    players = pd.concat([players,df], axis=0)

    # Team 1 Goalies
    html_table = str(soup.find_all('table')[gp+1])
    df = pd.read_html(StringIO(html_table))[0]
    df.columns = df.columns.droplevel(0)
    df['Team'] = teams[0]
    df = df.iloc[:-1]
    goalies = pd.concat([goalies,df], axis=0)

    # Team 2 Players
    html_table = str(soup.find_all('table')[gp+2])
    df = pd.read_html(StringIO(html_table))[0]
    df.columns = df.columns.droplevel(0)
    df['Team'] = teams[1]
    df = df.iloc[:-1]
    players = pd.concat([players,df], axis=0)

    # Team 2 Goalies
    html_table = str(soup.find_all('table')[gp+3])
    df = pd.read_html(StringIO(html_table))[0]
    df.columns = df.columns.droplevel(0)
    df['Team'] = teams[1]
    df = df.iloc[:-1]
    goalies = pd.concat([goalies,df], axis=0)
```

##### Create Plots

```python
for i in range(0,len(team_stats)):
    if int(team_stats.loc[i,'W']) == 4:
        team_stats.loc[i,'Result'] = 'W'
    else:
        team_stats.loc[i,'Result'] = 'L'
color_map = {'W': 'lightblue', 'L': 'lightcoral'}
colors = team_stats['Result'].map(color_map)
```

```python
bars = plt.bar(team_stats['Team'], team_stats['G'], color=colors)
plt.xlabel('Team')
plt.ylabel('Goals Scored')
plt.title('2025 Playoffs Goals Scored')

plt.xticks(rotation=45, ha='right', va='top')

for bar in bars:
    height = bar.get_height()
    plt.text(bar.get_x() + bar.get_width() / 2, height, f'{int(height)}', ha='center', va='bottom', fontsize=9)

plt.tight_layout()
plt.ylim(0,30)
plt.show()
# repeat for goals against and goal differential
```

In the first round, there were 3 teams which scored more goals than their opponents but still lost the series - the Colorado Avalanche, the St. Louis Blues, and the Minnesota Wild. This is because in the games they won, they won by alot, but when they lost, it was by a small margin.

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/playoff_goals.png" style="width:400px; vertical-align: top;" />
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/playoff_goals_against.png" style="width:400px; vertical-align: top;" /> 
</p>
<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/playoff_goal_diff.png" alt="" style="width:400px;">

## Player Stats

##### Create/Load Data

```python
pts_leaders = players.sort_values(by=['PTS','G','A'], ascending=[False, False, False]).head(10).reset_index(drop=True)
```

Mikko Rantanen is the points leader of the playoffs so far, with 5 goals and 7 assists for 12 points. Kyle Connor is tied in points, with 4 goals and 8 assists. 

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/playoff_points.png" alt="" style="width:900px;">

```python
top_goalies = goalies.sort_values(by='SV%', ascending=False).loc[goalies['DEC']!='0-0-0',].head(10).reset_index(drop=True)
```

Frederik Andersen is the top goalie of the playoffs so far, with a save percentage of 93.6% and 1.5 Goals Against Average.

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/playoff_goalies.png" alt="" style="width:900px;">

## Team Stats

I used data from [this](https://stathead.com/hockey/) website to perform my analysis.

##### Create/Load Data

```python
games = pd.read_excel({insert filepath here})
games = games.iloc[:,np.r_[1:3,4:20]]
games.columns = ['Team', 'Date', 'Loc', 'Opp', 'Result', 'G', 'PPG', 'SHG', 'SOG',
       'PIM', 'GA', 'PP GA', 'SH GA', 'SOGA', 'OppPIM', 'PPO', 'PKO', 'Diff']
games['Loc'] = games['Loc'].fillna('Home').replace('@','Away')
games['Res'] = [games['Result'][i].split()[0] for i in range(0,len(games))]
```

```python
stats = pd.DataFrame(list(dict.fromkeys(games['Team'])), columns=['Team'])

stats['PPO'] = list(games.groupby('Team')['PPO'].sum())
stats['PPG'] = list(games.groupby('Team')['PPG'].sum())
stats['PP%'] = round((stats['PPG']/stats['PPO'])*100,2)

stats['PKO'] = list(games.groupby('Team')['PKO'].sum())
stats['PPGA'] = list(games.groupby('Team')['PP GA'].sum())
stats['PK%'] = round(((stats['PKO']-stats['PPGA'])/stats['PKO'])*100,2)
```

```python
home = games.loc[games['Loc']=='Home',].reset_index(drop=True)
away = games.loc[games['Loc']=='Away',].reset_index(drop=True)

all_teams = home['Team'].unique()
stats['HW'] = list(home[home['Res'] == 'W'].groupby('Team')['Res'].count().reindex(all_teams, fill_value=0))
stats['HL'] = list(home[home['Res'] == 'L'].groupby('Team')['Res'].count().reindex(all_teams, fill_value=0))
stats['AW'] = list(away[away['Res'] == 'W'].groupby('Team')['Res'].count().reindex(all_teams, fill_value=0))
stats['AL'] = list(away[away['Res'] == 'L'].groupby('Team')['Res'].count().reindex(all_teams, fill_value=0))

stats['HG'] = list(games.loc[games['Loc']=='Home',].groupby('Team')['Loc'].count())
stats['AG'] = list(games.loc[games['Loc']=='Away',].groupby('Team')['Loc'].count())

stats['H%'] = round((stats['HW']/stats['HG'])*100,2)
stats['A%'] = round((stats['AW']/stats['AG'])*100,2)

stats['Result'] = ['Advanced' if stats.loc[i, 'HW'] + stats.loc[i, 'AW'] == 4 else 'Eliminated' for i in range(len(stats))]
```

The Carolina Hurricanes have the best penalty kill of the playoffs so far, with 100% success. The Los Angeles Kings have the best powerplay, scoring on 40% of their man-advantages. Several teams won all of their home games, and the Florida Panthers won all of their away games. 

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/playoff_stats.png" alt="" style="width:900px;">
