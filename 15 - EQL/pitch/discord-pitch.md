Hey everyone!

I've been researching ways to build a SQL-like language on top of Ethereum and other L2s. This main goal of EQL is to make on-chain data more accessible for researchers and dapp developers by enabling relation queries on the main blockchain entities (accounts, blocks, and transactions).

Ethereum's storage model makes it really hard to create relation queries, since objects are stored in key-value data bases, so they can only be accessed directly, which make searches based on internal properties extremely slow.

The main challenge at the moment is find and implement a good enough data structure do accommodate all L1 data, that is around [1TB](https://ycharts.com/indicators/ethereum_chain_full_sync_data_size), such that relation queries are efficient and at the same time can be distributed across peers on a network, no one needs to a fully synced copy of Ethereum running locally.

A few papers were written in the this field, most notably [Ethereum Query Language](https://sci-hub.se/https://dl.acm.org/doi/abs/10.1145/3194113.3194114) which implement the SQL syntax on top of Ethereum L1. It leveraged a binary tree to create indexes to make queries faster, which is interesting, but binary trees requires a lot of disk/network accesses if the data is far way from the root node. And the paper didn't described in depth how those indexes were created.

Another related work is [EBTrees](https://sci-hub.se/https://dl.acm.org/doi/10.1145/3399871.3399892), a data structure based on B+ trees, which are commonly used in database engines, that take small storage space and have fast access to the leaves of the tree (where the data is). The problem with EBTrees is that they're built on top of a Geth client which needs to be fully synced, which is a deal breaker for adoption. 

Still have a few papers to review before moving on:
- [Towards Interoperability of Open and Permissionless Blockchains: A Cross-Chain Query Language](https://arxiv.org/pdf/2209.07224)
- [Towards Scalable Blockchain Analysis](https://sci-hub.se/https://ieeexplore.ieee.org/document/8823909)
- [A Practical Scalable Distributed B-Tree](https://sci-hub.se/https://dl.acm.org/doi/10.14778/1453856.1453922)

I've created a v0 of EQL (https://github.com/iankressin/eql) which supports basic, non-relation queries, like:
- `GET nonce, balance FROM account 0x000...0000 ON ethereum`
- `GET timestamp, size, hash FROM block 123 ON optimism`
- `GET from, to, data FROM tx 0x000...0000 ON base`

I'm finishing a REPL at the moment, before "launching" it publicly, which means being more vocal about it on other discord servers and twitter.

I'd love to hear any ideas or criticism about the project :) 