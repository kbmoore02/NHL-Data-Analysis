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

## Code

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
    record = pd.read_excel({insert filepath here}, sheet_name=dates[i])
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

# Season Progression

My next step was to combine these layouts to show the progression of the season. I generated a timelapse where you can see how each team's position changed.

## Code

##### Import Libraries
```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from PIL import Image
import time
import os
from natsort import natsorted
import cv2
```

##### Set-up Processes
```python
def bold_rows(s):
    return ['font-weight: bold' if s.name in [0, 4, 8, 19, 23, 27] else '' for _ in s]

# Tables of teams which have clinched a playoff spot or been eliminated from contention
clinch = pd.read_excel({insert filepath here}, sheet_name='Clinched')
elim = pd.read_excel({insert filepath here}, sheet_name='Eliminated')
```

##### Create Tables
```python
for j in range(0, len(dates)):
    stand = pd.read_excel({insert filepath here}, sheet_name=dates[j])
    stand = stand.iloc[:,0:5]
    
    # Playoff Standings
    stand['Playoffs'] = np.repeat('', 32, axis=0)
    dt = datetime.strptime(dates[j], '%Y-%m-%d')
    for team in stand['Team']:
        if team in clinch['Team'].tolist():
            team_date = clinch.loc[clinch['Team'] == team, 'Date'].values[0]
            if dt >= pd.to_datetime(team_date):
                stand.loc[stand['Team']==team, 'Playoffs'] = 'X'
        if team in elim['Team'].tolist():
            team_date = elim.loc[elim['Team'] == team, 'Date'].values[0]
            if dt >= pd.to_datetime(team_date):
                stand.loc[stand['Team']==team, 'Playoffs'] = 'E'
    
    # Positions
    list1 = ['1- ', '2- ', '3- ', '1- ', '2- ', '3- ', '1- ', '2- ', '', '', '', '', '', '', '', '',
             '1- ', '2- ', '3- ', '1- ', '2- ', '3- ', '1- ', '2- ', '', '', '', '', '', '', '', '']
    list2 = stand.iloc[:,1]
    combined = [a + b for a, b in zip(list1, list2)]
    stand['Team'] = combined
    stand = stand.iloc[:, 1:6]
    
    # Central (West)
    section1_heading = pd.DataFrame([['Central', ' ', ' ', ' ', ' ']], columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])
    section1_data = pd.DataFrame(stand.iloc[0:3,:].values.tolist(), columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])

    # Pacific (West)
    section2_heading = pd.DataFrame([['Pacific', ' ', ' ', ' ', ' ']], columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])
    section2_data = pd.DataFrame(stand.iloc[3:6,:].values.tolist(), columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])

    # Wild Card (West)
    section3_heading = pd.DataFrame([['Wild Card W', ' ', ' ', ' ', ' ']], columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])
    section3_data = pd.DataFrame(stand.iloc[6:16,:].values.tolist(), columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])

    # Atlantic (East)
    section4_heading = pd.DataFrame([['Atlantic', ' ', ' ', ' ', ' ']], columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])
    section4_data = pd.DataFrame(stand.iloc[16:19,:].values.tolist(), columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])

    # Metropolitan (East)
    section5_heading = pd.DataFrame([['Metropolitan', ' ', ' ', ' ', ' ']], columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])
    section5_data = pd.DataFrame(stand.iloc[19:22,:].values.tolist(), columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])

    # Wild Card (East)
    section6_heading = pd.DataFrame([['Wild Card E', ' ', ' ', ' ', ' ']], columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])
    section6_data = pd.DataFrame(stand.iloc[22:32,:].values.tolist(), columns=[' ', 'PTS', 'GP', 'RW', 'Playoffs'])

    # Combine all
    df = pd.concat([section1_heading, section1_data, section2_heading, section2_data, section3_heading, section3_data, 
                    section4_heading, section4_data, section5_heading, section5_data, section6_heading, section6_data], ignore_index=True)
    
    # East and West
    west = df.iloc[0:19,].style.apply(bold_rows, axis=1).hide(axis='index').set_properties(subset=[' '], **{'text-align': 'left'}).set_caption(dates[j])
    east = df.iloc[19:,].style.apply(bold_rows, axis=1).hide(axis='index').set_properties(subset=[' '], **{'text-align': 'left'}).set_caption(dates[j])
    
    east.set_table_styles([
        {'selector': 'caption',
         'props': [('caption-side', 'top'), ('text-align', 'left'), ('font-weight', 'bold')]}
    ])
    west.set_table_styles([
        {'selector': 'caption',
         'props': [('caption-side', 'top'), ('text-align', 'left'), ('font-weight', 'bold')]}
    ])
    
    # Create Images
    my_list = [west, east]
    my_index = ['west','east']
    series = pd.Series(my_list, index=my_index)
    
    for b in range(0,len(series)):
        html = series[b].to_html()
        with open("temp_styled.html", "w", encoding='utf-8') as f:
            f.write(html)

        options = Options()
        options.headless = True
        driver = webdriver.Chrome(options=options)
        driver.get("file://" + os.path.abspath("temp_styled.html"))
        time.sleep(2)

        screenshot_path = f"{insert filepath here}{dates[j]}-{series.index[b]}.png"
        driver.save_screenshot(screenshot_path)
        driver.quit()

        img = Image.open(screenshot_path)
        cropped = img.crop((0, 0, img.width - 1050, img.height - 50))
        cropped.save(f"{insert filepath here}{dates[j]}-{series.index[b]}.png")
    
    # Save Table
    styled_df = df.style.apply(bold_rows, axis=1).hide(axis='index').set_properties(subset=[' '], **{'text-align': 'left'})
    excel_file_path = {insert filepath here} 
    new_sheet_name = dates[j]
    export_dataframe_to_excel_sheet(styled_df, excel_file_path, new_sheet_name)
```

##### Create Time Lapse
```python
# Settings
image_folder = {insert pathfile here}
output_video = '{insert pathfile here}\\Standings Timelapse East.mp4'
fps = 2

# Get image files
images = [img for img in os.listdir(image_folder) if img.endswith(('east.png'))]
images = natsorted(images)

if not images:
    raise ValueError("No images found in the directory.")

# Read first image to get size
first_image_path = os.path.join(image_folder, images[0])
frame = cv2.imread(first_image_path)
height, width, _ = frame.shape

# Initialize video writer
fourcc = cv2.VideoWriter_fourcc(*'mp4v')
video = cv2.VideoWriter(output_video, fourcc, fps, (width, height))

# Write each image to video
for image in images:
    image_path = os.path.join(image_folder, image)
    frame = cv2.imread(image_path)
    if frame is None:
        print(f"Warning: Skipping {image_path}")
        continue
    video.write(frame)

video.release()
# Repeat for western conference
```

Here are the results (click to watch): 

Eastern Conference - 
[![Watch the video](https://www.youtube.com/shorts/QX3W60ugI68)](https://www.youtube.com/shorts/QX3W60ugI68)

Western Conference - 
[![Watch the video](https://www.youtube.com/shorts/ZZcYDMrLTpA)](https://www.youtube.com/shorts/ZZcYDMrLTpA)
