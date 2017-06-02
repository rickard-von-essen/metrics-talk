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

Chronograf
==========

Kapacitor
=========

