## EQL intro
- What  EQL is
- Structure of post
## The problem
- Ethereum storage model doesn't allow for relational queries
- It's hard to explore the blockchain when you're not sure what data you want
- It's hard to explore the blockchain when you want to analyze ranges of data
- It's not trivial/performant to look for stuff (linear searches are not great)
- EQL syntax it's easier to learn than javascript/python
## The current landscape
- Current solutions
	- Etherscan
	- ~~The Graph~~
		-   Focus on contracts. 
		-  Not free, despite free tier
		-  Too much heavy lift for simple queries
	- ~~JSON-RPC~~
	- ~~Dune~~
	- Cryo
Each solution should follow the pattern above
- General overview
- Benefits
- Problems => why they are related to EQL
## ~~Acknowledge past work on query languages (Should be another post)~~
- ~~Ethereum Query Language~~
- ~~Cross-chain Query Language~~
- ~~EBTrees~~
## How EQL wants solve it
- We have the vision where we want to be, but just don't know how yet
- Distributed BTrees (distributed data aligns with Ethereum values)

## Details of current implementation
- GET queries
- Supported entities
- Supported chains
- Examples
## Minor goals
- relational query language
- simulation support
- extend syntax
## Development philosophy
-  the efforts to reach end game cannot be estimated time-wise, since were not sure which solution should be used. During the research phase, EQL will deliver intermediary solutions to abstract away from devs and researchers the complexity of querying EVM-chain data.
-  EQL will never require an API key