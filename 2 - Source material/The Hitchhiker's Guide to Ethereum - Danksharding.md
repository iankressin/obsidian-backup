2024-04-26 16:15

Link: https://members.delphidigital.io/reports/the-hitchhikers-guide-to-ethereum

# The Hitchhiker's Guide to Ethereum - Danksharding

- Danksharding aims to improve rollups validation and data availability, ending in rollups scalability.
  
- Rollup-centric roadmap optimize for data-hungry rollups. This is achieved via data sharding or big blocks
	- data sharding is scalable
	- big blocks are scalable, until the new transaction limit is reached, then they're not

- On danksharding, **validators** conduct **DAS** to confirm that all data is available and a single entity proposes a large block with the beacon block and all the shard data together. **(PBS)**
  
- **DAS** allow nodes (even light clients) to verify that the data is available, without downloading it all.
  
- Through a polynomial property that states that we can reconstruct => This post doesn't clarify how polynomials are used to reconstruct the original data with the samples available. Also it has some magical number being thrown around. SHOULD LOOK MORE IN DEPTH INTO Reed-Solomon code. => The polynomial is reconstructed via Lagrange interpolation.

- When validating the data sample collected from nodes, we need to guarantee that data made available by a node is a valid erasure coded data. Normally when we want to guarantee the inclusion of some data within a hash, we use merkle roots. But in this case this isn't possible since who produced the hash could have post 50% of a valid data and the other 50% of junk, so the junk would be valid within the hash posted.

- KZG commitments allows to commit to a polynomial.
	- The polynomial is used to evaluate the chunks of data from DAS through LaGrange Interpolation
	- The *block proposer* (Q4) creates a commitment saying that all the chunks of data sampled will be a (x, y) point in a given polynomial
	- This commitment uses an elliptic curve created using a secret number S
	- Elliptic curves allows someone to know all the points in the graph but not be able to retrieve S
	- The numbers created by S allows to evaluation of the polynomial at this secret S point
	- The property above matters because while evaluating a polynomial, we don't care about the value of S, we just need to know if the values passed to the polynomials lies on the graph of that polynomial
	- S should be properly discarded after the creation of the curve
- DAS allows us to check that erasure coded data was made available. KZG commitments prove to us that the original data was extended properly and commit to all of it.

- PBS supports the vision of a centralized block building and a decentralized block validation. PBS makes the proposer role as easy as possible to support the validator decentralization.

- Epochs are 32 slots long. 1/32 are designated to attest the data availability of each slot.

- Validators attest the validity of 2 rows and 2 columns in their assigned slot => more than 50% of each row is available, more than 50% of each column is available
  

### Questions
1. When we talk about data availability sampling (DAS), what data exactly are we talking about?
2. When the blocks are verified in DAS
3. Why is it safe to delete the blobs in the blob space after some time?
4. Who creates the KZG commitment?