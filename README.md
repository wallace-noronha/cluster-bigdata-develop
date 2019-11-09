### About

This docker-compose file make a mini fast-data cluster.
I'am runnig this compose file in a virtual-box machine with ubuntu an the follow configuration:
- Intel Core i5- 8350U 1.70GHz
- 16Gb Ram

# Tools



# Zookeeper
Single zookeeper instance to orchestrate Apache kafka, Schema-registry and Apache nifi
#### Hostname: zookeeper
#### exposeds ports: 2181
#### imagem: confluentinc/cp-zookeeper:latest

# Kafka
Single instance of Apache kafka for data stream.
#### Hostname: kafka
#### exposeds ports: 9091
#### imagem: confluentinc/cp-kafka:latest
#### dependencies: zookeeper


# Schemaregistry
Single instance of Schema-Registry for storing and retrieving avro schemas.
#### Hostname: schemaregistry
#### exposeds ports: 8081
#### imagem: confluentinc/cp-schema-registry:latest
#### dependencies: zookeeper

# Nifi
Apache nifi for data movement
#### Hostname: nifi
#### exposeds ports: 8080, 8888
#### imagem: apache/nifi:latest
#### dependencies: zookeeper

# Elasticsearch
Elasticsearch NOSQL database, DOC based.
Elasticsearch with ssl and authentication enabled for testing Apache Nifi processor.
username and password = elastic

#### Hostname: elasticsearch
#### exposeds ports: 9200
#### imagem: docker.elastic.co/elasticsearch/elasticsearch:7.4.2

# Kibana
Kibana for overview elasticsearch indexes and management.
#### Hostname: kibana
#### exposeds ports: 5601
#### imagem: docker.elastic.co/kibana/kibana:7.4.2
#### dependencies: elasticsearch



# Nifi Flow

The file kafka_to_elatic.xml it's a template for apache nifi, to import template open Apache Nifi and select Upload Template, instantiate chose template.
Before open the instantiate template start the controller services.
Start all processors and be happy.

The flow is the follow:

                
### Processors: Publish Kafka
The Flow open port 8888 of Apache Nifi with context /test

``` flow
http=>start: ListenHttp
avro=>operation: convertRecord - Avro
hex=>operation: Base64EncodeContent
kafka=>end: PublishKafka

http->avro->hex->kafka
```
### Processors: Put Elasticsearch
 ``` flow
kafka=>start: ConsumeKafka
hex=>operation: Base64EncodeContent
avro=>operation: convertRecord - Json
putTimestamp=>operation: ExecuteGroovyScript - Put timestamp
elasticsearch=>end: PutElasticsearchHTTP

kafka->hex->avro->putTimestamp->elasticsearch

```


## Testing:
```bash
curl -X POST  http://localhost:8888/test  -H 'Content-Type: application/json' -d '{"hostname":"teste", "ipaddress":""}' 
```

