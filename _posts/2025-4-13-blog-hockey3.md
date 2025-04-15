---
layout: post
title: "Stanley Cup Playoff Bracket"
author: Kelsey Moore
description: Visualize the bracket for the 2024-25 Stanley Cup playoffs
image: /assets/images/hockey3-header.png
---

# Introduction

Now that we can see what teams qualify for the Stanley Cup playoffs, I wanted to visualize the bracket, and see which teams would be playing against each other.

## Code

##### Load Libraries
```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
from openpyxl import load_workbook
import matplotlib.pyplot as plt
import matplotlib.image as mpimg
from matplotlib.offsetbox import OffsetImage, AnnotationBbox
from PIL import Image
import cv2
import os
from natsort import natsorted
```

##### Set-up Processes
```python
# Get Dates
start_date = datetime(2024, 10, 13)
end_date = datetime(2025, 4, 13)
date_list = [start_date + timedelta(days=x) for x in range((end_date - start_date).days + 1)]

formatted_dates = [f"{date.year}-{date.month}-{date.day}" for date in date_list]
comma_separated_dates = ",".join(formatted_dates)
dates = comma_separated_dates.split(",")

def add_logo(ax, path, x, y, zoom=None, width=None, height=None):
    img = Image.open(path)
    
    # Resize if width/height provided
    if width and height:
        img = img.resize((width, height), Image.ANTIALIAS)
    
    imagebox = OffsetImage(img, zoom=zoom if zoom else 1)
    ab = AnnotationBbox(imagebox, (x, y), frameon=False)
    ax.add_artist(ab)

def draw_bracket(ax, matchups, x_start):
    y_start = 10
    spacing = 3
    for t, (team1, team2) in enumerate(matchups):
        y = y_start - t * spacing
        ax.text(x_start, y+0.25, team1, ha='left', va='center', fontsize=10, fontweight='bold')
        ax.text(x_start, y - 0.75, team2, ha='left', va='center', fontsize=10, fontweight='bold')
        # Clinched
        ax.text(x_start-0.05, y+0.25, bracket.loc[bracket[' ']==team1.split('-')[1].strip(), 'Playoffs'].reset_index(drop=True)[0], ha='left', va='center', fontsize=10, fontweight='bold')
        ax.text(x_start-0.05, y - 0.75, bracket.loc[bracket[' ']==team2.split('-')[1].strip(), 'Playoffs'].reset_index(drop=True)[0], ha='left', va='center', fontsize=10, fontweight='bold')
    # Draw lines for matchup
        ax.plot([x_start, x_start + 0.75], [y-1, y-1], color='gray')
        ax.plot([x_start, x_start + 0.75], [y, y], color='gray')
        ax.plot([x_start + 0.75, x_start + 0.75], [y-1, y], color='gray')
```

##### Create Brackets
```python
for d in range(0,len(dates)):
    # Load Data
    bracket = pd.read_excel({insert filepath here}, sheet_name=dates[d])
    bracket = bracket.iloc[np.r_[1:4,5:8,9:11,20:23,24:27,28:30],:].fillna('').reset_index(drop=True)
    for i in range(0,len(bracket)):
        bracket.iloc[i,0] = bracket.iloc[i,0].replace('1- ','').replace('2- ','').replace('3- ','')

    # Create Tables
    w = bracket.iloc[np.r_[0,3],:].sort_values(['PTS','GP','RW'], ascending=[False, True, False]).reset_index(drop=True)
    e = bracket.iloc[np.r_[8,11],:].sort_values(['PTS','GP','RW'], ascending=[False, True, False]).reset_index(drop=True)
    e1 = e.iloc[0,0]
    e2 = e.iloc[1,0]
    w1 = w.iloc[0,0]
    w2 = w.iloc[1,0]

    # Sort Wildcards
    if bracket.iloc[0,0] == w1:
        ww1 = bracket.iloc[7,0]
    else:
        ww1 = bracket.iloc[6,0]
    if bracket.iloc[3,0] == w1:
        ww2 = bracket.iloc[7,0]
    else:
        ww2 = bracket.iloc[6,0]
    if bracket.iloc[8,0] == e1:
        ew1 = bracket.iloc[15,0]
    else:
        ew1 = bracket.iloc[14,0]

    if bracket.iloc[11,0] == e1:
        ew2 = bracket.iloc[15,0]
    else:
        ew2 = bracket.iloc[14,0]  
    if bracket.iloc[0,0] == w1:
        ww1n = 2
        ww2n = 1
    else: 
        ww1n = 1
        ww2n = 2
    if bracket.iloc[8,0] == e1:
        ew1n = 2
        ew2n = 1
    else:
        ew1n = 1
        ew2n = 2

    # East and West
    west_matchups = [
        (f'C1- {bracket.iloc[0,0]}', f'WC{ww1n}- {ww1}'),
        (f'C2- {bracket.iloc[1,0]}', f'C3- {bracket.iloc[2,0]}'),
        (f'P1- {bracket.iloc[3,0]}', f'WC{ww2n}- {ww2}'),
        (f'P2- {bracket.iloc[4,0]}', f'P3- {bracket.iloc[5,0]}')
    ]
    east_matchups = [
        (f'A1- {bracket.iloc[8,0]}', f'WC{ew1n}- {ew1}'),
        (f'A2- {bracket.iloc[9,0]}', f'A3- {bracket.iloc[10,0]}'),
        (f'M1- {bracket.iloc[11,0]}', f'WC{ew2n}-  {ew2}'),
        (f'M2- {bracket.iloc[12,0]}', f'M3- {bracket.iloc[13,0]}')
    ]

    # Chart Matchups
    teams = [item for tup in west_matchups for item in tup] + [item for tup in east_matchups for item in tup]
    for i in range(0, len(teams)):
        teams[i] = teams[i].split('-')[1].strip()
    logos = []
    for i in range(0, len(teams)):
        logos.append(f"{insert filepath here}{teams[i]}.png")
        
    # Create the figure
    fig, ax = plt.subplots(figsize=(12, 8))
    ax.axis('off')
    draw_bracket(ax, west_matchups, 2)
    draw_bracket(ax, east_matchups, 3)

    # Add logos
    add_logo(ax, logos[0], 2.7, 10.25)
    add_logo(ax, logos[1], 2.7, 9.25)
    add_logo(ax, logos[2], 2.7, 7.25)
    add_logo(ax, logos[3], 2.7, 6.25)
    add_logo(ax, logos[4], 2.7, 4.25)
    add_logo(ax, logos[5], 2.7, 3.25)
    add_logo(ax, logos[6], 2.7, 1.25)
    add_logo(ax, logos[7], 2.7, 0.25)

    add_logo(ax, logos[8], 3.7, 10.25)
    add_logo(ax, logos[9], 3.7, 9.25)
    add_logo(ax, logos[10], 3.7, 7.25)
    add_logo(ax, logos[11], 3.7, 6.25)
    add_logo(ax, logos[12], 3.7, 4.25)
    add_logo(ax, logos[13], 3.7, 3.25)
    add_logo(ax, logos[14], 3.7, 1.25)
    add_logo(ax, logos[15], 3.7, 0.25)

    # Title and Plot
    ax.set_title(f'NHL Playoff Bracket {dates[d]}\n', fontsize=16, loc='center')
    plt.savefig(f"{insert filepath here}{dates[d]}.png")
    plt.close()
```

##### Make Time Lapse
```python
# Settings
image_folder = {insert filepath here}
output_video =f'{image_folder}Bracket Timelapse.mp4'
fps = 2

# Get image files
images = [img for img in os.listdir(image_folder) if img.endswith(('.jpg', '.png', '.jpeg'))]
images = natsorted(images)
if not images:
    raise ValueError("No images found in the directory.")
first_image_path = os.path.join(image_folder, images[0])
frame = cv2.imread(first_image_path)
height, width, _ = frame.shape

# Initialize video writer
fourcc = cv2.VideoWriter_fourcc(*'mp4v')  # Use 'XVID' for .avi
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
```

Here is an example of the bracket:

<img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/sample-bracket.png" alt="" style="width:750px;">

Here is the season progression: [![Watch the video](https://www.youtube.com/watch?v=13hbnnlQAls)](https://www.youtube.com/watch?v=13hbnnlQAls)
