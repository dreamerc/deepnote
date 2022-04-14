# IFTTT Replacement

## Softwares

* [Apache Airflow](https://airflow.apache.org/)
* [Huginn](https://github.com/huginn/huginn)
* [Active Workflow](https://github.com/automaticmode/active_workflow)
* [Node Red](https://nodered.org/)
* [Home Assistant](https://www.home-assistant.io/) , with support

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

Clear stuck job (Problem Cause : Database IO Lock) , or set **Keep events** lower as possible (1~2 days).

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
- preconfig (rootless)
```bash
chown 100999:100999 docker-node_red
```
- docker run
```bash
docker run -it --rm -p 1880:1880 -v ~/docker-node_red:/data:rw --name nodered nodered/node-red
```

# Home Assistant
- Google , Alexa , Apple support with payment
```bash
docker pull ghcr.io/home-assistant/home-assistant:stable
docker run -it --rm --name homeassistant -v ~/docker-home-assistant/config:/config -v /etc/localtime:/etc/localtime:ro -p 8123:8123 ghcr.io/home-assistant/home-assistant:stable
```