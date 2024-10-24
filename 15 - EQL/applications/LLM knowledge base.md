# EVM Query Language
![cover image](./preview.png)

EVM Query Language (EQL) is a SQL-like language designed to query EVM chains, aiming to support complex relational queries on EVM chain first-class citizens (blocks, accounts, and transactions). It provides an ergonomic syntax for developers and researchers to compose custom datasets without boilerplate code.


[Try it here](https://eql-app.vercel.app/)

## Goals
EQL's primary goal is to support relational queries for Ethereum entities such as blocks, accounts, and transactions. The challenge lies in Ethereum's storage model, which stores these entities under key-value databases, indexing values by a single key. Linear searches over RPCs can be extremely slow, so research is being done to find the best way to distribute Ethereum state data and allow performant relational queries.

Additionally, EQL aims to extend its syntax beyond simple read operations, empowering developers and researchers with tools to compose custom datasets efficiently.
## The Problem
Ethereum clients store blocks, transactions, and accounts using a key-value model, making complex blockchain data analysis non-trivial. For instance, fetching all transaction values from a given block using TypeScript requires a disproportionate amount of code. Developers and researchers face productivity hindrances due to boilerplate code and public RPC rate limits.

## How EQL Wants to Solve It
EQL aims to bridge the gap between data exploration and developer/researcher experience by focusing on two main areas:
### Query Performance
Research is being conducted to provide fully relational queries for Ethereum's first-class citizens. Key principles guiding R&D include data access without requiring a full-node and avoiding centralization.
### Ergonomics
EQL aims to provide a small footprint on existing codebases and enhance productivity for beginners with a simple yet powerful syntax.
## Interpreter
EQL is an interpreted language mapping structured queries to JSON-RPC providers. The interpreter is divided into two phases: frontend and backend.
- **Frontend:** Takes the source of the program (queries), splits it into tokens, assesses the correctness of the expressions provided, and returns an array of structured expressions.
- **Backend:** Receives the array of expressions, maps them to JSON-RPC calls, and formats the responses to match the query requirements.
This allows interaction with EVM chain data and various operations on entities like accounts, blocks, and transactions.
## Usage
Queries can be run by executing `.eql` files with the `run` command:
```bash
eql run <file>.eql
```

Using the language REPL:
```sh
eql repl
```

Or incorporating the interpreter directly in your app. [See](https://github.com/iankressin/eql/blob/main/crates/core/README.md):
```sh
[dependencies]
eql_core = "0.1"
```

## Installation
To install the CLI you will need `eqlup`, the EQL version manager:
```sh
curl https://raw.githubusercontent.com/iankressin/eql/main/eqlup/install.sh | sh
```

Next, install the latest version of EQL:
```sh
eqlup
```

### Updating EQL
To update EQL to the latest version, run `eqlup` again:
```
eqlup
```

## Roadmap
![roadmap image](./roadmap.png)
Check out the full roadmap [here](./ROADMAP.md)

## Expressions

### GET
_Description:_ Read one or more fields from an entity, given an entity and an id.

_Production:_ `GET <[fields, ]> FROM <entity> <entity_id> ON <chain>`

_Example:_ `GET nonce, balance FROM account vitalik.eth ON base`
### SEND (Soon)
_Description:_ Sends a transaction to the network.

_Production:_ `SEND <type> to=<address>, value=<ether>, data=<bytes> ON <chain>`

_Example:_ `SEND TX to=vitalik.eth, value=1, data=0x0...000 ON arbitrum`
### MATH (Soon)
_Description:_ Supports basic math operations like SUM, SUB, DIV, TIMES.

_Production:_ `<operator>(<[expr, ]>)`

_Example:_ `SUM(GET balance FROM vitalik.eth ON base, GET balance FROM vitalik.eth ON ethereum)`
## Entities
These are the entities that can be queried using the EQL language, each addressed by its name and an id:
### Account
- address [id]
- nonce
- balance
- code
### Block
- number [id]
- timestamp
- size
- hash
- parent_hash
- state_root
- transactions_root
- receipts_root
- logs_bloom
- extra_data
- mix_hash
- total_difficulty
- base_fee_per_gas
- withdrawals_root
- blob_gas_used
- excess_blob_gas
- parent_beacon_block_root
### Transaction
- hash [id]
- transaction_type 
- from 
- to 
- data 
- value 
- gas_price 
- gas 
- status 
- chain_id 
- v 
- r 
- s 
- max_fee_per_blob_gas 
- max_fee_per_gas 
- max_priority_fee_per_gas 
- y_parity

## Release post

Today marks the release of EVM Query Language (EQL), a SQL-like language built to query EVM chains with the final goal of supporting complex relational queries on EVM chains first-class citizens (blocks, accounts, and transactions), whilst provide devs and researchers with a language with ergonomic syntax for simple everyday use. 

In this post we will cover the problems EQL is trying to solve, explore the current landscape EVM queries and do a quick overview of past work on query languages designed for Ethereum. In a second moment it will go through possible solutions for the problems listed previously, present the momentary solution and its limitations, and finally set milestones for future development.

This release, beyond a beta, it is a commitment to make EVM data more accessible for researchers, analysts and  developers with the goal of accelerate Ethereum development and adoption by giving builders tools to make them move faster.  
## The problem
Ethereum and other major L2s are open and permissionless data platforms,  which means anyone can access, manipulate, and analyze the ever-growing collection of bytes.
Ethereum clients stores data for blocks, transactions, and accounts using a key-value model, where an instance of any first-class entity can be access directly by using an unique key. The information of an account for example, can be retrieved by providing the key-value store the distinctive id of this account, its address.

Hash-map-based databases are the perfect choice when you have an id and need to fetch its content, and Ethereum and EVM clients doesn't need anything beyond that.
Although this storage model fits the interior node operations, it make complex analysis of blockchain data not trivial. 
Key-value stores are not designed with relational look-ups in mind, which means that when we want to retrieve all transactions that sent more than 1 ETH in last hour we need to:
- Run a linear search on the block hash-map to find timestamps are greater than `now - 1 hour`
- Run another linear search on each block to filter out all transactions which value is different of 1 ETH
The above is just a simple example, and it is easy to imagine how poorly this approach scales in terms of performance when more complex analysis stacks up.

Beyond the storage model, a minor inconvenience of accessing on-chain data for developers and researchers is the amount of boilerplate code involved.
If one simply wants to fetch all transaction values from a given block they need to setup a disproportionate amount of code for a simple query. A typescript implementation would look something like this:
```typescript
import { createPublicClient, http } from "viem";
import { mainnet } from "viem/chains";

async function fetchTransactionsFromBlock(
  blockTag: BlockTag,
): Promise<BigInt[]> {
  const client = createPublicClient({
    chain: mainnet,
    transport: http(),
  });

  const block = await client.getBlock({ blockTag });
  const values: BigInt[] = [];

  for (const hash of block.transactions) {
    const tx = await client.getTransaction({ hash });
    values.push(tx.value);
  }

  return values;
}
```
*Listing 1 - Fetching values from all tx in a block*

Notice that the in the code above we could switch from a `for of` loop to a `await Promise.all(tx.map(...))` in order to have better performance. But that doesn't take in consideration public RPCs rate limits,  and the "faster" version would likely crash due to excessive calls in a short period of time.

Dapp developers and researchers shouldn't be exposed to this class of problems as it hinders  productivity by forcing them to go out of their lane to deal with minor hiccups.
## Current landscape
There are lots of different options  when it comes to accessing blockchain data from EVM chains. 
### JSON-RPCs
First, we have the native alternative to all EVM nodes: JSON-RPCs. Nodes expose to the outer world an API to read blockchain's state and logs. RPCs are widely used as the backbone of virtually every application that requires some kind of interaction with on-chain data.
The methods provided by the RPC interface to read state, such as `eth_getBlockByNumber` or `eth_getBalance` receive an identifier, block number and account address respectively, and return their values. Internally these methods reads from some sort of key-value store and return the value to the user.

 Since JSON-RPCs are part of Ethereum specs, they are widely available for anyone through both free and payed service providers such as  [llama rpc](https://llamarpc.com/), [ankr](https://www.ankr.com/), [infura](https://www.infura.io/). The free tiers available often provide a good enough service for small scale apps and analysis of a a small section of the blockchain. 

When it comes accessing large sums of data, this approach can show its limitations. 
As discussed in *Listing 1*, RPC providers implement rate limiters, so this should be taken in consideration when trying to fetch data in sequential calls. Strategies like alternating between different providers or use a rate-limiter on the client-side can be used to avoid being banned from those services, but they are not targeting the disease, solely the symptom.
JSON-RPCs imposes the same limitations of key-value stores discussed in the previous section. Since they are a thin layer over a node storage, complex queries require linear searches over the data-structure which can has potential to deliver a terrible performance.
### The Graph
Popular solution for querying, aggregating and indexing data from EVM and non-EVM, [The Graph](https://thegraph.com/) allows to map-reduce EVM events from smart-contracts into custom structures that made easier for querying and searching data on-chain than it would be with public/view methods of contracts.

Subgraphs shines when it comes to parsing EVM events, indexing the results and make them available through a GraphQL API. No need to create and maintain the whole map-reduce-index structure manually. 

Some of its limitations come from the fact that the protocol heavily relies on smart-contract events, which means that a subgraph requires a smart-contract address to listen events from. So if one wants to create queries that depends strictly on EOAs, blocks and transactions, The Graph  doesn't support that out-of-the-box.
Subgraphs are built using [AssemblyScript](https://www.assemblyscript.org/introduction.html), a variant of TypeScript that compiles down to WebAssembly. Despite of being really similar to TS, AS has its own limitations that new developers have to acknowledge and deal with.
### Dune
Dune is the industry standard when it comes to data analysis. It offers a platform where users can query blockchains with a SQL-like language called DuneSQL, fine tuned for blockchain needs. Results can be streamed into dashboard to create data visualization. 

The platform provides an extensive data catalog which users can leverage to create complex relational queries  
On the down-side, DuneSQL doesnt have a native way to fetch the nonce of an account.  The query below was generate by the Dune's AI assistant by using the prompt: `fetch the nonce of account 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 on ethereum`
```SQL
SELECT 'The nonce value for an account cannot be determined using SQL queries on the provided tables. Nonce retrieval requires access to node or API data that specifically tracks the state and transaction count for each account.' AS message
  ```
  *List 2 - Dune AI assistant response* 

It's worth mentioning that the AI assistant is still in beta, so this can be a false positive, but scanning through the collections, I wasn't able to find  `nonce` nor `transaction_count` fields.

Another drawback of Dune is that queries should be defined  in their web-app and them be called from an application, passing in the parameters. Defining the query directly into the app can save some devs some head-space from needing to manage yet another platform and API key while building their software. 
## How EQL wants to solve it?
In a way or another all the tools and products explored above has some kind of limitation when it comes to complex queries and/or ergonomics.
The end-goal of EVM Query Language is to close that gap between data exploration and developer/researcher experience. With that in mind EQL needs to split efforts into two lanes.
### Query performance
As of now, the language doesn't support relational queries, since a regular JSON-RPC is still being used as the backbone of the [execution engine](https://github.com/iankressin/eql/blob/7fee5f9fc62bfe13ef8a0b753ab043e90b19329f/crates/core/src/interpreter/backend/execution_engine.rs#L34), and with it,  we inherit all its limitations listed above.

Research is being done to determine what is the path to be taken in  order to provide fully relational queries for Ethereum's first-class citizens. Down that road, a few principles will guide R&D:
- **Data access:** data should be easy to access to everyone online. No one should be required to run a full-node to be able to fetch the blockchain state.
-  **Avoid centralization:** no single entity should hold all data being queried. This means centralized databases are not an option here.

An instinctive approach to achieve relational queries is to parse Ethereum's state into a B+ Tree, the same data structure used by a number of SQL databases. B+ Trees are great for:
- Minimize number of disk reads
- Efficient range queries
Such idea was explored in a paper called [EBTrees](https://dl.acm.org/doi/10.1145/3399871.3399892), where the authors propose a B+ Tree-like data structure to index Ethereum state. The proposed storage model has fast insertion, searches and take relatively low storage space when compared to a regular disk usage of a regular Geth node.
Besides the promising results, all the data collected by this was done a machine running a forked Geth client, with the EBTree stored in the disk, which hurts the first principle listed above, since running a full-node is prohibitive, given the specs and time to sync.

The first line of research uses EBTrees as a starting point, but wants to be able to distribute the nodes of the tree in a network of peers, ideally an existing one (e.g. IPFS) and build the tree or sub-tree representation locally. This way, client would only hold internal nodes, which take less disk space than leaf nodes, whilst the actual data is stored remotely.

This work is early stage so no experiment was done with distributed EBTrees. As R&D moves forward, articles will be written to share the progress. 
### Ergonomics
The amount of languages, frameworks, libraries, and services that developers have to master and manage in order to put together consistent and scalable applications is already reasonably large, resulting in steep learning curves, and hindering new comers productivity.

The most valuable work of researchers is done when they are analyzing data and write about their methods and conclusions.  Every minute spent learning a new tool and all its intricacies is a minute wasted on the actual research of the subject.

- Small footprint on codebases
- Productivity for beginners
## The language
EQL is being designed to be a simple yet powerful query language to interact with EVM chains.  

The simplicity comes from its entity-based syntax where each query follows the same structure: `GET, FROM, ON`.
`GET` is similar to `SELECT` in other query languages, where it is used to list all the fields one wants to retrieve from a given query. In the case of EQL, users can list all the fields of the target entity that w

## Old Discord pitch

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

## Alpha Roadmap

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
 - [ ] Documentation

This benchmark aims to compare the core features of EQL (Ethereum Query Language) and Dune. One question that arises frequently after initial contact with EQL is how it stacks up against Dune for blockchain data retrieval. Although these tools have different end goals, they share a foundational similarity—both rely on a query language designed to retrieve blockchain data.

## Purpose and Approach
While Dune is widely known as a data analysis platform, EQL focuses on efficient data extraction, particularly within the Ethereum ecosystem. It’s important to note that this comparison is not a comprehensive evaluation of Dune’s capabilities, especially in analytics, as that falls outside the scope of what EQL aims to achieve. Instead, we are focused solely on the querying and data extraction capabilities of both tools.

In this series of tests, we will evaluate Dune as a data extraction tool, a role in which it overlaps with EQL. The goal is to compare performance under this specific use case. Dune provides pre-indexed and pre-populated databases, making it easier to access complex relationships within the data, while EQL is designed for more direct interaction with Ethereum’s raw state through JSON-RPC providers.
## What We’re Testing
The tests will focus on extracting specific Ethereum entities such as blocks, transactions, accounts, and logs. Each test uses Ethereum Layer 1 (L1) as the data source, and the primary metric we’ll consider is the time it takes for each tool to return results.

Additionally, it’s worth pointing out that while EQL has a tighter integration with Ethereum, Dune offers a more comprehensive data model through its pre-indexed infrastructure, making complex queries more accessible. However, EQL’s direct approach offers a more intuitive SQL-like API for Ethereum users, which may translate into efficiency for those familiar with blockchain data.

## Test Setup
The following tests were carried out based on the available resources and setups for each tool

• **Dune:** Queries were executed through both the Dune interface and its API.
• **EQL:** Tests were run on a MacBook Pro M2 Pro with a 500mb internet connection.

It is worth mentioning EQL’s performance is influenced by processing power, network conditions, and RPC (Remote Procedure Call) performance.

As of [current date], these tests were conducted, and it’s important to note that Dune’s performance may fluctuate during peak usage times as any other web application. If any of the results seem to fall outside the expected average performance, please let me know, and I will rerun the tests to ensure accuracy.

## Benchmarks

### Fetch all fields for blocks 100 blocks
The first test is to fetch 100 blocks from Ethereum. EQL performance is bounded by the RPC provider being used. For this test we're using dRPC



Despite being tools with different goals (explore this more in depth), EQL and Dune share similar features, since both have in it's core a query language to retrieve blockchain data.

In this series of tests we are going to use Dune as a data extraction tool, which this is what EQL is. Dune is broadly used as a data analysis platform, and we cannot compare that portion of the Dune ecosystem to EVM Query Language, since they are completely different things.

EQL is a tool design, in this first iteration, to extract parts of the Ethereum state subjected to the user interest, while Dune provide a pre-indexed and pre-populated databases, so access to complex relations is made easy.

The performance benchmark only considers the amount of time before each of the projects to yield the result of the queries

The test will analyze the performance of fetching single and groups of the considered first-class entities in EQL (blocks, transactions, accounts, and logs) all the tests will be use Ethereum L1 as data source.

EQL is able to provide a better SQL language API than Dune, since EQL is designed with Ethereum in mind.
### Setup
Dune queries were ran using the Dune interface and Dune API

EQL test were ran in a macbook pro with m2 pro chip and a internet connection of 27mb lol
EQL performance is highly bound in the processing power, network connection and RPC performance

Which is the fastest RPC?
Which one is the slowest RPC?

## Benchmarks

### Fetch all fields for blocks 100 blocks
The first test is to fetch 100 blocks from Ethereum. EQL performance is bounded by the RPC provider being used. For this test we're using dRPC

#### Dune
Query:
```sql
SELECT * FROM ethereum.blocks b WHERE b.number BETWEEN 1 AND 100
```

Performance:
![[Pasted image 20240914115654.png]]

#### EQL
Query:
```SQL
GET * FROM block 1:100 ON eth
```

Performance:

### Account's balance and nonce
#### Dune
Query:
```SQL
WITH account_balance as (
    SELECT balance
    FROM tokens_ethereum.balances_daily as b
    WHERE b.address = 0xc71048d303920c73c29705192393c567ac4e6c67
    AND token_standard = 'native'
    ORDER BY b.day DESC
    LIMIT 1
),
account_nonce as (
    SELECT nonce
    FROM ethereum.transactions t
    WHERE t."from" = 0xc71048d303920c73c29705192393c567ac4e6c67
    ORDER BY t.block_number DESC
    LIMIT 1
),
account as (
    SELECT nonce, balance FROM account_nonce, account_balance
)
SELECT * FROM account
```

Performance:
![[Pasted image 20240915180001.png]]

#### EQL
Query:
```SQL
GET nonce, balance FROM account 0xc71048d303920c73c29705192393c567ac4e6c67 ON eth
```

Performance:
### Account's balance and nonce using ENS
Different from the other queries, this one had to be ran using the "medium model" that costs 10 credits, since it wasn't able to  

#### Dune
Query:
```SQL
WITH account_address as (
    SELECT address
    FROM ens.reverse_latest e
    WHERE name = 'vitalik.eth'
    ORDER BY e.latest_tx_block_time ASC
    LIMIT 1
),
account_balance as (
    SELECT balance
    FROM tokens_ethereum.balances_daily as b
    WHERE b.address = (SELECT address FROM account_address)
    AND token_standard = 'native'
    ORDER BY b.day DESC
    LIMIT 1
),
account_nonce as (
    SELECT nonce
    FROM ethereum.transactions t
    WHERE t."from" = (SELECT address FROM account_address)
    ORDER BY t.block_number DESC
    LIMIT 1
),
account as (
    SELECT nonce, balance, address FROM account_nonce, account_balance, account_address
)
SELECT * FROM account
```


#### EQL
Query:
```SQL
GET nonce, balance FROM account vitalik.eth ON eth
```

Performance:

### Logs

#### Dune
Query:
```sql
SELECT *
FROM ethereum.logs l
WHERE l.block_number BETWEEN 4638657 AND 4638758
AND l.contract_address=0xdAC17F958D2ee523a2206206994597C13D831ec7
AND l.topic0=0xcb8241adb0c3fdb35b70c24ce35c5eb0c17af7431c99f827d44a445ca624176a
```

Perf:
![[Pasted image 20240915225140.png]]
#### EQL
Query:
```SQL
GET *
FROM log
WHERE 
    block 4638657:4638758,
    address 0xdAC17F958D2ee523a2206206994597C13D831ec7,
    topic0 0xcb8241adb0c3fdb35b70c24ce35c5eb0c17af7431c99f827d44a445ca624176a
ON eth
```

Objecoes:
	- Tratar de antemao a objecao e trazer uma argumentacao antecipada dos principais  problemas do EQL contra Dune

### Questions
1. Does Dune provide a way to dump the query result to a file?
	**R:** Only to CSV and only with a paid plan.
	

The title of this article captures the initial reaction many people have when they first learn about EVM Query Language (EQL). Although there are surface-level similarities between EQL and Dune, these two projects are fundamentally distinct in their goals, design, and use cases. This article seeks to address the common question of how EQL differs from Dune, while also examining the points where the two projects overlap.

The analysis will begin with a high-level overview of both EQL and Dune, providing essential background information on each project. Following this, it will delve deeper into a detailed comparison, highlighting both the differences and the similarities in their approaches to querying blockchain data. Finally, the post will conclude with a benchmark comparison, evaluating the performance of EQL and Dune in specific use cases, allowing readers to better understand their respective strengths and limitations in practical scenarios.
## Outlining the projects
Dune is a data analytics platform designed for querying and visualizing blockchain data, primarily focused on Ethereum and other related blockchains such as Binance Smart Chain, Polygon, and Optimism. It provides users with the ability to run SQL queries on publicly available on-chain data, making it possible to extract, analyze, and visualize information from these blockchains. Users interact with the data primarily through a SQL-based query engine that supports custom and predefined queries, enhanced by caching and parallel processing for performance. The platform offers interactive dashboards for visualization, supporting various chart types and dynamic filtering, and facilitates collaboration through public sharing and forking of dashboards.

EVM Query Language (EQL) is a data extraction tool that offers users a SQL-like language to execute queries and retrieve data from EVM chains. The syntax is currently under active development to simplify access to on-chain data for researchers and developers. At its present stage, EQL translates user queries into JSON-RPC requests, providing an efficient and straightforward method for querying the blockchain. The ultimate objective of the project is to create a fully decentralized storage engine, enabling anyone to query EVM chains using a relational approach similar to SQL databases. Unlike Dune, EQL does not aim to index or decode various smart contracts and on-chain data.
## How are the projects different?
As mentioned above, Dune is a platform focused on efficient querying and data visualization. It originally centered on the Ethereum ecosystem but has since expanded its operations across the broader blockchain space, from Bitcoin to Solana. In terms of storage, the platform uses a fork of Trino as its query engine, which supports Ethereum types natively, such as addresses and transaction hashes. Trino is a query engine where the execution layer is decoupled from the storage layer. Unlike traditional SQL databases, where query execution and storage are built into a monolithic system, Trino is well-suited for distributing both processing and storage across a cluster of servers. As a result, scaling the number of nodes also increases the amount of data the platform can process in parallel. The Trino syntax is similar to regular the regular SQL syntax used by most of the relational databases such as MySql and Postgres.

Although EQL's long-term goal is to develop its own storage engine to efficiently distribute Ethereum data across the network and enable efficient P2P relational queries, the project currently relies on JSON-RPC providers as the "storage engine." The downside is that EVM Query Language (EQL) cannot process large numbers of blocks in a single request due to the rate limits imposed by public RPC providers. However, there is a workaround for this data limitation in development.
EQL provides a SQL-like syntax that leverages the predictable relationships between key Ethereum entities—blocks, accounts, transactions, and logs—to create a more direct way of querying on-chain data. One example of how this syntax can be more efficient compared to SQL syntax—which is designed to handle all types of table relationships—is a simple query to fetch account information. For instance, let’s examine how to retrieve the nonce and address of Vitalik's account using Dune and Trino's SQL syntax:
```SQL
WITH account_balance as (
    SELECT balance
    FROM tokens_ethereum.balances_daily as b
    WHERE b.address = 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
    AND token_standard = 'native'
    ORDER BY b.day DESC
    LIMIT 1
),
account_nonce as (
    SELECT nonce
    FROM ethereum.transactions t
    WHERE t."from" = 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
    ORDER BY t.block_number DESC
    LIMIT 1
),
account as (
    SELECT nonce, balance FROM account_nonce, account_balance
)
SELECT * FROM account
```
*Listing 1.0 - Trino syntax to fetch and account's nonce and balance*

The query above needs to retrieve data from two different tables, `transactions` and `balances_daily`, and join them into a single record to return the desired information. This is necessary because Dune does not provide a table that contains account data directly, requiring us to combine data from these two sources to obtain the information we need.

![[Pasted image 20240921193647.png]]
*Listing 1.1 - Available "raw" Ethereum tables*

Unlike SQL, which must accommodate a wide range of use cases, EQL is specifically designed for EVM blockchains. As a result, it offers first-class support for queries like the one mentioned above.
```SQL
GET nonce, balance 
FROM account 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
ON eth
```
*Listing 1.2 - EQL syntax to fetch and account's nonce and balance*

It also offers first-class support for ENS, which in Trino would require an additional `SELECT` from another table. Moreover, EQL allows you to fetch all the related to an account, including `code`,  `address`, `nonce`, and `balance` — a process that would otherwise require separate queries in Trino.
```SQL
GET * 
FROM account vitalik.eth
ON eth


| nonce | balance | address | code |
| ----- | ------- | ------- | ---- |
| 1320  | 1231... | 0xd7... | 0x   |

```

In contrast, Dune and EQL diverge in several key aspects. Dune is a comprehensive platform that emphasizes data visualization and user collaboration, making it an ideal tool for analysts and researchers looking to query and visualize data from multiple blockchains, not limited to the Ethereum ecosystem. Its use of Trino as a query engine, along with interactive dashboards, allows users to handle complex queries and create rich visual reports. EQL, on the other hand, is narrowly focused on querying EVM blockchains with a simplified, domain-specific syntax. It is designed to be lightweight, free, and open-source, with a goal of decentralizing data access. Unlike Dune, EQL does not prioritize visualizations or user interfaces, but instead focuses on efficient blockchain data extraction and providing flexibility in data exports. Thus, while both platforms enable users to query and extract blockchain data, Dune excels in visualization and cross-chain functionality, whereas EQL specializes in efficient querying of Ethereum-based data with a more direct, customizable approach.
## How are the projects similar?
While EQL and Dune differ significantly in their design and long-term objectives, they do share several common features, particularly in their ability to extract and export blockchain data. Both platforms allow users to run queries that extract on-chain information, though the exact methods and formats differ.

In terms of data extraction, both Dune and EQL enable users to retrieve information from blockchain ecosystems using SQL-like query languages. Dune users write SQL queries to access data from various raw Ethereum tables, combining different datasets to obtain the desired insights. EQL, by contrast, offers a specialized SQL-like syntax tailored specifically to the Ethereum Virtual Machine (EVM), streamlining access to crucial blockchain data such as accounts, transactions, and balances. This specialization allows for more direct and efficient querying of on-chain data.

Both platforms also provide options for exporting query results, though they differ in terms of formats and pricing. Dune allows users to export query results directly through its interface, but this is limited to CSV format and only available with a paid plan. Free users must export data via API requests, which requires some additional technical steps. On the other hand, EQL, being open-source, offers greater flexibility and no cost barriers. Users can export data in a variety of formats, including JSON, CSV, and Parquet, without any restrictions, making it a more flexible and accessible option for developers and researchers.

In summary, while Dune and EQL differ in their broader goals and the scope of their query languages, they share important functionalities in data extraction and export capabilities. Dune excels in user-friendly visualization and cross-chain functionality, while EQL stands out for its flexible query syntax and free access to multiple export formats, appealing to users with a more technical or development-oriented focus.

## When should you use EQL over Dune
- Quick exploration of data
- Quickly check Ethereum state
- Check data about one or many accounts
- Query different objects that can't be cached

## Benchmarks
Dune offers an extensive range of datasets for users to build and customize their queries, allowing flexible data cross-referencing. However, for the purpose of these benchmarks, we are specifically focusing on the common elements between EQL and Dune, which include core Ethereum components such as blocks, transactions, accounts, and logs (traces _soon™_).

In this section, we'll compare the types of queries that both Dune and EQL enable.
### Test Environment
The tests were performed on a system with the following specifications:
- **Processor**: Ryzen 9 5900x
- **RAM**: 16GB HyperX Fury (Dual Channel)
- **Storage**: 1TB Kingston NV2 M.2 2280 SSD
- **Internet Connection**: 300 Mbps
### Testing Methodology
For each platform, ten query executions were collected to gather performance metrics. While executing a larger number of queries, such as one hundred, would provide more comprehensive data, the manual method employed to collect Dune metrics made expanding the number of records impractical and unproductive. Although Dune offers an API for fetching data from endpoints, it only returns data from the most recent execution of a specified query. This behavior effectively caches the last result, undermining the ability to perform a meaningful comparison between the performance of EQL and Dune.

Consequently, Dune queries were executed directly through the application's web user interface without any caching mechanisms or materialization configurations. The metrics were gathered solely based on the data provided by the app. In contrast, EQL metrics were collected using the `eql_core` Rust crate. The `eql` function within this crate was utilized to execute queries by passing them as parameters and retrieving the results. To ensure accurate and reliable performance measurements, the Criterion benchmarking library was employed to collect and analyze the metrics.

Dune offers three query engine options: Free, Medium, and Large. The Free engine is the slowest but has no cost. The Medium engine costs 10 credits per query, with a monthly limit of 2,500 credits. The Large engine, which requires a subscription, provides 25,000 monthly credits. It costs 20 credits per query and is priced at $349 per month, billed annually. The tests collected data separately from both the Free and Medium plans, as these plans are offered at no cost. This allows for a fair comparison, since EQL is also a free option for querying on-chain data.

### Testing Conditions and Considerations
These tests were conducted as of 2024-09-21. It is essential to acknowledge that Dune’s performance may vary during peak usage times, similar to other web applications. Such fluctuations could affect the consistency of the results. If any of the outcomes appear to deviate significantly from the expected average performance, it is recommended to rerun the tests to verify their accuracy and ensure the reliability of the comparative analysis.
EQL’s performance is influenced by several factors, including processing power, network conditions, and the efficiency of Remote Procedure Calls (RPC). Ensuring optimal conditions in these areas is crucial for achieving the best possible performance outcomes with EQL.
### The tests
- **Block Range Fetch**: 100 Blocks
- **Sequential Block Fetch**: 100 Blocks Individually
- **Transaction Lookup by Hash**: 100 Transactions
- **Account State Fetch**: Nonce and Balance
- **Event Log Fetch**: USDC Transfers in 100 Block Range

#### **Block Range Fetch**: 100 Blocks
Queries:
```sql
# Dune
SELECT * FROM ethereum.blocks b WHERE b.number BETWEEN 1 AND 100

##############################################

# EQL
GET * FROM block 1:100 ON eth
```

Performance:

|               | EQL      | Dune Free | Dune Medium |
| ------------- | -------- | --------- | ----------- |
| **Mean**      | 813.6 ms | 218.9 ms  | 544.8 ms    |
| **Median**    | 776.6 ms | 208.9 ms  | 432.4 ms    |
| **Std. Dev.** | 130.2 ms | 43.9 ms   | 41.0 ms     |

**Analysis:**
- **Dune Free** is the fastest, with both the lowest mean and median execution times.
- **EQL** is the slowest in this category.
- **Standard deviation** is lowest for Dune Free, indicating consistent performance.

**Conclusion:**
- For fetching a range of blocks, **Dune Free** offers the best performance.
- **EQL** may not be the optimal choice for this specific query type.

#### **Sequential Block Fetch**: 100 Blocks Individually
Queries:
```sql
# Dune
SELECT * FROM ethereum.blocks b WHERE b.number IN (1, 2, 3, ..., 100)

##############################################

# EQL
GET * FROM block 1,2,3,...,100 ON eth
```

Performance:

|               | EQL      | Dune Free | Dune Medium |
| ------------- | -------- | --------- | ----------- |
| **Mean**      | 804.8 ms | 313.0 ms  | 259.3 ms    |
| **Median**    | 107.7 ms | 214.0 ms  | 243.4 ms    |
| **Std. Dev.** | 775.6 ms | 317.2 ms  | 129.9 ms    |

**Analysis:**
- **EQL** has the lowest median time but the highest standard deviation, indicating inconsistent performance.
- **Dune Medium** has the lowest mean time and a relatively low standard deviation.
- **Dune Free** performs in between the other two.

**Conclusion:**
- **Dune Medium** offers the most consistent and fastest average performance.
- While **EQL** can be faster in some instances (as indicated by the median), its high variability might affect reliability.

#### **Transaction Lookup by Hash**: 100 Transactions
```SQL
# Dune
SELECT *
FROM ethereum.transactions t
WHERE t.hash 
IN (
	0xfffc07e7ff65a5ff7f8496042f85fc4e1d6bd29e012e776b970f4414c07d4d41,
	...
)

##############################################

# EQL
GET *
FROM tx 
0xfffc07e7ff65a5ff7f8496042f85fc4e1d6bd29e012e776b970f4414c07d4d41,
...
ON eth
```

Performance:

|               | EQL       | Dune Free | Dune Medium |
| ------------- | --------- | --------- | ----------- |
| **Mean**      | 1.14 s    | 7.93 s    | 30.05 s     |
| **Median**    | 1.08 s    | 4.09 s    | 26.84 s     |
| **Std. Dev.** | 142.24 ms | 8.13 s    | 18.33 s     |

**Analysis:**
- **EQL** significantly outperforms both Dune tiers.
- **Dune Medium** is the slowest, with a mean time almost 30 times that of EQL.
- **Standard deviation** for EQL is much lower, indicating consistent performance.

**Conclusion:**
- **EQL** is the superior choice for transaction lookups by hash, offering much faster and more consistent execution times.

####  **Event Log Fetch**: USDC Transfers in 100 Block Range
Queries:
```SQL
# Dune
SELECT *
FROM ethereum.logs l
WHERE l.block_number BETWEEN 4638657 AND 4638758
AND l.contract_address=0xdAC17F958D2ee523a2206206994597C13D831ec7
AND l.topic0=0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef

##############################################

# EQL
GET * 
FROM log 
WHERE
block 4638657:4638758,
address 0xdAC17F958D2ee523a2206206994597C13D831ec7,
topic0 0xcb8241adb0c3fdb35b70c24ce35c5eb0c17af7431c99f827d44a445ca624176a
ON eth
```

Performance:

|               | EQL      | Dune Free | Dune Medium |
| ------------- | -------- | --------- | ----------- |
| **Mean**      | 344.1 ms | 436.6 ms  | 759.2 ms    |
| **Median**    | 339.6 ms | 246.5 ms  | 472.4 ms    |
| **Std. Dev.** | 20.2 ms  | 58.1 ms   | 916.2 ms    |

**Analysis:**
- **EQL** has the lowest mean and standard deviation, indicating both speed and consistency.
- **Dune Free** has a lower median time than EQL but higher mean and standard deviation.
- **Dune Medium** shows high variability, as indicated by a high standard deviation.

**Conclusion:**
- **EQL** provides the best overall performance for event log fetching.
- **Dune Free** is competitive in median time but less consistent.

#### **Account State Fetch**: Nonce and Balance
Query:
```SQL
# Dune
WITH account_balance as (
    SELECT balance
    FROM tokens_ethereum.balances_daily as b
    WHERE b.address = 0xc71048d303920c73c29705192393c567ac4e6c67
    AND token_standard = 'native'
    ORDER BY b.day DESC
    LIMIT 1
),
account_nonce as (
    SELECT nonce
    FROM ethereum.transactions t
    WHERE t."from" = 0xc71048d303920c73c29705192393c567ac4e6c67
    ORDER BY t.block_number DESC
    LIMIT 1
),
account as (
    SELECT nonce, balance FROM account_nonce, account_balance
)
SELECT * FROM account

##############################################

#EQL
GET nonce, balance
FROM account 0xc71048d303920c73c29705192393c567ac4e6c67
ON eth
```

Performance:

|               | EQL       | Dune Free | Dune Medium |
| ------------- | --------- | --------- | ----------- |
| **Mean**      | 939.03 ms | 48.99 s   | 3.81 min    |
| **Median**    | 904.04 ms | 40.87 s   | 2.64 min    |
| **Std. Dev.** | 93.925 ms | 18.46 s   | 3.14 min    |

**Analysis:**
- **EQL** vastly outperforms both Dune tiers, completing queries in under a second.
- **Dune Free** and **Dune Medium** take significantly longer, with execution times ranging from nearly a minute to several minutes.
- High standard deviations for Dune indicate inconsistent performance.

**Conclusion:**
- For account state fetching, **EQL** is dramatically faster and more reliable.
- **Dune** may not be practical for time-sensitive applications requiring account state data.

### Benchmark analysis
EQL outperforms Dune in transaction lookups by hash, account state retrievals, and event log fetches, offering faster and more consistent performance across these query types. It also shows more reliable execution times with lower variability, making it a better choice for users requiring predictability.

Dune Free excels in block range fetches, particularly when scanning 100 blocks. Its infrastructure seems optimized for this type of query, making it more efficient than both EQL and Dune Medium. Despite being a free tier, it consistently outperforms the paid Dune Medium option in this area.

Dune Medium, despite being a larger query engine, shows high variability and does not offer better performance than Dune Free in block range queries. Its inconsistency across event log fetches and account state queries limits its reliability.

The unexpected performance gap between Dune Free and Dune Medium in block fetches remains unexplained.

## Final thoughts
EQL and Dune are both tools for querying blockchain data, each catering to different needs and use cases. Understanding their strengths and limitations is crucial in selecting the right tool for your specific requirements.

EQL offers a more intuitive and user-friendly SQL-like syntax tailored specifically for EVM chains, making it easier for developers and researchers to write queries without deep knowledge of traditional SQL. It is ideal for quick exploration of Ethereum’s state, checking data about one or multiple accounts, or querying specific objects that can’t be cached. Benchmark tests show that EQL outperforms Dune in transaction lookups by hash, account state retrievals, and event log fetches, offering faster and more consistent execution times. Being open-source, EQL allows users to export data in various formats like JSON, CSV, and Parquet without any cost barriers or restrictions.

On the other hand, Dune excels at processing large numbers of blocks or extensive datasets, making it more suitable when dealing with queries that involve significant amounts of data. It provides robust data visualization and collaboration features, offering interactive dashboards, rich visual reports, and cross-chain functionality, which is beneficial for analysts and researchers interested in multiple blockchain ecosystems. For users comfortable with traditional SQL, Dune’s use of Trino and standard SQL syntax allows for complex queries and familiar operations.

It’s important to note that EQL currently cannot process large numbers of rows in a single query due to dependencies on JSON-RPC providers and their rate limits. If your use case involves querying vast amounts of data, Dune is the better option at this time. However, the EQL project is actively working on a solution to overcome this limitation.

The benchmark tests highlight the strengths of each platform. EQL demonstrated superior performance in transaction lookups by hash, account state retrievals, and event log fetches, with faster execution times and consistent performance. Dune outperformed EQL in block range fetches, especially when scanning a large number of blocks, due to its optimized infrastructure and caching mechanisms.

While both EQL and Dune enable users to extract and export blockchain data using SQL-like query languages, their approaches differ. EQL focuses on providing a lightweight, free, and open-source tool specifically for EVM blockchains. Its syntax leverages the predictable relationships between Ethereum entities, offering a more direct and efficient querying method. Dune offers a comprehensive platform with an emphasis on data visualization, cross-chain functionality, and user collaboration, leveraging existing SQL constructions through Trino to provide familiarity and flexibility for complex queries.

Choosing between EQL and Dune ultimately depends on your specific needs. Opt for EQL if you need a user-friendly syntax for quick and efficient querying of EVM blockchain data and don’t require processing large datasets. Choose Dune if you need robust data processing capabilities, advanced visualization tools, and the ability to work across multiple blockchain ecosystems. Both platforms have their unique advantages, and understanding these will help you select the tool that best aligns with your project’s goals.