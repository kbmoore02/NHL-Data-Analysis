---
layout: post
title: "Playoff Contention"
author: Kelsey Moore
description: Mathematically determine Stanley Cup playoff clinching or elimination
image: /assets/images/hockey4-header.jpg
---

# Introduction

In the previous posts, I gathered data about whether a team was eliminated from playoffs or had clinched a spot by simply searching online and compiling my findings in a table. Now, however, I wanted to dive deeper into the math behind how those statuses are determined.

Just as before, rankings are determined first by points (PTS), then regulation wins (RW). When it is impossible for a team to catch up to the 2nd wild card spot, they are eliminated. When it is impossible for a team to be beaten out of the 2nd wild card spot, they have clinched.

## Code

##### Set-up Processes
```python
# Initiate Tables
elim = pd.DataFrame(columns=['Team', 'Date', 'Conf'])
clinch = pd.DataFrame(columns=['Team', 'Date', 'Conf'])
```

##### Determine Eliminations
```python
for d in range(0,len(dates)):
    # Load Data
    bracket = pd.read_excel({insert filepath here}, sheet_name=dates[d]) # NHL Records.xlsx
    bracket_west = bracket.iloc[np.r_[3:8,11:16],].sort_values(by=['PTS','GP','RW','W'], ascending=[False, True, False, False]).reset_index(drop=True)
    bracket_east = bracket.iloc[np.r_[19:24,27:32],].sort_values(by=['PTS','GP','RW','W'], ascending=[False, True, False, False]).reset_index(drop=True)
    for team in bracket_west['Team'].values:
        if team in elim['Team'].values:
            bracket_west = bracket_west[bracket_west['Team'] != team]
    bracket_west = bracket_west.reset_index(drop=True)
    for team in bracket_east['Team'].values:
        if team in elim['Team'].values:
            bracket_east = bracket_east[bracket_east['Team'] != team]
    bracket_east = bracket_east.reset_index(drop=True)    

    # Comparison Values
    elim_comp_pts_east = bracket_east.iloc[1,5]
    elim_comp_rw_east = bracket_east.iloc[1,6]
    elim_comp_w_east = bracket_east.iloc[1,2]
    elim_comp_pts_west = bracket_west.iloc[1,5]
    elim_comp_rw_west = bracket_west.iloc[1,6]
    elim_comp_w_west = bracket_west.iloc[1,2]

    # Check Status
    ## East
    for i in range(2,len(bracket_east)):
        team = bracket_east.iloc[i,0]
        games_left = (82-bracket_east.iloc[i,1])
        max_pts = games_left*2 + bracket_east.iloc[i,5]
        max_rw = games_left + bracket_east.iloc[i,6]
        max_w = games_left + bracket_east.iloc[i,2]
        if max_pts < elim_comp_pts_east:
            elim.loc[len(elim)] = [team, dates[d], 'East']
        elif max_pts == elim_comp_pts_east:
            if max_rw < elim_comp_rw_east:
                elim.loc[len(elim)] = [team, dates[d], 'East']
            elif max_rw == elim_comp_rw_east:
                if max_w < elim_comp_w_east:
                    elim.loc[len(elim)] = [team, dates[d], 'East']
    
    ## West
    for i in range(2,len(bracket_west)):
        team = bracket_west.iloc[i,0]
        games_left = (82-bracket_west.iloc[i,1])
        max_pts = games_left*2 + bracket_west.iloc[i,5]
        max_rw = games_left + bracket_west.iloc[i,6]
        max_w = games_left + bracket_west.iloc[i,2]
        if max_pts < elim_comp_pts_west:
            elim.loc[len(elim)] = [team, dates[d], 'West']
        elif max_pts == elim_comp_pts_west:
            if max_rw < elim_comp_rw_west:
                elim.loc[len(elim)] = [team, dates[d], 'West']
            elif max_rw == elim_comp_rw_west:
                if max_w < elim_comp_w_west:
                    elim.loc[len(elim)] = [team, dates[d], 'West']
```

##### Determine Clinches
```python
for d in range(0, len(dates)):
    # Load Data
    bracket = pd.read_excel({insert filepath here}, sheet_name=dates[d]) # NHL Records.xlsx
    bracket_west = bracket.iloc[np.r_[0:16],].sort_values(by=['PTS','GP','RW','W'], ascending=[False, True, False, False]).reset_index(drop=True)
    bracket_east = bracket.iloc[np.r_[16:32],].sort_values(by=['PTS','GP','RW','W'], ascending=[False, True, False, False]).reset_index(drop=True)
    clinch_teams = clinch['Team'].values
    teams_lw = bracket_west.iloc[0:8,0].values
    teams_w = [team for team in teams_lw if team not in clinch_teams]
    teams_le = bracket_east.iloc[0:8,0].values
    teams_e = [team for team in teams_le if team not in clinch_teams]
    spots = [8,7,6,5,4,3,2,1]

    # Calculate "Magic Numbers"
    ## West
    for team in teams_w:
        # Initialize Values
        beat = 0
        pos = int(bracket_west.loc[bracket_west['Team']==team].index[0])
        pts = bracket_west.iloc[pos,5]
        rw = bracket_west.iloc[pos,6]
        w = bracket_west.iloc[pos,2]

        # Compare Scores
        for i in range(pos+1,len(bracket_west)):
            gl = 82 - bracket_west.iloc[i,1]
            max_pts = gl*2 + bracket_west.iloc[i,5]
            max_rw = gl + bracket_west.iloc[i,6]
            max_w = gl + bracket_west.iloc[i,2]
            if pts < max_pts:
                beat += 1
            elif pts == max_pts:
                if rw < max_rw:
                    beat += 1
                elif rw == max_rw:
                    if w < max_w:
                        beat += 1
        if beat < spots[pos]:
            clinch.loc[len(clinch)] = [team, dates[d], 'West']

    ## East
    for team in teams_e:
        # Initialize Values
        beat = 0
        pos = int(bracket_east.loc[bracket_east['Team']==team].index[0])
        pts = bracket_east.iloc[pos,5]
        rw = bracket_east.iloc[pos,6]
        w = bracket_east.iloc[pos,2]

        # Compare Scores
        for i in range(pos+1,len(bracket_east)):
            gl = 82 - bracket_east.iloc[i,1]
            max_pts = gl*2 + bracket_east.iloc[i,5]
            max_rw = gl + bracket_east.iloc[i,6]
            max_w = gl + bracket_east.iloc[i,2]
            if pts < max_pts:
                beat += 1
            elif pts == max_pts:
                if rw < max_rw:
                    beat += 1
                elif rw == max_rw:
                    if w < max_w:
                        beat += 1
        if beat < spots[pos]:
            clinch.loc[len(clinch)] = [team, dates[d], 'East']
```

Here are the results: 

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/clinched.png" style="width:400px; vertical-align: top;" />
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/eliminated.png" style="width:400px; vertical-align: top;" /> 
</p>
