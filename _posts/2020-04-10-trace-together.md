---
layout: post
title: "Is TraceTogether really useful in tracing contacts?"
author: "Yu Jia Cheong"
categories: personal
tags: [tracetogether singapore data covid19]
image: city-2.jpg
---

# TraceTogether
In the midst of the COVID19 crisis, one of the teams in the Singapore government has developed an app called[ TraceTogether](https://www.tracetogether.gov.sg/), on which [much](https://splira.com/2020-03-28/) [ink](https://digitalreach.asia/tracetogether-disassembling-it-wasnt-easy-to-confirm-the-governments-claims-on-privacy/) [has](https://www.reddit.com/r/singapore/comments/fno0me/tracetogether_doesnt_seem_to_collect_much_info/) [already](https://github.com/opentrace-community) [been](https://www.tech.gov.sg/media/technews/tracetogether-behind-the-scenes-look-at-its-development-process) (spilt)[https://www.reddit.com/r/singapore/comments/ftpc3r/oc_i_extracted_my_own_tracetogether_data_from_my/]. The goal of the app is to aid in contact tracing by keeping track of people you might come into close contact with unknowingly (for example, on the train, at a hawker center) via tracking bluetooth signals. I won't go into more detail on how the app works and privacy concerns since those are not my areas of "expertise", but merely be content with examining the data collected in my TraceTogether app.

As I had never extracted data from my phone before, I found the instructions found in the last link above was pretty helpful. The app data contained several sqlite databases, but the two relevant ones were record_database and trace_together_status. Unlike the author of the post who used grafana as their data visualization tool, I decided to get onto more familiar ground by exporting the relevant extracted sqlite database as a csv file to examine it in a standard jupyter notebook.

At the time of extraction, I had installed the app for a little more than 2 weeks, during which time I was mostly working from home (during which my bluetooth was off), with a few outings for meals with friends and to the gym (pre "Circuit Breaker" measures) (for which I turned bluetooth back on).

# Data
Here's how the first few lines of data from the record_database look like:

|    |   id |     timestamp |   v | msg                                                                                  | org    | modelP   | modelC     |   rssi |   txPower |
|---:|-----:|--------------:|----:|:-------------------------------------------------------------------------------------|:-------|:---------|:-----------|-------:|----------:|
|  0 |    1 | 1585305371488 |   1 | 2k7qJt6lIT7Ys/EXxuG/Gb7QLNKyWRII+VVHYCY74rv6oHK8wYhJ6Fae2I9W1nzwQ0UwIQbYmOgEo2Ymcg== | SG_MOH | Pixel    | SM-N960F   |    -69 |       nan |
|  1 |    2 | 1585305437590 |   1 | TsPI8/n1kKnezeGgFv8OtY9ZlbpFXbNAAATV9eFWiKp4Jnjp/Oj4iBf/50yJ0MN4kLAninEnV9BECSQ3uQ== | SG_MOH | Pixel    | SM-G935F   |    -75 |       nan |
|  2 |    3 | 1585305455172 |   1 | duEIK+7NFOt4bTT+9jQCfYk8/Rmea3PNYM72CNlLxDTA6+Ai1bjUunu0G6lwufYD5/dj3Pa9GBCOXMr+og== | SG_MOH | Pixel    | Pixel 3 XL |    -76 |       nan |
|  3 |    4 | 1585305488374 |   1 | PpPIgfXSKgeOje1rUxG4mZBzlme3P3xhwnhGNEI6KmC6aF2jymvy5eawSZGWxqtz7JbBd/YNictWogVMJg== | SG_MOH | Pixel    | SM-G960F   |    -87 |       nan |
|  4 |    5 | 1585305498986 |   1 | 2k7qJt6lIT7Ys/EXxuG/Gb7QLNKyWRII+VVHYCY74rv6oHK8wYhJ6Fae2I9W1nzwQ0UwIQbYmOgEo2Ymcg== | SG_MOH | Pixel    | SM-N960F   |    -65 |       nan |

Right away we can figure out what some of these columns mean:
* id simply increases
* timestamp is in standard UNIX time (number of milliseconds since 1 Jan 1970 midnight UTC)
* v is always 1
* msg is probably the encrypted phone number of the other party encountered
* org is always "SG_MOH"
* modelP is my phone (Pixel)
* modelC is the model of the other phone encountered (as read elsewhere, probably tracked to aid in app debugging purposes)
* rssi is [Received Signal Strength Indicator](https://en.wikipedia.org/wiki/Received_signal_strength_indication), which when in negative form represents a stronger signal the nearer to 0 it is.
* txPower is mostly NaNs, with a few ones sprinkled in between, not too sure what this can be.

First things first - let's convert that timestamp into a more readable form. After a first quick look at the data and trying to piece together where I was for each of the entries, I couldn't figure out why there were readings at 10.30am when I was ostensibly working from home. Digging a little deeper, I realized that there were also readings at midnight when I was definitely at home and asleep - so something was definitely up! The answer is actually pretty simple - the timestamps were saved in UTC, so I needed to add 8 hours to it for the data to make sense. Keeping only the interesting columns and adding a quick parser on the phone model to get the brand of the phone:

|    | timestamp                  | msg                                                                                  | modelC     | modelC_parsed   |   rssi |
|---:|:---------------------------|:-------------------------------------------------------------------------------------|:-----------|:----------------|-------:|
|  0 | 2020-03-27 18:36:11.488000 | 2k7qJt6lIT7Ys/EXxuG/Gb7QLNKyWRII+VVHYCY74rv6oHK8wYhJ6Fae2I9W1nzwQ0UwIQbYmOgEo2Ymcg== | SM-N960F   | Samsung         |    -69 |
|  1 | 2020-03-27 18:37:17.590000 | TsPI8/n1kKnezeGgFv8OtY9ZlbpFXbNAAATV9eFWiKp4Jnjp/Oj4iBf/50yJ0MN4kLAninEnV9BECSQ3uQ== | SM-G935F   | Samsung         |    -75 |
|  2 | 2020-03-27 18:37:35.172000 | duEIK+7NFOt4bTT+9jQCfYk8/Rmea3PNYM72CNlLxDTA6+Ai1bjUunu0G6lwufYD5/dj3Pa9GBCOXMr+og== | Pixel 3 XL | Pixel           |    -76 |
|  3 | 2020-03-27 18:38:08.374000 | PpPIgfXSKgeOje1rUxG4mZBzlme3P3xhwnhGNEI6KmC6aF2jymvy5eawSZGWxqtz7JbBd/YNictWogVMJg== | SM-G960F   | Samsung         |    -87 |
|  4 | 2020-03-27 18:38:18.986000 | 2k7qJt6lIT7Ys/EXxuG/Gb7QLNKyWRII+VVHYCY74rv6oHK8wYhJ6Fae2I9W1nzwQ0UwIQbYmOgEo2Ymcg== | SM-N960F   | Samsung         |    -65 |

Doing the same thing for the trace_together_status database, we see that it keeps a record of when scanning is started and stopped on the phone:
|    |   id | timestamp                  | msg              |
|---:|-----:|:---------------------------|:-----------------|
|  0 |    1 | 2020-03-21 11:48:26.902000 | Scanning Started |
|  1 |    2 | 2020-03-21 11:48:34.837000 | Scanning Stopped |
|  2 |    3 | 2020-03-21 11:48:52.103000 | Scanning Started |
|  3 |    4 | 2020-03-21 11:48:55.535000 | Scanning Stopped |
|  4 |    5 | 2020-03-21 17:40:50.266000 | Scanning Started |


# Encounters
Each entry in the record_database is a recorded ping from another phone, therefore the total number of entries in the database would include multiple entries from the same individual encountered. Since the encrypted identifiant in the 'msg' column is unique for each individual, a groupby by date and 'msg' would give us the total number of unique encounters I had. Grouping by date would enable us to account for the possibility of running into the same person twice on different days, although this subsequently holds the assumption that all entries with the same individual encountered in the same day are from the same encounter (although in fact there were no repeated encounters in my set of data).

Some quick numbers to start things off:
* Number of entries: 248
* Number of unique encounters: 155

The simplest plot to make would be a histogram of the number of pings recorded per individual encounter. Since I have made sure that no individual was encountered on more than one day, I simply did a groupby on the 'msg' column for this. Making the (reasonable) assumption that each encounter is continuous, we can find the length of the encounter by looking at the difference between the timestamps of the first and last ping per individual.

[Histogram of Pings](/_includes/trace/ping_histogram.png)
[Histogram of duration of encounters](/_includes/trace/time_histogram.png)

As can be expected, most encounters were short ones with only one ping, and lasted less than a minute.

To get a sense of the density of encounters (did I encounter a lot of people at once? Did I encounter a few people over a longer period of time?), we can look at the total number of unique encounters per 30 minute blocks, and the cumulative plot of the total number of unique encounters over the two weeks.

[Unique encounters by 30 minute block](/_includes/trace/encounters_30.png)
[Cumulative Encounters](/_includes/trace/encounters_cumulative.png)

# Signal Strength
But the number of people I come into contact with is less interesting than knowing how long I was in contact with them and at what "proximity". Unfortunately, as the RSSI signal doesn't have standard units, I am unable to calculate the distance of the other party from my phone simply based on the RSSI data. The data can however give me a sense of how these distances are distributed.

We can plot the received signal strength of each individual with a different colour to get a sense of how different encounters moved around me (my phone) each time I turned my bluetooth on. A dinner at Clementi hawker center and subsequent quick jaunt in a supermarket with three friends looks like this:

[RSSI of encounters at Clementi](/_includes/trace/rssi_hawker.png)

This outing contained the longest encounter I had of 45 minutes (the xx line from xx to xx).
A HIIT session in the gym, where my phone was left in my bag at the side of the room:

[RSSI of encounters at Move to Live gym](/_includes/trace/rssi_gym.png)

Plotting the rssi of each encounter over a background of when scanning started and stopped, we see the following:
[Encounters and scanning over whole period](/_includes/trace/rssi_scanning.png)
[Encounters and scanning over one day](/_includes/trace/rssi_scanning_day.png)

# Phone Models
Others who've extracted their data have also looked at the distribution of phone models encountered. From my data, Samsung appears to have a disproportionately large amount of TraceTogether users, and this is in line with observations from others too. This is likely due to complications of using the app with iPhone resulting in slower uptake among iPhone users (complications such as having to keep it in the foreground, having to keep the screen on - if not for the latest version of the app, at least at the start).

[RSSI of encounters at Move to Live gym](/_includes/trace/phone_brand.png)
[RSSI of encounters at Move to Live gym](/_includes/trace/phone_models.png)
