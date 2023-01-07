#### Kafka Connect

**Source** Connectors to get data from common data source (databases,hdfs,...)
**Sink** Connectors to publish data in common data source (databases,hdfs,...)

```             Worker + Worker + Worker            Broker + Broker + Broker
Sources (1)=> <--------------------------> (2)=> <--------------------------->  (3) => Stream App 1
        <=(5)         Connect Cluster       <=(4)         Kafka Cluster             => Stream App 2
                                                                                    => Stream App 3
```

#### Kafka Security

* Authentication
* Authorization
* Encryption

#### Kafka Chage Topic Config

Describe config of topics

```
kafka-configs --zookeeper 127.0.0.1:2181 --entity-type topics --entity-name topic-name --describe
```

For **add** config for specific **topic**:
```
kafka-configs --zookeeper 127.0.0.1:2181 --entity-type topics --entity-name topic-name --add-config min.insync.replicas=2 --alter
```


For **delete** config for specific **topic**:
```
kafka-configs --zookeeper 127.0.0.1:2181 --entity-type topics --entity-name topic-name --delete-config min.insync.replicas=2 --alter
```

#### Segments

* Topic made of partitions and **partitions** made of **Segments**
* Segments is **file**
* Segments come with two indexes (files) (kafka know where to find data in a constant time)
   * an Offset to position of index (where to read to find message)
   * a timestamps to offset index (kafka find message with timestamps)

```
       Segment 0           Segment 1           Segment 2       Segment 3 (last segment)
<------------------><-------------------><-------------------><------------------------>
    Offsets 0-957      Offsets 958-1675    Offsets 1676-2453    Offsets 2454-? (Active)

  Position Index 0    Position Index 1       Position Index 2       Position Index 3
<------------------><-------------------><---------------------><------------------------>
  Timestamp Index 0   Timestamp Index 1      Timestamp Index 2      Timestamp Index 3
<------------------><-------------------><---------------------><------------------------>
<-------------------------------------------------------------------------------------->
                                         Partitions
```

**log.segment.bytes** => the max size of a single segment in bytes (1GB)
**log.segment.ms** => the time kafka will wait before committing the segment if not full (1 week)


#### Log Cleanup Policies

* Policy1: **log.cleanup.policy=delete** (default is week) => 
* Policy2:**log.cleanup.policy** (kafka default for topic __consumer_offsets) => will delete old duplicate keys after the actibe segment commited































