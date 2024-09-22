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

// !TODO finish this paragraph
Dune provides query engines in three different "sizes" — Free, Medium, and Large — with Free being the slowest, but as the name implies, free from any cost, Medium costing 10 credits and having a monthly limit of 2500, and Large being the  
### Testing Conditions and Considerations
These tests were conducted as of 2024-09-21. It is essential to acknowledge that Dune’s performance may vary during peak usage times, similar to other web applications. Such fluctuations could affect the consistency of the results. If any of the outcomes appear to deviate significantly from the expected average performance, it is recommended to rerun the tests to verify their accuracy and ensure the reliability of the comparative analysis.
EQL’s performance is influenced by several factors, including processing power, network conditions, and the efficiency of Remote Procedure Calls (RPC). Ensuring optimal conditions in these areas is crucial for achieving the best possible performance outcomes with EQL.
### The tests
- Fetching 100 blocks in a range
- Fetching 100 blocks one by one
- Fetching 100 transactions by hash
- Fetching account's nonce and balance
- Fetching logs from USDC transfers in 100 blocks range

####  100 blocks in a range
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

#### 100 blocks one by one
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

#### 100 transactions by hash
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

|               | EQL | Dune Free | Dune Medium |
| ------------- | --- | --------- | ----------- |
| **Mean**      |     | 7.93 s    | 30.05 s     |
| **Median**    |     | 4.09 s    | 26.84 s     |
| **Std. Dev.** |     | 8.13 s    | 18.33 s     |


####  Logs from USDC transfers in 100 blocks range

Queries:
```SQL
# Dune
SELECT *
FROM ethereum.logs l
WHERE l.contract_address=0xdAC17F958D2ee523a2206206994597C13D831ec7
AND l.topic0=0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
AND l.block_number BETWEEN 4638657 AND 4638758

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



#### Account's balance and nonce

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

|               | EQL       | Dune Free | Dune Medium  |
| ------------- | --------- | --------- | ------------ |
| **Mean**      | 939.03 ms | 48.99 s   | 4 min 49 sec |
| **Median**    | 904.04 ms | 40.87 s   | 2 min 39 sec |
| **Std. Dev.** | 93.925 ms | 18.46 s   | 3 min 9 sec  |

