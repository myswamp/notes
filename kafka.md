# delivery semantic
at least once - filtering on the consumer ends
at most once - no acks 
exactly once - producer, consumer, topic, broker all invovled 

# why fast
- batch write to the end of log
- use OS page cache instead of JVM heap to avoid GC pauses
- serve latest messages from page cache

# problems kafka can solve
- data silos
- data replay

# kafka stream
- a high level layer on top of producer/consumer
- light weight, run in your application, no separate processing cluster
- local state with fault-tolerance, stateful processing, distributed joins
- one at a time message processing
- exactly once

# kafka connect
- source connector: import to kafka
- sink connector: export from kafka

# ksqlDB
- continuous query over a data stream

# where kafka shines
- large volumes of data
- real-time streaming

# when not to use kafka
- random access pattern
- strict ordering, one producer thread, one partition in topic, one consumer per group
- large message, default 1 MB 

# message structure 
- optional key, value (timestamp, optional header?)

# replication
- only one replica leader in a replica group

# consumer
- polls one or more topics

# partition
- exists on one broker, cannot split
- consists of segments

# commit log 
- append only
- data retention factor: 1. size 2. time

# java client lib
- consumer is not thread-safe

# kafka application design considerations
- event loss  (ack)
- grouping (key)
- ordering
- last value only (compact)
- how many consumers?
  
