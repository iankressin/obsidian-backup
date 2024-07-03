---
date: 2024-07-03 10:19
tags:
  - final
source: https://www.blocknative.com/blog/blobsplaining
---


# Blobsplanning

- EIP-4844 introduced a new type of transaction, type-3 transactions, that uses the newly introduced storage for short-lived data, the blobspace.

- A type-3 transactions can contains 1 to 6 blobs, each blob holding 128Kib of data

- Type-3 transaction fees are calculated differently from the regular type-2 transactions.
	- For EIP-1559: $$Total Fee=(BF×CG)+(PF×CG)$$
	- For EIP-4844:$$Total Fee=(BF × CG)+(PF × CG)×(BBF + BG)$$

- As seeing in the equations above, type-3 transaction users still need to pay gas fees on any calldata gas used.
