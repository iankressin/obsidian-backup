---
date: 2024-06-03 10:11
tags:
  - final
source: https://ethereum.org/en/developers/docs/scaling/optimistic-rollups/
---
# Ethereum.org - Optimistic rollups

- Optimistic rollups executes transactions off-chain, and post their outcome to the layer 1

- To prove that data is valid, optimistic rollups implements a dispute system where after a bundle of transactions is posted on the layer 1, anyone can verify the validity of those transactions.
	- If someone spots an invalid transaction, a dispute between the sequencer and the challenger begins
	- The challenger produces a fraud-proof, where is proves the result of a transaction is invalid
	- Then a third-party runs the proof, and if the challenger is correct, the sequencer is forced to recalculate the computation and post the correct data on-chain. Also he's forced to pay a sum of ether to the challenger
	- If the challenger is wrong, they also need to pay a sum of ether to the sequencer

- This approach is called "Optimistic" because if, in a window of time, no validator challenge to the on-chain data is presented, the data is considered valid and settled on-chain, and the block is considered valid.

- Block produces can continue to build on top of unconfirmed blocks, but if a block is considered invalid, then all the blocks built on top of the invalid block are also considered invalid.

- Optimistic rollups run a set of smart-contracts of Ethereum L1 that keeps track of blocks, state updates and track users' deposits.

- Optimistic rollups are considered a hybrid scaling solution, because while their leverage Ethereum security to store their execution data, they still exists as a separated protocol

- Censorship resistance is achieved in op rollups by forcing operators (sequencers) to post data to the L1
	- If an operator start to censor user transactions or halt the block production and halt the chain completely, another operator can pick up the chain state from the L1 and use the onchain data to resume the block production.
	- The censor node, will need to pick up with the new blocks produced, so they can resume their block production as well, being forced to accept the transactions previously censored 

- Some optimistic rollups choose to not run the L2 permissionlessly. Instead of using nodes and operators, the chain uses a single sequencer only it has the permission to build blocks, process transactions and interact with the L1 contracts. It also has priority on the rollup chain, that is, if a sequencer wants to execute a transaction, its included before anyone else's.

- Rollups use either calldata or blobs to send compressed batched transactions to their on-chain contracts. Calldata shouldn't be used any longer in the future, since blobs are much cheaper

- State transaction happens on the L1, when an operator posts the new state root along with the old state root. If the old state root is the same as the one on the L1, the new state root substitutes the old one. The contract accepts new state roots immediately after they're posted, but if challenged and proven wrong, the new state root can be reverted to a previous correct version.

- There are two ways of fraud proofs among the rollups ecosystem: single-round and multi-round interactive proving
	- Single-round: a asserter posts the data on-chain, a challenger disagrees with the data posted and start a dispute. The challenger provides a new state root, and to validate which new state root is correct the whole transaction is executed on-chain. This fraud-proof has expensive execution, since all the transaction needs to be re-executed.
	- Multi-round: After challenged, an asserter bisects the transaction data, and the challenger picks where in the two halves the transaction has an invalid state transition. This process then repeats, until there's a single step of execution. Then this step is executed on-chain, and the contract picks the winner of the dispute. This is called multi-round, because asserters and challengers need to respond to one another on-chain, within a window of time. If one of the parties fail to respond during this period, the contract considers a forfeit from that party.

- To enter in an optimistic rollup, a user make a transaction depositing tokens on the bridge contract for that specific rollup, this transaction is then queued and later executed by the sequencer on the L2, which will mint the tokens to the user on that chain.

- To withdraw the tokens to the L1 again, the users need to interact with the bridge in the L2, which will burn the tokens on that chain, and wait for the dispute time to finish on the L1 (7 days), so after that the withdraw is complete. In order to have a better UX, there are service that act as liquidity providers for cross-chain liquidity. They assume a transaction from the user, asking to withdraw their funds, re-execute this transaction to validate it, and if valid they provide the users the tokens on the L1. The tokens on the user's transaction will go then to the LP

### Questions
*Why rollups only need blobs to be available for a given period of time?*
Optimistic rollups uses a fraud-proof mechanism to guarantee that the transactions being posted on chain are valid, after a window of time, if no challenge arrives, the block the data is considered valid, and the new state root hash is calculated. After that, the all the transaction data is not relevant anymore, so it can be safely discarded. But it needs to live on-chain for sometime so challengers can reconstruct the state and check if the state hashes being posted are valid 
  
*Where the dispute system mechanism takes place on the L1? does it require some kind of protocol validator intervention?*
It all happens on a smart-contract, it doesn't require any protocol intervention. Once a data is posted on chain by a sequencer or an operator, anyone can verify the validity of the data and challenge that data providing a new state root. The smart-contract uses a multi-round fraud proof system, where between a message exchange (using the contract) between challenger and asserter, the challenger points which step of the transaction is invalid. Then the contract re-executes only that piece of transaction and decides who's the one lying

*Who resolves the dispute between sequencer and challenger?*
The fraud-proof smart-contract

*What are exactly fraud-proofs?*
Fraud proof is the process of proving that the current state root hash is wrong, and provide evidence to a smart-contract that will judge which party is correct. 

*Can sequencers extract MEV? Do them extract MEV?*

*During message exchanges between L1 and L2s who picks up the messages on the L1 to deliver to L2?*