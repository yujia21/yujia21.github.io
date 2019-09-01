---
layout: post
title: "USEP Exploratory Data Analysis"
author: "Yu Jia Cheong"
categories: energy
tags: [energy singapore]
image: cuba-4.jpg
---
The following data is retrieved from the Energy Market Company website and is available for free.

This dataset comprises of the Uniform Singapore Energy Price (USEP), in $/MWh, which changes at every half hour and is dependent on the demand and supply of the period, as well as the overall demand in Singapore in MW.

Below is data from the year of 2018, the first plot shows the evolution of the prices over the year. While the prices are generally around 100$/MWh, there are clear peaks.

{% include usep/usep_2018.html %}

Plotting the average USEP over the week, we observe no clear weekday and weekend difference. Notice that the mean is much higher than the median due to extreme valued outliers.
{% include usep/usep_ave.html %}

To better get a sense of the outliers, we look at the scatter plot of the prices with respect to the periods. The following two graphs show the spread of the prices in each period of the day (48 in all), as well as each period of the week (such that period 0 on Monday is 0 and Period 48 on Sunday is 336).
{% include usep/usep_scatter_period.html %}
{% include usep/usep_scatter_week.html %}

Extremely high prices generally occur in the day during working hours, and seems to be pretty independent of the day of the week.

Looking now at the average weekly demand, we see this time a clear weekday and weekend difference, as is expected.
{% include usep/demand_ave.html %}

Plotting a scatter of demand vs USEP, we can observe a generally linear trend with the outliers in the USEP tending to be when the demand is sufficiently high. Again, this is as expected.
{% include usep/demand_vs_usep.html %}
