---
title: UK Temperature Trends
date: 2023-04-12 00:00:00
description: Utilizing python to investigate UK weather trends.
featured_image: '/images/stock_uk_weather.jpg'
---

The weather in the UK is famously unpredictable, with four distinct seasons that can often feel like they occur all in one day! Summers are generally mild and pleasant, with occasional heat waves and long daylight hours. Autumn is characterized by crisp, cool air and stunning foliage colors. Winters can be cold and damp, with frequent rain and occasional snowfall. Spring brings a sense of renewal with blossoming flowers and longer days, but can also be quite rainy. Overall, the weather in the UK is a topic of constant conversation and surprise, with locals often needing to carry a jacket and umbrella with them at all times just in case.

In this post I investigate some weather trends in two UK locations: Braemar, in the highlands of Scotland, and Heathrow in Southern England.

I use standard Python data visualization libraries (matplotlib and Seaborn), and attempt to emulate the data visualization best practices laid out by Edward Tufte in his excellent 1983 book [The Visual Display of Quantitative Information](https://www.edwardtufte.com/tufte/books_vdqi). I hope to get around to writing a blog post discussing some of the information contained within this book later in the year.

# Data Background

The dataset is available [for free from the met office](https://www.metoffice.gov.uk/research/climate/maps-and-data/historic-station-data). Braemar has data from 1959 - 2023, and Heathrow from 1948 - 2023. The two weather stations are in completely different parts of the UK and see widely different weather trends.


The dataset consists of
* Year
* Month
* Mean daily maximum temperature (tmax)
* Mean daily minimum temperature (tmin)
* Days of air frost (af)
* Total rainfall (rain)
* Total sunshine duration (sun)

Example data for January 1957 at Heathrow can be seen below:

| Year | Month | Max Temp  | Min Temp | af | Rain (mm) | Sun (hrs) |
|------|-------|-----------|----------|----|-----------|-----------|
| 1957 | 1     | 26        | 11       | 9  | 56        | 40        |










