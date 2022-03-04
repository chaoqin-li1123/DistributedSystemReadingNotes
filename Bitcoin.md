# Describe the Problem
Build a digital currency system without any central authority.

# How They Solve The Problem
- Digital Coin: a chain of signed digital signature. The last node of the chain contains the public key of the current owner and a signature signed by the private key of the last owner.
- Block Chain: each block contains hash of the previous block, a nonce, some transactions. 
- Proof of Work: Sign a block by finding a nonce to make the hash of the block have some leading zeros.
- Concensus: always accept the longest block chain. Because the majority of CPU power generates block faster.
- Privaty: The owner of the public key is anonymous.  
- Extra Privace: Generate a new key pair for each transaction.
- Combine/Split Value of a Coin: a transaction contain multiple input/output. Multiple input combine value, multiple output split value.
- Incentive: Each block contains a transaction to the block creator.(the coin may come from transaction fee or newly issue coins per block)

# Engineering Tricks
- Each block uses a merkle tree for hashing. Discards node without affecting the hash.
