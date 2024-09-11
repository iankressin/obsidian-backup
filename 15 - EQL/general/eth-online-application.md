I've been researching ways to build a SQL-like language on top of Ethereum and other L2s with the ultimate goal of make on-chain data more accessible for researchers and dapp developers by enabling relation queries on the main blockchain entities (accounts, blocks, and transactions).

I've put together an alpha release of such language, called EVM Query Language (EQL), with limited capabilities. At the moment the language can only map syntax to RPC calls.
The repo can be found in the link below:
https://github.com/iankressin/eql

For this hackathon I want to give another two steps in the direction of the language's end-game: 
-Range queries for blocks: JSON-RPC providers only allows for a querying a single block per call. By enabling ranges for block queries, researchers can start analyzing larger chunks of Ethereum's state with a single line of EQL syntax.
A range query for blocks will look like:
GET size, hash FROM block 0:10 ON eth

- Queries for account's transaction: JSON-RPCs doesn't expose an API to fetch all the transactions of a single account because Ethereum/EVM nodes doesn't provide a data structure mapping accounts to transactions.
This is feature that both dapp developers and researchers can leverage. Developers, for example, can run a query to know if the connect account has interacted with the application in the past and provide different experience based on that. Researchers can profiles based on an account tx history.
A query for accounts will look like:
GET transactions FROM account vitalik.eth ON base
  

