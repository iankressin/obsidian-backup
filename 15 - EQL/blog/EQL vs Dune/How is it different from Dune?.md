The title of this article captures the initial reaction many people have when they first learn about EVM Query Language (EQL). Although there are surface-level similarities between EQL and Dune, these two projects are fundamentally distinct in their goals, design, and use cases. This article seeks to address the common question of how EQL differs from Dune, while also examining the points where the two projects overlap or share common features.

The analysis will begin with a comprehensive, high-level overview of both EQL and Dune, providing essential background information on each project. Following this, it will delve deeper into a detailed comparison, highlighting both the differences and the similarities in their approaches to querying blockchain data. Finally, the post will conclude with a benchmark comparison, evaluating the performance of EQL and Dune in specific use cases, allowing readers to better understand their respective strengths and limitations in practical scenarios.
## Outlining the projects
Dune is a data analytics platform designed for querying and visualizing blockchain data, primarily focused on Ethereum and other related blockchains such as Binance Smart Chain, Polygon, and Optimism. It provides users with the ability to run SQL queries on publicly available on-chain data, making it possible to extract, analyze, and visualize information from these blockchains. Users interact with the data primarily through a SQL-based query engine that supports custom and predefined queries, enhanced by caching and parallel processing for performance. The platform offers interactive dashboards for visualization, supporting various chart types and dynamic filtering, and facilitates collaboration through public sharing and forking of dashboards.

EVM Query Language (EQL) is a data extraction tool that offers users a SQL-like language to execute queries and retrieve data from EVM chains. The syntax is currently under active development to simplify access to on-chain data for researchers and developers. At its present stage, EQL translates user queries into JSON-RPC requests, providing an efficient and straightforward method for querying the blockchain. The ultimate objective of the project is to create a fully decentralized storage engine, enabling anyone to query EVM chains using a relational approach similar to SQL databases. Unlike Dune, EQL does not aim to index or decode various smart contracts and on-chain data.
## How are the projects different?
Dune is a platform focused on efficient queries and data visualization. The platform started focused solely on the Ethereum ecosystem but has expanded their operations across all the blockchain space, from Bitcoin do SQLana. 1

## How are the projects similar?
When people hear Etherum and SQL in the same sentence they immediately think about Dune, this is proven through the frequently asked questions about the differences between EQL and Dune.


## When should you use EQL
- Quick exploration of data
- Quickly check Ethereum state
- Check data about one or many accounts
- Query different objects that can't be cached
- 


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
	