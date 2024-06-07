---
date: 2024-05-31 07:45
tags:
  - Ethereum
  - final
description: The Ethereum scaling solution
---
# Danksharding

### Intro
Danksharding is a way to scale Ethereum TPS without hurting the blockchain trilemma. It envision a new form to distribute data around the blockchain that does not require validators to download all the blocks in order to get up to speed with the chain history. The block data is distributed across validators in  a way that if someone needs to verify the integrity of the chain and validate its state, it can query the validators for the pieces of data, and then rebuild the blocks history.
### Motivation
The original vision of Ethereum was scale the blockchain through sharding chains, that is, splitting the Ethereum chain into smaller chains, and each of these chains will be responsible for processing a smaller portion of the transactions/blocks. Sharding the L1 would potentially increase in orders of magnitude the TPS of Ethereum. For the end users, there would be no difference interacting with the chain before and after sharding, as this complexity wouldn't be exposed beyond the mainnet.
Despite the benefits, the original idea of sharding, also brought challenges. As the chain is breakdown into smaller sets of validators, in order to increase the transaction throughput, these sub-chains, are a more susceptible target to attackers, since they are a narrower attack surface. It also presented challenges in syncing the shard data between shards, which could be resource intensive and re-allocate the processing power from validating blocks and transactions to communication within the chain.

Danksharding emerged as a solution to the problems above. Instead of having execution and validation designated to subsets of validators, this approach proposed that the L1 would focus on scale validation and data availability, while the L2 would scale the execution.
These approach solves the shard centralization problem, since all the validators on Ethereum would be involved in the process. The inter-shard communication is solved by a complex set of zero knowledge proofs, polynomials and data verification.
This solution reshapes the Ethereum end game and gives birth to a rollup-centric roadmap, where, as mentioned above, the execution scalability is pushed to the L2s.
### Full Danksharding
Once this vision is full implemented, the blocks would be split into smaller chunks of data, which will go through an encoding algorithm called Reed-Solomon, which extends original data by including other pieces of data, and if the original chunks become unavailable, the full content of the block can still be rebuilt with only 50% of the extended data.
Reed-Solomon takes advantage of a polynomial property that allows a polynomial of degree *N* can be recreated through *N+1* points of the graph. 

This data is then distributed through the network of validators and validate by *sampling validators*, which are regular validators who gets assigned to the task of validating that at least 50% of the chunks were made available to the chain.

To avoid attacks where malicious validators encode the data with 50% of real data and 50% of junk, along with the data, they should also post a commitment to the polynomial which was used to produced the erasure coded chunks using KZG commitments. 

To solve the execution scalability, Danksharding pushes the execution scalability to the edge via L2. These L2 are able to provide cheaper gas fees for their users thanks to EIP-4844 (Proto-Danksharding), that includes a new block space, called blob space, which is an ephemeral storage on the blockchain, where data can be posted much cheaper when compared to CALLDATA, since validators don't need to store them for long periods. This new space on the chain can be used by rollups to post the data necessary to validate their execution. The only thing that L2 post on chain and will live forever is a commitment to the data posted on the blob, which is a small proof and irrelevant in size.

### The problems with Danksharding

In my viewpoint, Danksharding brings fragmentation problems to the table when compared to the old shard proposal.

Since execution is pushed to the L2, the protocol encourages transactions to the edge as well. With the explosion of rollups and the native support of the protocol, users move from L1 to L2, in order to avoid the high gas prices that mainnet imposes. Along with the users, the liquidity also migrates from a single source, to a wide variety of chains, which end up causing a sub-optimal liquidity for tokens on DEXes, hurting the trading experience of users.

Also, protocols are required to support a lot of different chains in order to capture this liquidity. This can lead to poor onbording for new crypto users.

![[Pasted image 20240531102553.png]]
![[Pasted image 20240531102533.png]]
![[Pasted image 20240531102819.png]]

## References
