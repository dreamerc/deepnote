# IFTTT Replacement

## Softwares

* [Apache Airflow](https://airflow.apache.org/)
* [Huginn](https://github.com/huginn/huginn)
* [Active Workflow](https://github.com/automaticmode/active_workflow)
* [Node Red](https://nodered.org/)

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

# Node Red
- preconfig (rootless)
```bash
chown 100999:100999 docker-node_red
```
- docker run
```bash
docker run -it --rm -p 1880:1880 -v ~/docker-node_red:/data:rw --name nodered nodered/node-red
```