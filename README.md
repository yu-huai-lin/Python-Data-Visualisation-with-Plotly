# Python Data Visualisation with Plotly

Plotly and plotly Express is one of the best packages for building visualisations in python. It takes care of a lot of painful axes formatting while allowing easy and convenient customisation. It also has an interative interface that allows you to zoom in/out of the charting, which can be embedded in html files. 

In this repository, I will show how I make use of the hotel booking demand dataset to do some explotary data analysis with plotly and playing around with its amazing and super easy to learn features!

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

#### 1.2 Faceted Line plot -  Cancellation Rate by Hotel

![](/images/line2.png)

#### 1.3  ADR by Hotel and Deposit Type


![](/images/line3.png)




