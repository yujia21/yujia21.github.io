---
layout: post
title: "Singapore's Weather"
author: "Yu Jia Cheong"
categories: weather
tags: [weather singapore EDA]
image: rain-1.jpg
---
# Dataset
Real time weather data can be retrieved from this [API provided by NEA and the Singapore government](https://data.gov.sg/dataset/realtime-weather-readings) and is available for free. We have air temperature, rainfall, windspeed, and wind direction data.

Data can only be fetched day by day or for a specific timestep through this API. It is pretty simple to write a quick script to request data from the API using Python's "requests" library. Data is returned in json format and subsequently converting the data into a pandas dataframe just takes a little more work (a naive conversion into a dataframe results in two columns: one for the timestamp and one where the entry for each timestep is a list of dictionaries of the form {"sensor id": xx, "value", yy}).

## Rainfall
Once we have split the sensors up into different columns, we can look at our data properly. The meta data in the retrieved json also gives the longitude and latitude of the sensor, along with the units the dataset uses (mm). We will not keep the metadata in our dataframe, but for a quick sense of where the sensors are located we have the following plot.

{% include weather/geojson_simple.html %}

Here we have the first few timesteps of the rainfall dataset for 2019, with data from 54 different sensors across Singapore. For the sake of space only the first few sensors are shown - not very interesting as they're all 0 on the timesteps considered!

|                           |   S07 |   S08 |   S11 |   S24 |   S29 |   S33 |   S35 |   S36 |
|:--------------------------|------:|------:|------:|------:|------:|------:|------:|------:|
| 2019-01-01 00:05:00+08:00 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |
| 2019-01-01 00:10:00+08:00 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |
| 2019-01-01 00:15:00+08:00 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |
| 2019-01-01 00:20:00+08:00 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |
| 2019-01-01 00:25:00+08:00 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |

Rest assured though, the dataset does not only contain zeros - plotting the total rainfall in Singapore at each time step over the data we have retrieved, we have the following graph:

{% include weather/rainfall_all.html %}

Does it rain more or less the same amount everywhere in Singapore? Summing the total rainfall for each day and taking the mean of this value over all the days in our dataset gives the following graph, showing a difference of 4mm all the same between the sensor with the most and the least rain!

{% include weather/rainfall_total_daily_mean.html %}

Looking at the average maximum daily rainfall over the whole dataset at each sensor gives us a similar curve. (Here we take the daily maximum, and then the average over the whole dataset, as compared to the daily total).
{% include weather/rainfall_sensors_max.html %}

Now a natural question to ask might be whether the ranking of the sensors for the above two plots are about the same - that is, would we expect the sensors with the most average daily rainfall to also be the sensors with the most maximum daily rainfall? The answer is not always! (A zero value here would mean that the position of the sensor for the above two plots are the same.)
{% include weather/rainfall_positional_difference.html %}

Does it rain more often at night or in the day? Here we have a scatter plot of the hour of day that each datapoint (five minute timesteps) belongs to and total rainfall across Singapore recorded at that time. There doesn't seem to be a significant difference, although more detailed plots might show seasonal trends (for example, there might be some months where it only rains in the afternoon, or at night, etc.)
{% include weather/scatter_hour.html %}

Let's focus our attention on data from the sensor with the smallest label number (S07), which is incidentally located at Lornie Road (around the center of Singapore). The mean total daily rainfall is 3.95mm, which places it at around the median of all the sensors. Something every foreigner always asks me and that I never know how to answer any more (climate change!) is when the rainy season is in Singapore - so let's look at how the rainfall looks over the nine months or so of data. The first plot shows this for each five minute time step while the second is a scatter plot that shows the total daily rainfall for each day.

{% include weather/rainfall_S07.html %}
{% include weather/rainfall_daily_S07.html %}

We notice a few days with missing data, but otherwise rainfall seems pretty regular from January to Sept 2019. Several consecutively dry days are scattered here and there, and rainy days seem to be more dense in the middle of the year.

According to your gut feeling, how often does it rain in Singapore? Asked more quantitatively, what percentage of timesteps would rainfall be non zero? Calculating the number of non zero entries in the five minute timestep data, we realize that only 1031 of the 68544 timesteps for which we have data are non zero - a mere 1.5%!

This might seem unintuitive as it seems to rain pretty often in Singapore. In fact, the issue here is that we are considering the data at the very fine timestep of 5 minutes. Once we group the data by day and take the maximum rainfall of each day, we have the following ratio: 90 out of 245 days, or 36.7%. This feels more reasonable - this equates to there being rain about 2 to 3 days each week. We realize also that this naturally means that we often have rain - no rain patches within the same day, in order for the percentages to differ as such.

Zooming back out, we can take a look at how this percentage varies across the island. Let's also throw in the maximum recorded rainfall at each sensor from our dataset.
{% include weather/geojson.html %}
