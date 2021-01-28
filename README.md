# Elastik Search Stack

Docker compose file dan folder konfigurasi untuk menjalankan elastic search services.   
ELK = Elasticsearch, Kibana, Logstash.

### Run containers   
How to run:   
`docker-compose --project-name elk_services -f ./docker-compose-elk.yml up -d`

### Stop containers
also Remove containers for services not defined in the Compose file and named volumes declared in the `volumes` section of the Compose file and anonymous volumes attached to containers.   
`docker-compose --project-name elk_services -f ./docker-compose-elk.yml down --remove-orphans --volumes`
   
How to stop and remove containers  as well as any networks that were created:   
`docker-compose --project-name elk_services -f ./docker-compose-elk.yml down`   
   
How to stop but do not remove containers:   
`docker-compose --project-name elk_services -f ./docker-compose-elk.yml stop`   

#### Jalankan service satu-satu:
##### Elastic service
`docker-compose --project-name elk_services -f ./docker-compose-elk.yml up --detach elasticsearch-service`

##### Kibana service
`docker-compose --project-name elk_services -f ./docker-compose-elk.yml up --detach kibana-service`


### Check services

Cek melalui [http://localhost:9200/_cat/health](http://localhost:9200/_cat/health) untuk health status  
Cek melalui [http://localhost:9200/_cat/indices](http://localhost:9200/_cat/indices) untuk daftar index yg terdapat di elastic

Cek juga pada log:   
`docker-compose --project-name elk_services -f ./docker-compose-elk.yml logs --timestamp --follow elasticsearch-service`

#### Akses Kibana
Kibana bisa diakses di [http://localhost:5601](http://localhost:5601)


## referensi

 - https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html
 - https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-options.html
 - https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html