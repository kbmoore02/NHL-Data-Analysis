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

## Individual Game Stats

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

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/playoff_stats_period.png" alt="" style="width:400px;">

## Series Stats

##### Load/Create Data - Game Results

```python
cookie_string = "insert cookie string here"
headers = {"User-Agent": "Mozilla/5.0","Cookie": cookie_string}
base_url = "https://stathead.com/hockey/goal_finder.cgi"
params = {"request": 1,"season_end": -1,"grouping": "p","order_by": "date_game","year_min": 1968,"is_playoffs": "Y",
"season_start": 1,"year_max": 2025,"order_by_asc": 1,"match": "goallist","age_max": 100,"offset": 0}
all_rows = []
max_pages = 249

for page in range(max_pages):
    params["offset"] = page * 100
    response = requests.get(base_url, headers=headers, params=params)
    soup = BeautifulSoup(response.text, "html.parser")

    table = soup.find("table", {"id": "stats"})
    if not table:
        print(f"No table found on page {page + 1}.")
        continue

    rows = table.find_all("tr")
    for row in rows:
        cells = [cell.get_text(strip=True) for cell in row.find_all(["th", "td"])]
        if cells:
            all_rows.append(cells)

    time.sleep(1)

with open("insert filepath here", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    for row in all_rows:
        writer.writerow(row)
```

```python
goals = pd.read_csv("insert filepath here")
goals = goals.loc[goals['Date']!='Date',].reset_index(drop=True)
goals.columns = ['Date', 'Tm', 'Loc', 'Opp', 'Result', 'Scorer', 'Assist1',
       'Assist2', 'Goalie', 'Period', 'Time', 'Strength']
goals['Loc'] = goals['Loc'].fillna('')
goals['Year'] = [goals.loc[i,'Date'].split('/')[2] for i in range(0,len(goals))]

for i in range(0,len(goals)):
    team1 = goals.loc[i,'Tm']
    team2 = goals.loc[i,'Opp']
    teams = sorted([team1, team2])
    ser = goals.loc[i,'Year'] + '-' + teams[0] + '-' + teams[1]
    goals.loc[i,'Series'] = ser
```

```python
ser = goals['Series'].unique()
all_series = pd.DataFrame(columns=['Series','Team1Wins','Team2Wins'])
all_series['Series'] = ser
all_series['Team1Wins'] = all_series['Team1Wins'].fillna(0)
all_series['Team2Wins'] = all_series['Team2Wins'].fillna(0)

for i in range(0,len(all_series)):
    dates = goals.loc[goals['Series']==ser[i],'Date'].unique().tolist()
    for d in dates:
        score = pd.DataFrame(
            goals.loc[(goals['Series'] == ser[i]) & (goals['Date'] == d), 'Tm']
            .value_counts()
            .reset_index()
            .rename(columns={'Tm': 'Team','count': 'Score'})
        ).sort_values('Team').reset_index(drop=True)

        team1 = all_series.loc[i,'Series'].split('-')[1]
        team2 = all_series.loc[i,'Series'].split('-')[2]
        team1_score = score.loc[score['Team']==team1,'Score'].reset_index(drop=True)
        team1_score = int(team1_score.values[0]) if not team1_score.empty else 0
        team2_score = score.loc[score['Team']==team2,'Score'].reset_index(drop=True)
        team2_score = int(team2_score.values[0]) if not team2_score.empty else 0

        if team1_score > team2_score:
            all_series.loc[i,'Team1Wins'] += 1
        else:
            all_series.loc[i,'Team2Wins'] += 1

for i in range(0,len(all_series)):
    if all_series.loc[i,'Team1Wins'] == 4:
        team = all_series.loc[i,'Series'].split('-')[1]
        all_series.loc[i,'Winner'] = team
    elif all_series.loc[i,'Team2Wins'] == 4:
        team = all_series.loc[i,'Series'].split('-')[2]
        all_series.loc[i,'Winner'] = team
```

```python
all_games = pd.DataFrame(columns=['Series','Date','Team1','Team2','Winner'])
for s in range(0,len(ser)):
    games = goals.loc[goals['Series']==ser[s],]
    dat = games['Date'].unique().tolist()
    for d in range(0,len(dat)):
        new_row = pd.DataFrame({'Series': [ser[s]], 'Date': [dat[d]]})
        all_games = pd.concat([all_games, new_row], ignore_index=True)

for s in range(0,len(ser)):
    games = goals.loc[goals['Series']==ser[s],]
    dates = games['Date'].unique()
    team1 = ser[s].split('-')[1]
    team2 = ser[s].split('-')[2]
    for d in range(0,len(dates)):
        g = games.loc[games['Date']==dates[d],]
        score_dict = g['Tm'].value_counts().to_dict()
        t1 = score_dict.get(team1, 0)
        t2 = score_dict.get(team2, 0)
        all_games.loc[(all_games['Series']==ser[s]) & (all_games['Date']==dates[d]), 'Team1'] = t1
        all_games.loc[(all_games['Series']==ser[s]) & (all_games['Date']==dates[d]), 'Team2'] = t2

for i in range(0,len(all_games)):
    if all_games.loc[i,'Team1'] > all_games.loc[i,'Team2']:
        team = all_games.loc[i,'Series'].split('-')[1]
        all_games.loc[i,'Winner'] = team
    else:
        team = all_games.loc[i,'Series'].split('-')[2]
        all_games.loc[i,'Winner'] = team
```

```python
all_games['Team1Wins'] = 0
all_games['Team2Wins'] = 0
all_games.loc[0,'Team1Wins'] = 1

for i in range(1,len(all_games)):
    winner = all_games.loc[i,'Winner']
    team1 = all_games.loc[i,'Series'].split('-')[1]
    team2 = all_games.loc[i,'Series'].split('-')[2]
    if all_games.loc[i,'Series'] == all_games.loc[i-1,'Series']:
        if winner == team1:
            all_games.loc[i,'Team1Wins'] = all_games.loc[i-1,'Team1Wins'] + 1
            all_games.loc[i,'Team2Wins'] = all_games.loc[i-1,'Team2Wins']
        else:
            all_games.loc[i,'Team1Wins'] = all_games.loc[i-1,'Team1Wins']
            all_games.loc[i,'Team2Wins'] = all_games.loc[i-1,'Team2Wins'] + 1
    else:
        # new series
        if winner == team1:
            all_games.loc[i,'Team1Wins'] = 1
        else:
            all_games.loc[i,'Team2Wins'] = 1
```

```python
all_games['SeriesWinner'] = ''

for s in range(len(ser)):
    mask = all_games['Series'] == ser[s]
    idx = all_games[mask].index[-1]  # Get the index of the last row in this series
    winner = all_games.loc[mask, 'Winner'].value_counts().idxmax()
    all_games.loc[idx, 'SeriesWinner'] = winner
```

##### Load/Create Data - Series Records

```python
series_winners = all_games[all_games['SeriesWinner'].notna()]['SeriesWinner'].reset_index(drop=True)
ser = all_games['Series'].unique()
```

```python
won_game1 = []
for i in range(0,len(ser)):
    first_game = all_games.loc[all_games['Series']==ser[i],].reset_index(drop=True).iloc[0]
    winner = first_game['Winner']
    won_game1.append(winner)
```

```python
won_first2 = []
for i in range(0,len(ser)):
    series_games = all_games.loc[all_games['Series'] == ser[i]].reset_index(drop=True)
    if len(series_games) < 2:
        won_first2.append(None)
        continue
    second_game = series_games.iloc[1]
    if ((second_game['Team1Wins']==2) | (second_game['Team2Wins']==2)):
        winner = second_game['Winner']
        won_first2.append(winner)
    else:
        won_first2.append(None)
indexes = [i for i, item in enumerate(won_first2) if item is not None]
won2 = [won_first2[i] for i in indexes]
ser2 = [series_winners[i] for i in indexes]
# repeat for won_first3
```

```python
record_21 = []
for i in range(0,len(ser)):
    series_games = all_games.loc[all_games['Series'] == ser[i]].reset_index(drop=True)
    if len(series_games) < 3:
        record_21.append(None)
        continue
    third_game = series_games.iloc[2]

    if third_game['Team1Wins']==2:
        winner = ser[i].split('-')[1]
        record_21.append(winner)
    elif third_game['Team2Wins']==2:
        winner = ser[i].split('-')[2]
        record_21.append(winner)
    else:
        record_21.append(None)
    
indexes = [i for i, item in enumerate(record_21) if item is not None]
won21 = [record_21[i] for i in indexes]
ser21 = [series_winners[i] for i in indexes]
# repeat for record_31 and record_32
```

##### Create Table

```python
stats = pd.DataFrame([
    ['1-0',sum([won_game1[i]==series_winners[i] for i in range(len(won_game1))])/len(won_game1)],
    ['2-0',sum([won2[i]==ser2[i] for i in range(0,len(won2))])/len(won2)],
    ['3-0',sum([won3[i]==ser3[i] for i in range(0,len(won3))])/len(won3)],
    ['2-1',sum([won21[i]==ser21[i] for i in range(len(won21))])/len(won21)],
    ['3-1',sum([won31[i]==ser31[i] for i in range(len(won31))])/len(won31)],
    ['3-2',sum([won32[i]==ser32[i] for i in range(len(won32))])/len(won32)]
], columns = ['Rec.','Win %'])
stats['Win %'] = stats['Win %']*100
stats['Win %'] = stats['Win %'].round(2)
```

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/playoff_stats_games.png" alt="" style="width:250px;">

