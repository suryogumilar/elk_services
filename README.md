# Elastik Search Stack

Docker compose file dan folder konfigurasi untuk menjalankan elastic search services.   
ELK = Elasticsearch, Kibana, Logstash.

#### Logstash environment
untuk menjalankan docker-compose terlebih dahulu set environment workdir yang berisi log untuk diproses
```bash
# pada windows
SET workdir=C:\Worksite\

# pada linux
workdir=C:\Worksite\

```

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

Cek melalui [http://localhost:9200/_cat/health](http://localhost:9200/_cat/health?pretty=true) untuk health status  
Cek melalui [http://localhost:9200/_cat/indices](http://localhost:9200/_cat/indices) untuk daftar index yg terdapat di elastic   
Cek melalui [http://127.0.0.1:9200/_cat/nodes?v&pretty](http://127.0.0.1:9200/_cat/nodes?v&pretty) untuk daftar nodes dan loadnya   
Untuk info cluster, jenis dan versi menggunakan [http://127.0.0.1:9200/](http://127.0.0.1:9200/)

Cek juga pada log:   
`docker-compose --project-name elk_services -f ./docker-compose-elk.yml logs --timestamp --follow elasticsearch-service`

#### Akses Kibana
Kibana bisa diakses di [http://localhost:5601](http://localhost:5601)

### Grok
grok didefinisikan pada filter di logstash.conf 
#### Contoh dari grok pattern   

pattern grok untuk TIME:   
`%{YEAR:year}-%{MONTHNUM:month}-%{MONTHDAY:date}` applied to string: `2021-01-29` maka struktur datanya menjadi:
```
{
  "date": "29",
  "month": "01",
  "year": "2021"
}
```
##### menggunakan custom pattern
pattern timestamp   
jika kita definisikan pattern:   
`MYDATESTAMP %{YEAR:year}-%{MONTHNUM:month}-%{MONTHDAY:date}`   
dan pada grok :   
`%{MYDATESTAMP:datetimestamp}` applied to string: `2021-01-29` maka struktur datanya menjadi:
```
{
  "date": "29",
  "month": "01",
  "year": "2021",
  "datetimestamp": "2021-01-29"
}
```
namun jika pattern yang didefinisikan :   
`MYDATESTAMP %{YEAR}-%{MONTHNUM}-%{MONTHDAY}`   
dan pada grok :   
`%{MYDATESTAMP:datetimestamp}` applied to string: `2021-01-29` maka struktur datanya menjadi:
```
{
  "datetimestamp": "2021-01-29"
}
```
   
jika pattern yang didefinisikan :   
`MYDATESTAMP %{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME}`   
dan pada grok :   
`%{MYDATESTAMP:datetimestamp}` applied to string: `2021-01-29 17:08:49,025` maka struktur datanya menjadi:
```
{
  "datetimestamp": "2021-01-29 17:08:49,025"
}
```

Untuk pattern:   
`%{MYDATESTAMP:datetimestamp}%{SPACE}%{LOGLEVEL:loglevel}%{SPACE}%{WORD:service}%{SPACE}l:%{INT:linenumber}%{SPACE}%{GREEDYDATA:logmessage}`   
diaplikasikan ke string:   
`2021-01-29 17:08:49,024   DEBUG    unmask_neta l:88 query done`   
maka struktur datanya:   
```
{
  "linenumber": "88",
  "service": "unmask_neta",
  "loglevel": "DEBUG",
  "logmessage": "query done",
  "datetimestamp": "2021-01-29 17:08:49,024"
}
```

#### Grok Debuger
bisa diakses melalui:   
[http://localhost:5601/app/dev_tools#/grokdebugger](http://localhost:5601/app/dev_tools#/grokdebugger)

## referensi

 - https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html
 - https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-options.html
 - https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html

### grok
 - https://grokdebug.herokuapp.com/patterns#