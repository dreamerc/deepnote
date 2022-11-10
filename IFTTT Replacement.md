# IFTTT Replacement

## Software

- [Apache Airflow](https://airflow.apache.org/)
- [Huginn](https://github.com/huginn/huginn)
- [Active Workflow](https://github.com/automaticmode/active_workflow)
- [Node Red](https://nodered.org/)
- [Home Assistant](https://www.home-assistant.io/), with support (Smart Home Devices)
- [Domoticz](https://domoticz.com/) (Smart Home Devices)

### Huginn

#### Docker Install (Development)

```bash
$ docker run -d --rm -v ~/docker-huginn:/var/lib/mysql --name huginn huginn/huginn
```
#### Docker install (local Production)
```bash
git clone https://github.com/huginn/huginn.git
```
then add the patch :
```diff
diff --git a/Procfile b/Procfile
index 266e43a8..f6ea3623 100644
--- a/Procfile
+++ b/Procfile
@@ -3,8 +3,8 @@
 ###############################
 
 # Procfile for development using the new threaded worker (scheduler, twitter stream and delayed job)
-web: bundle exec rails server -p ${PORT-3000} -b ${IP-0.0.0.0}
-jobs: bundle exec rails runner bin/threaded.rb
+#web: bundle exec rails server -p ${PORT-3000} -b ${IP-0.0.0.0}
+#jobs: bundle exec rails runner bin/threaded.rb
 
 # Old version with separate processes (use this if you have issues with the threaded version)
 # web: bundle exec rails server
@@ -21,8 +21,8 @@ jobs: bundle exec rails runner bin/threaded.rb
 # https://github.com/huginn/huginn/doc
 
 # Using the threaded worker (consumes less RAM but can run slower)
-# web: bundle exec unicorn -c config/unicorn.rb
-# jobs: bundle exec rails runner bin/threaded.rb
+web: bundle exec unicorn -c config/unicorn.rb
+jobs: bundle exec rails runner bin/threaded.rb
 
 # Old version with separate processes (use this if you have issues with the threaded version)
 # web: bundle exec unicorn -c config/unicorn.rb
diff --git a/app/concerns/web_request_concern.rb b/app/concerns/web_request_concern.rb
index 331dfea4..ead57151 100644
--- a/app/concerns/web_request_concern.rb
+++ b/app/concerns/web_request_concern.rb
@@ -131,6 +131,8 @@ module WebRequestConcern
         builder.options.params_encoder = DoNotEncoder
       end
 
+      builder.options.timeout = (Delayed::Worker.max_run_time.seconds - 2).to_i
+
       if userinfo = basic_auth_credentials
         builder.request :basic_auth, *userinfo
       end
```
run :
```bash
cp config/unicorn.rb.example bin/unicorn.rb
docker build --rm --tag huginn -f docker/multi-process/Dockerfile .
docker run -d --rm -v ~/huginn:/var/lib/mysql --name huginn huginn
```
#### API
* [Shopify Liquid](https://shopify.dev/api/liquid)
* [Github Liquid](https://shopify.github.io/liquid/)

#### Troubleshooting

Clear stuck job (Problem Cause : Database IO Lock), or set **Keep events** lower than possible (1~2 days).

```bash
$ docker exec -it huginn /bin/bash
Docker$ source .env
Docker$ bundle exec rails console --environment=prod
Rails$ Delayed::Job.where("locked_at IS NOT NULL AND locked_by IS NOT NULL AND failed_at IS NULL").destroy_all
```

MySQL DB lock Monitoring & Suggestion
```bash
docker exec -it huginn /bin/bash -c 'mysql -uroot -ppassword -e "SHOW FULL PROCESSLIST;"'
docker exec -it huginn /bin/bash -c 'mysql -uroot -ppassword -e "show engine innodb status;"'
docker exec -it huginn /bin/bash -c 'mkdir -p ~/tmp/sqltest; curl https://raw.githubusercontent.com/major/MySQLTuner-perl/master/mysqltuner.pl -o ~/tmp/sqltest/tun.pl; perl ~/tmp/sqltest/tun.pl --user root --pass password'
```

MySQL DB Transaction Tunning : 
```bash
docker exec -it huginn /bin/bash -c 'mysql -uroot -ppassword -e "show global variables;"'
docker exec -it huginn /bin/bash -c 'mysql -uroot -ppassword -e "set global max_heap_table_size=536870912;"'
docker exec -it huginn /bin/bash -c 'mysql -uroot -ppassword -e "set global tmp_table_size=536870912;"'
docker exec -it huginn /bin/bash -c 'mysql -uroot -ppassword -e "set global key_buffer_size=49152;"'
docker exec -it huginn /bin/bash -c 'mysql -uroot -ppassword -e "set global interactive_timeout=600;"'
docker exec -it huginn /bin/bash -c 'mysql -uroot -ppassword -e "set global wait_timeout=600;"'
docker exec -it huginn /bin/bash -c 'mysql -uroot -ppassword -e "show global variables;"' 
```

Notice:
1. Default MySQL need tunning in mass events ; reduce events and reduce 

WebClient cause job stuck
https://github.com/huginn/huginn/pull/1892/files

# Apache Airflow
- Docker-compose
```yaml
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
#

# Basic Airflow cluster configuration for CeleryExecutor with Redis and PostgreSQL.
#
# WARNING: This configuration is for local development. Do not use it in a production deployment.
#
# This configuration supports basic configuration using environment variables or an .env file
# The following variables are supported:
#
# AIRFLOW_IMAGE_NAME           - Docker image name used to run Airflow.
#                                Default: apache/airflow:master-python3.8
# AIRFLOW_UID                  - User ID in Airflow containers
#                                Default: 50000
# AIRFLOW_GID                  - Group ID in Airflow containers
#                                Default: 50000
#
# Those configurations are useful mostly in case of standalone testing/running Airflow in test/try-out mode
#
# _AIRFLOW_WWW_USER_USERNAME   - Username for the administrator account (if requested).
#                                Default: airflow
# _AIRFLOW_WWW_USER_PASSWORD   - Password for the administrator account (if requested).
#                                Default: airflow
# _PIP_ADDITIONAL_REQUIREMENTS - Additional PIP requirements to add when starting all containers.
#                                Default: ''
#
# Feel free to modify this file to suit your needs.
---
version: '2.4'
x-airflow-common:
  &airflow-common
  image: ${AIRFLOW_IMAGE_NAME:-apache/airflow:2.2.4}
  environment:
    &airflow-common-env
    AIRFLOW__CORE__EXECUTOR: CeleryExecutor
    AIRFLOW__CORE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
    AIRFLOW__CELERY__RESULT_BACKEND: db+postgresql://airflow:airflow@postgres/airflow
    AIRFLOW__CELERY__BROKER_URL: redis://:@redis:6379/0
    AIRFLOW__CORE__FERNET_KEY: ''
    AIRFLOW__CORE__DAGS_ARE_PAUSED_AT_CREATION: 'true'
    AIRFLOW__CORE__LOAD_EXAMPLES: 'true'
    AIRFLOW__API__AUTH_BACKEND: 'airflow.api.auth.backend.basic_auth'
    _PIP_ADDITIONAL_REQUIREMENTS: ${_PIP_ADDITIONAL_REQUIREMENTS:-}
  volumes:
    - ./dags:/opt/airflow/dags
    - ./logs:/opt/airflow/logs
    - ./plugins:/opt/airflow/plugins
  user: "${AIRFLOW_UID:-50000}:0"
  depends_on:
    redis:
      condition: service_healthy
    postgres:
      condition: service_healthy

services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    volumes:
      - ./postgressql-dada:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "airflow"]
      interval: 5s
      retries: 5
    restart: always

  redis:
    image: redis:latest
    ports:
      - 6379:6379
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 30s
      retries: 50
    restart: always

  airflow-webserver:
    <<: *airflow-common
    command: webserver
    ports:
      - 8080:8080
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://localhost:8080/health"]
      interval: 10s
      timeout: 10s
      retries: 5
    restart: always

  airflow-scheduler:
    <<: *airflow-common
    command: scheduler
    healthcheck:
      test: ["CMD-SHELL", 'airflow jobs check --job-type SchedulerJob --hostname "$${HOSTNAME}"']
      interval: 10s
      timeout: 10s
      retries: 5
    restart: always

  airflow-worker:
    <<: *airflow-common
    command: celery worker
    healthcheck:
      test:
        - "CMD-SHELL"
        - 'celery --app airflow.executors.celery_executor.app inspect ping -d "celery@$${HOSTNAME}"'
      interval: 10s
      timeout: 10s
      retries: 5
    restart: always

  airflow-init:
    <<: *airflow-common
    command: version
    environment:
      <<: *airflow-common-env
      _AIRFLOW_DB_UPGRADE: 'true'
      _AIRFLOW_WWW_USER_CREATE: 'true'
      _AIRFLOW_WWW_USER_USERNAME: ${_AIRFLOW_WWW_USER_USERNAME:-airflow}
      _AIRFLOW_WWW_USER_PASSWORD: ${_AIRFLOW_WWW_USER_PASSWORD:-airflow}

  flower:
    <<: *airflow-common
    command: celery flower
    ports:
      - 5555:5555
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://localhost:5555/"]
      interval: 10s
      timeout: 10s
      retries: 5
    restart: always
```
- postconfig (rootless)
```bash
chown 149999:149999 -R .
```
# Node Red
- Pre-config (rootless)
```bash
chown 100999:100999 docker-node_red
```
- docker run
```bash
docker run -it --rm -p 1880:1880 -v ~/docker-node_red:/data:rw --name nodered nodered/node-red
```

# Home Assistant
- Google, Alexa, Apple support with payment
## Request
- If using Chrome Cast, Apple Cast, and some of those device, those require IP-LAN Broadcast feature (```--network host``` option in docker). Others is using Virtual Machine.
## Run
```bash
docker pull ghcr.io/home-assistant/home-assistant:stable
docker run -it --rm --name homeassistant -v ~/docker-home-assistant/config:/config -v /etc/localtime:/etc/localtime:ro -p 8123:8123 ghcr.io/home-assistant/home-assistant:stable
```
## Community Store
- https://hacs.xyz/

## Hardware
### Wifi

| Name                                                                                      | Spec                                                                                                                                              | Image                     | Software                                                        | Vendor                    | FCC                  |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- | --------------------------------------------------------------- | ------------------------- | -------------------- |
| Yeelight LED 智慧燈泡 W3 (彩光版) <br> Yeelight LED RGB W3 Lightbulb <br> Model : YLDP005 | Wifi 802.11 b/g/n 2.4G <br> E27 (W3) Mount <br> 110V 60Hz 0.13A <br> Maximum : F4000 Color<br> 8W Power with 106lm/W <br> 15,000 Hours Life cycle | ![](images/yeelight_YLDP005.png) | [yeecli](https://gitlab.com/stavros/yeecli) ,<br> yeelight(Offical) | Yeelight <br>(acquire Xiaomi) | NCC : CCAH21LP4160T5 |
| Yeelight LED 智慧燈泡 W3 (色溫版) <br> Yeelight LED W3 Lightbulb <br> Model : YLDP006 | Wifi 802.11 b/g/n 2.4G <br> E27 (W3) Mount <br> 110V 60Hz 0.13A <br> Maximum : F4000 Color<br> 8W Power with 106lm/W <br> 15,000 Hours Life cycle | ![](images/yeelight_YLDP006.png) | [yeecli](https://gitlab.com/stavros/yeecli) ,<br> yeelight(Offical) | Yeelight <br>(acquire Xiaomi) | NCC : CCAH21LP4160T2 
| Wemo WiFi 智慧插座 <br> Wemo WiFi Smart Plug <br> Model : WSP080 | Wifi 802.11 b/g/n 2.4G <br> Electrical Rating : 120V~/15A/60Hz/1800W | ![](images/wemo_WSP080.png) | [wemo-cli](https://github.com/matthewhuie/wemo-cli) ,<br> wemo(Offical) | Belkin Wemo | NCC : CCAI21LP0480T0 |        

- Wemo 
  - add device without auto discovery (DHCP Discovery)

manual with config/configuration.yaml
```yaml
wemo:
  discovery: false
  static:
    - 192.168.the.foo
    - 192.168.the.bar
```

- Yeelight
   - some bulb mount are unable to hold this device, but regular bulbs are fine.

### Zigbee
#### Restriction
- There are Zigbee coordinator (gateway), router (repeater), and device limitation amounts by the chipset.
- Zigbee using 2.4G Hz may cause problem with your Wi-Fi, and reducing the distance of connection.
- Wall and floor can cause the connection going none static strongly. 

#### Devices
- Zigbee Z-Wave
   - Gateway
   - Dongle Gateway (CC2531) - install firmware : [zigbee2mqtt.io/flashing the cc2531](https://www.zigbee2mqtt.io/guide/adapters/flashing/flashing_the_cc2531.html) [https://github.com/Koenkk/Z-Stack-firmware](https://github.com/Koenkk/Z-Stack-firmware)
```bash
# upload coordinator/router firmware
sudo ./cc-tool -e -w firmware.hex
#postconfig
sudo chmod +0666 /dev/ttyACM0
#run rootless docker
docker run -it --rm --name homeassistant -v /dev/ttyACM0:/dev/ttyACM0 \
			-v ~/docker_home-assistant/config:/config \
			-v /etc/localtime:/etc/localtime:ro \
			-p 8123:8123 ghcr.io/home-assistant/home-assistant:stable
```
- Sensor
   - Hack
      - Action Sensor Delay-jumpper [破解小米米家人體感應器反應時間限制](https://droidcookie.blogspot.com/2020/01/zigbeehassiozigbee2mqtt.html) [Video](https://www.youtube.com/watch?v=TAstPtsmjl0)

| Name                                                        | Spec                         | Image                            | Software              | Vendor | FCC                  |
| ----------------------------------------------------------- | ---------------------------- | -------------------------------- | --------------------- | ------ | -------------------- |
| 米家人體感應器 <br> Mi Motion Sensor <br> Model : RTCGQ01LM | Zigbee <br> Battery : CR2450 | ![](images/mi_motion_sensor.png) | Xiaomi Zigbee Gateway | Xiaomi | NCC : CCAK17LP1420T7 |

#### All-in-One
- Home Assistant Core
```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y python3 python3-dev python3-venv python3-pip libffi-dev libssl-dev libjpeg-dev zlib1g-dev autoconf build-essential libopenjp2-7 libtiff5 libturbojpeg0-dev tzdata
sudo useradd -rm homeassistant
sudo -u homeassistant -H -s
cd /srv/homeassistant
python3 -m venv .
source bin/activate

python3 -m pip install wheel

pip3 install homeassistant
pip3 install jinja2==3.0.3
```