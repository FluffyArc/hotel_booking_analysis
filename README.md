# Investigate Hotel Business using Data Visualization

**Tool:** Python<br>
**Visualization:** Python(Matplotlib/Seaborn)<br>
**Data Source:** Rakamin Academy<br>

**Table of Contents:**
- **Stage 0: Problem Statement**
    - Overview
    - Objective
- **Stage 1: Data Preprocessing**
    - Data Cleansing
- **Stage 2: Data Analysis**
    - Monthly Hotel Booking Analysis Based on Hotel Type.
    - Impact Analysis of Stay Duration on Hotel Bookings Cancelation Rate.
    - Impact Analysis of Lead Time on Hotel Bookings Cancelation Rate.
- **Stage 3: Summary**

# **Stage 0: Problem Statement**

## Overview:
This project aims to identify the customer's behaviour in the context of hotel bookings, and the correlation with the number of cancelation requests been made. The discovered insight will be present in the form of graphs visualization, so the data can be more understandable and more persuasive.

## Objective:
Visualize the insight of three primary metrics:<br>
1. Monthly Hotel Booking Analysis Based on Hotel Type.
2. Impact Analysis of Stay Duration on Hotel Bookings Cancelation Rate.
3. Impact Analysis of Lead Time on Hotel Bookings Cancelation Rate.
---
# **Stage 2: Data Preprocessing**
The dataset that will be used today is a dummy data contains a single file which compares various booking information between two hotels: a City Hotel and a Resort Hotel.

## Identify Missing Value
Before conducting a treatment for missing value variables, we are required to identify which feature contains the missing value. The following line of codes are used to performed the mentioned action.
<details>
<summary>Click here to expand the code</summary>

```python
columns = [x for x in df.columns]
percent=[]
for kolom in columns:
    percent.append(round(df[kolom].isnull().sum()/df[kolom].shape[0]*100, 2))
    
explore = df.describe(percentiles = [], include = 'all').T 
explore['missing'] = len(df) - explore['count'] 
explore['%'] = percent
explore = explore[['missing','%','min','max']]

explore = explore.replace(np.nan, '-', regex=True)
explore
```
</details>