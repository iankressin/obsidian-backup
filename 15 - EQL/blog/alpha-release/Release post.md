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
