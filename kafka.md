### Kafka

#### Topics, Partitions, Offset

particular stream of data
    
```
topic => partitions 0 => offset [0 1 2 3 4 5 6]
      => partitions 1 [0 1 2 3 4 5 6 7 8 9]
      => partitions 2 [0 1 2 3 4 5 6 7 8 9 10]
```

##### Offset:

* data is kept only for one week (default)
* once the data is written to a partition **it cant be changed**
* data is assigned randomly to a partition unless a key is provided

#### Brokers

* each broker has **id**
* each broker contains topic partitions
* connect to any broker (bootstrap broker) connect to entire cluster

#### Topic Replication factor

topicA with replica 2:
 
```
Broker101 => partitions0 TopicA

Broker102 => partitions1 TopicA, partitions0 TopicA

Broker103 => partitions1 TopicA
```

#### Leader for a Partitions

* only leader can receive and serve data for a partitions other brokers sync the data
* if leader lost the replica become leader

``` 
Broker101 => partitions0 TopicA Leader

Broker102 => partitions1 TopicA Leader, partitions0 TopicA Replica

Broker103 => partitions1 TopicA Replica
```

#### Producers  

* write data to topics 
* automatically know to which broker and partition to write
* if broker fail , producers auto recover
* loadbalace to brokers

##### receive acknowledgment of data writes:
```
acks=0 => won't waite for acknowledgment (possible data write)
acks=1 => wait for leader acknowledgment (default)(limited data loss)
acks=all => Leader + replica acknowledgment (no data loss)
```

```
           => Broker101 => partitions0 TopicA 

Producers  => Broker102 => partitions1 TopicA

           => Broker103 => partitions1 TopicA
```

##### Message Key

* Producers can choose to send a **key** with message (string, number,....)
* if **key=null** data send **round robin** to brokers
* all message for key always go to the same partition

#### Consumers
 
* read data from a topics
* consumers know which broker to read from
* in case broker fail, consumers khow how to recover
* read data in order each partition
* read parallel between two partions 

```
Broker101 => partitions0 TopicA =>
                                    Consumers
Broker102 => partitions1 TopicA =>
```

##### Consumers Group 

* each consumers in group reads from exclusive partitions
* too many Consumers, more than partitions, consumers will be inactive
 
```
partitions0 TopicA <== [Consumer1, Consumer2] Consumers Group1
partitions1 TopicA <==
partitions3 TopicA <== [Consumer1, Consumer2, Consumer3] Consumers Group2
```

#### Consumer Offsets

* kafka restore the offsets which consumer group has been reading
* offsets commited live in **topic** named **_consumer_offsets**
* if consumers die , it will be able to read back from where it left off

```
  commit offsets   read
         |-------------------->|
[0,1,2,3,4,5] offsets <====== [Consumer1, Consumer2] Consumers Group
```

#### Delivery Semantics

* consumers choose when to commit offsets

* **At most once**  => if processing goes wrong message will be lost
* **At least once** => if processing goes wrong message will be read again, idempotent
* **Exactly once** => kafka to external system (databases) idempotent(duplicate message), can achieved for kafka

#### Kafka Broker Discovery

* Every broker called **bootstarp server**
* connect to any broker (bootstrap broker) connect to entire cluster
* Each broker knows about all brokers, topics, partitions(metadata)

```
              connections + metadata request
kafka client ===============================>   Kafka Cluster [Broker1, Broker2, ...]
                   list of all brokers
             <===============================
               can connect to needed brokers
             ================================>
```

#### Zookeeper
 
* Manages brokers (keeps a list of them)
* Helps in performing leader election for **partitions**
* Send notifications to kafka in case of chnages (new topics, brokers die, ...)
* Zookeeper design operates odd number of servers)
* leader(handle writes), followers(handle reads)

```
Zookeeper(Follower)  <=========> Zookeeper(Leader) <=========> Zookeeper(follower)

 Broker1 Broker2                  Broker3 Broker4               Broker5 Broker6
```
