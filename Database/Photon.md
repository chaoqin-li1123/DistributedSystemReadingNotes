# Motivation
- Build a vectorized query engine for data lakehouse.

# Requirement
1. High and consistent performance query on both structured and unstructured data
2. Integrate with Apache Spark(Databricks Runtime)
3. Good observability and debugability.

# Design Decision
1. C++ over jvm
    * jvm gc and object metadata overhead
    * lack of control over low level optimization in JVM
    * workload become cpu bounded because: 
        * cache and data skipping reduce IO cost; 
        * processing unstructured data in data lake can be cpu expensive
2. interpreted vectorization vs volcano
    * batching amortize virtual function call costs 
    * better data locality and CPU prediction
3. interpreted vectorization over code generation
    * easy to develop and debug
    * good observability, modularity for operators
    * runtime adaptivity to changing data, choose optimal code paths for different batches dynamically
    * It is possible to get the benefit of codegen by creating fused operator in vectorized query engine
4. partial rollout: 
    * Invoke photon operators through JNI and can fallback to spark operator for unimplemented operator.
    * Only invoke photon operators in the bottom of the execution plan. Avoid regression caused by row to column pivot.

# Benchmark
* Cluster setup: a single i3.2xlarge Amazon EC2 instance with 11GB of memory for the JVM and 35GB of off-heap memory for Photon
* Workload: data(TPC-H) and query type(aggregation, join, expression evaluation, parquet write)
* Comparison against: non-photon DBR and open source spark


