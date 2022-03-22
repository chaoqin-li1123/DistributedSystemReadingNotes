# Problem
A decentralized, turing complete smart contract system. 

# Concept
- Account: an entity that has id and balance.
- External account: an entity that can send message.
- Contract account: an entity with code and storage. Receive message and invoke code execution.
- Gas: currency in Ethereum.

# Solution
- Turing complete programming environment: Prevent infinite execution. Each code execution has initial gas. If gas is exhausted, code execution stops and the state change is rolled back.(fees are still deduced)
- Virtual machine: stack, memory, persistent storage, code, program counter. While there is remaining gas, fetch the intruction at PC offeset and execute it.

# Storage Optimization(How each block carries the full state?)
- State stored in a tree, only the modified subtree needs to be stored in this block, the unmodified tree can be referenced by its hash value.
- Only store the hash/fingerprint on chain.
