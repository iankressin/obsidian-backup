---
date: 2024-06-11 15:35
tags:
  - "#inprogess"
source: https://ethereum.org/pt-br/developers/docs/scaling/zk-rollups/
---


# Ethereum.org - ZK

- The main difference between ZK and Optimistic rollups is the approach to guarantee the integrity of the off-chain computation result.
	- Optimistic rollups => Fraud proofs
	- ZK rollups => ZK proofs

- The state of a ZK rollup is stored on the L1 through a smart-contract.
	- In order to update this state, a rollup opertator needs to post the compressed batch of transactions along with a validity proof for verification.
	- The state transition of a rollup only depends on the validity proof being verified.
	- Because of the fast block finality, there are no delays from L2 -> L1 messages. 

- Like Optmistic rollups, ZK rollup's core components are:
	- **On-chain contracts:** store blocks, track deposits, and monitor state updates
	- **Off-chain VM:** environment where transactions are executed 

- Anyone can recreate the current root hash through the transaction batches posted on the L1 *(is it true after EIP-4844?)*

- Most ZK-rollups use a supernode to produce blocks, which is efficient but centralized. If users feel that they are being censored, the L1 contract accept bundled transactions directly.
#### ZK-SNARKs
- ZK-SNARKs require a Common Reference String (CRS), which are public parameters for proving and verifying statements. If the parameters used to create CRS becomes public, then malicious actors can create false proofs.
  
- Multi-party computation ceremony (MPC) can be used to generate CRS, where each participant contributes with some randomness and destroy the data immediately after.
  
- As long as one participant discard their input, the security is guaranteed
  
- ZK-SNARKs are popular for the small proofs and constant-time verification
#### ZK-STARKs
- ZK-STARKs doesn't require a trusted setup (like CRS for ZK-SNARKs)

- Rely on **public verifiable randomness** to create proofs and verifications

- the time needed to verify a ZK-STARK scales quasi-linearly while in a ZK-SNARK scales linearly, which makes a ZK-STARK more efficient when large datasets are involved.

   - Produces large proofs, therefore is more expensive to verify on Ethereum
   
   - Secure against quantum computers

#### ZK-Rollups and EVM Compatibility
- Proving EVM computation in circuits is more difficult than proving simple computations

- zkEVMs recreates existing EVM opcodes for proving/verification circuits, allowing ZK-Rollups to have smart-contracts

- zkEVMs produces ZK proofs on each step of the program's execution

#### Questions

*Do ZK rollups take advantage of blob space?*

*Can anyone reconstruct the current root hash with the L1 data?*