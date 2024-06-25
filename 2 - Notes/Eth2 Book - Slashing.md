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

- 