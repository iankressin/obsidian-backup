In this phase EQL will expand beyond the syntax and will focus on improving the data extraction performance and capabilities.

As shown in the [[EQL vs Dune]] benchmark, EQL is behind Dune in terms of performance in many queries. To become the defacto Ethereum extraction tool, we need to make sure that we're doing better than the data analysis tools.
#### v0.1.8-alpha
- Extract large amount of blocks: Dune respond the block queries for at most 1,000,000 blocks (test this info)
- Subscription based extraction: subscribe to a given query, such as `GET * FROM transaction WHERE block=1:10 AND transaction.value > 100 ON eth > tx.csv`, and append each new result to the dump file `tx.csv`