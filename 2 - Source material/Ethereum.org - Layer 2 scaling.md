---
date: 2024-06-03 09:08
tags:
  - final
source: https://ethereum.org/en/developers/docs/scaling/#layer-2-scaling
---
# Ethereum.org - Layer 2 scaling

- Scaling is part of an off-chain scaling solutions are Ethereum-compatible scaling solutions that aim to scale the transaction throughput without sacrificing decentralization (to a level).

- The type of scaling solutions that derive their security of Ethereum, or in some form uses the ecosystem created by the mainnet is called Layer 2

- The most adopted Layer 2 solution are rollups

- Rollups execute transactions off-chain and only post the transaction data to Ethereum, which off-loads the computation of mainnet, while still leveraging the security of the L1.

- There are two types of rollups: Optimistic and ZK rollups

- Optimistic rollups use a fraud proof mechanism to validate the validity of the data being posted on L1.
	- Every time a new data is posted on-chain, an external observer checks if the data is valid or not. If it isn't a dispute begins on L1, where the party with the correct computation receives a reward from the other participant of the dispute. The sequencer then, recalculates the and post the correct execution transaction.

- ZK rollups, as the name denotes, uses zero knowledge proofs to attest the integrity of the data being posted to the L1. This proofs can be quickly validated and are much faster then the dispute mechanism implemented by Optimistic rollups.
