We currently have four options for escrowing payments from the sponsors to bounty hunters: DEAL and Allo.
For each possible solution, we are going to provide a high-level overview, list the benefits and drawbacks and a review.
### DEAL
_Link:_ [https://deal.lextek.eth.limo/](https://deal.lextek.eth.limo/)
_By:_ [@kali__gg](https://twitter.com/kali__gg)

Digital Escrow Agreement for Labor (DEAL) is a solution for service providers to generate an on-chain invoice or escrow for services rendered. It supports ENS names and USDC payments.
#### Benefits
• Verified on Basescan, so if we need to deploy to other chains, it would be easy.
• No fees incurred.
#### Drawbacks
• Documentation is [nonexistent](https://lextek.gitbook.io/lextek/ex-e-k/deal).
• Undocumented contract.
• Currently only available on Base.
• Isn’t audited or battle-tested enough. As of the time of writing, the contracts don’t have [any transactions](https://basescan.org/address/0xde001DC6a918070f00e5B3676700050000787800).
• Use of unchecked blocks and inline assembly, which increases the chances of bugs.
![[Pasted image 20240917160402.png]]
![[Pasted image 20240917160153.png]]
• Uses an [unaudited library](https://github.com/NaniDAO/ie) for transferring tokens and wrapping ETH.
• Unable to create a service due to UNPREDICTABLE_GAS_LIMIT error on the UI.
![[Pasted image 20240917162258.png]]
#### Review
This solution doesn’t provide a way for the platform to take a cut from the amount being escrowed by the contract, so a wrapper around this contract would need to be implemented, at least for the creation function, in order to redirect fees to the platform treasury.

During the analysis of DEAL contracts, it was noticed that the project uses an [Escrow](https://basescan.org/address/0x00000000000044992CB97CB1A57A32e271C04c11#code) contract to secure the funds and release them to the service provider once the service is completed. I believe that’s the only part that interests us because the invoice framework provided by DEAL implies that both the client and service providers are known at the time of issuance, but only the client is. The service provider will only be known when a sponsor picks the winner of a bounty.

Both contracts share the same deployer, so likely the Escrow contract is from KaliDAO as well, but the project wasn’t available on their GitHub. Can we get in touch with KaliDAO to get more info about the Escrow contract?

DEAL doesn’t seem to provide the robustness we’re looking for, since the contracts were recently deployed, don’t have any transactions, and are not audited.
### Allo
_Link:_ https://github.com/allo-protocol/allo-v2,  https://docs.allo.gitcoin.co
_By:_ https://gitcoin.co

Allo is a protocol developed by Gitcoin that enables communities to create and manage their own funding mechanisms for public goods projects. By providing a modular and programmable framework, Allo allows users to create pools with custom strategies or leverage pre-existing and audited ones.
#### Benefits
• Project is audited by the Sherlock community.
• Good documentation and examples.
• Flexible enough to build any use case on top of.
#### Drawbacks
• For each pool, a new strategy needs to be created to define how funds are managed. A strategy is a smart contract defined by us, which inherits all the risks associated with deploying a new contract on-chain. Allo provides a handful of strategy contracts, but none of them fit exactly our use case.
• The protocol has a base fee for deploying new pools, which is currently set to zero on all chains but can be used in the future.
• The protocol has a percent fee, which is an amount that will be deducted from every deposit to a pool. As with the base fee, it is currently zero but can be changed in the future.
#### Review
Allo provides a flexible enough framework to build any use case on top of. Despite none of the pre-defined strategies fitting exactly our use case, I’m confident we can use one of the pre-existing ones, such as the [Direct Grants Strategy](https://github.com/allo-protocol/allo-v2/blob/main/contracts/strategies/direct-grants-lite/README.md) to build our bounties around. It needs further exploration and conversation with more experienced people in the Allo ecosystem.

We need to ensure the Allo team doesn’t have plans to turn on the fees; otherwise, it could have a significant impact on the revenue model of 2077 Bounty.

Below is a high-level communication scheme between sponsors, users, the platform, and Allo:

**Create New Bounty**
![[Pasted image 20240917195639.png]]
  
**Bounty Review / Reward Distribution**
![[Pasted image 20240917195539.png]]