# Python Data Visualisation with Plotly

Plotly and plotly Express is really convenient for building visualisations in python. It takes care of a lot of painful axes formatting while allowing easy customisation. It also has an interative interface that allows you to zoom in/out of the charting, which can be embedded in html files. 

In this repository, I will show how I make use of the hotel booking demand dataset play around with plotly while doing EDA

## Data

I'm using the Hotel Booking Demand dataset, which is downloadable from Kaggle

https://www.kaggle.com/jessemostipak/hotel-booking-demand

This data set contains booking information for City hotel and Resort hotel, and includes information such as when the booking was made, if the booking is cancelled, length of stay, the number of adults, children, and/or babies, and the number of available parking spaces, etc.

### Understanding the Data
some column definition in order to understand the dataset better (Data source: Kaggle):

| Column Name | Description|
|:----|:-----------|
| #hotel | Hotel (H1 = Resort Hotel or H2 = City Hotel)|
| #is_canceled| Value indicating if the booking was canceled (1) or not (0)|
| #lead_time| Number of days that elapsed between the entering date of the booking into the PMS and the arrival |
| #market_segment | Market segment designation. In categories, the term “TA” means “Travel Agents” and “TO” means “Tour Operators” |
| #country  | Country of origin. Categories are represented in the ISO 3155–3:2013 format |
| #ADR |Average Daily Rate as defined by dividing the sum of all lodging transactions by the total number of staying nights|
| #reservation_status |Reservation last status, assuming one of three categories: Canceled – booking was canceled by the customer; Check-Out – customer has checked in but already departed; No-Show – customer did not check-in and did inform the hotel of the reason why|
| #deposit_type|Indication on if the customer made a deposit to guarantee the booking. This variable can assume three categories: No Deposit – no deposit was made; Non Refund – a deposit was made in the value of the total stay cost; Refundable – a deposit was made with a value under the total cost of stay.|
|#room_type|Code of room type reserved. Code is presented instead of designation for anonymity reasons.|
| #Adults | Number of adults |


## Required Packages
```
import pandas as pd
from plotly.subplots import make_subplots
import plotly.graph_objects as go
import pandas as pd
import numpy as np
import datetime
import plotly.express as px
```

## Data Clean-up and pre-processing

```
dt = pd.read_csv('hotel_bookings.csv')

# 1. convert to date time and add extract month from ymd
dt['reservation_status_date'] = pd.to_datetime(dt['reservation_status_date'], format='%Y-%m-%d')

# 2. Remove duplicating rows
df = df.drop_duplicates( keep='first')

# 3. Remove average daily hotel rate <0
df = df[df.adr > 0]

# 4. Removing the outliners found in adr:`
df =df[(df['adr'] < 1000)]
```

![](src/images/outliner.png)


## Data Visualisation

### Line Chart
#### 1.1 Line chart - Monthly Hotel booking trend

![](/images/fig1_line.png)


```
trend_month = df.groupby(['reservation_status_month']).count().reset_index()
trend_month.sort_values('reservation_status_month', inplace=True)
trend_month['reservation_status_month'] = trend_month['reservation_status_month'].astype(str)
trend_month['reservation_status_month'] = pd.to_datetime(trend_month['reservation_status_month'])
```

```
fig = px.line(trend_month, x='reservation_status_month', y="hotel",
             height=600, width=1000, template = 'plotly_white',
             title = "Monthly Hotel booking trend")
fig.update_layout(
    xaxis_title="date",
    yaxis_title="bookings",
    font=dict(family="Avenir",))

fig.show()
```
---

#### 1.2 Faceted Line plot -  Cancellation Rate by Hotel

![](/images/line_2.png)

```
pv =df.pivot_table(values='lead_time', index=['reservation_status_month', 'hotel'], 
               columns='is_canceled', aggfunc='count').rename(columns={0:'bookings', 1:'cancel'}).reset_index('hotel')
pv['cancellation_rate'] = pv['cancel'] / (pv['bookings'] + pv['cancel'] )

```

```
fig = px.scatter(pv, x=pv.index, y="cancellation_rate",
             height=600, width=1000, template = 'plotly',
             title = "Monthly Hotel Cancellation Rate by Hotel", facet_col="hotel").update_traces(mode='lines+markers')
fig.update_layout(
    xaxis_title="date",
    yaxis_title="Cancellation Rate",
    font=dict(
        family="Avenir"))
fig.update_xaxes(tickangle=90,
                 ticks="outside",
                 tickwidth=2,
                 tickfont=dict(size=10))

fig.show()
```





#### 1.3  ADR by Hotel and Deposit Type

![](/images/line_3.png)

```
sc3 = df.groupby(['reservation_status_month', 'hotel', 'deposit_type']).agg({'adr':'mean'})\
.reset_index(['hotel', 'deposit_type'])
fig = px.line(sc3, x=sc3.index, y="adr",
             height=600, width=1000, template = "plotly_white", color_discrete_sequence=px.colors.qualitative.Safe,
             title = "ADR by Hotel and Deposit Type", facet_col = "deposit_type", facet_col_wrap = 3, color = "hotel")
fig.update_layout(
    yaxis_title="ADR",
    font=dict(
        family="Avenir"
    )
)
fig.update_xaxes(ticks="outside",
                 tickwidth=2,
                 tickfont=dict(size=10))

fig.show()
```




### Stacked Bar Chart 

#### Monthly bookings and Cancellations

![](/images/bar_1.png)


```
bc = df.groupby(['reservation_status_month', 'reservation_status']).size().reset_index().rename(columns={0: 'count_bookings'})

fig = px.bar(bc, x='reservation_status_month', y='count_bookings', color='reservation_status', 
             title="Monthly Bookings and cancellations",
            height=500, width=900, template = 'plotly_white', color_discrete_sequence=px.colors.qualitative.Set2)
fig.update_layout(xaxis_title="month", yaxis_title="bookings",font=dict(family="Avenir",))
fig.show()
```



### Count Plot 

####bookings & cancellations by hotel type

![](/images/count.png)



```
df['bin'] = pd.cut(df['lead_time'], [0, 50, 100,200, 300, 400, 500, 600, 700, 800], 
                   labels=['0-50', '50-100', '100-200','200-300', '300-400','400-500', '500-600', '600-700', '700-800'  ])
c =df.groupby(['bin', 'hotel', 'is_canceled']).size().reset_index().rename(columns={0:'bookings'})
c['Cancellation Status'] = np.where(
    c['is_canceled'] ==0, "canceled", "bookings") 
```

```
fig = px.bar(c, x="bin", y="bookings",
             height=600, width=1000, template = "plotly_white", color = "Cancellation Status",color_discrete_sequence=px.colors.qualitative.Safe,
             title = "Bookings and cancellations by lead time Bin", facet_col="hotel")
fig.update_layout(
    xaxis_title="Lead Time",
    yaxis_title="Bookings",
    font=dict(
        family="Avenir" ))
fig.update_xaxes(ticks="outside",
                 tickwidth=2,
                 tickfont=dict(size=10))

fig.show()
```



### Multi chart type 
#### Bar + line with sedondary y axis - Monthly Bookings & Daily Avg Hotel Rate

![](/images/multi_1.png)


```
d = df.groupby(['reservation_status_month']).agg({'hotel':'count', 'adr':'mean'})\
.rename(columns={'hotel':'bookings'}).reset_index()
d.sort_values('reservation_status_month', inplace=True)
```

```
# Create figure with secondary y-axis
fig = make_subplots(specs=[[{"secondary_y": True}]])

# Add traces
fig.add_trace(
    go.Bar(x=d['reservation_status_month'], y=d['bookings'], name="bookings"),
    secondary_y=False)

fig.add_trace(
    go.Scatter(x=d['reservation_status_month'], y=d['adr'], name="adr"),
    secondary_y=True)

# Add figure title
fig.update_layout(
    title_text="Monthly Bookings & ADR")

# Set layout
fig.update_xaxes(title_text="Month")
fig.update_yaxes(title_text="bookings", secondary_y=False)
fig.update_yaxes(title_text="Average Daily Rate", secondary_y=True)
fig.update_layout(font=dict(family="Avenir"))

fig.show()
```



### Pie chart 

### Bookings by Market Segment

![](/images/pie_1.png)

```
ms = df.groupby(['market_segment']).size().reset_index().rename(columns={0: 'count_bookings'})
fig = px.pie(ms, values='count_bookings', names='market_segment', 
             color_discrete_sequence=px.colors.qualitative.Safe, title="Bookings by Market Segment")
fig.update_layout(font=dict(family="Avenir"))
fig.show()

```


### Donut chart 

#### bookings by Market Segment and hotel type

![](/images/donut_1.png)

```
pf = df.pivot_table(values='lead_time', index=['market_segment'], 
               columns='hotel', aggfunc='count').reset_index('market_segment')
pf=pf.fillna(0)


labels = ms['market_segment']

# Create subplots: use 'domain' type for Pie subplot
fig = make_subplots(rows=1, cols=2, specs=[[{'type':'domain'}, {'type':'domain'}]])
fig.add_trace(go.Pie(labels=labels, values=pf['City Hotel'], name="City Hotel"),
              1, 1)
fig.add_trace(go.Pie(labels=labels, values=pf['Resort Hotel'], name="Resort Hotel"),
              1, 2)

# Use `hole` to create a donut-like pie chart
fig.update_traces(hole=.4, hoverinfo="label+percent+name")

fig.update_layout(font=dict(family="Avenir"),
    title_text="Market Segment Share - City and Resort Hotel",
    # Add annotations in the center of the donut pies.
    annotations=[dict(text='City Hotel', x=0.16, y=0.5, font_size=19, showarrow=False),
                 dict(text='Resort Hotel', x=0.85, y=0.5, font_size=19, showarrow=False)])
fig.show()
```



### Box Plot 
####Lead time distribution by cancellation status and hotel type


![](/images/box_1.png)

```
fig = px.box(df, x="is_canceled", y="lead_time", color = "hotel", template = "plotly_white", 
             title="Lead time distribution by cancellation status and hotel type")
fig.update_xaxes(ticks="outside",
                 tickwidth=2,
                 tickfont=dict(size=10))

fig.show()
```



### Scatter Plot
#### Relaitonship between leadtime and ADR

![](/images/scatter_1.png)

```
df['cancellation_status'] = np.where(
    df['is_canceled'] ==0, "canceled", "bookings") 
sc = df.groupby(['reservation_status_month', 'hotel', 'deposit_type']).agg({'adr':'mean', 'lead_time': 'mean', 'stays_in_week_nights': 'mean'})\
.reset_index(['hotel', 'deposit_type'])


fig = px.scatter(sc, x="lead_time", y="adr",
             height=600, width=1000, template = "plotly_white", color_discrete_sequence=px.colors.qualitative.Safe,
             title = "Lead Time versus ADR - by Deposit type", facet_col = "deposit_type")
fig.update_layout(
    xaxis_title="Lead Time",
    yaxis_title="ADR",
    font=dict(
        family="Avenir"
    )
)
fig.update_xaxes(ticks="outside",
                 tickwidth=2,
                 tickfont=dict(size=10))

fig.show()
```


