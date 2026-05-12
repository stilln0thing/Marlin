# PROMPT PREPARATION

## Repository Context
`aiokafka` is a python library that developers import and take use of it. It lets python apps talk to Apache kafka. Kafka is used when we need to pass large volumes of data reliabily between services . The key here is that `asyncio` lets python apps talk to kafka asynchronously, without blocking the event loop.

In kafka, messages are organised into topics (channels). And each topic is then split into partitions for parallel processing. This increases the processing speed of kafka by a lot. There are 2 main terms in kafka, producers and consumers.
`Producers` write to topics and `Consumers` read from topics. Kafka is heavily used in industries where a lot of data is produced and needs to be processed, e.g. banking, finance, e-commerce, etc.

`aiokafka` uses `cpython` instead of python for better performance. It directly implements the Kafka protocol, reducing any dependency on `kafka-python`. The traditional Python kafka clients used blocking i/o, due to which the whole thread freesez. 
`aiokafka` does this in a different way, it uses non blocking i/o, so that while waiting for these other tasks can run.

## Pull Request Description
In `aiokafka/structs.py` file there is a struct named `RecordMetaData` which is returned as an result when a producer pushes an event to kafka. Before this PR it has fields like topic, partition, topic_partition and offset. So the developer can know where the message is and not when the message was created.
The PR #273 bridges this gap.

In kafka 0.10.0 patch, kafka introduced a new message format that included a timestamp on every message. There are 2 timestamps at the broker level, i.e. :-
- `CreateTime`:- broker uses the timestamp producer sent
- `LogAppendTime`:- ignores the producer timestamp and uses its own timestamp

This choice is decided by the broker and the producer has no command over it. Before this PR, the producer only got an offset and no timestamp.

In this PR, we added 2 new fields to `RecordMetaData` struct, i.e. `timestamp` and `timestamp_type`. `timestamp` stores the actual timestamp and `timestamp_type` stores from where the timestamp data came from.
If `timestamp_type = 1`, then it means the broker is set to `LogAppendTime` and the broker overrode the timestamp that the producer sent and if the `timestamp_type = 0`, then the broker is set to `CreateTime` and it stores the timestamp that the broker sent. This is critical for time sensitive application like financial systems and audit logs.

## Acceptance Criteria
