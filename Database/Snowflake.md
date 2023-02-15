# Motivation / Problem to Solve
A cloud native database: use cloud object store as storage and EC instances as compute.

# Requirement
1. Relational interface.
2. Serverless, no cluster provisioning and performance tuning.
3. Handle bot structured and semi-structured data
4. Elastic and scalable, can scale storage and compute easily
5. Fault tolerant and high availability.
6. Secure

# Design Decision
1. Architecture: decouple storage and compute, database metadata and virtual clusters managed by a multi-tenant cloud service layer.
    * Scale compute and storage separately.
    * No eager data migration when resizing compute.
    * Cons: Early filtering/ predicate pushdown support is limited in cloud object store.
3. Storage format: use hybrid columnar format, row groups and columns in a file
    * Good for vectorized execution. Data and instruction locality, SIMD
    * Reduce total I/O cost by only reading necessary column in a row group. Read header of a file to determine the byte range of data that needs to be read.
    * Better compression
    * Data skipping with per column stats for each row group
4. Query Planning and Execution: 
    * Top-down cost based optimization
    * Vectorized execution
    * Compute instance spawn a new worker process for each task from a query, retry for fault tolerance.
    * Work assignment
        * Use consistent hashing to assign workload.(Worker instances are stateless, no eager data migration) 
        * Use work stealing to handle straggler.
5. Version management: A version of control plane only send tasks to the same version of data plane. Old versions stop accepting traffic and retire after all running workers finish.
6. Concurrency control protocol: MVCC concurrency control.
    * Cloud object files are immutable. Version -> files mapping stored as metadata.
    * Efficient time travel and clone.
    * Parallel query from a immutable snapshot without synchronization.
    * OLAP workload, mostly bulk read, insert, update
 7. Security. Hierachical key model: root key, accout key, table key, file key. Key rotation and rekeying in the background. Role based access control.
    
 # Benchmark
 1. Workload: structured and unstructured TPC-H like data.
 2. Cluster setup / resources: medium standard warehouse.
