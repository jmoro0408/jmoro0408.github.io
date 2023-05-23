---
title: Parkrun Event Notifier
subtitle: Using Python and Apache Airflow to detect and notify new Parkrun events
date: 2023-04-20 00:00:00
description: Trying to collect all the parkrun events in a specific city? This workflow emails users every Sunday of newly published parkrun events and their locations.
featured_image: jogging_unsplash.jpg
accent_color: '#4C60E6'
gallery_images:
  - jogging_unsplash.jpg
---

## Background

[Parkrun](https://www.parkrun.org.uk/) is a free, community event where you can walk, jog, run, volunteer or spectate. parkrun is 5k and takes place every Saturday morning, junior parkrun is 2k, dedicated to 4-14 year olds and their families, every Sunday morning.

There are currently more than 2,000 parkrun locations taking part in over 22 countries.

To try and "complete" different parkrun challenges is fairly common, including running all the events within a city, or trying to complete a different event for each letter of the alphabet.

New parkrun events are always springing up, however a list of new events is not published by parkrun itself, mainly to avoid very large turnouts to new events.

## Project


![architecture diagram](https://raw.githubusercontent.com/jmoro0408/parkrun/main/readme_visuals/architecture.png)

The workflow automatically runs weekly, every Sunday morning. It uses the Python [requests](https://requests.readthedocs.io/en/latest/) library to scrape parkrun event json data and store it locally.

All UK events are then filtered out and the latest event data compared to that of the previous week. All new events are collected and their locations plotted on an interactive map of the UK, as shown below: (you may need to refresh the page to load the interactive map).

{% include parkrun_map.html %}


Finally the new events are emailed via Gmail along with the interactive map to a distribution list.

These tasks are orchestrated via [Apache Airflow](https://airflow.apache.org/docs/apache-airflow/stable/index.html), contained within
Docker, hosted on a raspberry pi.
This is almost certainly overkill for what could feasibly automated with a simple cron job,
however it was a good opportunity to expand both my Docker and Airflow knowledge.

## Notes on the environment

Note that in order to run this on your own system, several credentials are required, and a custom Airflow container needs to be built.
A credentials.toml file is used to hold the following private (and some public) information:
*  parkrun json access point url
* save directories for jsons and UK-filtered jsons
* mapbox token for plotly - [see here for more info](https://plotly.com/python/mapbox-layers/)
* gmail credentials -
  * sender address
  * sender password - note this is an "app password", **not** your usual login password. [See here for more info](https://stackoverflow.com/questions/33286691/gmail-smtp-requires-an-app-password)
  * recipiant addresses - defined as an array in the .toml

A custom Docker image also needs to be built. I've included the docker-compose yaml file in the [github repository](https://github.com/jmoro0408/parkrun). This is just the standard apache airflow image extended to include requests, plotly, pandas, and tomli.

If you want to recreate this project and have questions, just send me a message via the "Get in touch" button at the top of this page.




