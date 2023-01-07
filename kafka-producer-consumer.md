#### min.insync.replicas

we can set **min.insync.replicas** set in broker or topic level (override)
if we have min.insync.replicas=2 , web nead at least 2 broker (include leader) to respond data.

```
min.insync.replicas=2 + acks=all

                                            <=======>  Broker102 (down)
             send data to leader
Producer -----------------------------> Broker101
         <-----------------------------
          EXEPTION(NOT_ENOUGH_REPLICAS)
                                            <=======>  Broker103 (dwon)
```

#### Producer Retries

```
retries = 2147483647 # default retries in kafka (kafka > 2.1)
retry.backoff.ms = 100ms # retry every 100 milliseconds producer sned to kafka
delivery.timeout.ms = 120000 ms (2 min) # until 2 min retry and retry after 2min get timeout and not send to kafka anymore
max.in.flight.requests.per.connection=1 # (default 5) how many request producer can be parallel in single broker
```

#### Idempotent Producer

the producer can introduce duplicate messages in kafka due to network err

```
enable.idempotence=true (producer level) # Detect duplicate don't commit twise
```

##### Good Request

```
                     1.Produce
Producer  ---------------------------------->  Kafka  2.commit
         <-----------------------------------
                       3.aks
```

##### Duplicate Request

```
                     1.Produce
         ---------------------------------->          2.commit
          <-------------!!!-----------------
Producer      3.aks never reaches(net err)     Kafka
         ----------------------------------->
                    4.Retry Produce                   5.commit(Detect duplicate don't commit twise)
         <-----------------------------------
                       6.aks
```

#### Message Compression

* compression in producer level, dont require change in broker or consumer
* **compression.type** can be **none** (default), **gzip**, **lz4**, **snappy**

```
Producer Batch   Message1, Message2, Message3, ... Message100    Kafka
                 <------------------------------------------->
                                Compressed Message
```

#### linger.ms, batch.size

Kafka by default 5 messages individually send at same time , if more message have to be send while 
others are in flight,kafka batching them wait and send all toghether

**linger.ms:** number of miliseconds a producer is willing to wait before sending batch out (default 0)
**batch.size:** Max number of bytes that will be included in batch (default 16KB) 

**Any message that is bigger than the batch size will not be batched**

```
Producer Message             Message1, Message2, Message3, ... Message100
                          <-------------------------------------------------->
wait up to linger.ms       One Batch (max size is batch.size) / One Request
                               <-------------------------------------->
                                     Compressed Message if enabeld
```



#### Default producer partitioner and how keys are hashed 

* by default keys are hashed using the **murmur2** algorithma

```
targetPartition = Math.abs(Utils.murmur2(keyBytes)) % (numPartitions - 1)
```

that means same keys will go to the same partition


#### Max.block.ms, buffer.memory

if the producer produce faster than the broker can take the records will buffered in **producer** memory

**buffer.memory=33554432 (32MB)** the size of send buffer
**Max.block.ms=60000** the time until throw excpetion 
   * producer filled up its buffer
   * the broker not accept new data (down)


#### Delivery Semantics

* consumers choose when to commit offsets

* **At most once**
   * if processing goes wrong message will be lost
   * offsets are committed as soon as massage batch is recieved

```         
            o         o         x        o               o
                                           Committed Offset
                                     <----------------------->
Kafka   Message1, Message2, Message3, Message4, ... Message100   Consumer from
                                                                 Consumer Groups (Proccess data)
        ----------------------------> ----------------------->
                 Read Batch                Read start from 
                                           commit after restart
```


* **At least once**
   * if processing goes wrong message will be read again, idempotent
   * Offests are commited after the message is proccessed

```         
                                o        o      
            o         o         o        o               o
                                     Committed Offset
                            <-------------------------------->
Kafka   Message1, Message2, Message3, Message4, ... Message100   Consumer from
                                                                 Consumer Groups (Proccess data, Upserts in database)
        -------------------> -------------------------------->
                 Read Batch           Read start from 
                                      commit after restart
```


#### Consumer Poll Behavior

* kafka consumer have **poll** model ,other message bus use **push** model
* allow consumer to control where in the log they want to consume, how fast, and give ability to replay events

```
               poll(Duration timeout)
       <--------------------------------------
Broker ---------------------------------------> Comsumer
          return data immediately if possible
           return empty after timeout
```

**Fetch.min.bytes** 
   * Controls how much data you want to pull at least on each request (default 1 byte)

**Max.poll.records** 
   * Controls how many records are receive per poll request (default 500)

**Max.partitions.fetch.bytes (default 1MB)** => partitions * 1MB
   * max data returned by the broker per partitions
   * if you read from 100 partition , you will need a lot of memory(RAM)

**Fetch.max.bytes (default 50MB)** 
   * Max data returned for each fetch request (covers multiple partitions)
   * The consumer performs multiple fetches in parallel


#### Consumer offset reset behaviour

* auto.offset.reset=latest => will read from the end of the log
* auto.offset.reset=earliest => will read from start of the log
* auto.offset.reset=none => will throw exceprion if no offset is found

**offset.retention.minutes** => retention for offsets

#### Consumer offset Commits Strategies

* enable.auto.commit = true && synchronus proccessing of batches
* enable.auto.commit = false && manual commit of offsets

**auto.commit.interval.ms=5000 by default** offset auto committed for regular intervals

#### Controlling Consumer Liveliness

```
                |Consumer1   <------------ poll thread ---------->    Broker

Consumer Group  |Consumer2                                               |
                                                                         
                |Consumer3   <--------- Heartbeat Thread -------->    Consumer Coordinator (Consumer health check)                                                                   
```

**session.timeout.ms (default 10 seconds)**
   * Heartbeat are send periodically to ther broker
   * if not heartbeat is send during that period, the consumer is considered dead

**heartbeat.interval.ms (default 3 seconds)**
   * how often to send hearbeats
   * usually to 1/3d of session.timeout.ms

**max.poll.interval.ms (default 3 minetes)**
   * maximum amount of time between two poll() calls before declaring the consumer dead
