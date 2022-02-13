# Describe the Problem
Balance TCP/UDP connections(uniquely identified by 5 tuple) to a set of backends. 
- load balance: make the load distribution as balanced as possible.(matric: max traffic / avg traffic)
- connection affinity: when an endpoint is removed or added, minimize the probability that the backend selection result changes.

# How They Solve the Problem
- Process packets with commodity computers, no special hardware, bypass kernel networking stacks, copy packets to user space directly.
- maglev hash algorithm, hosts take turns fill in a fix size hash table
~~~
array<host, M> maglev_table;
func build_table() -> void {
  while (!maglev_table.empty()) {
    for (host: hosts) {
      maglev_table[empty_entry] = host;
    }
  }
}

func lookup(hash) -> host {
  return maglev_table[hash % M];
}
~~~
- Connection affinity guaranteed by hash consistency and connection table. if connection not found in the connection table, compute the hash of the connection 5 tuple and look up the maglev table.

# Implementation Details
- Control plane/ data plane architecture: use RPC to update backend pool.
- Bypass expensive kernel networking stack
- Batch process packets, start processing a batch when it is large enough or a timer expire
- Use hash to dispatch packets packets into queues: packets for the same connection always handled by the same thread to prevent packet reordering. fall back to round robin if the destination queue is full.
- Each queue is handled by a worker thread which is assigned a dedicated cpu, no resource sharing, no synchronization overhead.

