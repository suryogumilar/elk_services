# Elastik Search Stack

Docker compose file dan folder konfigurasi untuk menjalankan elastic search services.   
ELK = Elasticsearch, Kibana, Logstash.

#### set environment
untuk menjalankan docker-compose terlebih dahulu set environment workdir yang berisi log untuk diproses
```bash
#### untuk elasticsearch environment

# pada windows 
SET elasticsearch_data=./../elastic_search
# pada linux
elasticsearch_data=./../elastic_search


### untuk environment logstash
# pada windows
SET workdir=C:\Worksite\

# pada linux
workdir=C:\Worksite\


## setting port:

### in windows
SET elastic_port1=9200
SET elastic_port2=9300
SET kibana_port=5601
SET logstash_port1=9600
SET logstash_port2=5044

# next, coba pake file .env
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

#### cek config melalui docker-compose
   
`docker-compose --project-name elk_services -f ./docker-compose-elk.yml config`

#### dengan akses url
   
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

### Setting Kibana dashboard
berikut contoh langkah setting dashboard (pie chart presentase)
 - akses [home dashboard kibana](http://localhost:5601/app/home) 
 - akses [index management](http://localhost:5601/app/management/data/index_management/indices)   
   pastikan log dari logstash ada dalam daftar indices
 - akses [index pattern](http://localhost:5601/app/management/kibana/indexPatterns) untuk membuat entry (Create index pattern)   
     - Define an index pattern: pilih index logstash yang tersedia 
     - pilih primary time field yang akan digunakan sebagai global time filter. Lalu klik 'Create Index Pattern'
     - edit fields yang ada dalam index yang telah dibuat
 - akses [dashboard](http://localhost:5601/app/dashboards) untuk membuat tampilan
     - create new dashboard
     - Add an existing or new object to dashboard
     - pilih visualisasinya, misal piechart
     - pilih indexnya, misal logstash
     - add bucket untuk piechart berdasar kebutuhan, misal hitung loglevel error, info dll
         - split slices untuk tampilkan persentase tiap log level
         - pilih terms sebagai agregasi dan pada field pilih keyword
         - orderby adalah metric: count 
         - klik update
         - save

## referensi

 - https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html
 - https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-options.html
 - https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html
 - https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html

### grok
 - https://grokdebug.herokuapp.com/patterns#