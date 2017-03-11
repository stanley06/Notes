## APACHE CASSANDRA 

 - no SPOF, P2P distributed system across its nodes, each node is independent and interconnected to other nodes as well
 each node can accept read/write, no matter where the data is in the cluster
 mutil-master, multi-DC
 linearly scalable
 larger-than-memory datasets
 best-in-class performance (not just writes, using commit logs)
 fully durable
 integrated caching
 tunable consistency

## components
- data partition: how to partition data across cluster->data center->node
- consistent hashing: row keys->physical nodes
- data replication
	- simple strategy: replica for data determined by the partitioner
 	- network topology strategy: data center/rack aware to replicate data
- eventual consistency
- tunable consistency: tune consistency level
- consistency level: multiple # of replicas exist, level means how many of the replicas should be updated before confirmming read/write successful
- gossip protocol: used for node discovery
- bloom filter: allow false positive, but not false negative, to avoid I/O operation
- Merkle tree:
- SSTable: sorted string table ordered immutable key value map, efficient way to store large sorted data segments in a file
- Memtable: write back in-memory cache that has not been flushed to disk yet
	when to flush
	 - maximum allocated size in memory reached
	 - TTL reached
	 - manually (before turnning down a node)
- keyspace: counterpart of schema in RDMBS, at definition time, need to provide replication starategy and replication factor
- column family: counterpart of table in RDMBS, is in fact a map of sorted map
- row key: a sorted map, aka partition key, used to decide data distribution across cluster


 commit log
 mem-table
 sstable
 bloom filter

## Write process
- client write request --> cluster node 
- inside cluster node, writes go to 'commit log' and MemTable in parallel. when Memtable is full, it is flushed to disk, i.e., SSTable, which is immutalbe. So each time a change is made, new SSTable is written on disk. To avoid extensive IO, we have in-memory MemTable.
  write to commit log is optimized, and less than storing in SSTable. 
  so when turnning down 
