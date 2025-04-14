---
layout: post
title: "NHL Playoff Standings"
author: Kelsey Moore
description: Configure Stanley Cup Playoff standings over the course of the 2024-25 NHL season
image: /assets/images/hockey2-header.png
---

# Introduction

Now that I've gathered the data, it's time to organize it. The NHL is divided into two conferences, East and West, each of which are divided into two divisions, Atlantic and Metropolitan, and Pacific and Central, respectively. The top 3 teams in each division go to the playoffs, and out of the remaining teams, the top 2 in each conference go as well, making a total of 16 teams in the playoffs, 8 from each conference.

Rankings are determined first by points (PTS), then by games played (GP), then regulation wins (RW). I sorted the teams by these metrics to determine their standings.

# Code

##### Import Libraries
```python
import pandas as pd
import numpy as np
from openpyxl import load_workbook
from datetime import datetime, timedelta
```

##### Set-up Processes
```python
# Function to Export File
def export_dataframe_to_excel_sheet(df, excel_path, sheet_name):
    try:
        book = load_workbook(excel_path)
        with pd.ExcelWriter(excel_path, engine='openpyxl') as writer:
            writer.book = book
            df.to_excel(writer, sheet_name=sheet_name, index=False)
    except FileNotFoundError:
        print(f"Error: The file '{excel_path}' was not found.")
    except Exception as e:
        print(f"An error occurred: {e}")

# Get Dates
start_date = datetime(2024, 10, 13)
end_date = datetime(2025, 4, 13)

date_list = [start_date + timedelta(days=x) for x in range((end_date - start_date).days + 1)]
formatted_dates = [f"year={date.year}&month={date.month}&day={date.day}" for date in date_list]

comma_separated_dates = ",".join(formatted_dates)
dates = comma_separated_dates.split(",")

for i in range(0,len(dates)):
    dates[i] = dates[i].replace('year=','').replace('&month=','-').replace('&day=','-')
```

##### Create Standings Tables
```python
for i in range(0,len(dates):
    # Load Data
    record = pd.read_excel('C:\\Users\\kelse\\Pictures\\2008 (first camera)\\NHL Records.xlsx', sheet_name=dates[i])
    west = record[record['Conf'].str.contains('West', na=False)]
    east = record[record['Conf'].str.contains('East', na=False)]
    
    comb_w = pd.concat([west.iloc[np.r_[0:3, 8:11], 0:7], west.iloc[np.r_[3:8, 11:16], 0:7].sort_values(by=['PTS', 'GP', 'RW'], ascending=[False, True, False])], ignore_index=True)
    comb_e = pd.concat([east.iloc[np.r_[0:3, 8:11], 0:7], east.iloc[np.r_[3:8, 11:16], 0:7].sort_values(by=['PTS', 'GP', 'RW'], ascending=[False, True, False])], ignore_index=True)

    # Add Position
    stand = pd.concat([comb_w, comb_e], ignore_index=True)
    row_names = ['Central 1','Central 2','Central 3','Pacific 1','Pacific 2','Pacific 3','W Wild Card 1','W Wild Card 2',
                 'WWC 3','WWC 4','WWC 5','WWC 6','WWC 7','WWC 8','WWC 9','WWC 10',
                 'Atlantic 1','Atlantic 2','Atlantic 3','Metropolitan 1','Metropolitan 2','Metropolitan 3','E Wild Card 1','E Wild Card 2',
                 'EWC 3','EWC 4','EWC 5','EWC 6','EWC 7','EWC 8','EWC 9','EWC 10']
    stand.index = row_names
    
    stand['Position'] = stand.index
    stand = stand[['Position', 'Team', 'PTS','GP', 'RW', 'W', 'L', 'OL']]
    
    # Export to Excel
    excel_file_path = {insert filepath here}
    new_sheet_name = dates[i]
    export_dataframe_to_excel_sheet(stand, excel_file_path, new_sheet_name)
```
I now have the playoff standings for every day of the 2024-25 season. Here's an example:

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/sample-layout-stacked.png" alt="" style="width:500px;">
