---
layout: post
title: "Which boat should my rowing club buy?"
author: "Yu Jia Cheong"
categories: personal
tags: [exploratory data analysis]
image: row.jpg
---

# Dataset
As one of the resident data scientists of my [rowing club](http://easterrowing.club), I decided to play a little with the data generated by boat booking over the year 2019. Data was collected from a shared online Google Sheets file, where members wrote down their names for the boat they intended to use and the date/time where they were rowing. Since this analysis was done, stricter logging policies were implemented (standardizing names used by members by forcing the choice to be made from a drop down list, and enforcing full list of crew names for crew boats), so next year's of data analysis should be less painful! Of course, this dataset doesn't take into account those with private single boats - they appear on the sheet only when they row in crew boats.

This is the first time I've played with data with some "natural language" properties outside of school since my work consists mainly of IoT time series data. To nobody's surpise this means loads of preprocessing and cleaning. A quick look at data from the first day of the year:

|    | Day         | Time          |                    #09 - Single |   #11 - Single |                            #14 - Single |   #15 - Single | New LW Single Suu Balm Sponsored            60-75 kg   |   Birra Viola - Single |   4 Fingers - Single |   ERC blue #1 - training single, priority to beginners |   ERC blue #2 - training single, priority to beginners |                    SRA 1; Double  |                          SRA 2; Double  |           Double MW;  | New Double           |   Pair | SRA 7;  Pair   |     #01 - Four |   #02 - Four |   Nelly - Quad   |   Aisyah - Quad |       Eights A  | Datetime     |
|    |             |               |                         LW; <75 |      MW; 75-85 |                               MW; 75-85 |        HW; 85+ |                                                        |                HW; 85+ |          MW; 75-85   |                                                        |                                                        |   HW; (Out of action for repairs) |   LW; (Now available for booking again) |   Brotzeit Sponsored  | Osteria Sponsored    |    HW; |                |       HW;  85+ |              |                  |         HW; 85+ |   pre-payment   |              |
|    |             |               |   the bottom teeth tracks slide |                |   the footplate is only in low position |                |                                                        |                        |                      |                                                        |                                                        |                                   |                                         |               75-85kg | MW: 70-85 kg         |        |                |   Concord nose |              |                  |    Wintech Oars |          rq SRA |              |
|    |             |               |                                 |                |                                         |                |                                                        |                        |                      |                                                        |                                                        |                                   |                                         |                       |                      |        |                |                |              |                  |                 |                 |              |
|---:|:------------|:--------------|--------------------------------:|---------------:|----------------------------------------:|---------------:|:-------------------------------------------------------|-----------------------:|---------------------:|-------------------------------------------------------:|-------------------------------------------------------:|----------------------------------:|----------------------------------------:|----------------------:|:---------------------|-------:|:---------------|---------------:|-------------:|-----------------:|----------------:|----------------:|:-------------|
|  0 | Tue, 1/1/19 | 6.30 - 8.00   |                             nan |            nan |                                     nan |            nan | Yu Jia (21k)                                           |                    nan |                  nan |                                                    nan |                                                    nan |                               nan |                                     nan |                   nan | nan                  |    nan | Waiting        |            nan |          nan |              nan |             nan |             nan | 1/1/19 6.30  |
|    |             |               |                                 |                |                                         |                |                                                        |                        |                      |                                                        |                                                        |                                   |                                         |                       |                      |        | for            |                |              |                  |                 |                 |              |
|    |             |               |                                 |                |                                         |                |                                                        |                        |                      |                                                        |                                                        |                                   |                                         |                       |                      |        | rigging        |                |              |                  |                 |                 |              |
|  1 | Tue, 1/1/19 | 8.00 - 9.30   |                             nan |            nan |                                     nan |            nan | nan                                                    |                    nan |                  nan |                                                    nan |                                                    nan |                               nan |                                     nan |                   nan | nan                  |    nan | nan            |            nan |          nan |              nan |             nan |             nan | 1/1/19 8.00  |
|  2 | Tue, 1/1/19 | 9.30 - 11.00  |                             nan |            nan |                                     nan |            nan | nan                                                    |                    nan |                  nan |                                                    nan |                                                    nan |                               nan |                                     nan |                   nan | nan                  |    nan | nan            |            nan |          nan |              nan |             nan |             nan | 1/1/19 9.30  |
|  3 | Tue, 1/1/19 | 16.00 - 17.30 |                             nan |            nan |                                     nan |            nan | nan                                                    |                    nan |                  nan |                                                    nan |                                                    nan |                               nan |                                     nan |                   nan | Olga/Jason           |    nan | nan            |            nan |          nan |              nan |             nan |             nan | 1/1/19 16.00 |


Yikes. The first thing I did was to standardize the date and time columns and clean up the column names:

| Datetime            |   #09 - Single LW |   #11 - Single MW |   #14 - Single MW |   #15 - Single HW | Suu Balm - Single LW   |   Birra Viola - Single HW |   4 Fingers - Single MW |   ERC blue 1 - Training Single |   ERC blue 2 - Training Single |   SRA 1 - Double HW |   SRA 2 - Double LW |   Brotzeit - Double MW | Osteria - Double MW   |   N/A - Pair HW; |   SRA 7 - Pair |   #01 - Four HW |   #02 - Four |   Nelly - Quad |   Aisyah - Quad HW |   Eights SRA |
|:--------------------|------------------:|------------------:|------------------:|------------------:|:-----------------------|--------------------------:|------------------------:|-------------------------------:|-------------------------------:|--------------------:|--------------------:|-----------------------:|:----------------------|-----------------:|---------------:|----------------:|-------------:|---------------:|-------------------:|-------------:|
| 2019-01-01 06:30:00 |               nan |               nan |               nan |               nan | Yu Jia (21k)           |                       nan |                     nan |                            nan |                            nan |                 nan |                 nan |                    nan | nan                   |              nan |            nan |             nan |          nan |            nan |                nan |          nan |
| 2019-01-01 08:00:00 |               nan |               nan |               nan |               nan | nan                    |                       nan |                     nan |                            nan |                            nan |                 nan |                 nan |                    nan | nan                   |              nan |            nan |             nan |          nan |            nan |                nan |          nan |
| 2019-01-01 09:30:00 |               nan |               nan |               nan |               nan | nan                    |                       nan |                     nan |                            nan |                            nan |                 nan |                 nan |                    nan | nan                   |              nan |            nan |             nan |          nan |            nan |                nan |          nan |
| 2019-01-01 16:00:00 |               nan |               nan |               nan |               nan | nan                    |                       nan |                     nan |                            nan |                            nan |                 nan |                 nan |                    nan | Olga/Jason            |              nan |            nan |             nan |          nan |            nan |                nan |          nan |

Now we can get to work!

# Analysis
Counting the number of non-null entries per boat seemed like the most obvious and helpful thing to do - this allows the club to know which boats are most used and in highest demand.

{% include row/overall_boat_usage.html %}

A clear preference for singles and doubles - understandable due to our small club sizes! Some of the needs surfaced from this analysis confirmed some of the felt needs of the club and translated to new boats bought over the year (the ERC blues, towards the end of the year).

I then regrouped the data and did the same plot by weight class and boat type.
{% include row/boat_usage_by_weight.html %}
{% include row/boat_usage_by_type.html %}

At first glance it might seem that singles are the most popular boat, but remember that we should in fact correct for the number of people in the boats:
{% include row/boat_usage_by_type_corrected.html %}

Even then, this isn't exactly fair as we have more singles and doubles than fours and quads in the club.

When do our rowers get on the water? Doing the same count but by time slot of the week:

{% include row/boat_usage_by_slot.html %}

As expected, weekends are when we get the most rowers down. There's also a clear preference for morning rowing rather than afternoon rowing.

# Rower statistics
I was tasked with finding the most frequent rowers of the club who are not already in the club committee and who might be susceptible to take up some club responsibility. This was difficult because members used different names when booking, for example, I personally used "Yu Jia" and "YJ" interchangeably. To confound matters, crew boats were booked with a single entry, depending on style it could be "A + B", "A & B", "A / B", or quite simply "A B"! There was also no guarantee that all members of the crew boat would be listed - the ladies often booked boats with "Ladies", and similarly in other cases only two or three names out of four would be present. Luckily our club boasts only about 70 members, with some 50 active rowers - I created a standardized list of users and did some search and replace, dropping spaces, symbols, cases... not the most exact analysis, but with the standardized name list next year, things should be easier!

Anonymizing slightly the names, we have:

{% include row/boat_usage_by_rower.html %}

Unfortunately, our most frequent rowers were all in our under 18 junior team, or already have served / are serving in the committee, which means no rest for the current committee yet...

As a side note, I had personally counted 82 rows in 2019. Barring two days of regatta and an outing in France, the above count is still missing a good number of rows! This is probably due to the couple of weeks where fumigation was being carried out at the reservoir and rowing times were shifted, resulting in the boat usage data being temporarily logged on a separate Google Sheet.

Time to give myself a pat on the back for being one of the most frequent rowers in the club, and retire my analysis notebook till the end of 2020...
