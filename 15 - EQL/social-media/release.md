**Today marks the alpha release of the EVM Query Language (EQL)**

 
EQL is a SQL-like language for querying EVM chains, designed to handle complex relational queries on blocks, accounts, and transactions with an ergonomic syntax.

### Problems EQL Aims to Solve

- **Complex Data Analysis:** The current key-value model in Ethereum clients is inefficient for complex queries.
- **Boilerplate Code:** Developers often write excessive code for simple queries.
- **Performance Issues:** Sequential RPC calls are slow and rate-limited.

### How EQL Addresses These Issues

- **Query Performance:** Research is focused on B+ Tree structures for efficient data access without centralization.
- **Ergonomics:** The goal is to provide a simple syntax to enhance productivity and reduce the learning curve for developers and researchers.

### EQL Syntax and Capabilities

EQL uses a straightforward syntax to simplify querying EVM data:

`GET <[fields, ]> FROM <entity> <entity_id> ON <chain>`

#### Examples:

- **Fetch an account's `nonce` and `balance` on Ethereum:**
`GET nonce, balance FROM vitalik.eth ON eth`

- **Retrieve block `size` and `timestamp` on Base:**
`GET size, timestamp FROM block 1 ON base`

- **Get transaction `from`, `value`, and `to` on Arbitrum:**
`GET from, value, to FROM tx 0x... ON arb`

#### Supported Entities:

- **Accounts:** `GET nonce, balance FROM <account> ON <chain>`
- **Blocks:** `GET size, timestamp FROM <block> ON <chain>`
- **Transactions:** `GET from, value, to FROM <tx> ON <chain>`

A full list of supported entities, fields, and chains is available [here](https://github.com/iankressin/eql).

### Try the Alpha Now

- **Web-based REPL:** [eql-app.vercel.app](https://eql-app.vercel.app/)
- **CLI and REPL:** [github.com/iankressin/eql](https://github.com/iankressin/eql)
- **Rust:** [crates.io/crates/eql_core](https://crates.io/crates/eql_core)
- **Typescript:** Coming soon

This release aims to make EVM data more accessible and support Ethereum development. For more information or to discuss EQL, you can DM me on [X](https://x.com/iankguimaraes).
 

# THIS GOES TO THE FIRST COMMENT
 