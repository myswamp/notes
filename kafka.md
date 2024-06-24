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
- replica.lag.time.max.ms default to 30000. specifies what replicas are considered stuck or lagging

# consumer
- polls one or more topics

# partition
- exists on one broker, cannot split
- consists of segments

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
  - many partitions might increase end-to-end latency. having more brokers to migrate in case of a broker failure
  - only one consumer per consumer group can read one partition
  - partition.assignment.strategy: Range, RoundRobin, Sticky, CooperativeSticky
  - reading from a compacted topic, consumers can still get multiple entries for a single key, Because compaction runs on the log files that are on disk, compaction may not see every message that exists in memory during cleanup
  - offsetsForTimes, times might appear out of order
  - the offset sent to broker is supposed to be the future index
 
  # broker
  - rack awareness
  - Only one broker in the cluster acts as the controller: cluster management, partition reassignment
  - each partition has a single leader replica, ISR list is maintained by the leader
  - unclean.leader.election.enable is true, the controller selects a leader for a partition even if it is not up to date
  - Kafka replicas do not heal themselves automatically
  - JMX
  - upgrading strategy: rolling restart, controlled.shutdown.enable. Setting this to true enables the transfer of partition leadership before a broker shuts down
  - backup: MirrorMaker, Confluent Replicator. preferred option is for a cluster to be backed by a second cluster
  - --under-replicated-partitions
  - reducing number of partitions is not currently supported
  - delete.topic.enable default to fasle
  - better to set auto.create.topics.enable to false
  - Replication factors <= no. of brokers
  - segments make up a partition 
  - segment has .log .index .timeindex
  - segment name should be the same as the first offset in that file
  - cleanup.policy=compact
  - compact topic example: kafka's internal topic "__consumer_offsets"
  - tombstone, message value of null

  # schema registry
  - internal topic for schema registry default to "_schemas"
  - Alternative to a schema registry: produce data on a different topic with a breaking change; transform existing data to new topic if reprocessing old data is required.
  - Compatibility rules: BACKWARD (the default type, adding non-required fields or removing fields, consumer upgrade first), BACKWARD_TRANSITIVE, FORWARD(add new fields ), FORWARD_TRANSITIVE, FULL,FULL_TRANSITIVE, and NONE
 
  # storage
  - logically sits between the long-term storage solutions of a database and the transient storage of a message broker
  - default 7 days
  - data retention factor: 1. size 2. time
  - turn off deletion log.retention.bytes and log.retention.ms to –1
  - for long term storage, move data out of kafka and store it to db, hdfs, cloud storage etc.
  - keep raw, original data format
  - treat data as an infinite series of events, move away from a batch mindset
  - goood tip: create a new topic to reload archived data in s3 using kafka collect
  - tiered storate: local, remote for older data
  - Data retention should be driven by business needs. Decisions to weigh include the cost of storage and the growth rate of our data over time.

  # tools
  - debezium, secor, flume

  # architecture with kafka
  - ? lambda architecture: unites data from the serving layer and the speed layer to answer requests with a complete view of all recent and past data.
  - kappa architecture: using Kafka Streams or ksqlDB to read all the events in near-real time and creating a view
 
  # cluster size -
    Kafka scales well, and it is not unheard of to reach hundreds of brokers for a single cluster

  # scaling
   - add brokers
   - multiple clusters . e.g. CQRS write cluster and read cluster

  # monitoring
  - top3: UnderMinIsrPartitionCount, UnderReplicatedPartitions, UnderMinIsr
  - Kafka interceptors for tracing
  - cluster monitoring & management: CMAK AKA kafka manager, Confluent Control Center, Cruise Control
  - listeners vs advertised listeners

  # protection
  - quota 

  # setup
  LISTENERS: are what interfaces Kafka binds to.
  ADVERTISED_LISTENERS: are how clients can connect.

  # stream processing
  - Kafka Streams is for data transformations with potentially complex logic consuming and producing data back into Kafka
  - high level layer on top of producer/consumer
  - light weight, run in your application, no separate processing cluster
  - local state with fault-tolerance, stateful processing, distributed joins,
  - The state stores in use are backed by a replicated Kafka topic that is partitioned
  - one at a time message processing
  - exactly once
  - KStreams: model a data-processing process as a graph of nodes, source/transform/filter/sink processor
  - StreamsBuilder: starting point for building our topology
  - KTable: add events to view, draw a parallel to a database table that deals with updates in place
  - GlobalKTable: consumes all partitions, make the data available to our application regardless of which partition it is mapped to
  - processor API: create topology explicityly
