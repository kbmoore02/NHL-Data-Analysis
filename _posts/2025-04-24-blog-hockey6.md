---
layout: post
title: "Miscellaneous Analysis"
author: Kelsey Moore
description: A few random stats and charts from this season
image: /assets/images/hockey4-header.jpg
---

```python
import requests
import pandas as pd
import numpy as np
from openpyxl import load_workbook
from datetime import datetime, timedelta
import matplotlib.pyplot as plt
```

```python
start_date = datetime(2024, 10, 4)
end_date = datetime(2025, 4, 16)

date_list = [start_date + timedelta(days=x) for x in range((end_date - start_date).days + 1)]
formatted_dates = [date.strftime("%Y-%m-%d") for date in date_list]

comma_separated_dates = ",".join(formatted_dates)
dates = comma_separated_dates.split(",")
```

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

<p float="left">
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/clinched.png" style="width:400px; vertical-align: top;" />
  <img src="https://raw.githubusercontent.com/kbmoore02/NHL-Data-Analysis/main/assets/images/eliminated.png" style="width:400px; vertical-align: top;" /> 
</p>
