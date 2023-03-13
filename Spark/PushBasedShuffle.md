# Current Spark Shuffle Mechanism
1. Each executor registers itself with the External Shuffle Service(ESS) located on the same node.
2. A map task produces an index file and shuffle file. Index file contains offsets of shuffle blocks inside the shuffle file.
3. Reduce tasks query driver for locations of input shuffle blocks, establish connection to corresponding ESS instances in order to fetch input data.

# Limitation of Current Implementation
1. Low io efficiency: random disk read IO of small shuffle blocks(KBs). HDD random write throughput is much higher than random read throughput
2. Reliability issue caused by large connection number(numExternalShuffleServiceInstances * numExecutors) in shuffle stage
3. Poor data locality for reduce tasks

# Requirement
1. Avoid IO efficiency caused by small shuffle blocks random read
2. Alleviate potential Spark ESS connection failures
3. Cope with potential stragglers and data skews
4. Low memory and cpu overhead

# Design Sketch
1. Spark driver coordinate map tasks, reduce tasks and shuffle services
2. Shuffle map tasks push shuffle blocks to remote shuffle services(map task duration don't increase with async rpc)
3. Shuffle service accept remotely pushed shuffle blocks and merge them into merged shuffle files
4. Reduce tasks use the merged shuffle files(usually co-located with the reduce task)

# Design
# Push-Merge Shuffle
1. In the pusher side: read shuffle file sequentially and push chunks that contain shuffle blocks.
2. In the receiver side: (application id, shuffle id and shuffle partition id) uniquely identify a merged shuffle file. shuffle file metadata: mapperId -> offset of the block

## Block Pushing Algorithm
1. A chunk contain continuous blocks
2. Shuffle blocks for the same shuffle partitions go to the same shuffle service
3. Should distribute workload across time and workers, avoid shuffle service receiving all chunks at the same time with randomness.

## Fault Tolerance
1. If a map task failed, normal spark task retry.
2. Give up shuffle chunk push attempt after max retries.
3. Reduce task can read shuffle blocks from partially merged shuffle files or fallback to fetching from original location.

## Replicate shuffle blocks in both original location and destination location
### Pros: 
1. Better fault tolerance, can always fallback to existing mechanism
### Cons:
1. Storage amplification(not an issue, temporary shuffle file is only a small fraction of cluster storage capacity)
2. Write amplification(not the bottleneck for shuffling)

# Evaluation & Benchmark
## Cluster Setup
56 cores, 256GB RAM per node, 6HDDs with 100MB sequential read/write IO. YARN NodeManager instances
## Workload 
Different total shuffle IO, CPU intensive and IO intensive queries
## Metrics
1. Resource usage: CPU, on heap / off heap memory consumption
2. Query execution time,duration of different stages


