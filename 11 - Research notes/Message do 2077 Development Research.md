Hey everyone! As Emmanuel mentioned I've been working on a few research ideas but for the sake simplicity, I'll be narrowing down to the one that has seen more work, EVM Query Language (EQL).

I've been researching ways to build a SQL-like language on top of Ethereum and other L2s. This main goal of EQL is to make on-chain data more accessible for researchers and dapp developers by enabling relation queries on the main blockchain entities (accounts, blocks, and transactions).

Ethereum's storage model makes it really hard to create relation queries, since objects are stored in key-value data bases, so they can only be accessed directly, which make searches based on internal properties extremely slow.

The main challenge at the moment is find and implement a good enough data structure do accommodate all L1 data, that is around [1TB](https://ycharts.com/indicators/ethereum_chain_full_sync_data_size), such that relation queries are efficient and at the same time can be distributed across peers on a network, no one needs to a fully synced copy of Ethereum running locally.

A few papers were written in the this field, most notably [Ethereum Query Language](https://sci-hub.se/https://dl.acm.org/doi/abs/10.1145/3194113.3194114) which implement the SQL syntax on top of Ethereum L1. It leveraged a binary tree to create indexes to make queries faster, which is interesting, but binary trees requires a lot of disk/network accesses if the data is far way from the root node. And the paper didn't described in depth how those indexes were created.

Another related work is [EBTrees](https://sci-hub.se/https://dl.acm.org/doi/10.1145/3399871.3399892), a data structure based on B+ trees, which are commonly used in database engines, that take small storage space and have fast access to the leaves of the tree (where the data is). The problem with EBTrees is that they're built on top of a Geth client which needs to be fully synced, which makes it unaccessible  

