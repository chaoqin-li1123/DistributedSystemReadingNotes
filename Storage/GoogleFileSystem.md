# Describe the Problem
- Design a fault tolerant, consistent distributed file system for large scale data processing workload.
- Workload pattern: huge file(GB), appending write and sequential read are more common than random read/write. Throughput more important than latency.

# How They Solve the Problem
- Architecture: master(metadata) and chunk servers(data), divide files into chunks(64MB), store N(default: 3) replicas. 
- Chunk server is the only source of truth of chunk status. Chunk server check data integrity locally by checksum. Master poll chunk server at startup and periodically to update its view of chunk status and do replication upon failure dection.
- Master apply operation logs sequentially to mutate the file namespace.(atomically). Replicate the operation logs on multiple machines, restart a new master when the master is down.
- Support atomic append(only guarantee append at least once, it is up to client to remove duplicate and padding)
- A chunk server is granted a lease to a chunk(primary server), mutations are pushed to all replicas, primary server specify an order these mutations are applied.

# Engineering Tricks
### Garbage Collection vs Eager Deletion
Pros
- Explicit delete message may be lost.
- Batch deletion for lower overhead and don't interfere with normal traffic.

Cons
- Can not fine tune the create-delete from the client side. Maybe client create a lot of temp file and explicitly delete to reclaim storage.
### Data Replication
- Replicate across rack for failure isolation.
- Pipeline data transfer over TCP connection to reduce latency.
- Use version number to detect stale replicas.
### Snapshot
- Duplicate metadata on master first.
- Data copy on write: upon request to mutate snapshotted chunk, replicate the chunk locally before serving the request. 
### Efficient Namespace Lookup and Mutation
- Lookup table with prefix compression that maps a full path to a metadata.(not a recursive data structure)
- There is a read-write lock associated with each node.
