---
layout: post
title: "Miscellaneous Analysis"
author: Kelsey Moore
description: A few random stats and charts
image: /assets/images/hockey6-header.jpg
---

```python
schedule = pd.read_excel({insert pathfile here})
canes = pd.concat([schedule.loc[schedule['Visitor']=='Carolina Hurricanes'],schedule.loc[schedule['Home']=='Carolina Hurricanes']]).fillna('R').sort_values(by='Date').reset_index(drop=True)
```

```python
canes_goals = []
opp_goals = []
for i in range(0,len(dates)):
    if dates[i] in canes['Date'].astype(str).values:
        if canes.loc[canes['Date']==dates[i],'Visitor'].values[0] == 'Carolina Hurricanes':
            canes_goals.append(int(canes.loc[canes['Date']==dates[i],'VG'].reset_index(drop=True)[0]))
            opp_goals.append(int(canes.loc[canes['Date']==dates[i],'HG'].reset_index(drop=True)[0]))
        else:
            opp_goals.append(int(canes.loc[canes['Date']==dates[i],'VG'].reset_index(drop=True)[0]))
            canes_goals.append(int(canes.loc[canes['Date']==dates[i],'HG'].reset_index(drop=True)[0]))
    else:
        canes_goals.append(0.1)
        opp_goals.append(0.1)
```

```python
diff = []
for i in range(0,len(canes_goals)):
    diff.append(canes_goals[i] - opp_goals[i])
colors = ['green' if d >= 1 else 'red' if d < 0 else 'black' for d in diff]
```

```python
plt.figure(figsize=(25, 6))
bars = plt.bar(dates, diff, color=colors, width=1)
plt.xlabel('Date')
plt.ylabel('Point Diff')
plt.title('Point Differentials - Carolina Hurricanes')

missing_dates = [dates[i] for i in range(len(canes_goals)) if canes_goals[i] == 0.1]
missing_diff = [0] * len(missing_dates)
plt.scatter(missing_dates, missing_diff, color='black', zorder=5, s=10, marker='*')

plt.xticks(ticks=range(0, len(dates), 15), labels=[dates[i] for i in range(0, len(dates), 15)])

plt.tight_layout()
plt.show()
```

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/canes_pts_diff.png" style="width:1000px; vertical-align: top;" />
<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/jets_pts_diff.png" style="width:1000px; vertical-align: top;" />
<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/sharks_pts_diff.png" style="width:1000px; vertical-align: top;" />

```python
data = pd.read_csv({insert filepath here})
location = data[['arenaAdjustedXCord','arenaAdjustedXCordABS','arenaAdjustedYCord','event','teamCode','shooterName','isHomeTeam', 'homeSkatersOnIce','awaySkatersOnIce','goalieNameForShot']].reset_index(drop=True)

for i in range(0,len(location)):
    if location.loc[i,'arenaAdjustedXCord']<0:
        location.loc[i,'reflectedYCord'] = location.loc[i,'arenaAdjustedYCord']*-1
    else:
        location.loc[i,'reflectedYCord'] = location.loc[i,'arenaAdjustedYCord']
```

```python
player = location.loc[location['shooterName']=='Alex Ovechkin',]
player = player.reset_index(drop=True)
p_shots = player.loc[player['event']!='GOAL',]
p_goals = player.loc[player['event']=='GOAL',]
```

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
plt.title('Alex Ovechkin Attempted Shots 2007-2025')
plt.show()
```

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/ovi_goals.png" style="width:600px; vertical-align: top;" />
