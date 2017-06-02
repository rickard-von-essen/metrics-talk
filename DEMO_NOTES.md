Demo Notes
==========

Preparations
------------

```
cd sandbox
./sandbox up
./sandbox enter influxdb
curl https://s3.amazonaws.com/noaa.water-database/NOAA_data.txt -o /tmp/NOAA_data.txt
influx -import -path=/tmp/NOAA_data.txt -precision=s
```

Telegraf
========

```
./sandbox enter telegraf
telegraf config > /tmp/telegraf.conf; vim /tmp/telegraf.conf
telegraf --config-directory ~/.telegraf.d
telegraf --test --input-filter system
```

InfluxDB
========

Schema Exploration
------------------

```
./sandbox enter influx
influx
```

```
help
precision rfc3339

SHOW DATABASES

USE NOAA_water_database
SHOW MEASUREMENTS

SHOW FIELD KEYS FROM "h2o_feet"
SHOW TAG KEYS FROM "h2o_feet"
SHOW TAG VALUES FROM "h2o_feet" WITH KEY = "location"

SHOW SERIES
```

Data Exploration
----------------

```
SELECT * FROM "h2o_feet"
SELECT * FROM "h2o_feet" LIMIT 4

SELECT * FROM "h2o_feet" WHERE "location" = 'santa_monica' LIMIT 4
SELECT * FROM "h2o_feet" WHERE "location" =~ /santa/ LIMIT 4
SELECT * FROM "h2o_feet" WHERE "location" !~ /santa/ LIMIT 4

SELECT * FROM "h2o_feet" GROUP BY "location" LIMIT 4

SELECT MAX("water_level")  FROM "h2o_feet"

SELECT MEAN("water_level") AS mean_water_level_1d FROM "h2o_feet" WHERE time < 1442534400000000000 GROUP BY location, time(1d) LIMIT 2
SELECT MIN("water_level"), MEAN("water_level"), MAX("water_level") FROM "h2o_feet" WHERE time < 1442534400000000000 GROUP BY location, time(1d) LIMIT 2

SELECT SUM("max") FROM (SELECT MAX(water_level) FROM "h2o_feet" GROUP BY location)
```

Retention Policies - RP's
-------------------------

```
SHOW RETENTION POLICIES
```

Continuous Queries - CQ's
-------------------------

```
SHOW CONTINUOUS QUERIES

CREATE CONTINUOUS QUERY "cq_max_usage_percent_10m" ON "telegraf" BEGIN SELECT MAX("usage_percent") AS "max_usage_percent" INTO "docker_container_cpu_10m" FROM "docker_container_cpu" GROUP BY "container_name", TIME(10m) END

CREATE CONTINUOUS QUERY "cq_max_usage_percent_10m" ON "telegraf"
BEGIN
  SELECT MAX("usage_percent") AS "max_usage_percent" INTO "docker_container_cpu_10m" FROM "docker_container_cpu" GROUP BY "container_name", TIME(10m)
END

SELECT * FROM "docker_container_cpu_10m" WHERE "container_name" =~ /influxdb/ LIMIT 10
```

Chronograf
==========

```
open http://localhost:8888
```

Show:

- Host List
- Admin
- Data Explorer
- Dashboards


Kapacitor
=========

Show in Chronograf:

- Alert History
- Alert Rules

```
./sandbox enter kapacitor
```

```
kapacitor help
kapacitor list tasks
```

```
stream
    |from()
        .measurement('cpu')
    |alert()
        .crit(lambda: "usage_idle" < 70)
        .log('/tmp/alerts.log')
```

```
kapacitor define cpu_alert -type stream -tick cpu_alert.tick -dbrp telegraf.autogen
export rid=$(kapacitor record stream -task cpu_alert -duration 20s)
kapacitor list recordings
kapacitor replay -recording $rid -task cpu_alert
```

(Adjust cpu_usage to 100%)

```
kapacitor define cpu_alert -tick cpu_alert.tick
kapacitor replay -recording $rid -task cpu_alert
cat /tmp/alerts.log
```

(Revert back to 70% and add slack)

(Configure Slack)[https://docs.influxdata.com/kapacitor/v1.3/nodes/alert_node/#slack] this is simplest done from Chronograf.

```
stream
    |from()
        .measurement('cpu')
    |alert()
        .crit(lambda: "usage_idle" < 70)
        .stateChangesOnly()
        .message('{{.Level}}: {{.ID}} Idle CPU: {{ index .Fields "value" }}%')
        .id('cpu-idle')
        .slack()
        .channel('#kapacitor')
        .iconEmoji(':smile:')
```

```
kapacitor define cpu_alert -tick cpu_alert.tick
kapacitor enable cpu_alert
while true; do i=0; done
```
