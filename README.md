### Kafka Connect Field and Time Based Partitioner

####  Summary
- Partition initially by custom fields and then by time.
- It extends **[TimeBasedPartitioner](https://github.com/confluentinc/kafka-connect-storage-common/blob/master/partitioner/src/main/java/io/confluent/connect/storage/partitioner/TimeBasedPartitioner.java)**, so any existing time based partition config should be fine i.e. `path.format` will be respected.
- In order to make it work, set `"partitioner.class"="com.liveperson.TimeBasedPartitione"` and `"partition.field.name"="ields in your recor"` in your connector config.


    ```bash
    {
         "partitioner.class": "com.liveperson.TimeBasedPartitioner",
         "custom.partition.field.name.in.path": "event_type",
         "custom.partition.field.name": "commonHeader.eventType",
         "timestamp.extractor": "RecordField",
         "path.format": "'year'=YYYY/'month'=MM/'day'=dd'",
         "timestamp.field": "commonHeader.eventTimeStamp"
     }          
    ```
    will produce an output in the following format : 
    
    ```bash
    /data/event_type=XXXXX/year=2020/month=11/day=30
    ```  

####  Example

```bash
KCONNECT_NODES=("localhost:18083" "localhost:28083" "localhost:38083")

for i in "${!KCONNECT_NODES[@]}"; do
    curl ${KCONNECT_NODES[$i]}/connectors -XPOST -H 'Content-type: application/json' -H 'Accept: application/json' -d '{
        "name": "connect-s3-sink-'$i'",
        "config": {
    "name": "s3-parquet-partition-connector-1",
    "connector.class": "io.confluent.connect.s3.S3SinkConnector",
    "s3.region": "us-east-2",
    "topics.dir": "parquet-1",
    "flush.size": "10000",
    "auto.register.schemas": "false",
    "tasks.max": "1",
    "s3.part.size": "5242880",
    "timezone": "UTC",
    "enhanced.avro.schema.support": "true",
    "value.converter.value.subject.name.strategy": "io.confluent.kafka.serializers.subject.RecordNameStrategy",
    "locale": "en-GB",
    "value.subject.name.strategy": "io.confluent.kafka.serializers.subject.RecordNameStrategy",
    "custom.partition.field.name": "commonHeader.eventType",
    "schema.registry.url": "https://dev-us-east-2-schema-registry-internal.mavenapp.com",
    "s3.credentials.provider.class": "com.amazonaws.auth.DefaultAWSCredentialsProviderChain",
    "format.class": "io.confluent.connect.s3.format.parquet.ParquetFormat",
    "aws.access.key.id": "AKIAZRNMXWZF3UK3S7KV",
    "value.converter": "io.confluent.connect.avro.AvroConverter",
    "s3.bucket.name": "prd-dwh",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "custom.partition.field.name.in.path": "event_type",
    "partition.duration.ms": "86400000",
    "schema.compatibility": "NONE",
    "topics": "public-flow-1,public-profile-consumer,public-profile-brand,public-messaging-core-events,public-report-events,public-help-others,public-commerce-events,public-search-events,public-flow-1",
    "parquet.codec": "snappy",
    "aws.secret.access.key": "xxx",
    "value.converter.schema.registry.url": "https://dev-us-east-2-schema-registry-internal.mavenapp.com",
    "partitioner.class": "com.liveperson.TimeBasedPartitioner",    
    "storage.class": "io.confluent.connect.s3.storage.S3Storage",
    "timestamp.extractor": "RecordField",
    "rotate.schedule.interval.ms": "30000",
    "path.format": "'year'=YYYY/'month'=MM/'day'=dd'",
    "timestamp.field": "commonHeader.eventTimeStamp"
}
    }'
done
```
