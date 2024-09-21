The title of this article captures the initial reaction many people have when they first learn about EVM Query Language (EQL). Although there are surface-level similarities between EQL and Dune, these two projects are fundamentally distinct in their goals, design, and use cases. This article seeks to address the common question of how EQL differs from Dune, while also examining the points where the two projects overlap or share common features.

The analysis will begin with a comprehensive, high-level overview of both EQL and Dune, providing essential background information on each project. Following this, it will delve deeper into a detailed comparison, highlighting both the differences and the similarities in their approaches to querying blockchain data. Finally, the post will conclude with a benchmark comparison, evaluating the performance of EQL and Dune in specific use cases, allowing readers to better understand their respective strengths and limitations in practical scenarios.
## Outlining the projects
Dune is a data analytics platform designed for querying and visualizing blockchain data, primarily focused on Ethereum and other related blockchains such as Binance Smart Chain, Polygon, and Optimism. It provides users with the ability to run SQL queries on publicly available on-chain data, making it possible to extract, analyze, and visualize information from these blockchains. Users interact with the data primarily through a SQL-based query engine that supports custom and predefined queries, enhanced by caching and parallel processing for performance. The platform offers interactive dashboards for visualization, supporting various chart types and dynamic filtering, and facilitates collaboration through public sharing and forking of dashboards.

EVM Query Language (EQL) is a data extraction tool that offers users a SQL-like language to execute queries and retrieve data from EVM chains. The syntax is currently under active development to simplify access to on-chain data for researchers and developers. At its present stage, EQL translates user queries into JSON-RPC requests, providing an efficient and straightforward method for querying the blockchain. The ultimate objective of the project is to create a fully decentralized storage engine, enabling anyone to query EVM chains using a relational approach similar to SQL databases. Unlike Dune, EQL does not aim to index or decode various smart contracts and on-chain data.
## How are the projects different?
Dune is a platform focused on efficient querying and data visualization. It originally centered on the Ethereum ecosystem but has since expanded its operations across the broader blockchain space, from Bitcoin to Solana. In terms of storage, the platform uses a fork of Trino as its query engine, which supports Ethereum types natively, such as addresses and transaction hashes. Trino is a query engine where the execution layer is decoupled from the storage layer. Unlike traditional SQL databases, where query execution and storage are built into a monolithic system, Trino is well-suited for distributing both processing and storage across a cluster of servers. As a result, scaling the number of nodes also increases the amount of data the platform can process in parallel. The Trino syntax is similar to regular the regular SQL syntax used by most of the relational databases such as MySql and Postgres.

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

It also offers first-class support for ENS, which in Trino would require an additional `SELECT` from another table. Moreover, EQL allows you to fetch the `code` of an account using the wildcard operator `*` along with `address`, `nonce`, and `balance`—a process that would otherwise require separate queries in Trino.
```SQL
GET * 
FROM account vitalik.eth
ON eth
```

## How are the projects similar?
While EQL and Dune differ significantly in their design and long-term objectives, they do share several common features, particularly in their ability to extract and export blockchain data. Both platforms allow users to run queries that extract on-chain information and provide options for exporting query results, though the exact methods and formats differ.

1. **Data Extraction**:  
    Both Dune and EQL enable users to extract data from blockchain ecosystems using familiar, SQL-like query languages. In Dune, users write SQL queries to access data from various raw Ethereum tables, combining different datasets as needed. EQL, on the other hand, offers a specialized SQL-like syntax tailored to the Ethereum Virtual Machine (EVM), streamlining access to key blockchain data like accounts, transactions, and balances.
    
2. **Exporting Query Results**:  
    Both platforms also provide options to export the results of queries, though with some differences in the formats supported and pricing models:
    - **Dune** allows users to export query results directly from its user interface, but this feature is limited to CSV format and is only available with a paid plan. For users on the free plan, there is still the option to export data via API requests, albeit with some technical overhead.
    - **EQL**, being an open-source project, offers more flexibility in terms of export formats and cost. Users can export data in multiple formats, including JSON, CSV, and Parquet, directly from the platform. Since EQL is open source, all these export features are freely available to any user.
      
3. **User-Friendly Interfaces**:  
    Both projects aim to simplify the process of querying blockchain data. Dune provides an interactive interface for building queries, visualizing data, and sharing insights through dashboards. EQL, while still under development, is designed to reduce the complexity of querying EVM chains by offering a language that directly maps to blockchain entities, enabling developers to fetch key data in fewer steps than traditional SQL.
    

In summary, while Dune and EQL differ in their broader goals and the scope of their respective query languages, they share core functionalities in data extraction and export options. Both platforms empower users to access and analyze blockchain data, with Dune excelling in user-friendly visualization and EQL standing out for its flexibility in query syntax and free access to export capabilities in multiple formats.

## When should you use EQL
- Quick exploration of data
- Quickly check Ethereum state
- Check data about one or many accounts
- Query different objects that can't be cached


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
	