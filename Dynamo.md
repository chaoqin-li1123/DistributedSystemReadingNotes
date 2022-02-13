# Describe the Problem
A highly available(always writable) and fault tolerant key-value store.

# How They Solve the Problem
~~~
term
coordinator: node that handle read or write request
~~~
- load balance: consistent hash with virtual nodes, virtual node number proportional to load assignment.
- data replication: replicate an object at N node in the preference list of the coordinator, can spread preference list across data centers or zones.
- hinted handoff: if a node in the preference is unhealthy, send object to an alternate node along with metadata of the intended recipient, real recipient retry to deliver to the intended recipient.
- data versioning: use vector clock, if vector A is a subsequent vector of vector B, then B is the ancestor of A, otherwise there is conflict.
- eventual consistency: can read multiple version of data, have the reader do the read repair.
- sloppy quorum: R + W > N, but R and W don't need can be nodes outside of the preference list if some nodes are unhealthy.
- membership: use anti-entropy gossip protocol to propagate menbership change, send to a alternate peer if current peer is unhealthy, retry the unhealthy peer periodically.

# Some Implementation Details & Engineering Tricks
- Balance background tasks(hinted handoff, replica synchronization) and foreground tasks(put, get), use a controller with a feedback loop. Controller measure request latencies and failures and allocate time slices to background tasks accordingly.
- Amazon SLA: latency percentile of 99.9% requests
- Cooridinator of a request is determined by key, a generic http load balancer is not appropriate(introduce extra hop if routed to node that is not the coordinator). Use a membership aware client to route requests instead.
- quorum configuration:: a typical setting N = 3, R = 2, W = 2. set R to be small in favor of read availability; set W to be small in favor of write readability.
- If data durability is unimportant, buffer the writes in the memory and flush to disk periodically.
