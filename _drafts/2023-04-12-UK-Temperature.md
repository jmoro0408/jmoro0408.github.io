---
title: UK Temperature Trends
date: 2023-04-12 00:00:00
description: Utilizing python to investigate UK weather trends.
featured_image: '/images/stock_uk_weather.jpg'
---

The weather in the UK is famously unpredictable, with four distinct seasons that can often feel like they occur all in one day! Summers are generally mild and pleasant, with occasional heat waves and long daylight hours. Autumn is characterized by crisp, cool air and stunning foliage colors. Winters can be cold and damp, with frequent rain and occasional snowfall. Spring brings a sense of renewal with blossoming flowers and longer days, but can also be quite rainy. Overall, the weather in the UK is a topic of constant conversation and surprise, with locals often needing to carry a jacket and umbrella with them at all times just in case.

In this post I investigate some weather trends in two UK locations: Braemar, in the highlands of Scotland, and Heathrow in Southern England.

I use standard Python data visualization libraries (matplotlib and Seaborn), and attempt to emulate the data visualization best practices laid out by Edward Tufte in his excellent 1983 book [The Visual Display of Quantitative Information](https://www.edwardtufte.com/tufte/books_vdqi). I hope to get around to writing a blog post discussing some of the information contained within this book later in the year.

Code snippets are provided here but I have omitted some for brevity, the full notebooks can be found [on my Github](https://github.com/jmoro0408/DataVis/tree/main/UK%20Temperature).

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
| 1957 | 1     | 8.7       | 2.7      | 5  | 39.5      | 53.0      |
| 1958 | 2     | 8.9       | 1.9      | 10 | 58.7      | 45.7      |
|------|-------|-----------|----------|----|-----------|-----------|

## Cleaning

Before exploring the data we perform some basic cleaning:
1. converting "---" to np.nan values,
2. removing footnote asterisks,
3. mapping month numbers to names,
4. converting series values to their appropriate datatypes

Post cleaning we can see that the data for each site is comprised of the following:

| df.info()|                |                 |          |
| -------  | -------------- | --------------- | ---------|
| Column   | Braemar count  | Heathrow count  |   Dtypes |
| -------  | -------------- | --------------- | ---------|
| mm       | 770            | 902             |  object  |
| tmax     | 766            | 902             |  float64 |
| yyy      | 770            | 902             |  int64   |
| tmin     | 766            | 902             |  float64 |
| af       | 766            | 890             |  float64 |
| rain     | 743            | 902             |  float64 |
| sun      | 552            | 794             |  float64 |

Heathrow records start much earlier than Braemar which is why there are ~150 more records.

## Plotting

Let's look at some trends over time, starting with the minimum and maximum temperature for both sites:

<details>
  <summary>Code Snippet</summary>

  ```python
  fig, ax = plt.subplots(figsize = (12,10))
sns.lineplot(data = heathrow_df,
             y = 'tmax',
             x = 'mm',
             hue = 'yyyy',
             palette= ['gray'],
             legend = None,
             alpha = ALPHA,
             ax = ax)

sns.lineplot(data = heathrow_df[heathrow_df['yyyy'].isin([2022, 2021, 2020,2019])],
             y = 'tmax',
             x = 'mm',
             hue = 'yyyy',
             palette= COLORS,
             ax = ax)

sns.lineplot(data = heathrow_df,
             y = 'tmin',
             x = 'mm',
             hue = 'yyyy',
             palette= ['gray'],
             legend = None,
             alpha = ALPHA,
             ax = ax)

sns.lineplot(data = heathrow_df[heathrow_df['yyyy'].isin([2022, 2021, 2020,2019])],
             y = 'tmin',
             x = 'mm',
             hue = 'yyyy',
             legend = None,
             style = 'yyyy',
             palette= COLORS,
             ax = ax)

ax.set_ylabel("Temperature")
ax.set_xlabel("Month")
ax.set_title("Heathrow Min and Max Temperature from 1959 - 2023")

  ```
</details>

![Braemar min max lineplot](/images/blog_temperature/Braemar/min_max_lineplot.svg "Braemar min max lineplot")

![Heathrow min max lineplot](/images/blog_temperature/Heathrow/min_max_lineplot.svg "Heathrow min max lineplot")

Here, the latest four years are highlighted, and we can clearly see that for both min and max the latest years are on the upper end of the spectrum, as would be expected with climate change.


This can be seen more clearly by plotting the boxplots for each month and highlighting the recent years.

<details>
  <summary>Code Snippet</summary>

```python
melted = heathrow_df.melt(id_vars = 'mm', value_vars='tmax')
fig, ax = plt.subplots(figsize = (10,8))

PROPS = {
    'boxprops':{'facecolor':'none', 'edgecolor':'black'},
}

sns.boxplot(data = melted,
            x = 'mm',
            y = 'value',
            ax = ax,
            zorder = 0,
            **PROPS)

sns.scatterplot(data = heathrow_years_tmax,
                x = 'mm',
                hue = 'yyyy',
                y = 'tmax',
                style = 'yyyy',
                palette = COLORS,
                s = MARKERSIZE,
                zorder = 1)

sns.scatterplot(data = heathrow_max_temp_year_month_df,
                x = 'mm',
                y = 'tmax',
                color = 'black',
                label = 'Year of\nhighest temp',
                marker = "^",
                s = MARKERSIZE,
                zorder = 1)

arrowprops=dict(arrowstyle="-",color = 'black',
                            connectionstyle="arc3")

for i in range(len(heathrow_max_temp_year_month_df)):
    skip_months = ['Aug', 'Oct', 'Nov']
    if heathrow_max_temp_year_month_df.iloc[i]['mm'] in (skip_months):
        pass
    else:
        ax.annotate(text = heathrow_max_temp_year_month_df.iloc[i]['yyyy'],
                xy = (heathrow_max_temp_year_month_df.iloc[i]['mm'],
                    heathrow_max_temp_year_month_df.iloc[i]['tmax']),
                xytext = (heathrow_max_temp_year_month_df.iloc[i]['mm'],
                    heathrow_max_temp_year_month_df.iloc[i]['tmax']+1),
                arrowprops=arrowprops,
                ha='center'
                )

multiple_month_df = (heathrow_max_temp_year_month_df[heathrow_max_temp_year_month_df['mm']
                                                     .isin(skip_months)]
                     .sort_values(by = 'mm')
                     .reset_index())

ax.annotate(text = multiple_month_df.iloc[0]['yyyy'],
                xy = (multiple_month_df.iloc[0]['mm'],
                    multiple_month_df.iloc[0]['tmax']),
                xytext = (multiple_month_df.iloc[0]['mm'],
                    multiple_month_df.iloc[0]['tmax']+1),
                arrowprops=arrowprops,
                ha='left'
                )

ax.annotate(text = multiple_month_df.iloc[1]['yyyy'],
                xy = (multiple_month_df.iloc[1]['mm'],
                    multiple_month_df.iloc[1]['tmax']),
                xytext = (multiple_month_df.iloc[1]['mm'],
                    multiple_month_df.iloc[1]['tmax']+2),
                arrowprops=arrowprops,
                ha='right'
                )

ax.annotate(text = multiple_month_df.iloc[2]['yyyy'],
                xy = (multiple_month_df.iloc[2]['mm'],
                    multiple_month_df.iloc[2]['tmax']),
                xytext = (multiple_month_df.iloc[2]['mm'],
                    multiple_month_df.iloc[2]['tmax']+1),
                arrowprops=arrowprops,
                ha='left'
                )

ax.annotate(text = multiple_month_df.iloc[3]['yyyy'],
                xy = (multiple_month_df.iloc[3]['mm'],
                    multiple_month_df.iloc[3]['tmax']),
                xytext = (multiple_month_df.iloc[3]['mm'],
                    multiple_month_df.iloc[3]['tmax']+2),
                arrowprops=arrowprops,
                ha='right'
                )

ax.annotate(text = multiple_month_df.iloc[4]['yyyy'],
                xy = (multiple_month_df.iloc[4]['mm'],
                    multiple_month_df.iloc[4]['tmax']),
                xytext = (multiple_month_df.iloc[4]['mm'],
                    multiple_month_df.iloc[4]['tmax']+1),
                arrowprops=arrowprops,
                ha='right'
                )

ax.annotate(text = multiple_month_df.iloc[5]['yyyy'],
                xy = (multiple_month_df.iloc[5]['mm'],
                    multiple_month_df.iloc[5]['tmax']),
                xytext = (multiple_month_df.iloc[5]['mm'],
                    multiple_month_df.iloc[5]['tmax']+2),
                arrowprops=arrowprops,
                ha='left'
                )



ax.set_ylabel("Temperature (C)")
ax.set_ylim(0,32)
ax.set_xlabel("")
ax.set_title("Heathrow Monthly Max Temperature Boxplots 1959 - 2023")

if SAVE:
    plt.savefig(Path(save_dir,
                     "tmax_boxplot.svg"),
                     format = "svg",
                     facecolor = 'white',
                     dpi = 300)

  ```
</details>

![Braemar max boxplot](/images/blog_temperature/Braemar/tmax_boxplot.svg "Braemar max boxplot")

![Braemar max boxplot](/images/blog_temperature/Braemar/tmin_boxplot.svg "Braemar max boxplot")

![Heathrow max boxplot](/images/blog_temperature/Heathrow/tmax_boxplot.svg "Heathrow max boxplot")

![Heathrow max boxplot](/images/blog_temperature/Heathrow/tmin_boxplot.svg "Heathrow max boxplot")

Here we can see clearly how often the latest years are on the upper end of the boxplots. For heathrow, the maximum temperatures for 10/12 months have all occurred within the last 20 yers. As the lowest recorded temperatures are typically 40+ years ago, it can be inferred that London is heating up overt time.