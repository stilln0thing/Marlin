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
Below are some of the acceptance criteria, that must be fulfilled :-
- When a message is successfully produced, the `RecordMetaData` should contain non-null and valid timestamp field, following all the timestamp data type protocols.
- The `timestamp_type` field in `RecordMetaData` should be set to `1` if the broker is set to `LogAppendTime`.
- The `timestamp_type` field in `RecordMetaData` should be set to `0` if the broker is set to `CreateTime`.
- When an message is sent with an explicit timestamp via `send()`, the returned struct should have a valid `RecordMetaData.timestamp` and should reflect the actual timestamp the broker stored.
- Rest everything should continue working as it was working before, without any changed to it.

## Edge Cases
### EdgeCase 1:- Timestamp value of Zero 
If the producer sends a timestamp value of `0`. The consumer should take the timestamp value as it is. Since the timestamp value of 0 is a valid timestamp (1 Jan, 1970), there should not be any error. But if the producer sends a value of `-1`, then there the broker should return error, saying `INVALID TIMESTAMP`.

### EdgeCase 2 :- Network Interruption between Produce
If there is a network error and the connection drops after the broker stores the data before before sending the response back.
The implementation should be such that it returns what the broker returned and not what was originally sent.

### EdgeCase3 - Override by LogAppendTime
If the broker is configured to `LogAppendTime`, then the broker ignores the time sent by the producer and uses its own time. The `timestamp_type` should be set to `1`. Proper error handling should be there.

## Initial Prompt
### Context
`asyncio` is a python library that lets python apps to talk to Apache Kafka aysnchronously, without blocking the event loop. Apache Kafka is used in distributed services, when we need to pass huge volumes of data reliably between sevices.

### Task
The goal is to implement the issue #218 (https://github.com/aio-libs/aiokafka/issues/218). The implementation is to add `timestamp` and `timestamp_type` to `RecordMetaData` struct, which is returned by the `AIOKafkaProducer` after a successful message is produced.

### Repository Structure 
- `aiokafka/producer/producer.py` :- update producer response parsing logic
- `aiokafka/structs.py` :- Add `timestamp` and `timestamp_type` to `RecordMetaDate` struct
- `tests/test_producer.py` :- add tests for validating the producer logic

### Specific Changes Required
- Add `timestamp` and `timestamp_type` in the `RecordMetaData` tuple in the `structs.py`.
- Update the producer reponse extracting code to extract the timestamp and timestamp type.
- Ensure not to break anything else or any code using the previous version


### Acceptance Criteria
Below are some of the acceptance criteria, that must be fulfilled :-
- When a message is successfully produced, the `RecordMetaData` should contain non-null and valid timestamp field, following all the timestamp data type protocols.
- The `timestamp_type` field in `RecordMetaData` should be set to `1` if the broker is set to `LogAppendTime`.
- The `timestamp_type` field in `RecordMetaData` should be set to `0` if the broker is set to `CreateTime`.
- When an message is sent with an explicit timestamp via `send()`, the returned struct should have a valid `RecordMetaData.timestamp` and should reflect the actual timestamp the broker stored.
- Rest everything should continue working as it was working before, without any changed to it.

### Edge Cases
#### EdgeCase 1:- Timestamp value of Zero 
If the producer sends a timestamp value of `0`. The consumer should take the timestamp value as it is. Since the timestamp value of 0 is a valid timestamp (1 Jan, 1970), there should not be any error. But if the producer sends a value of `-1`, then there the broker should return error, saying `INVALID TIMESTAMP`.

#### EdgeCase 2 :- Network Interruption between Produce
If there is a network error and the connection drops after the broker stores the data before before sending the response back.
The implementation should be such that it returns what the broker returned and not what was originally sent.

#### EdgeCase3 - Override by LogAppendTime
If the broker is configured to `LogAppendTime`, then the broker ignores the time sent by the producer and uses its own time. The `timestamp_type` should be set to `1`. Proper error handling should be there.

### Testing Requirements
- Write unit tests verifying timestamp is returned 
  correctly for CreateTime
- Write unit tests verifying timestamp_type returns 
  correct value for both LogAppendTime and CreateTime
- Write a test verifying old broker compatibility 
  (timestamp = -1 case)
- Ensure all existing producer tests still pass

### Constraints
- Do not break existing RecordMetadata field access
- Follow aiokafka's existing async patterns
- Handle brokers older than Kafka 0.10 gracefully