# TECHNICAL COMMUNICATION

I already had prior exposure to Kafka and how it works through my project GoKart(https://github.com/stilln0thing/GoKart). Due to this the repo `aiokafka` felt relatable and approchable. That context also made the PR's implementation instantly understandble. I knew how producers and consumer worked in kafka and why confirmation matters. The timestamp implementation done by PR #237(https://github.com/aio-libs/aiokafka/pull/237) was solid. The scope of the PR was also very contained and not all over the repo.


As I had already some idea how kafka works and its producer/consumer pattern. When I saw `RecordMetaData`, I immediately recognised it as a confirmation message sent after a message is successfully produced. The before and after was clear and just adding two fields to a struct felt approachable and architecturally not hard. The `timestamp_type` field made sense. `CreateTime` and `LogAppendTime` are two different usecases and the developer should be able to tune them to his need.

Working through this PR bridges the gap between using Kafka at a surface level in Gokart and understanding how it works internally. Implementing it will push me to read actual protocol-level code,  something I have not done before. That is exactly the kind of learning I was looking for when I chose this PR.