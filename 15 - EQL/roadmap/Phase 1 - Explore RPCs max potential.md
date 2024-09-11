  Despite the fact that the main goal of EQL is move away from public JSON-RPCs, as they can be seen as centralization vectors in the space, extracting the maximum amount of capabilities from RPCs can support the long term vision of the project in two ways.
1. Act as middle ground while a better and more decentralized solution is being built
2. Help shape the language syntax

> [!NOTE]
> RPC is a good collection of Ethereum high-level structures, so covering most of RPCs means that we covered most the structures that matters.

While EQL doesn't fulfill its long term goal of providing a decentralized solution for querying EVM chains, we need to play with the cards that we have. Using RPCs as the backbone of the `execution_engine` enables the team to focus on other parts of the language.

One of these other components is syntax. In its first version, EQL only supported a hand full of fields for each of the first-class entities. At the time of writing 0.1.1-alpha version is right around the corner, adding support for lots of new fields and block range queries. But we're far from covering all the fields available through RPCs, so we should keep pushing to add all the fields available.

Exhausting RPCs doesn't mean to squeeze every RPC method into the language. Instead we should go through every method and try to understand how it can spark up new syntax possibilities, always keeping in mind the commitment with the user experience and language ergonomics.
### Short term problems
- RPC rate limits
### Alpha roadmap
#### v0.1.3-alpha
- [ ] Logs and filters
	- `GET topic0, topic1 FROM log WHERE block latest address 0x0 ON eth`
- [ ] Dump query results to json, parquet, and, csv
	- `GET nonce, balance FROM vitalik.eth, iankguimaraes.eth ON eth > data.csv`
- [ ] List of entity ids
	- `GET value, to, timestamp FROM tx 0x..., 0x..., 0x... ON polygon`
- [ ] Add support for more EVM chains
#### v0.1.4-alpha
- [ ] Get transactions from blocks
	- `GET from, to FROM transaction WHERE tx.value 1, block 1:10 ON eth`
- [ ] Wildcard operator for both fields and chains
	- `GET * FROM account vitalik.eth ON *`
- [ ] REPL improvements
	- Save query history
	- Fix minor bugs
#### v0.1.5-alpha
 - [ ] User configurable RPC list
 - [ ] Support to transaction receipt fields under transaction entity
 - [ ] Smart-contract support
	 - `GET balanceOf(0x...) FROM contract 0x... ON polygon`
#### v0.1.6-alpha
- [ ] Add, div, sub, mul
	- `ADD(GET nonce FROM account vitalik.eth ON eth, polygon, base)`
- [ ] Get account balance, nonce at specific block and range
	- `GET nonce, balance FROM vitalik.eth WHERE block 1:10 ON base`
- [ ] Support for custom chains:
	- `GET nonce, balance FROM vitalik.eth WHERE block 1:10 ON custom-1`
#### v0.1.7-alpha
 - [ ] Codebase refactor