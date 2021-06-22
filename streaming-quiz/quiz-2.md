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
    log.retention.hours=168

#### 2.Run below code on your client machine.
##### Just remind that replace MSK broker string to yours.

```python

import random
import json
from kafka import KafkaProducer
from kafka.errors import KafkaError

##Producer config

producer = KafkaProducer(bootstrap_servers='b-1.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-2.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-3.mskname.xxxxx.kafka.region.amazonaws.com:9092'
        ,retries=5,retry_backoff_ms=6000,request_timeout_ms=60000
        ,value_serializer=lambda m: json.dumps(m).encode('ascii'))


for i in range(10000000):
    key=random.randint(0,99)
    value=random.randint(0,99)
    future=producer.send('els-test',{key:value},timestamp_ms=0)
    record_metadata = future.get(timeout=10)
    str = 'key:{0},value:{1},topic:{2},partition:{3},offset:{4}'.format(key,value,record_metadata.topic,record_metadata.partition,record_metadata.offset)
    print(str)

```
#### 3.Running code for 3 minutes and stop it, then wait about abount 5 minutes and use console-consummer to consume data from beginning.

```bash
./kafka-console-consumer.sh --bootstrap-server b-1.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-2.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-3.mskname.xxxxx.kafka.region.amazonaws.com:9092 --topic els-test --from-beginning
```

## Questions

#### 1. What's your observation from console-conumer result?
#### 2. Describe the possible reason for your observation.
#### 3. How to mitigate the issue from your observation?

Tip: Check the relationship between message arrvial timestamp and log.retention.hours