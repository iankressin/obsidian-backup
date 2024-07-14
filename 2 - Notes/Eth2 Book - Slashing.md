---
date: 2024-06-25 08:44
tags:
  - "#inprogess"
source: https://eth2book.info/capella/part2/incentives/slashing/
---
# Eth2 Book - Slashing

- Slashing is a punishment applied to validators who try to cheat the consensus rules. The protocol slashes a piece of the offender's stake and can even result in the ejection of the bad actor from the protocol.
  
- Slashing offences related to Casper FFG consensus:
	- two different attestations for the same checkpoint
	- attest with source and target votes surround those in another attestation from the same validator

- Slashing offences related to LMD GHOST consensus:
	- proposing more than one *distinct* block at the same height
	- attest to different block heads with the same source and target checkpoints

- All the offences listed above are related to "equivocation", which happens when a validator contradicts something it was previously advertised to the network by themselves.
	- This type of offence shouldn't happen to a validator running a regular consensus client, but it can happen if the same validator is running more than one consensus client with the same validator keys. This is awful strategy to increase uptime

- Any slashed value from validators is burned

- After the first slashable offence, `1/32` of the max effective balance (32 ETH) is deducted from the offender. A withdrawable period is set to 36 days.

- During these 36 days, the validator is still part of the validator set, but its unable to fulfill its duties of signing attestations and proposing blocks, therefore it continues to bleed ethers away

- A third penalty is applied 18 days after being slashed, which is called correlation penalty. It punish mass attack events to the chain, by applying greater slashing punishments when more slashable offences are committed in a similar window of time 
  - `Correlation penalty = min (B, 3SB / T)`
  - `3`: the slashing multiplier constant
  - `S`: sum of increments in the list of slashed validators over the last 36 days
  - `B`: validator effective balance
  - `T`: total increments

   