---
date: 2024-06-18 11:20
tags:
  - "#inprogess"
source: https://epf.wiki/#/eps/week2
---
# EPFsg - Week 2

### Block validation
- The consensus layer communicates with execution layer through a function defined in the  consensus spec as `process_execution_payload` which does a series of checks and calls internally `notify_new_payload`, that will notify the execution engine of a new payload that needs to be processed 

- The execution client then, runs the state transition function
	- If headers are invalid the block is rejected
	- If the transaction is invalid the block is rejected
	- If block and transactions are valid, the is state is updated

### Block building
- Nodes are always gossiping transactions over the p2p network. These transactions form the mempool
- These transactions are valid to be included in a block: nonce in correct, account has enough balance to pay for the tx
- 