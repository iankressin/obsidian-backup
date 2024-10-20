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