# Segment
  - minimun unit of a lucene index which is mapped to a shard for an ES index
  - segment = inverted index + term index + stored fields + doc values
  - read only once genrated
  - segment merging


# Inverted index 
  - inverted index = term dictionary + post listing
  - term dictionary is sorted, able to binary search
  - post listing contains term frequency and term offset

# Term index
  - FST to expedite search, help to locate term in the term dictionary

# Stored fields 
  - document content

# doc values
  - column origented 
  - exchange space for time
  - used for ordering and aggregation

# Node roles
  - one node can have multiple roles
  - coordinating node: serve read and write requests 
  - master node: cluster management 
  - data node

# Write flow
  1. requests reach coordinateing node first
  2. coordinateing node routes requests to target primary shard in data node
  3. primary shard sync data to replica
  4. once replica was written, replay ack to coordinating node
  5. coordinateing node responds to client

# Read flow
  query phase + fetch phase
  1. requests reach coordinating node first
  2. based on index name info, coordinating node knows the number of shards and data nodes they reside
  3. concurrently search segments and shards
  4. coordinating node receives IDs from multiple shards, orders and aggreates then discards data
  5. fetch document content from multiple shards based using doc ID
  6. coordinateing node responds to client
