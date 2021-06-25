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

#### 2.Create the topic with below setting.

```bash

./kafka-topics.sh --create --bootstrap-server b-1.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-2.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-3.mskname.xxxxx.kafka.region.amazonaws.com:9092 --topic els-test-3 --partitions 1 --replication-factor 1

```

#### 3.Run below code on your client machine.
##### Just remind that replace MSK broker string to yours.

```python
import random
import json
from kafka import KafkaProducer
from kafka.errors import KafkaError

##Producer config

producer = KafkaProducer(bootstrap_servers='b-1.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-2.mskname.xxxxx.kafka.region.amazonaws.com:9092,b-3.mskname.xxxxx.kafka.region.amazonaws.com:9092'
        ,value_serializer=lambda m: json.dumps(m).encode('ascii')
        ,acks='all')


for i in range(10000000):
    key=random.randint(0,99)
    value=random.randint(0,99)
    future=producer.send('els-test-3',{key:value})
    record_metadata = future.get(timeout=10)
    str = 'key:{0},value:{1},topic:{2},partition:{3},offset:{4}'.format(key,value,record_metadata.topic,record_metadata.partition,record_metadata.offset)
    print(str)

```

## Questions

#### 1. Which error messages you received from above code execution?
#### 2. Describe the possible reasons from this issue.
#### 3. How to mitigate the issue from your observation?