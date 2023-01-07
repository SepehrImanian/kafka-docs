### Kafka CLI

##### Create Topic with one partition 

```bash
./kafka-topics.sh --zookeeper localhost:2181 --create --topic testTopic --replication-factor 1 --partitions 1
```
**replication-factor = number of brokers**


##### List all topics

```bash
./kafka-topics.sh --zookeeper localhost:2181 --list
```

##### Describe Topic
```
./kafka-topics.sh --zookeeper localhost:2181 --topic testTopic --describe
```
 
##### Delete Topic
```
./kafka-topics.sh --zookeeper localhost:2181 --topic testTopic --delete
```

##### Produce Message
```
./kafka-console-producer.sh --broker-list 172.16.21.122:9092 --topic test
```

**with acknowledgment** 
```
./kafka-console-producer.sh --broker-list 172.16.21.122:9092 --topic test --producer-property acks=all
```

**config/server.properties**
```
num.partitions=1 # number of default partition
```

##### show all message from beginning of topic (show all message)
```
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic TutorialTopic --from-beginning
```

#### Consumer Group

Each consumber read from different partition , consumer group **rebalance and shared load** 
```
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic TutorialTopic --group GroupName1
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic TutorialTopic --group GroupName1
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic TutorialTopic --group GroupName1
```
if all GroupName1 consumer group be down , ofset have been committed in kafka , GroupName2 group read new message
```
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic TutorialTopic --group GroupName2
```

##### CLI

```
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list # List of all Consumer Groups
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group GroupName1 # descibe consumber group
```

#### Reset Offsets

if message producers send to queue not consume by consumer , **lag** of this topic become increase
for unread message from be beginning or spesify time we use reset offsets after that lag become increase

```
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --topic TutorialTopic --group GroupName1 --reset-offsets --to-earliest --execute # from beginning
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --topic TutorialTopic --group GroupName1 --reset-offsets --shift-by -2 --execute # unread message 2 each partition means 6 offsets
```
