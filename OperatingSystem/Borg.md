# Problem
A cluster manager that can schedule tasks in a set of hetergenious machines.

# Concept & Interface
- Task: a Linux containerized process.
- Task configuration: CPU cores, RAM, disk space, TCP port.
- Job: a set of tasks.
- Job configuration: name, owner, number of tasks, machine type.

# Solution
### Scheduling: Which Task to be Executed
- Tasks are divided into classes of different priority(prod or latency-sensitive, non-prod or batch).
- Tasks in prod must be scheduled. This is achieved by admission control: reject job config submission that cause the prod quota to exceed capacity. Evict non-prod tasks if there isn't enough resources.
- Task in non-prod will be scheduled at best efforts. Quota for non-prod level can be oversold to maximize efficiency. Each non-prod priority level has a separate queue with a round robin scheduler.
### Scheduling: Which Machine to Allocated
- Calculate a score that max different criteria.
- Minimize low priority tasks preemption
- Minimize cost of package distribution
- Spread tasks for a job across failure domains.
- Avoid resource fragmentation
### Architecture
- Borgmaster manages the state machine of the cluster. The master process is replicated 5 times and reach state consistency through Paxos protocol.
- Borglet is an agent process in a machine. It start and stop tasks, manage resources, report state to Borgmaster.
- Each running task runs a http server that report states for monitoring.

# Engineering Tricks
### Scale the Borgmaster
- A master process that receive RPCs and manage the state macine.
- A scheduler process that operate on a stale copy of state, poll the master for state update from the master repeatedly.
- Read-only requests and communication with Borglet are sharded to the paxos follower replicas.
