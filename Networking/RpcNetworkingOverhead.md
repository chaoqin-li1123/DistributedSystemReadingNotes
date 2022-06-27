# Why
Where the latency of rpc come from?
Understand Rpc overhead in different layer for performance optimization.

# What Contribute to RPC Latency
- Protocol overhead. Framing, handshake, encrypt/decrypt, encode/decode. The overhead will be significant if we have a large number of small size packet.
- Data copy from kernel to user space. (RAM bandwidth and CPU cycles)
- RTT and bandwidth limit.
- Network congestion and retransmission.
- Flow control. Socket buffer size limit bound the receive rate. 
- Lock contention in user application or rpc thread pool.

# Maximize Throughput, Minimize Latency
- Choose a low overhead protocol or tune the protocol configuration.
- Bypass kernel networking stack, zero copy.
- Batch messages to combat fragmentation.
- Design message proto to be compact.
- Prefer streaming to unary RPC calls.
- Data compression to avoid sending redundant information.
- Use message passing and lock free shared memory for inter-thread communication. Make state thread local.
- Inline light weight processing to avoid message passing and synchronization cost.
Delecgate heavy weight processing to a separate coprocessor thread in order not to block critical operation.
- If possible, pin worker threads to a delicated core.
- Upgrade network devices.
