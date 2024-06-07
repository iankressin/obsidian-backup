2024-05-10 08:03

Link: https://www.youtube.com/watch?v=N5p0TB77flM&t=1814s

# Bankless 119 - Sharding

- The Ethereum roadmap which originally proposed L1 sharding, now proposes a rollup-centric roadmap. Where thr L2 converts is non-scalable data into blobs and post it to the L1

- Ethereum provides a scalable data and non-scalable computation and L2 converts scalable data and non-scalable computation into scalable computation.

- The rollup-centric roadmap, still leaves the door open for L1 sharding execution

- Idea that the block builders have to track all the shards because of the separation between proposers and builders.

- Proposer builder separation (PBS). Is a technique where validators on the beacon chain only have to worry about proposing a block, whilst a new entity called block builder will aggregate all the transactions on the mempool and build the most MEV-efficient block, which will be available for block proposers to propose.

- Proposer uses data availability sampling to ensure the blocks being offered are valid and correct.
  
- Danksharding => DAS, PBS

- Danksharding introduces a new market on Ethereum: blob market. Beyond the bids for including the transactions on chain, now we have a separate market from agent who wants to include their data on the Ethereum blob space. 

#### DAS
- you're able to download some amount of some data if you want it to in a scalable way.
  
- danksharding introduces a new way to encode the data where nodes can verify the availability of the data using only pieces of it
  
- the data is encoded through an algorithm called Reed-Solomon, where you can verify the integrity of the whole data only with only 50% of it, and reconstruct the whole data from those pieces