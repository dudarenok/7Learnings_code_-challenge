# "Will it snow tomorrow?" - The time traveler asked

In this coding challenge I had to request and clean weather data. Then I should use these data to predict snowfall tomorrow 11 years ago.

This repository contains a jupyter notebook Part_1.ipynb with my solution for the first part and jupyter notebook Part_2.ipynb for the second part, as well all .csv-files that I created in the 1st part.

## Part 1

### 1. Task

Change the date format to 'YYYY-MM-DD' and select the data from 2005 till 2010 for station numbers including and between 725300 and 726300, and save it as a pandas dataframe. Note the maximum year available is 2010.


First of all, I selected data (from 2005 till 2010 for station numbers including and between 725300 and 726300) and saved it as pandas dataframe "weather_data_1". Then I created new column with the date in format 'YYYY-MM-DD'.

### 2. Task

From here want to work with the data from all stations 725300 to 725330 that have information from 2005 till 2010. Do a first analysis of the remaining dataset, clean or drop data depending on how you see appropriate. 


In "weather_data_1" are already saved all data from 2005 till 2010 and from stations 725300 to 726300. So, we need just reduce the stations to 725330.

Using the info method, it is convenient to quickly look at the gaps in the data, in our case there are some columns with much less non-null-values than 19189. We can first go drop this columns:

    mean_station_pressure,
    num_mean_station_pressure_samples,
    max_gust_wind_speed,
    min_temperature,
    min_temperature_explicit,
    snow_depth.

Also we can drop column 'max_temperature_explicit', because it values have type object, and if we try to convert it in expected type float, remain this 3 values: 0., 1., nan, so we can not use this feature, so that it makes sense.

Values from columns 'year', 'month' and 'day' are already saved in new column 'date', so we can drop them too.

Some columns are not informative or helpfull for this task, e.g. all columns with samples number:

    num_mean_temp_samples,
    num_mean_dew_point_samples,
    num_mean_sealevel_pressure_samples,
    num_mean_visibility_samples,
    num_mean_wind_speed_samples.

Some infos to 'station_number' and 'wban_number':

    (station_number?) WMO Identifiers - The World Meteorological Organization relies on a 5-digit numeric code to identify a weather station. It is widely used in synoptic and upper air reports. The entire identifier is often called the "index number". The first two digits are sometimes referred to as the "block number" and refer to the geographic area (00-29 Europe, 30-59 Asia, 60-69 Africa, 70-79 North America, 80-89 South America, 90-99 Oceania). The last three digits are loosely referred to as the "station number" in the context of "block numbers".
    WBAN - The WBAN (Weather Bureau Army Navy) identifier is a 5-digit identifier developed in the 1950s to augment the system of longline identifiers. It is still used by NCDC to identify many of its climatological datasets and thus continues to be important.

[http://www.weathergraphics.com/identifiers/index.htm]

We need one of this features as identifier, where exactly will we predict the weather.

station_number's look more logical (the same prefix indicating the location of stations), so I will use it and drop the another identifier - wban_number.

Then, I drop all rows that have any NaN values.

### 3. Task

Now it is time to split the data, into a training, evaluation and test set. As a reminder, the date we are trying to predict snow fall for is the following, and hence should constitute your test set.


The available dates are between 2005-01-01 and 2010-04-16.
So, the latest date is 2010-04-16 and it is earlier as our wanted date 2010-05-25.

This means that we do not have to pay attention to which data is saved in which set.

My idea is to make the snowfall forecast "overnight". So, we have all the data from day (n) and want to know whether it will snow on the (n + 1)-th day. For this I have to expand the data with the column "snow tomorrow", where it is saved whether it will snow the next day. Then I can randomly split the data into subsets.

There are significantly fewer days with snowfall than without. It is important that this ratio is preserved after dividing into subsets. I use for that train_test_split()-function with 'snow_tomorrow' column as stratify-column.

In the end I saved these dataframes as .csv-files to use it in part 2.

## Part 2

If you made it up to here all by yourself, you can use your prepared dataset to train an Algorithm of your choice to forecast whether it will snow on the following date for each station in this dataset: '2010-05-25'.

You are allowed to use any library you are comfortable with such as sklearn, tensorflow keras etc. If you did not manage to finish part one feel free to use the data provided in 'coding_challenge.csv' Note that this data does not represent a solution to Part 1.


I loaded dataframes from .csv-files (saved in Part 1).

The available dates are between 2005-01-01 and 2010-04-15.
So, the latest date is 2010-04-15 and it is earlier as our wanted date 2010-05-25.

Because of what we don't have any data for the day before the desired forecast day, we could try to generate this data. Unfortunately, only 6 years of data are available, which is not enough to generalize. But we can try, for example, to calculate the mean or median of values from known years for float values, and for boolean use the values that occur most frequently in earlier years.

Another possibility would be to use only 'station_number' and 'date' as features, but then we ignore all weather parameters, which cannot be a good solution.

In the algorithms below I used the data from training and test sets that I saved in part 1.

I change the datatype of 'station_number' from integer to category, because these data are discrete, not continuous. Another possibility would be to divide the train and test sets into subsets, individually for each station and then fit each model with data from each station separately. But in this case we will have too small data and cannot generalize weather-data combinations between the stations.

I tried to predict the snowfall with 3 algorithms: logistic regression, SVM and random forest. 

Accuracy scores with original data:

Logistic regression score: 0.8738738738738738
SVM score: 0.8706442291347952
Random Forest score: 0.8791432942376338

All these algorithms have accuracy score around 87%. But are they really so good?..

The problem is the imbalance: we have much more days without snowfall as with snowfall. This means that even if these algorithms are always predicting "no snow", these predictions will be mostly correct.

To eliminate this imbalance, we can either delete random data with 'snow_tomorrow' == False, or copy random data with 'snow_tomorrow' == True several times, so that in the end there is the same amount of data with and without snow. Deleting will greatly reduce the number of data, so I'm trying to use the copy option.

Accuracy scores with balanced data:

Logistic regression score: 0.6654248069059518
SVM score: 0.5345751930940481
Random Forest score: 0.5769195820081781

As we can see, the accuracies have become significantly worse, but at least they are over 50%.

Alternatively, neural networks could be used, for example LSTM. But in order for it to make sense, I had first to adapt all datasets: use for input the weather data not from just one day, but from the last 3 or 7 days. 
