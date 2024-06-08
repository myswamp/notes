# Index
- leaf nodes contains metadata for row locking, transaction isolation etc.
- the rightmost of every secondary index is the primary key
- Use pt-duplicate-key-checker to find and report duplicate indexes.


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

