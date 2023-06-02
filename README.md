# Investigate Hotel Business using Data Visualization

**Tool:** Python<br>
**Visualization:** Python(Matplotlib/Seaborn)<br>
**Data Source:** Rakamin Academy<br>

**Table of Contents:**
- [**Stage 0: Problem Statement**](#stage-0-problem-statement)
    - [Overview](#overview)
    - [Objective](#objective)
- [**Stage 1: Data Preprocessing**](#stage-1-data-preprocessing)
    - [Identify Missing Values](#identify-missing-value)
    - [Handling Missing Values](#handling-missing-value)
    - [Removing the Unnecessary Features](#removing-the-unnecessary-features)
- [**Stage 2: Data Analysis**](#stage-2-data-analysis)
    - [Average Number of Bookings per Month for Every Hotel Category](#average-number-of-bookings-per-month-for-every-hotel-category)
    - [Ratio of Cancelation Bookings with the Number of Stays](#ratio-of-cancelation-bookings-with-the-number-of-stays)
    - [Impact Analysis of Lead Time on Hotel Bookings Cancelation Rate](#impact-analysis-of-lead-time-on-hotel-bookings-cancelation-rate)
- [**Stage 3: Summary**](#stage-3-summary)

---

# **Stage 0: Problem Statement**

## Overview:
This project aims to identify the customer's behaviour in the context of hotel bookings, and the correlation with the number of cancelation requests been made. The discovered insight will be present in the form of graphs visualization, so the data can be more understandable and more persuasive.

## Objective:
Visualize the insight of three primary metrics:<br>
1. Monthly Hotel Booking Analysis Based on Hotel Type.
2. Impact Analysis of Stay Duration on Hotel Bookings Cancelation Rate.
3. Impact Analysis of Lead Time on Hotel Bookings Cancelation Rate.
---
# **Stage 1: Data Preprocessing**
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

In order to understand the data more easily, the features are categorized into two groups **numericals** and **categoricals**.

<details>
<summary>Click here to expand the code</summary>

```python
num_dtypes = ['int32','int64','float64']
cat_dtypes = ['object']

num_df = df.select_dtypes(include=num_dtypes)
cat_df = df.select_dtypes(include=cat_dtypes)

numericals = num_df.columns
categoricals = cat_df.columns
```
</details>

## Handling Missing Value
Both numerical and categorical features will be treat with different method of imputation. **The missing value of numerical** features will be imputed by predicting based on the nearest neighbour using **KNN_Imputer**. Many studies shows that imputing value with a prediction method is more reasonable than using descriptive statistics (mean/median/mode).

<details>
<summary>Click here to expand the code</summary>

```python
from sklearn.impute import KNNImputer

df_num = df[numericals].copy()
impute_KNN = KNNImputer(n_neighbors=2)

df_num_imputed = pd.DataFrame(impute_KNN.fit_transform(df_num), columns=df_num.columns)
```
</details>

As for **the missing value of categorical** features will be treated by using **SimpleImputer** library, which allow us to add addition category for the missing value. In this project, the missing value for the categorical features will be categorized as **missing** category.

<details>
<summary>Click here to expand the code</summary>

```python
from sklearn.impute import SimpleImputer

df_cats = df[categoricals].copy()
imputer = SimpleImputer(strategy='constant', fill_value='missing')

df_cats_imputed = pd.DataFrame(imputer.fit_transform(df_cats), columns=df_cats.columns)
```
</details>

## Removing The Unnecessary Features
Feature ```adults```, ```children```, and ```babies``` will be merged into new variable as **```num_cust```**.

---

# **Stage 2: Data Analysis**
## Average Number of Bookings per Month for Every Hotel Category
First, the new dataframe will be created based on the require features needed for the analysis. The dataframe will be filtered using the ```groupby()``` function based on ```arrival_date_month```, ```hotel```, and ```arrival_date_year``` features (since the data contains multiple years) before it ```counts``` the value from the ```agent``` feature. The data then sorted based on the ```arrival_date_month``` in order to make the data more readable.

<details>
<summary>Click here to expand the code</summary>

```python
booking_trends = df_new.groupby(['hotel', 'arrival_date_month', 'arrival_date_year']).agg(
    hotel_count = ('agent','count')
).reset_index()

booking_trends_avg = booking_trends.groupby(['hotel', 'arrival_date_month']).agg({
    'hotel_count':'mean'
}).apply(lambda x: round(x,1)). reset_index()

booking_trends_avg = booking_trends_avg.sort_values('arrival_date_month', key=lambda x: x.apply(lambda x:month_dict[x]))
booking_trends_avg
```
</details>

![task2_result1](https://github.com/FluffyArc/hotel_booking_analysis/assets/40890491/6cecc472-8892-4e5e-b679-0f5c687bdd17)

Based on the new dataframe, the visualization is made using ```Seaborn Plot``` library.

<details>
<summary>Click here to expand the code</summary>

```python
plt.figure(figsize=(15,7))
fig_2 = sns.barplot(data=booking_trends_avg, x='arrival_date_month', y='hotel_count', hue='hotel', palette='deep')

plt.title('Average Number of Bookings per Month\nfor Every Hotel Category',
         fontsize=18, color='black', weight='bold', pad=50)
plt.text(x=-1, y=5450, s='The figure shows that the number of hotel bookings started to rise gradually ' 
         'from the beginning of the year and reached its peaked\nin July at 4116 bookings for the City Hotel. '
         'In addition, the number of bookings for City Hotel always becomes a favourite among the customers every month.',
        fontsize=10, style='italic')

for i in range(2):
    fig_2.bar_label(fig_2.containers[i])

plt.xlabel('Months')
plt.ylabel('Total Bookings')

plt.axvline(4.5, ls='--', color='green')
plt.axvline(6.5, ls='--', color='green')
plt.stackplot(np.arange(4.5,7.5), [[5000]], color='limegreen', alpha=0.3)
plt.text(x=4.85, y=4750, s='Peak Season', fontsize=14, color='green', va='center')

plt.axvline(10.5, ls='--', color='red')
plt.axvline(11.5, ls='--', color='red')
plt.stackplot(np.arange(10.5,12.5), [[5000]], color='#ed6a5e', alpha=0.3)
plt.text(x=10.6, y=4750, s='X-mas\nSeason', fontsize=14, color='red', va='center')
```
</details>

![task2](https://github.com/FluffyArc/hotel_booking_analysis/assets/40890491/a756d41b-462d-4d1e-8e90-03fd55d07a0e)

Based on the given figure above, **City Hotel** is the most favourite hotel type compared to the Resort Hotel every month. The number of bookings for City hotels **peaked at 4116** during the **July holiday season**. 
**The number of bookings also increased significantly during the Christmas Season**, which stood at **3802 bookings** compared to the other season (excluding the holiday season)

---
## Ratio of Cancelation Bookings with the Number of Stays
To retrieve the number of stay for every customer, we need to combine the ```stay_in_weekend_nights``` and the ```stay_in_weekday_nights``` features. Since the number of stays ranged from 0-69 nights, the data will be grouped into the following category:
 - 1 Week (number of stay from 1 - 7 days)
 - 2 Weeks (number of stay from 8 - 14 days)
 - 3 Weeks (number of stay from 15 - 21 days)
 - 4 Weeks (number of stay from 22 - 28 days)
 - \>5 Weeks (number of stay > 28).

<details>
<summary>Click here to expand the code</summary>

```python
df_new['weeks'] = df_new.apply(
    lambda x: '1 Week' if x['num_stays'] <= 7 else
    '2 Weeks' if x['num_stays'] <=14 else
    '3 Weeks' if x['num_stays'] <=21 else
    '4 Weeks' if x['num_stays'] <= 28 else
    '>5 Weeks'
    , axis=1)
df_new['num_stays'].value_counts()
```
</details>


 As for the customers with the number of stay 0 will be droped.<br><br>

 To calculate the ratio between the cancelation bookings and the number of stays, we need to create a new dataframe and ```grouped``` it based on the ```hotel```, ```weeks```, and the ```is_canceled``` features. And since we only want the non-canceled reservation, we need to make sure that our dataframe only contain the data with ```is_canceled``` value is **```1```**.

 <details>
 <summary>Click here to expand the code</summary>

 ```python
stay_to_cancel = df_new.groupby(['hotel','weeks','is_canceled']).agg(
    num_bookings = ('agent','count'),
).sort_values('weeks', key=lambda x: x.apply(lambda x:week_dict[x])).reset_index()

total_bookings = stay_to_cancel.groupby(['hotel','weeks']).agg({
    'num_bookings':'sum'
}).sort_values('weeks', key=lambda x: x.apply(lambda x:week_dict[x])).reset_index()

stay_to_cancel = stay_to_cancel.merge(total_bookings, on=['hotel','weeks'])
stay_to_cancel_true = stay_to_cancel[stay_to_cancel.is_canceled == 1]

stay_to_cancel_true['percentage'] = stay_to_cancel_true.apply(lambda x: round((x['num_bookings_x']/x['num_bookings_y'])*100), axis=1)
stay_to_cancel_true
 ```
 </details>

![task3_result1](https://github.com/FluffyArc/hotel_booking_analysis/assets/40890491/fa842a31-639f-4825-b7dd-bdc8336db1e0)

Based on the new dataframe, the visualization is made using ```Seaborn Plot``` library.

<details>
<summary>Click here to expand the code</summary>

```python
plt.figure(figsize=(12,7))
fig_3 = sns.barplot(data=stay_to_cancel_true, x='weeks', y='percentage', hue='hotel', palette='mako')

sns.regplot(
            data=stay_to_cancel_true[stay_to_cancel_true.hotel == 'City Hotel'], 
            x=np.arange(0, len(stay_to_cancel_true[stay_to_cancel_true.hotel == 'City Hotel'])), 
            y='percentage',
            scatter=False,
            truncate=False,
            label='City Hotel'
           )

sns.regplot(
            data=stay_to_cancel_true[stay_to_cancel_true.hotel == 'Resort Hotel'], 
            x=np.arange(0, len(stay_to_cancel_true[stay_to_cancel_true.hotel == 'Resort Hotel'])), 
            y='percentage',
            scatter=False,
            truncate=False,
            label='Resort Hotel'
           )

plt.xlabel('Weeks')
plt.ylabel('Cancelation Rate(%)')

plt.title('Ratio of Cancelation Bookings With The Number of Stays',
         fontsize=18, color='black', weight='bold', pad=60)
plt.text(x=-0.6, y=118, s='The figure shows the correlation between the cancelation requests and the number of staying.\n'
         'It can be seen that, there is a positive correlation between the two mentioned variables.\n' 
         'The longer the customers will stay at the hotel, the more probability of cancelation requests will be made by the customers'
         ,
        fontsize=10, style='italic')

plt.legend(fontsize=10)

for i in range(2):
    fig_3.bar_label(fig_3.containers[i])
```
</details>

![task3](https://github.com/FluffyArc/hotel_booking_analysis/assets/40890491/5ca655d9-488a-4829-8f38-d4068920a070)

The given figure describes the correlation between the number of stay duration and the cancelation rates. It can be seen that the mentioned variables have a **positive correlation** for the **City Hotel**, which means **the longer the customers decide to stay at the hotel, the probability that the customers will cancel the booking become higher too**. On the other hand, the correlation for the **Resort Hotel** almost remains flat in the given weeks.<br><br>


Since the City Hotel is the most favorite place to stay among customers, it makes sense if the highest cancelation rates come from the City Hotel, too (**accounted for 77 cancelation requests for a stay duration of more than equal five weeks**.)

## Impact Analysis of Lead Time on Hotel Bookings Cancelation Rate.

Firstly, we need to extract the monthly information of the customer ```lead_time``` by running the following command.

```python
df_new['lead_months'] = df_new['lead_time'].apply(lambda x: (x-(x//365)*365)//30)
```

The next step is to create new filtered dataframe based on the ```hotel```, ```is_canceled```, and ```lead_months``` features. Similar to the previous one, we need to make sure that our dataframe **only contains** values where ```is_canceled``` is **```1```**.

<details>
<summary>Click here to expand the code</summary>

```python
lead_to_cancel = df_new.groupby(['hotel','is_canceled','lead_months']).agg(
    num_books = ('agent','count')
).reset_index()

total_books = lead_to_cancel.groupby(['hotel','lead_months']).agg(
    total_books = ('num_books','sum')
).reset_index()

df_merge = lead_to_cancel.merge(total_books, on=['hotel','lead_months'])
lead_to_cancel = df_merge[df_merge.is_canceled == 1]
lead_to_cancel['percentage'] = lead_to_cancel.apply(lambda x: round((x['num_books']/x['total_books'])*100, 2), axis=1)
```
</details>

![task4_result1](https://github.com/FluffyArc/hotel_booking_analysis/assets/40890491/91e1df20-ff7f-4985-bbec-5677f12a049d)

Based on the new dataframe, the visualization is made using ```Seaborn Plot``` library.

<details>
<summary>Click here to expand the code</summary>

```python
plt.figure(figsize=(15,5))
fig_4 = sns.barplot(data=lead_to_cancel, x='lead_months', y='percentage', hue='hotel', palette='crest')

sns.regplot(data = lead_to_cancel[lead_to_cancel.hotel == 'City Hotel'],
            x = np.arange(0, len(lead_to_cancel[lead_to_cancel.hotel == 'City Hotel'])),
            y='percentage',
            scatter=False,
            truncate=False,
            label='City Hotel'
           )

sns.regplot(data = lead_to_cancel[lead_to_cancel.hotel == 'Resort Hotel'],
            x = np.arange(0, len(lead_to_cancel[lead_to_cancel.hotel == 'Resort Hotel'])),
            y='percentage',
            scatter=False,
            truncate=False,
            label='Resort Hotel'
           )

for i in range(2):
    fig_4.bar_label(fig_4.containers[i])

plt.xlabel('Lead Time (Month)')
plt.ylabel('Cancelation Rate (%)')
plt.xticks(np.arange(13),[
    '<1 Month', '1 Month', '2 Months', '3 Months', '4 Months', '5 Months', '6 Months', '7 Months', '8 Months',
    '9 Months', '10 Months', '11 Months', '12 Months'
           ])

plt.axvline(-0.5, ls='--', color='green')
plt.axvline(0.5, ls='--', color='green')
plt.stackplot(np.arange(-0.5,1.5), [[120]], color='limegreen', alpha=0.3)
plt.text(x=-0.4, y=110, s='Lowest\nRate', fontsize=14, color='green', va='center')

plt.axvline(11.5, ls='--', color='red')
plt.axvline(12.5, ls='--', color='red')
plt.stackplot(np.arange(11.5,13.5), [[120]], color='#ed6a5e', alpha=0.3)
plt.text(x=11.5, y=110, s='Highest\nRate', fontsize=14, color='red', va='center')

plt.title('Ratio of Cancelation Bookings With The Lead Time',
         fontsize=18, color='black', weight='bold', pad=60)
plt.text(x=-1.1, y=133, s='The figure shows the correlation between the cancelation requests and the lead time.\n'
         'It shows that, there is also a positive correlation between the two mentioned variables like the previous.\n' 
         'The longer the customers book the hotel reservation, the more probability of cancelation requests will also be made by the customers'
         ,
        fontsize=10, style='italic')
plt.legend(fontsize=10)
```
</details>

![task4](https://github.com/FluffyArc/hotel_booking_analysis/assets/40890491/83b958f9-cab9-4e4e-be66-6d07dac4ad5e)

The given chart above shows the correlation between the cancellation request and the lead time. The bar chart illustrates a **positive correlation** among the mentioned variables for the **City Hotel**. **The longer the customer’s lead time, the higher the probability that cancellation requests will also be made**. In contrast, the correlation for the **Resort Hotel** remained steady in the given period of time.

---
# Stage 3: Summary
Based on the analysis above, there are some key takeaways:
1. Since the City Hotel gained more popularity than the Resort Hotel, especially on the **Peak Seasons**, they need to maintained every resource they have (especially the human resources). Applying some promos in other season besides the holiday season could be one of the solutions to improve the customer retention. <br><br>
As for the **Resort Hotel** any strategies related to campaign need to be optimized more to draw the customers (ex: Buy 1 Get 1 promos).

2. Since the high cancelation requests will be affecting the Hotel's revenue and most of the cancelation requests are made because of the longer period of stays and the late bookings, the **City Hotel Management** needs to encourge customers to do more direct bookings as much as possible. One of the solutions is to change the **cancelation policy** such as:
    - Sell at refundable rates during low seasons, but offer no refunds during high seasons. 
    - Offer free cancellation on specific booking channels. For example, directly on your website. 
    - Request a deposit or credit card details as a guarantee at the time of booking.
    - Applying a non-refundable policy for the specifics period of staying (ex: > a week).
    - etc.