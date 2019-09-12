# Logstash Introduction
Logstash is a data collection engine with real-time.
Logstash can user ElasticSearch and Kibana analyze data.

## Install Logstash
If you want install logstash you can :
- yum
```
sudo yum install logstash
```
- rpm
```
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

## Simple run Logstash
This is helloword.
```
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```
This is run more *.config
```
./bin/logstash -f /etc/logstash/conf.d
```

## How Logstash Work

### Inputs
use inputs to get data into logstash.We can use more tool:
- file:reads from a file 
- redis:reads from redis server.
- sql:reads usr SQL from Mysql.
- beats: processes events sent by Beats.

### Filters
Filters is a intermediary processing devices in the Logstash pipeline.
>- grok: parse and structure arbitrary text. Grok is currently the best way in Logstash to parse unstructured log data into something structured and queryable. With 120 patterns built-in to Logstash, it’s more than likely you’ll find one that meets your needs!
>- mutate: perform general transformations on event fields. You can rename, remove, replace, and modify fields in your events.
>- drop: drop an event completely, for example, debug events.
>clone: make a copy of an event, possibly adding or removing fields.
>- geoip: add information about geographical location of IP addresses (also displays amazing charts in Kibana!)

For more information about the available filters, see [Filter Plugins](https://www.elastic.co/guide/en/logstash/6.2/filter-plugins.html).

### Outputs
Outputs are the final phase the Logstash pipeline. An event cna pass through multiple outputs.

>- elasticsearch: send event data to Elasticsearch. If you’re planning to save your data in an efficient, convenient, and easily queryable format… Elasticsearch is the way to go. Period. Yes, we’re biased :)
>- file: write event data to a file on disk.

For more information about the available outputs, see [Output Plugins](https://www.elastic.co/guide/en/logstash/6.2/output-plugins.html).