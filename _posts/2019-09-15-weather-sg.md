---
layout: post
title: "Rainfall Analysis"
author: "Yu Jia Cheong"
categories: weather
tags: [weather singapore]
image: rain-1.jpg
---
# Rainfall
## Dataset
The following data is retrieved from the API for [Realtime Weather Readings across Singapore](https://data.gov.sg/dataset/realtime-weather-readings) and is available for free.

Data can only be fetched day by day or for a specific timestep through this API. It is pretty simple to write a quick script to request data from the API using Python's "requests" library. Data is returned in json format and subsequently converting the data into a pandas dataframe just takes a little more work (a naive conversion into a dataframe results in two columns: one for the timestamp and one where the entry for each timestep is a list of dictionaries of the form {"sensor id": xx, "value", yy}).

Once we have split the sensors up into different columns, we can look at our data properly. Here we have the first few timesteps of the rainfall dataset for 2019, with data from 54 different sensors across Singapore. For the sake of space only the first few sensors are shown - not very interesting as they're all 0 on the timesteps considered!

|                           |   S07 |   S08 |   S11 |   S24 |   S29 |   S33 |   S35 |   S36 |
|:--------------------------|------:|------:|------:|------:|------:|------:|------:|------:|
| 2019-01-01 00:05:00+08:00 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |
| 2019-01-01 00:10:00+08:00 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |
| 2019-01-01 00:15:00+08:00 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |
| 2019-01-01 00:20:00+08:00 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |
| 2019-01-01 00:25:00+08:00 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |     0 |

## Lornie Road Sensor: S07
Let's focus our attention on data from the sensor with the smallest label number (S07), which is incidentally located at Lornie Road. The meta data in the retrieved json also gives the longitude and latitude of the sensor, along with the units the dataset uses (mm).

{% include weather/rainfall_S07.html %}

With a quick plot of data from this sensor over all the timesteps retrieved, we notice a few missing data, and see also that rain is pretty regular from January to Sept 2019. Several consecutively dry days are scattered here and there, and rainfall seems to be more densly packed in the middle of the year.

According to your gut feeling, how often does it rain in Singapore? Asked more quantitatively, what percentage of timesteps would rainfall be non zero? Calculating the number of non zero entries in the five minute timestep data, we realize that only 1031 of the 68544 timesteps for which we have data are non zero - a mere 1.5%!

This might seem unintuitive as it seems to rain pretty often in Singapore. In fact, the issue here is that we are considering the data at the very fine timestep of 5 minutes. Once we group the data by day and take the maximum rainfall of each day, we have the following ratio: 90 out of 245 days, or 36.7%. This feels more reasonable - this equates to there being rain about 2 to 3 days each week. We realize also that this naturally means that we often have rain - no rain patches within the same day, in order for the percentages to differ as such.

Below are some other interesting questions we might ask, and the resulting plots that can be used to examine the data.

What is the total rainfall per day then?
{% include weather/rainfall_daily_S07.html %}

Does it rain more often at night or in the day?
{% include weather/scatter_hour_S07.html %}
