2024-04-26 09:42

Status: #final 

Tags: [[6 - Tags/Ethereum|Ethereum]]

# Beacon Chain

The Beacon chain was introduced during in the The Merge section of Ethereum proof-of-stake roadmap. It was introduced first as a chain of "empty" blocks, back in 2020, and now it serves as the consensus layer for Ethereum.

The role of Beacon chain is to handle blocks and attestations, also it is responsible for rewards and penalties to stakers. It does not handle transactions, smart-contract interactions or any type of execution.

After the introduction of the Beacon chain, nodes were required to run two clients. One for the execution layer and other for the Beacon chain itself. Although these two clients are generally included in node software like geth and reth. 

Introducing the Beacon chain was a step forward in the direction of [[Danksharding]]
### Slots
Slots are the heartbeat of the Ethereum network. In the beginning of a new slot, a [[validator]] is randomly selected to propose a block and get the rewards of the proposition process.
But not all slots results in a new block. If the selected Validator is offline or fail to propose a block for some reason, the slot may pass without a block being included to the chain.
Each slot lasts 12 seconds.
### Epochs
An Epoch is a set of 32 Slots. Every new epoch triggers certain events
- **Validator shuffling and committee:** at the beginning of each new epoch, the set of Validators are shuffled and a new committee is assigned.
- **Finality and rewards:** the beacon chain uses epochs to achieve finality, a process where blocks and the beacon chain state are considered irreversible. Also at the end of an epoch is where the rewards are distributed to block proposers and committee members
### Block proposition
A block proposer is a Validator, randomly assign at the beginning of each slot, to propose a new block.

The pseudo-randomness of block proposer selection is done through an  algorithm called RANDAO that mixes a hash from the block proposer and a seed that gets updated every block, this value is used to pick a Validator from the total Validator set.

The chances of a Validator being selected is not simply `1 / N` where `N` is the total number of Validators. It is weighted by the effective balance of each Validator, the maximum effective balance being 32 ETH.

Only one block producer is selected each slot, and only one block should be included in this process. Including more than block results in a slash, often known as "equivocation".
### Validators committee
Every **slot** has it own attestation committee, which is responsible for attesting to the validity of the block produce in that slot. A committee is formed by a minimum of 128 Validators in being one of them responsible for collecting and aggregating all the signatures. Every Validator participates in only one committee at a time.

Committees are spread across the slots of an epoch, and are responsible for validating the block produced on that slot.

The composition of the committees for an epoch, is fully determined at the beginning of that epoch.
### Attestation
An attestation is a Validator's vote on the validity of a block, weighted by its ETH balance. The message signed by a Validator in order to attest a block contains the following fields:
- Slot number
- Beacon block root: a reference to the proposed block
- Source root and source epoch: the last root hash and epoch that the Attester is aware of
- Target root and epoch: the root and the epoch for which the attestation in being made
- Signature: the private key signature of the Attester
### Checkpoints
A checkpoint is a block in the first slot of an epoch. If there's no such block, then the checkpoint in the next most recent block. There is always one checkpoint per epoch. A block can be a checkpoint for multiple epochs (in extreme conditions).
![[Beacon-Chain-Checkpoints.jpg.webp]]

When casting a vote to attest the validity (LMD GHOST), the Validator also votes for the checkpoint in its current epoch, called the **target** (Casper FFG). This vote also include a prior checkpoint called source. LMD GHOST votes are only available for Validators assigned to a slot, but all Validators can cast Casper FFG.
### Finality
When an epoch ends, if its checkpoint has garnered a 2/3 supermajority, then the checkpoint becomes justified. If a checkpoint A is justified and the immediate next epoch becomes justified, then A becomes finalized. Typically, a checkpoint is finalized in two epochs, 12.8 minutes.

## References