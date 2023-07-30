# Mosquitto InfluxDB Setup

## Install

```bash
git clone git@github.com:maxjuhlke/mosquitto-influxdb-grafana.git
cd mosquitto-influxdb-grafana
```
> Setup the Repos

```bash
docker compose pull
```
> Pull the Images (Installation of `docker-ce` >= 22

```bash
docker compose up -d
```
> Start the Services

## Credentials

### Mosquitto

Mosquitto is configured with default-credentials `user:1q2w3e4r5t!`.
To set your own Credentials edit the config-file under `./mosquitto/pass/password.txt`.
Afterwards you need to encrypt the password. User the Mosquitto Container for this:

```bash
docker exec --entrypoint sh mosquitto mosquitto_passwd -U /mosquitto/pass/password.txt
```
> Encrypts the mqtt-password

### Telegraf

Also edit the Telegraf-Scrape-Config under `./telegraf/config/telegraf.conf`

```bash
sudo editor ./telegraf/conf/telegraf.conf
```

```toml
[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = ""
  debug = false
  quiet = false
  logfile = ""
  hostname = "telegraf"
  omit_hostname = false

[[outputs.influxdb_v2]]
  urls = ["http://influxdb:8086"]
  token = "gPi6WgDODAgjvEmwBeT51AYHH9NZL5ap"
  organization = "mqtt"
  bucket = "mqtt"

[[inputs.mqtt_consumer]]
  servers = ["tcp://mosquitto:1883"]
  username = "user"
  password = "1q2w3e4r5t!"
  topics = [
    "plugs/tasmota_079401/SENSOR",
    "plugs/tasmota_17CA0D/SENSOR"
  ]

  data_format = "json_v2"
  [[inputs.mqtt_consumer.json_v2]]
    [[inputs.mqtt_consumer.json_v2.object]]
      path = "ENERGY"
```
> edit `inputs.mqtt_consumer.username` and `inputs.mqtt_consumer.password`

The telegraf Config also holds the `outputs.influxdb_v2.token` for writing into the InfluxDB, which is then Scraped by Grafana.

Also change `inputs.mqtt_consumer.topics` to your mqtt-devices.

### InfluxDB

InfluxDB is configured with default-credentials `user:1q2w3e4r5t!`.

If you want to change these Update it in the `docker-compose.yml` under:

```yaml
services.infludb.environment.[DOCKER_INFLUXDB_INIT_USERNAME]
services.infludb.environment.[DOCKER_INFLUXDB_INIT_PASSWORD]
```

### Grafana

Grafana is not configured. You can Login with `admin:admin`. Grafana prompts you to change this on 1st Login.

Also there are no default Dashboards. Currently you are required to build your own Dashboard. This is easily done via the InfluxDB Data-Explorer. Then export the Script from InfluxDB to Grafna.
