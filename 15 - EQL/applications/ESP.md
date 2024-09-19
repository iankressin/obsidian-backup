**First name:**
Ian

**Last name:**
Guimarães

**Email**:
ianguimaraes31@gmail.com (I need a 2077.xyz email)

**Are you submitting on behalf of a team, or as an individual?**
 - [ ] Individual
 - [x] Team

**Individual or team summary:**
*Tell us about yourself, your experience, and your motivations. Feel free to link to any biography pages, LinkedIn pages, etc.*


**City:**
Florianópolis

**Website:**
https://github.com/iankressin https://2077.xyz

**Twitter:**
@iankguimaraes
@2077Collective

**Project name:**
EVM Query Language (EQL)

**Project repo:**
https://github.com/iankressin/eql

**Previous work:**
*Please provide links to published code, research, or other documentation of what you've worked on.*
- Release notes: [https://eql-app.vercel.app/blog/alpha-release-notes](https://eql-app.vercel.app/blog/alpha-release-notes "https://eql-app.vercel.app/blog/alpha-release-notes")
- Roadmap: [https://github.com/iankressin/eql/blob/main/ROADMAP.md](https://github.com/iankressin/eql/blob/main/ROADMAP.md "https://github.com/iankressin/eql/blob/main/ROADMAP.md")
- Web-based REPL: [https://eql-app.vercel.app/](https://eql-app.vercel.app/ "https://eql-app.vercel.app/")
- Last release: [https://github.com/iankressin/eql/releases/tag/0.1.1-alpha](https://github.com/iankressin/eql/releases/tag/0.1.1-alpha "https://github.com/iankressin/eql/releases/tag/0.1.1-alpha")

**What is the project?**
*Describe the main concept and components of the proposed work.*
The EVM Query Language (EQL) is a SQL-like language specifically designed to query Ethereum Virtual Machine (EVM) chains, including Ethereum and Layer 2. The primary goal of EQL is to enable complex relational queries on EVM chain first-class citizens—blocks, accounts, and transactions—while providing developers and researchers with an ergonomic syntax to compose custom datasets efficiently, without boilerplate code.

**What problem(s) are being solved by within the scope of the grant?**
*What are the specific problems, research questions, or needs you are trying to address?*
https://x.com/pumatheuma/status/1836278488527253651

EQL addresses a significant challenge in the Ethereum ecosystem: the difficulty of performing complex data analysis due to the blockchain’s key-value storage model. Traditional Ethereum clients store data in a way that is optimized for direct access via unique keys but is not conducive to relational queries or searches based on internal properties. This limitation hinders productivity for developers and researchers who need to extract and analyze blockchain data, often requiring disproportionate amounts of boilerplate code and facing rate limits with public JSON-RPC providers.

  Despite the fact that the main goal of EQL is move away from public JSON-RPCs, as
  they can be seen as centralization vectors in the space, extracting the maximum amount of capabilities from RPCs can support the long term vision of the project in two ways.
1. Act as middle ground while a better and more decentralized solution is being built
2. Help shape the language syntax

While EQL has not yet achieved its long-term goal of providing a fully decentralized solution for querying EVM chains, we must leverage the available resources effectively. By utilizing RPCs as the backbone of the execution_engine, the team can focus on enhancing other aspects of the language.

One key area of focus is syntax development. In its initial version, EQL supported only a limited set of fields for each of the first-class entities. With the upcoming release of version 0.1.1-alpha, we are expanding support to include additional fields and block range queries. However, there is still more work to be done to cover all available fields accessible through RPCs, and we are committed to pushing forward until full coverage is achieved.

Expanding RPC coverage does not mean indiscriminately incorporating every RPC method into the language. Instead, we will carefully evaluate each method to determine how it can introduce new syntax possibilities, while maintaining our commitment to user experience and language ergonomics.

**Why is your project important?**
*How do you know people need what you're making? Why is this project important for your target demographic/problem area?*
EQL (EVM Query Language) is vital for the Ethereum ecosystem because it addresses significant challenges that developers, researchers, and analysts face when accessing and analyzing blockchain data. By providing a SQL-like language tailored to EVM chains, EQL simplifies complex data retrieval processes and enhances productivity in a way that existing solutions do not. This project fills a clear need for efficient, decentralized, and user-friendly data querying tools in the blockchain space

The Ethereum ecosystem is a treasure trove of data, but accessing that data efficiently remains a significant challenge. Developers building decentralized applications (dApps) and researchers analyzing blockchain trends frequently encounter hurdles due to Ethereum’s key-value storage model. Existing methods for accessing this data are cumbersome, requiring large amounts of boilerplate code and leading to performance bottlenecks, especially when relying on public JSON-RPC providers that enforce rate limits.

• Developers currently spend disproportionate time and effort writing boilerplate code just to access simple blockchain data, such as transaction details or account balances.
• JSON-RPC rate limits and performance issues further hinder developers, forcing them to manage multiple services or platforms to overcome these restrictions.

**How does your project differ from similar ones?**
*What other solutions are being worked on, and what alternatives do people currently rely on? Do you have unique expertise/perspective?*

While there are several existing solutions for querying blockchain data, none fully address the problems that EQL is solving:

**The Graph**
While The Graph is excellent for querying smart contract event data, it falls short when dealing with native Ethereum entities like accounts and transactions. Additionally, The Graph requires users to write subgraphs in AssemblyScript, adding complexity for developers who want quick, direct access to EVM data.

**Dune**
Dune is powerful for pre-indexed blockchain data analysis, but using regular SQL and relational database structures come with its price. We can see below an example of Dune SQL query to fetch an account's nonce and balance
```SQL
-- Fetching account's nonce and balance with Dune SQL
WITH account_balance as (
    SELECT balance
    FROM tokens_ethereum.balances_daily as b
    WHERE b.address = 0x
    AND token_standard = 'native'
    ORDER BY b.day DESC
    LIMIT 1
),
account_nonce as (
    SELECT nonce
    FROM ethereum.transactions t
    WHERE t."from" = 0x
    ORDER BY t.block_number DESC
    LIMIT 1
),
account as (
    SELECT nonce, balance FROM account_nonce, account_balance
)
SELECT * FROM account
```

And this is the query that produces the same result as the above, but now using EQL syntax:
```SQL
-- Fetching account's nonce and balance using EQL
GET nonce, balance FROM account 0x ON eth
```

Dune also requires users to interact with a centralized platform and pay for larger queries, limiting its utility for decentralized, real-time querying needs.

EQL is different because it directly supports querying all Ethereum first-class entities, such as accounts, blocks, and transactions. It also introduces the ability to run queries directly in user applications via a CLI or REPL, reducing the need for additional platforms or services.

**Requested amount**
*Choose denominated currency and enter a whole number in the Amount field*
$30,000.00

**Proposed tasks, roadmap and budget**
*Give us an itemized breakdown of how you'll be using the requested funds. Provide a brief timeline of the expected work and estimated budget. For each month or stage of work, list: main objectives, tasks that need to be completed to reach each objective, deliverables, and anticipated budget.*

**v0.1.4-alpha**
- [ ] Get transactions from blocks:
`GET from, to FROM transaction WHERE tx.value 1, block 1:10 ON eth`

- [ ] Wildcard operator for both fields and chains:
 `GET * FROM account vitalik.eth ON *`
- [ ] REPL improvements: Save query history, fix minor bugs

**v0.1.5-alpha**
- [ ] User configurable RPC list
- [ ] Support to transaction receipt fields under transaction entity
- [ ] Smart-contract support:
 `GET balanceOf(0x...) FROM contract 0x... ON polygon`
 
**v0.1.6-alpha**
- [ ] Add, div, sub, mul:
`ADD(GET nonce FROM account vitalik.eth ON eth, polygon, base)`
- [ ] Get account balance, nonce at specific block and range:
 `GET nonce, balance FROM vitalik.eth WHERE block 1:10 ON base`
- [ ] Support for custom chains:
 `GET nonce, balance FROM vitalik.eth WHERE block 1:10 ON custom-1
 
**v0.1.7-alpha**
- [ ] Codebase refactor
- [ ] Project documentation

**Is your project a public good?**
*If so, how?*

**Is your project open source?**
*If not, why not?*

**What are your plans after the grant is completed?**
*How do you aim to be sustainable after the grant? Alternatively, tell us why this project doesn't need to be sustainable!*

**If you didn't work on this project, what would you work on instead?**

**Have you previously applied to ESP with this same idea or project?**
- [ ] No
- [ ] Yes

**Have you applied for or received other funding?**
*If so, where else did you get funding from?*

**Did anyone recommend that you submit an application to the Ecosystem Support Program?**
Perrie

**Anything else you'd like to share?**
*Is there anything we should know about that hasn't been covered by the questions above? You also have the option to link any supporting documents or relevant sites here.*

