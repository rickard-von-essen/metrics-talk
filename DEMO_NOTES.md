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
telegraf config > /tmp/telegraf.conf; vim /tmp/telegraf.conf
telegraf --config-directory ~/.telegraf.d
telegraf --test --input-filter system
```

InfluxDB
========

Schema Exploration
------------------

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
kapacitor list tasks

```
