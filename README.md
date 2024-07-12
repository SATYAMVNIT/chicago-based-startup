### Google Data Analytics Professional Certificate Capstone Project: Bike Sharing data

# First we import all the required libraries:

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime
import janitor
import numpy as np

# Importing the data sets:

data_sep2020 = pd.read_csv("D:\\google project data\\dataset_csv_format\\Sep2020-tripdata.csv")
data_oct2020 = pd.read_csv("D:\\google project data\\dataset_csv_format\\Oct2020-tripdata.csv")
data_nov2020 = pd.read_csv("D:\\google project data\\dataset_csv_format\\Nov2020-tripdata.csv")
data_dec2020 = pd.read_csv("D:\\google project data\\dataset_csv_format\\Dec2020-tripdata.csv")
data_jan2021 = pd.read_csv("D:\\google project data\\dataset_csv_format\\Jan2021-tripdata.csv")
data_feb2021 = pd.read_csv("D:\\google project data\\dataset_csv_format\\Feb2021-tripdata.csv")
data_mar2021 = pd.read_csv("D:\\google project data\\dataset_csv_format\\Mar2021-tripdata.csv")
data_apr2021 = pd.read_csv("D:\\google project data\\dataset_csv_format\\Apr2021-tripdata.csv")
data_may2021 = pd.read_csv("D:\\google project data\\dataset_csv_format\\May2021-tripdata.csv")
data_june2021 = pd.read_csv("D:\\google project data\\dataset_csv_format\\June2021-tripdata.csv")
data_july2021 = pd.read_csv("D:\\google project data\\dataset_csv_format\\July2021-tripdata.csv")
data_aug2021 = pd.read_csv("D:\\google project data\\dataset_csv_format\\Aug2021-tripdata.csv")

# The column names of all the dataset is checked for confirming consistency:

print(data_sep2020.columns)
print(data_oct2020.columns)
print(data_nov2020.columns)
print(data_dec2020.columns)
print(data_jan2021.columns)
print(data_feb2021.columns)
print(data_mar2021.columns)
print(data_apr2021.columns)
print(data_may2021.columns)
print(data_june2021.columns)
print(data_july2021.columns)
print(data_aug2021.columns)
import pandas as pd



# To check the structure of all the columns mentioned in the datasets:

for dataset in [data_sep2020, data_oct2020, data_nov2020, data_dec2020, data_jan2021, 
                data_feb2021, data_mar2021, data_apr2021, data_may2021, data_june2021, 
                data_july2021, data_aug2021]:
    print(dataset.info())



# Convert 'start_station_id' and 'end_station_id' to string type for all datasets

datasets = [data_sep2020, data_oct2020, data_nov2020, data_dec2020, data_jan2021, 
            data_feb2021, data_mar2021, data_apr2021, data_may2021, data_june2021, 
            data_july2021, data_aug2021]

for data in datasets:
    data['start_station_id'] = data['start_station_id'].astype(str)
    data['end_station_id'] = data['end_station_id'].astype(str)

    

# Now combining all the data for the last 12 months into a single dataframe named "trip_data":

trip_data = pd.concat(datasets, ignore_index=True)
import pandas as pd
import numpy as np
from datetime import datetime



# To check the value in the rideable_type column:

print(trip_data['rideable_type'])



# Now we need to separately create columns for months, year, day and day of the week:
trip_data['months'] = pd.to_datetime(trip_data['started_at']).dt.strftime('%m')
trip_data['year'] = pd.to_datetime(trip_data['started_at']).dt.strftime('%Y')
trip_data['day'] = pd.to_datetime(trip_data['started_at']).dt.strftime('%d')
trip_data['day_of_the_week'] = pd.to_datetime(trip_data['started_at']).dt.strftime('%a')



# Converting time into the format of hours and minutes:

trip_data['time'] = pd.to_datetime(trip_data['started_at']).dt.strftime('%H:%M')
trip_data['time'] = pd.to_datetime(trip_data['time'], format='%H:%M')



# Calculating the length of the ride(in mins) taken by the user:

trip_data['ride_length'] = (trip_data['ended_at'] - trip_data['started_at']).dt.total_seconds() / 60



# To convert ride_length column from "factor" to "numeric" so that we can use it 

trip_data['ride_length'] = pd.to_numeric(trip_data['ride_length'])
print(pd.api.types.is_numeric_dtype(trip_data['ride_length']))



# Now we need to check the minimum value of ride_length and need to see whether it has any negative values or not :

print(trip_data['ride_length'].min())



# Checking for trip lengths less than 0

print(len(trip_data[trip_data['ride_length'] < 0]))



# Checking for testrides that were made by company for quality checks

print(len(trip_data[trip_data['start_station_name'].str.contains('TEST', case=False, na=False)]))



# We create a new dataframe named "trip_data_pos" which will only have data containing positive ride_length values and are not observations of tests undertaken by the company 

trip_data_pos = trip_data[trip_data['ride_length'] >= 0]



# Removing the test rides

trip_data_pos = trip_data_pos[~trip_data_pos['start_station_name'].str.contains('TEST|test', case=False, na=False)]



# Checking the dataframe:

print(trip_data_pos.info())



# Checking count of distinct values

print(trip_data_pos['member_casual'].value_counts())



# Aggregating total trip duration by customer type

print(trip_data_pos.groupby('member_casual')['ride_length'].sum().reset_index().rename(columns={'ride_length': 'total_ride_length'}))



# To find the mean, median, maximum and minimum values of ride length for casual and regular members:

print(trip_data_pos.groupby('member_casual')['ride_length'].agg(['mean', 'median', 'max', 'min']))
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt



# Checking the dataframe trip_data_pos:

print(trip_data_pos.info())



# Creating a summary table for number of rides with respect to customer types and day of the week:

trip_data_pos_summary = trip_data_pos.groupby(['member_casual', 'day_of_the_week']).agg(
    no_of_rides=('ride_length', 'count'),
    average_duration=('ride_length', 'mean')
).reset_index().sort_values(['member_casual', 'no_of_rides'], ascending=[True, False])



# To check the new summary dataframe: 

print(trip_data_pos_summary.info())



# VISUALIZATIONS:

# Bar graph representing the relation between customer_type and day of the week:
summary = trip_data_pos.groupby(['member_casual', 'day_of_the_week']).size().reset_index(name='no_of_rides')
plt.figure(figsize=(12, 6))
sns.barplot(x='day_of_the_week', y='no_of_rides', hue='member_casual', data=summary, dodge=True)
plt.title('Total trips by customer type Vs. Day of the week')
plt.ylabel('Number of rides')
plt.ticklabel_format(style='plain', axis='y')
plt.xticks(rotation=45)
plt.show()

# Average number of trips by customer types and month:
monthly_summary = trip_data_pos.groupby(['member_casual', 'months']).agg(
    no_of_rides=('ride_length', 'count'),
    average_duration_mins=('ride_length', 'mean')
).reset_index().sort_values(['member_casual', 'no_of_rides'], ascending=[True, False])
print(monthly_summary)

# Visualization for total trips by customer type vs. Months
plt.figure(figsize=(12, 6))
sns.barplot(x='months', y='no_of_rides', hue='member_casual', data=monthly_summary, dodge=True)
plt.title('Total trips by customer type Vs. Months')
plt.ylabel('Number of rides')
plt.ticklabel_format(style='plain', axis='y')
plt.xticks(rotation=30)
plt.show()
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Average trip duration by customer type on each day of the week:
(trip_data_pos.groupby(['member_casual', 'day_of_the_week'])
 .agg({'ride_length': 'mean'})
 .reset_index()
 .pipe((sns.catplot, 'data'), x='day_of_the_week', y='ride_length', hue='member_casual', kind='bar', dodge=True)
 .set(title='Average trip duration by customer type Vs. Day of the week'))
plt.show()

# Average trip duration by customer type in each month:
(trip_data_pos.groupby(['member_casual', 'months'])
 .agg({'ride_length': 'mean'})
 .reset_index()
 .pipe((sns.catplot, 'data'), x='months', y='ride_length', hue='member_casual', kind='bar', dodge=True)
 .set(title='Average trip duration by customer type Vs. Month'))
plt.xticks(rotation=30)
plt.show()

# Demand of bikes over a period of 24 hours (1 day):
(trip_data_pos.groupby(['member_casual', 'time'])
 .size()
 .reset_index(name='no_of_trips')
 .pipe((sns.lineplot, 'data'), x='time', y='no_of_trips', hue='member_casual')
 .set(title='Demand over 24 hours of a day', xlabel='Time of the day'))
plt.xticks(rotation=90)
plt.show()

# Type of rides VS no of trips with respect to the customer types:
(trip_data_pos.groupby(['rideable_type', 'member_casual'])
 .size()
 .reset_index(name='no_of_trips')
 .pipe((sns.barplot, 'data'), x='rideable_type', y='no_of_trips', hue='member_casual')
 .set(title='Ride type Vs. Number of trips'))
plt.show()

# Now at the end, we create a csv file consisting of the clean data which can be used for further analysis and visualizations:
clean_data = (trip_data_pos.groupby(['member_casual', 'day_of_the_week'])
              .agg({'ride_length': 'mean'})
              .reset_index())
clean_data.to_csv("Clean Data.csv", index=False
