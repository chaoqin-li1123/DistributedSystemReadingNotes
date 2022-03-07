# Describe the Problem
Network virtualization for large scale cloud platform.

# How They Solve the Problem
- IP layer tunneling. Encapsulate the original ipv4 packets(virtual machine src/dst virtual ip, virtual network id) in ipv6 packets(physical host src/dst ip)
- Control plane: most routes go through a central gateway(Hoverboard), which knows the route between every 2 VMs. When the controller detects high bandwidth route, programme a direct path in the on-host switch.
~~~
Common Model in SDN
1 Preprogrammed Model: programme a full mesh of forwarding rules from each VM to every other VM, not scalable
2 On Demand Model: fetch path from controller upon the arrival of the first packet, large tail latency, control plane vulnerable to DoS attack
3 Gateway Model: all packets sent to a small number of gateways, easy to scale control plane, but gateways are bottleneck and needs to be overprovisioned.
Hoverboard model is a combination of the preprogrammed model and gateway model.
~~~
- On host data plane: a simple fast path(simple, high throughput and low latency) and a coprocessor(enable a large set of features) per host.

# Engineering Tricks
- Communication between process use single producer single consumer lock free packet rings in shared memory.
- Fairness and resource isolation: When the packet processing thread is shared by VMs(fast path), have per VM packet queues, scheduler find the queue that consumes the least CPU. Fairness in coprocessor is guaranteed by a dedicated thread per VM.
- Fail static: fallback to the last-known-good state when controller in unreachable.
- Coprocessor pattern: a shared processor and per customer coprocessor. Coprocessor usually has a large set of features. It perform cpu/memory intensive or risky tasks(running scripts, make system calls). This pattern enforce isolation and facilitate billing.  



