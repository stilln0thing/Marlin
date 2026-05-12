# PULL REQUEST ANALYSIS

## Summary
In this, I chose the repository `aiokafka` due to my personal interest with learning about `distributed systems` as well as my current project on which I am currently using `kafka` as its core requirement (https://github.com/stilln0thing/GoKart). 

The project `aiokafka` is a high level messages consumer and producer that aims to send messages to kafka asynchronously. It can be used mainly for distributed systems.

I have selected `2` PR's, i.e.
- https://github.com/aio-libs/aiokafka/pull/201
- https://github.com/aio-libs/aiokafka/pull/237


## 1. Added search_for_times API to search offsets based on timestamps (#201) :-
In this PR, the developer introduces a way to get a kafka message based on its timestamp. Before this, there was no way to replay a message from a specific point of time, e.g. if you want to replay all the messages from 3PM yesterday.

In kafka, every message has an offset numbers (its position) and a timestamp(the time it was created).
This PR added mainly 3 functions :- 
1. `offsets_for_times` :- get offset based on timestamps
2. `beginning_offsets` :- get offset to the very start
3. `end_offsets` :- get offset to the very end

Therefore, this PR added a full set of offset positioning api's, beginning, end and timestamp based, giving full control to developers for offsets positioning.

### Technical Changes :-
The files changed are :-
- `aiokafka/consumer.py` : added `offset_for_times()` in asyncioConsumer class
- `aiokafka/errors.py`
- `aiokafka/fetcher.py`
- `aiokafka/message_accumulator.py`
- `aiokafka/producer.py`
- `aiokafka/structs.py`
- `tests/_testutil.py`
- `tests/test_consumer.py`
- `tests/test_fetcher.py`
- `tests/testpep492.py`


### Implementation Details :- 
The issue got resolved by Kafka's broker level API called `listoffsets`. It allows fetching offsets based on a given timestamp.
This PR updated the consumer class to call this api asynchronously through a python method called `offset_for_times`. 


- Sample Pseudo-code example :- 

```python
import time
from aiokafka import TopicPartition

partition = TopicPartition("orders_topic", 0)

timestamp_ms = int((time.time() - 60 * 60) * 1000)
result = await consumer.offsets_for_times({
    partition: timestamp_ms
})

consumer.seek(partition, result[partition].offset)
async for record in consumer:
    print(record.value)
```

This is especially useful for replaying recent events, debugging production issues, or recovering missed messages without scanning the entire topic history.

### Potential Impact :-
With this PR, users can now replay the events from a particular timestamp using this. They can first get the offset using the `offset_for_times` method and then use `seek()` to display the events from that timestamp (see the code example above).
This made `aiokafka` closer to real world implementations.



## 2. Add timestamp to RecordMetadata  (#237) :- 
In this PR, the developer addressed a gap in how kafka producers recieved confirmations after sending a messgae. Earlier the
metadata returned to them only had topic, partition and offset of the message.
After some time, Kafka introduced timestamps as well for each message. But producers has no idea about the times assigned by brokers.
Therefore, this PR solves this by adding timestamp directly in `RecordMetadata`. Due to this, producers can now verify the exact timings of their messages.

### Technical Changes :-
The files changed are :-
- `aiokafka/producer/message_accumulator.py`
- `aiokafka/producer/producer.py` :- updated producer response parsing logic
- `aiokafka/record/legacy_records.py`
- `aiokafka/structs.py` :- Added `timestamp` and `timestamp_type` to `RecordMetaDate` struct
- `aiokafka/utils.py`
- `tests/record/test_legacy.py`
- `tests/test_message_accumulator.py`
- `tests/test_producer.py`

### Implementation Details :-
In this, the developer mainly focused on improving the metadata returned after a successful event is registered by Kafka. Before this PR, `asyncio` was not exposing timestamp information to devs. This PR fixed that by updating the producer logic to have timestamps and include them in `RecordMetaData`.

Two new fields were added in the `RecordMetaData` struct, i.e. `timestamp` and `timestamp_type`. The `timestamp` stored the actual timestamp, while the `timestamp_type` stored from where the timestamp data came from.

### Potential Impact :-
This PR directly affects the application relying on RecordMetadata. Developers now can directly access the accurate broker assigned timestamps just after publishing a message. This can help the developers in debugging and observability.
