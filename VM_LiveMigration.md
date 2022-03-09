# Problem
Migrate a live virtual machine to another physical host, minimize the downtime.

# Solution

* Networking: rewrite the virtual network routing rule.
* Memory copying(3 phases, but a real system only combine 1 or 2 phase)
  * Push: send memory pages to dst host
  * Stop and Copy: suspend the VM, copy the remaining dirty pages and the CPU states.
  * Pull: upon page fault on pages that haven't been copied, pull from src on demand
* Iterative Pre-copy: copy dirty pages in each round, terminate when the dirtying rate is larger than the maximum bandwidth.
~~~
Dynamic ratelimit:
dirtying_rate = pages_dirtied_in_a_round / round_interval
transfer_bandwith = dirtying_rate + constant
~~~
* Track dirty pages: trap on VM's write, then VMM can set the dirty bit on the bitmap.
* Optimization
  * Lower the priority of the processes that cause many write page faults.
  * Flush on-disk buffer to reduce amount of memory to be transferred.
  * Migrate VM with a small set of hot pages.
