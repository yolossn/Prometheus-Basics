# Prometheus-Basics
A beginner friendly introduction to prometheus.




## Table of Contents


# What is prometheus ?

Prometheus is a system monitoring and alerting system. It was opensourced by SoundCloud in 2012 and was incubated by Cloud Native Computing Foundation. Prometheus stores all the data as time series, i.e metrics information is stored along with the timestamp at which it was recorded, optional key-value pairs called as labels can also be stored along with metrics. 

# What is metrics and why should I worry about it ?

Metrics in layman terms is a standard for measurement. What we want to measure depends from application to application. For a web server it can be request times, for a database it can be CPU usage or number of active connections etc. 

Metrics play an important role in understanding why your application is working in a certain way. If you run a web application and someone comes up to you and says that the application is slow. You will need some information to find out what is happening with your application. For example the application can become slow when the number of requests are high. If you have the request count metric you can spot the reason and increase the number of servers to handle the heavy load. Whenever you are defining the metrics for your application you must put on your detective hat and ask this question **what all information will be important for me to help debug if any issue occurs in my application?**

Analogy: I always use this analogy to simply understand what a monitoring system does. When I was young I wanted to grow tall and to measure it I used height as a metric. I asked my dad to measure my height everyday and keep a table of my height on each day. So in this case my dad is my monitoring system and the metric was my height.

# Basic Architecture of Prometheus

The basic components of prometheus are:
- Prometheus Server (The server which scraps and stores the metrics data).
- Client Library which is used to calculate and expose the metrics.
- Alert manager to raise alerts based on preset rules.
  
(Note: Apart from this prometheus has push gateways which I am not covering here).

<p align="center">
  <img width="751" height="281" src="https://github.com/yolossn/Prometheus-Basics/blob/master/images/architecture.png">
</p>


So there is an application for example let us consider it to be a web server. I want to extract certain metric like the number of api calls processed by my web server. So I add certain instrumentation code using the prometheus client library and expose the metrics information. Now that my web server also exposes its metrics I want it to be scraped by prometheus. So I run prometheus in separately and configure it to fetch the metrics from the web server which is listening on xyz IP address port 7500 at specific time interval say every minute. 


I set prometheus up and expose my web server accessible for the frontend or other client to use.


At 11:00 PM when I make my web server open to consumption. Prometheus scrapes the count metric and stores the value as 0

By 11:01 PM one request is processed by my web server . The instrumentation logic in my code makes the count to 1. When prometheus scraps the metric the value of count is 1 now

By 11:02 PM two requests are processed by my web server and the request count is 1+2 = 3 now. Similarly metrics are scraped and stored

The user can control the frequency at which metrics are scraped by prometheus


Time Stamp | Request Count (metric)
-----------| -------------
11:00 PM   | 0
11:01 PM   | 1
11:02 PM   | 3

(Note: This table is just a representation for understanding purposes. Prometheus doesn’t store the values in the exact format)

Prometheus also has a server which exposes the metrics which are stored by scraping. This server is used to query the metrics, create dashboards/charts on it etc.

A simple Line chart created on my Request Count metric will look like this

<p align="center">
  <img width="580" height="400" src="https://github.com/yolossn/Prometheus-Basics/blob/master/images/graph.png">
</p>

I can scrape multiple metrics which will be useful to understand what is happening in my application and create multiple charts on them. Group the charts into a dashboard and use it to understand what is happening in my application.

# Show me how it is done.

Let’s get our hands dirty and setup prometheus. Prometheus is written using golang and all you need is the binary compiled for your operating system. Download the binary corresponding to your operating system from [here](https://prometheus.io/download/). 

Prometheus exposes its own metrics which can be consumed by itself or another prometheus server.

Now that we have Prometheus, the next step is to run it. All that we need is just the binary and a configuration file. Prometheus uses yaml files for configuration.

*prometheus.yml*
```
global:
 scrape_interval: 15s
 
scrape_configs:
 - job_name: prometheus
   static_configs:
       - targets: ["localhost:9090"]
```

In the above configuration file we have mentioned the scrape_interval i.e how frequently you want prometheus to scrape the metrics. We have added scrape_configs which has a name and target to scrape the metrics from. Prometheus by default listens on port 9090. So I have added it.

<p align="center">
  <img width="580" height="400" src="https://github.com/yolossn/Prometheus-Basics/blob/master/images/prometheus1.gif">
</p>

Now we have Prometheus up and running and scraping its own metics every 15s. Prometheus has standard exporters available to export metrics. Next we will run a node exporter which is an exporter for machine metrics and scrape the same using prometheus.
Download node metrics exporter from [here](https://prometheus.io/download/#node_exporter)

There are many standard exporters available like node exporter you can find them [here](https://github.com/roaldnefs/awesome-prometheus#exporters)

**Run the node exporter in a terminal.**
> ./node_exporter

<p align="center">
  <img width="1600" height="276" src="https://github.com/yolossn/Prometheus-Basics/blob/master/images/node_exporter.png">
</p>

Next Add node exporter to the list of scrape_configs

*node_exporter.yml*
```
global:
 scrape_interval: 15s
 
scrape_configs:
 - job_name: prometheus
   static_configs:
       - targets: ["localhost:9090"]
 - job_name: node_exporter
   static_configs:
       - targets: ["localhost:9100"]
```
<p align="center">
  <img width="580" height="400" src="https://github.com/yolossn/Prometheus-Basics/blob/master/images/prometheus2.gif">
</p>


# Type of metrics.

There are four types of metrics which prometheus client libraries support as of May 2020. They are:
1. Counter 
2. Gauge
3. Histogram
4. Summary

## Counter:

Counter is a metric value which can only increase or reset i.e the value cannot reduce than the previous value. It can be used for metrics like number of requests, no of errors etc.

> go_gc_duration_seconds_count
<p align="center">
  <img width="800" height="560" src="https://github.com/yolossn/Prometheus-Basics/blob/master/images/counter_example.png">
</p>

The rate() function takes the history of metrics over a time frame and calculates how fast value is increasing per second. Rate is applicable on counter values only.

> rate(go_gc_duration_seconds_count[5m])

<p align="center">
  <img width="800" height="560" src="https://github.com/yolossn/Prometheus-Basics/blob/master/images/rate_example.png">
</p>

## Gauge:
    
Gauge is a number which can either go up or down. It can be used for metrics like number of pods in a cluster, number of events in an queue etc.

> go_memstats_heap_alloc_bytes

<p align="center">
  <img width="800" height="560" src="https://github.com/yolossn/Prometheus-Basics/blob/master/images/gauge_example.png">
</p>

PromQL functions like max_over_time, min_over_time and avg_over_time can be used to query gauge metrics

## Histogram

Histogram is a little complex metric type when compared to the ones we have seen. Histogram can be used for any calculated value which is counted based on bucket values, the buckets can be configured by the user. It is a cumulative metric and provides sum of all the values by default. This is applicable for metrics like request time, cpu temperature  etc

Example:
I want to observe the time taken to process api  requests. Instead of storing the request time for each request, histogram metric allows us to approximate and store frequency of requests which fall into particular buckets. I define buckets for time taken like 0.3,0.5,0.7,1,1.2. So these are my buckets and once the time taken for a request is calculated I add the count to all the buckets which are higher than the value.

Lets say Request 1 for endpoint “/ping” takes 0.25 s. The count values for the buckets will be. 

> /ping

Bucket     | Count
-----------| -------------
0 - 0.3    | 1
0.3 - 0.5  | 1
0.5 - 0.7  | 1
0.7 - 1    | 1
1 - 1.2    | 1
+Inf       | 1
Note: +Inf bucket is added by default.


(Since histogram is a cumulative frequency 1 is added to all the buckets which are greater than the value)

Request 2 for endpoint “/ping” takes 0.4s The count values for the buckets will be this.

> /ping

Bucket     | Count
-----------| -------------
0 - 0.3    | 1
0.3 - 0.5  | 2
0.5 - 0.7  | 2
0.7 - 1    | 2
1 - 1.2    | 2
+Inf       | 2

Since 0.4 lies in the seconds bucket(0.3-0.5) all the buckets above the calculated values count is increased. 
Histogram is used to find average and percentile values.

> prometheus_http_request_duration_seconds_bucket{handler="/graph"}

<p align="center">
  <img width="800" height="560" src="https://github.com/yolossn/Prometheus-Basics/blob/master/images/histogram_example.png">
</p>

histogram_quantile() function can be used to calculate calculate quantiles from histogram

<p align="center">
  <img width="800" height="560" src="https://github.com/yolossn/Prometheus-Basics/blob/master/images/histogram_quantile_example.png">
</p>

The graph shows that the 90th percentile is 0.09, To find the histogram_quantile over last 5m you can use the rate() and time frame

> histogram_quantile(0.9, rate(prometheus_http_request_duration_seconds_bucket{handler="/graph"}[5m]))

<p align="center">
  <img width="800" height="560" src="https://github.com/yolossn/Prometheus-Basics/blob/master/images/histogram_rate_example.png">
</p>




