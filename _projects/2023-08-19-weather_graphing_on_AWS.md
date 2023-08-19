---
title: "UK Weather - Unleashing Insights with AWS"
subtitle: "Using AWS microservices to automatically deliver insights on the weather in the UK"
date: 2023-08-19 00:00:00
description: "Garnering hands-on experience with Amazon Web Services' Lambda, S3, EC2, and EventBridge microservices."
featured_image: london_weather.jpg
accent_color: '#39bf8e'
gallery_images:
  - london_weather.jpg
---

# TLDR; 
Didn't know any AWS before, know some AWS now. 
Built a pipeline with AWS to automatically pull weather data from the met office, visualized with streamlit. 
Check out the Streamlit app [here](https://weathergraphing-n3afxbnpm3dp7rdkmtot8r.streamlit.app/).

## Background
I've managed to gain experience with Microsoft's cloud offerings (Azure) through a mix of professional work and personal upskilling (I completed the Azure Data Scientist Associate certification last March), and have dabbled with GCP. However throughout my journey the use of Amazon Web Services (AWS) has somehow evaded me, despite AWS having 32% of the cloud market share [(source 2023/08/08)](https://www.statista.com/chart/18819/worldwide-market-share-of-leading-cloud-infrastructure-service-providers/).

<img src="https://github.com/jmoro0408/jmoro0408.github.io/blob/master/images/weather_AWS/cloud_market_share.jpeg?raw=true" alt="aws_market_share" width="500"/>

With this fact in mind, I decided to gain some experience with some of AWS's basic features. 

I recently stumbled upon a great statistics blog by Nigel Marriott [(link)](https://marriott-stats.com/nigels-blog/), and was inspired to try and recreate some of the weather graphs he creates every month, using AWS to automatically pull and update the data as it is published every month. As the creation of the visuals is fairly simple, this would allow me to focus mainly on the AWS and automation side of things.

The plots also provide some interesting insights - often a month will feel very warm/cold/wet, but how does it actually statistically compare to historic months? By plotting the min/10th/mean/90th/max values for various weather trends by month, we can check how the latest month's value compares. 

##  Methodology
- The met office publishes data on a monthly basis [(link)](https://www.metoffice.gov.uk/research/climate/maps-and-data/uk-and-regional-series), consisting of the UK average values for:
  - Maximum temperature
  - Minimum temperature
  - Average temperature
  - Hours of sunshine
  - mm of rainfall

- The data is in .txt format and ranges from the late 1800s to the previous month. 

After reading about AWS's offerings, I came up with the following pipeline to automatically ingest, transform, and analyse new data as it is published. 

![Flow Diagram](https://raw.githubusercontent.com/jmoro0408/jmoro0408.github.io/d887f7069535394a74f5fe706c4e076db52d9c13/images/weather_AWS/weather_flow_flow_dark.svg)

1. An EventBridge scheduler runs on the 1st of every month. This:
2. Triggers a Lambda function to turn on an EC2 instance which:
3. On startup, runs a bash script to `cd` into the appropriate folder, activate a virtual environment, and run a python script which in turn:
   1. Downloads the latest weather data from the met office
   2. Saves the raw `.txt` files in a temp folder
   3. Moves the raw files to an S3 bucket
   4. Deletes the temp files

4. After 15 minutes (plenty of time for the script to run to completion), a second EventBridge schedule runs to:
5. Trigger a second lambda function which:
6. Terminates the EC2 instance.

This pipeline allows for new data to be ingested and stored in an S3 bucket on a monthly basis. 

Once the data is in the bucket, I used a python script hosted on Streamlit cloud to grab it, undertake the necessary transformations and create interactive plots with plotly. 

## Outputs
Examples of rendered charts are shown below (you may need to refresh the page). 
Note these are static to July 2023, to see live updates please visit the streamlit [app](https://weathergraphing-n3afxbnpm3dp7rdkmtot8r.streamlit.app/).

{% include Max_Temp.html max-width="200px" %}
{% include Rainfall.html max-width="200px" %}
{% include Sunshine.html max-width="200px" %}

Here, the mean and 90% deciles are plotted, along with min/max values. Values for the current year provided as markers, with marker shape aligned to which decile the value lies within.


## Outcomes

Navigating my way around AWS took some work due to the sheer number of services available, but this was a very interesting exploration into some of their most basic offerings. Next up, deploying an ML solution on AWS!