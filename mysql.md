# Query
- Query metrics originate from the slow query log or the Performance Schema
- The slow query log can log all queries when long_query_time is set to zero, this can increase disk I/O and use a significant amount of disk space.
- Lock time is an inherent part of query time
- Direct query optimization is changes to queries and indexes.
- Indirect query optimization is changes to data and access patterns.

# Index
- leaf nodes contains metadata for row locking, transaction isolation etc.
- the rightmost of every secondary index is the primary key
- Use pt-duplicate-key-checker to find and report duplicate indexes.
- index pushdown: pushing down conditions to the storage engine level, Only the matching rows are then passed back to the MySQL server layer


# access methods
- index lookup: ref, eq_ref, range, and so forth
- index scan: full index scan, index-only scan(requires covering index)
- table scan: read all rows in pk order, terrible but easy to fix. should be avoid unless table is tiny and infrequently accessed or table selectivity is very low

# leftmost prefix requirement
  - because the underlying index stucture is ordered by the index column order
  - Indexes (a, b) and (b, a) are different indexes

# explain execution plan
- type: table access method or index lookup access type, ALL: full table scan, index: index scan, const/ref/range: index look up
- possible keys
- key
- ref
- rows: estimated number of rows, mysql uses index statistics to estimate
- extra: indicates query optimizations that can apply


# sharding
- vitess addresss challenges like resharding and rebalancing

# transaction
- Locking and transaction isolation levels are related.
- InnoDB: every query executes in a transaction by default, even a single SELECT statement
- Reads do not lock rows (except for SELECT...FOR SHARE and SELECT...FOR UPDATE), but writes always lock rows
- In a REPEATABLE READ transaction, InnoDB can lock more rows than it writes
- default isolation level: REPEATABLE READ, it uses next-key locks to prevent phantom rows
- READ COMMITTED disables gap locking, which includes next-key locks.
- READ COMMITTED reduces locks and undo logs, which helps improve performance.
- SET TRANSACTION applies once to the next transaction. After the next transaction, subsequent transactions use the default transaction isolation level
- large transaction(modifying too many rows, leads to locks contention, replication lag, slow to commit);
- long transaction (fix: limit the number of queries in the transaction),
- stalled transactions(time waiting between queries),
- Abandoned Transactions(an active transaction without an active client connection)
  

# lock
- MySQL detects and breaks deadlocks, but they kill performance (MySQL kills one transaction to break the deadlock) and they’re annoying.
- Locking reads should be avoided, especially SELECT...FOR UPDATE, because they don’t scale
- Whereas table locks and row locks control access to table data, metadata locks control access to table structures (columns, indexes, and so on) to prevent changes while queries are accessing the tables. Every query acquires a metadata lock on every table that it accesses. Metadata locks are released at the end of the transaction, not the query.
- SELECT queries must acquire shared metadata locks (MDL) on all tables accessed
- Record lock: Locks a single record
- Gaplock: Locks the gap before (less than) a record
- Next-key lock: Locks a single record and the gap before it, a combination of record lock and gap lock
- Insert intention lock: Allows INSERT into gap?
- Locks are released when a transaction ends
- exmaine locks using table 'performance_schema.data_locks'.
- (BETWEEN) accesses the gaps; therefore, it uses next-key locks to lock the gaps
- when (IN) does not access the gaps, it uses record locks, IN clause does not preclude gap locking when it access the gaps
- Gap locking is easy to disable by using READ COMMITTED
- the lower the selectivity, the larger the gaps
- insert intention lock is a special type of gap lock that means the transaction will insert a row into the gap when the gap is not locked by other transactions
- Gap locks prevent INSERT. Insert intention locks allow INSERT.
- Insert intention locks do not lock the gap
- Insert intention locks are created and reported only when they conflict with gap locks held by other transactions
- If an insert intention lock is created, it is used once and released immediately once granted; but InnoDB continues to report it until the transaction is complete
- Explicit locks exist as lock structures in memory; therefore, InnoDB can report them. But implicit locks do not exist: there is no lock structure; therefore, InnoDB has nothing to report

 # MVCC
 - Multiversion concurrency control means that changes to a row create a new version of the row
 - Undo logs record how to roll back changes to a previous row version
 - There are two sets of undo logs: insert undo logs for INSERT and update undo logs for UPDATE and DELETE
 - In a READ COMMITTED transaction, each read establishes a new snapshot
 - Snapshots only affect reads (SELECT)—they’re never used for writes. Writes always secretly read current rows, even if the transaction cannot “see” them with SELECT. e.g. duplicate key
 - If the application executing the transaction needs a newer snapshot, it must commit the transaction and begin a new one to establish a new snapshot.
 - undo logs are saved in the InnoDB buffer pool, they use memory and are periodically flushed to disk.
 - Data locks and undo logs are released when a transaction ends, with COMMIT or ROLLBACK.
 - as an engineer using MySQL, you only need to know and monitor one: HLL
 - History list length (HLL) gauges the amount of old row versions not purged or flushed.
 - Alert on HLL greater than 100,000.
 - The MySQL Performance Schema makes detailed transaction reporting possible
   

 # Access Pattern
 - Read/Write
 - Throughput burst/steady/cyclical:
 - Data Age: affects working set which contains frequently accessed data, frequently dredging up old data is problematic for performance
 - MySQL must evict old pages, which it tracks in a least recently used (LRU) list.
 - Occasionally accessing old data is not a problem
 - old data is relative to access, not time. e.g.The profile of a user who last logged in a week ago isn’t necessarily old by time, but their profile data is relatively old because millions of other profile data have since been accessed, which means their profile data was evicted from memory.
 - Data Model: determine the ideal data model for the access, then use a data store built for that data model
 - Transaction Isolation
 - Read Consistency: The duration of eventually is roughly equal to replication lag, which should be less than a second.
 - Replicas used to serve read access are called read replicas. (Not all replicas serve reads; some are only for high availability, or other purposes.)
 - concurrency
 - Always ensure that offload reads are acceptable with eventual consistency and not part of a multi-statement transaction.
 - do not offload all reads, e.g. read replicas with large storage but small CPU and memory (to save money)
 - MySQL has a built-in query cache: forget it and never use it. It was deprecated as of MySQL 5.7.20 and removed as of MySQL 8.0.
 - Enqueue Writes: Use a queue to stabilize write throughput, allow the application to respond gracefully and predictably to a thundering herd
 - For write-heavy applications, enqueueing writes is the best practice and practically a requirement.
 - Scale up hardware to improve performance after exhausting other solutions.

 # Replication
 - Replication affects high availability.
 - MySQL replication supports multiple writable sources, but this is rare due to the difficulty of handling write conflicts. A single writable source is the norm
 - Replicas should always be read-only to avoid split-brain
 - Replicas are not required to write binary logs, but it’s standard practice for high availability because it allows a replica to become the source
 - A row image is a binary snapshot of a row before and after modification. A single SQL statement can generate countless row images, which yields a large transaction that might cause lag as it flows through replication.
 - Asynchronous replication: default, transaction completes after binlogs are wrttien 
 - semi‐synchronous replication: for each transaction, MySQL waits for a replica to acknowledge that it has written the binary log events for the transaction to its relay logs
 - row-based replication: apply binary log events. faster, because they’re given the end result—data changes—and told where to apply them
 - statement based replication: more compact, less space.
 - Replication lag has three main causes: transaction throughput, a MySQL instance catching up after failure and rebuild (failover, legitimate, just be aware), and network issues
 - backfilling, deleting, and archiving data are common operations that can cause massive replication lag, solution: proper batch size, monitor replication lag and slow down when replicas begin to lag. It’s better for an operation to take one day than to lag a replica by one second
 - replication lag is data loss, use semisynchronous replication to avoid data loss!!
 - semisynchronous replication requires that the source and replicas are on a fast, local network because network latency implicitly throttles transaction throughput on the source.
 - Reducing Lag: Multithreaded Replication, ---Transaction dependency tracking
 - The de facto tool for monitoring MySQL replication is pt-heartbeat.
   • The MySQL metric for replication lag, Seconds_Behind_Source, can be misleading; avoid relying on it.
   • Use a purpose-built tool to measure and report MySQL replication lag at subsec‐ ond intervals.

   # Schema Changes
   - online schema change (OSC) tools: pt-online-schema-change/gh-ost/certain built-in online DDL operations can run for days or weeks while allowing the application to function normally
  
   # Sharding
   - limit the total data size of a single MySQL instance to 2 TB (date 2021 DEC) 
   - To determine whether sharding is needed, estimate data size and growth for the next four years, whether the data set is bounded or unbounded
   - application, not MySQL, is responsible for mapping and accessing data by shard key because MySQL has no built-in concept of sharding
   - An ideal shard key has three properties: High cardinality, Reference application entities(so access do not cross shard), Small
   - common strategies: hash, range, and lookup (or directory).
   - challenges:
      ^ transactions(avoid),
      ^ joins(applications do it),
      ^ Cross-shard queries(A moderate number of cross-shard queries is inevitable and acceptable, but scatter queries should be avoided)
      ^ Resharding: don’t wildly overestimate, but estimate generously.
         • An initial bulk data copy from old to new shards
         • Sync changes on old shard to new shards (during and after data copy)
         • Cutover process to switch to new shards
   - Rebalancing: handle hot shards
   - Online schema changes: least complex challenge


