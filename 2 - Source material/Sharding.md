2024-04-26 09:45

# Sharding

Sharding has emerge as one solution to scale Ethereum transactions per second, without loosing its security and decentralization properties.

In the current Ethereum architecture, each node stores the entire state of the chain and process all transactions. This provided a large amount of security, but also limits scalability: **a blockchain cannot process more transactions than a single node can**.

Sharding comes as a solution where only a subset of nodes process the transactions, as long as there are sufficiently many nodes processing the transactions. This allows the system to use the rest of the subset of validators to process other transactions in parallel. 
In simpler words, sharding wants to allow Ethereum to have 10,000+ TPS, without forcing each node to be a supercomputer or to store a terabyte of state data.

In Vitalik's words, on interview to Lex Friedman, sharding is: "I as an individual participant can be convinced that everything is going on in this distributed blockchain thing is correct, without actually personally check more than a percent of it.".
### Basic sharding
![[Pasted image 20240417060212.png]]
[[basic-sharding]]

### Danksharding and Proto-Danksharding
Danksharding is how Ethereum becomes a truly scalable blockchain, but there are several protocol upgrades required to get there. Proto-Danksharding is an intermidiate step along the way.
#### Proto-Danksharding
Proto-Danksharding is a way to rollups to add cheaper data to blocks. Untill EIP-4844, rollups used to post data on Ethereum L1 using CALLDATA, but this is an expensive transaction since needs to be processed by all Ethereum nodes and lives on-chain forever, even though rollups only need the data for a short period of time.

Proto-Danksharding adds a new space in the blocks called blobspace that are delete after a 4096 epochs, or about 18 days. This means rollups can send their data cheaply and pass savings on to the end user in the form of cheaper transactions.

#### Danksharding



##### Questions
- Is it possible to challenge committees on ethereum today?
- Are shards already working on ethereum today?
- Shards vs Roll ups? Does sharding mean the end of roll ups
##### Notes
the main problem sharding wants to solve is how can nodes that only verifies 100 txs/s can verify 10.000 txs/s => how to scale the blockchain not only scaling the parameters.

sharding is also the ability that a individual has to verify the integrity of a blockchain, without the need to recalculate all the blocks until present 

random sampling: the model that we have today

zero knowladge

##### Structure
- Intro => what's sharding and what compose the technique of sharding?
- Explore those techniques individually
- Is Sharding being used today on Ethereum? If yes explain where
- Potentially down sides of Sharding

## References