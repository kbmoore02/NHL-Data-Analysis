---
layout: post
title: "NHL Data Scraping"
author: Kelsey Moore
description: Gather data about the 32 NHL teams from the 2024-25 season
image: /assets/images/hockey1-header.png
---

# Introduction

For this project, I am analyzing the 2024-25 Stanley Cup Playoffs picture over the course of the season. To do this, I gathered data from every day of the season and determined what the playoff layout would be if the tournament started that day.

This is the website I used to gather the data: https://www.hockey-reference.com/boxscores/

# Code

##### Load Libraries
```python
import requests
import pandas as pd
from bs4 import BeautifulSoup as bs
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
start_date = datetime(2025, 4, 11)
end_date = datetime(2025, 4, 13)

date_list = [start_date + timedelta(days=x) for x in range((end_date - start_date).days + 1)]
formatted_dates = [f"year={date.year}&month={date.month}&day={date.day}" for date in date_list]

comma_separated_dates = ",".join(formatted_dates)
dates = comma_separated_dates.split(",")
```

##### Scrape Webpage

```python
for i in range(0, len(dates)):
    # Read Website
    url = f"https://www.hockey-reference.com/boxscores/?{dates[i]}"
    req = requests.get(url)
    soup = bs(req.text, 'html.parser')
    table = soup.find_all('table')[-1]
    rows = table.find_all('tr')

    # Western Conference
    html_table_w = str(soup.find_all('table')[-1])
    df_w = pd.read_html(html_table_w)[0]
    df_w = df_w.iloc[np.r_[1:9, 10:18],np.r_[0:6, 9:10]]
    df_w = df_w.rename(columns={'Unnamed: 0': 'Team'})
    df_w['Team'] = df_w['Team'].str.replace('*', '', regex=False)
    df_w['Div'] = np.repeat(np.array(["Central", "Pacific"]), [8, 8], axis=0)
    df_w['Conf'] = np.repeat('West', 16, axis=0)

    # Eastern Conference
    html_table_e = str(soup.find_all('table')[-2])
    df_e = pd.read_html(html_table_e)[0]
    df_e = df_e.iloc[np.r_[1:9, 10:18],np.r_[0:6, 9:10]]
    df_e = df_e.rename(columns={'Unnamed: 0': 'Team'})
    df_e['Team'] = df_e['Team'].str.replace('*', '', regex=False)
    df_e['Div'] = np.repeat(np.array(["Atlantic", "Metropolitan"]), [8, 8], axis=0)
    df_e['Conf'] = np.repeat('East', 16, axis=0)

    # Combine Tables
    df = pd.concat([df_w,df_e], ignore_index=True)

    # Export
    excel_file_path = {insert filepath here} 
    new_sheet_name = dates[i].replace('year=','').replace('&month=','-').replace('&day=','-')
    export_dataframe_to_excel_sheet(df, excel_file_path, new_sheet_name)
```

I now have all daily standings saved to an Excel spreadsheet, each on their own page. Here's a sample:

<img src="https://raw.githubusercontent.com/kbmoore02/Stat486-Final-Blog/main/assets/images/sample-standings-stacked.png" alt="" style="width:500px;">
