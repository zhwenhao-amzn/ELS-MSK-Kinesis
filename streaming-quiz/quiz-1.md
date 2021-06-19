## Prerequisites

#### 1.Set up custom configuration of MSK cluster as below

    auto.create.topics.enable=true
    default.replication.factor=3
    min.insync.replicas=2
    num.io.threads=8
    num.network.threads=5
    num.partitions=1
    num.replica.fetchers=2
    socket.request.max.bytes=104857600
    unclean.leader.election.enable=true
    log.retention.bytes=1073741824

#### 2.Copy below scripts and save it as kafka_topics_sizes.sh into the kafka_2.12-2.4.1/bin folder.

```bash

#!/usr/bin/env bash
topic_size()
{
bash ~/kafka_2.12-2.4.1/bin/kafka-log-dirs.sh --bootstrap-server b-1.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-2.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-3.mskname.xxxxx.kafka.region.amazonaws.com:9092 --topic-list ${1} --describe | tail -n1 | jq '.brokers[0].logDirs[0].partitions | map(.size/1000000000) | add' | xargs echo ${1} =;
}
list_topics() {
bash ~/kafka_2.12-2.4.1/bin/kafka-topics.sh --bootstrap-server b-1.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-2.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-3.mskname.xxxxx.kafka.region.amazonaws.com:9092 --list;
 }
export -f topic_size

TEMP_FILE=$(mktemp)

list_topics | xargs -I{} bash -c 'topic_size "{}"' > $TEMP_FILE
sort -g -k3 $TEMP_FILE
rm $TEMP_FILE

```

#### 3. Create topic `caculate-size` with 18 partitions and 3 replication-factor

```bash
./kafka-topics.sh --create --bootstrap-server b-1.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-2.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-3.mskname.xxxxx.kafka.region.amazonaws.com:9092 --topic caculate-size --partitions 18 --replication-factor 3`
```

#### 4. Push messages to MSK cluster 

```bash
./kafka-run-class.sh org.apache.kafka.tools.ProducerPerformance --print-metrics --topic caculate-size --num-records 10000000 --throughput 8000000 --record-size 1024 --producer-props bootstrap.servers=b-1.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-2.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-3.mskname.xxxxx.kafka.region.amazonaws.com:9092 buffer.memory=67108864 batch.size=32768  acks=1
```

#### 5. Run kafka_topics_sizes.sh to calcute size of topic `caculate-size`

```bash
./kafka_topics_sizes.sh
```

## Questions

#### 1. print the result of kafka_topics_sizes.sh
#### 2. I have configured log.retention.bytes as 1 GB, why log cleanup policy of MSK doesn't take affect?

Tips: check log.retention.bytes definitions.