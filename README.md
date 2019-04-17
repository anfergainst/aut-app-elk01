# aut-app-elk01

! https://docs.bitnami.com/containers/how-to/work-with-non-root-containers/
! https://www.elastic.co/guide/en/elasticsearch/reference/6.7/docker.html
! https://hub.docker.com/u/bitnami?page=1

1 - Adicionar o shared folder no docker...
/bitnami

6.7.0 por causa do plugin ReadOnlyRest!!!

! https://hub.docker.com/r/bitnami/elasticsearch
! https://hub.docker.com/r/bitnami/kibana
! https://hub.docker.com/r/bitnami/grafana
! https://hub.docker.com/_/logstash -> SYSLOG: https://www.elastic.co/guide/en/logstash/current/config-examples.html
docker pull bitnami/elasticsearch:6.7.0
docker pull bitnami/kibana:6.7.0
docker pull bitnami/grafana
docker pull logstash:6.7.0

mkdir -p /bitnami/elasticsearch/config
nano /bitnami/elasticsearch/config/readonlyrest.yml
readonlyrest:
    access_control_rules:

    - name: "Require HTTP Basic Auth"
      type: allow
      auth_key: user:password

! https://github.com/elastic/logstash/tree/master/config
nano /bitnami/logstash/config/logstash.yml
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.url: http://elasticsearch:9200

Executar no linux SERVER...
$ grep vm.max_map_count /etc/sysctl.conf
vm.max_map_count=262144

# sysctl -w

==== SET KIBANA PASSWORD ====

Colocar username e pass do ElasticSearch no kibana: (ou usar o kibana.yml do git)
nano /bitnami/kibana/data/kibana/conf/kibana.yml
elasticsearch.username: "user"
elasticsearch.password: "password"

==== TEST ELASTICSEARCH ====

curl -XGET 'localhost:9200/_cat/indices?v&pretty'

==== START/STOP COMPOSER ====

- Iniciar e parar Composer:
docker-compose up -d
docker-compose stop

==== Install ReadOnlyRest ====
! https://www.elastic.co/guide/en/elasticsearch/plugins/current/_other_command_line_parameters.html#_batch_mode
!
docker exec a2cd870eb91b elasticsearch-plugin install -b file:///files/readonlyrest-1.17.4_es6.7.0.zip

- Remove ReadOnlyRest:
docker exec a2cd870eb91b elasticsearch-plugin remove readonlyrest

- User management:
https://github.com/beshu-tech/readonlyrest-docs/blob/master/elasticsearch.md#user-management-with-groups

Sample file:
readonlyrest:
    access_control_rules:

    - name: "Require HTTP Basic Auth"
      type: allow
      auth_key: user:password

==========

elasticsearch.yml
http:
  port: 9200
path:
  data: /bitnami/elasticsearch/data
transport:
  tcp:
    port: 9300
network:
  host: 172.18.0.2
  publish_host: 172.18.0.2
  bind_host: 0.0.0.0
cluster:
  name: elasticsearch-cluster
node:
  name: db2261b943cc
  master: true
  data: true
  ingest: false

===== LOGS ====

docker logs grafana
docker logs kibana
docker logs elasticsearch
docker logs logstash

===== BACKUP ====

$ docker-compose stop
Next, take a snapshot of the persistent volume /path/to/elasticsearch-data-persistence using:

$ rsync -a /bitnami/elasticsearch /path/to/elasticsearch-data-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
$ rsync -a /bitnami/kibana /path/to/kibana-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
$ rsync -a /bitnami/grafana /path/to/kibana-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
$ rsync -a /bitnami/logstash /path/to/kibana-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
