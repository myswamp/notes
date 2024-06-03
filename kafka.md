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
- ordering : max.in.flight.requests.per.connection value to 1 and set acks (the number of brokers that send acknowledgments back) to all 
- last value only (compact)
- how many consumers?
- schema

# producer notes
- fetch metadata from cluster, to determine which broker to write to
- idepotent producer: enable.idempotence=true
- ack: all, 1, 0
- retries
- max.in.flight.requests.per.connection
- partitioner.class
- async: Callback

  # consumer notes
  - read from leader replica
  - many partitions might increase end-to-end latency.
  - only one consumer per consumer group can read one partition
  - partition.assignment.strategy: Range, RoundRobin, Sticky, CooperativeSticky
  - reading from a compacted topic, consumers can still get multiple entries for a single key, Because compaction runs on the log files that are on disk, compaction may not see every message that exists in memory during cleanup
  - offsetsForTimes, times might appear out of order
  - the offset sent to broker is supposed to be the future index
  
