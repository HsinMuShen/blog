# What is Blockchain? A Developer's Guide Beyond the Hype

```
           /\_/\              🔗 Block 1    →    🔗 Block 2    →    🔗 Block 3
          ( •.• )           (Genesis)         (Transactions)      (More Txs)
         / >🔗< \           |                 |                   |
                            └─ Hash: 0x000    └─ Prev: 0x000     └─ Prev: 0xabc
                            └─ Data: ...      └─ Hash: 0xabc     └─ Hash: 0xdef
```

## Table of Contents

1. [Introduction](#1-introduction)
2. [What Is a Blockchain?](#2-what-is-a-blockchain)
3. [Key Concepts](#3-key-concepts)
   - [3.1 Blocks](#31-blocks)
   - [3.2 Nodes](#32-nodes)
   - [3.3 Consensus](#33-consensus)
4. [Example: Blockchain Transaction](#4-example-blockchain-transaction)
5. [Why It Matters for Developers](#5-why-it-matters-for-developers)
6. [Example Code: Interacting with Ethereum](#6-example-code-interacting-with-ethereum)
7. [Key Takeaways](#7-key-takeaways)
8. [References](#8-references)

---

## 1. Introduction

You've probably heard the buzzwords: Bitcoin, Ethereum, Web3, NFTs. Everyone seems to be talking about blockchain. But what actually is blockchain? And why should web developers care?

At its core, a blockchain is not magic. It's a special type of database that's shared across many computers, with rules that make it hard to cheat. Once you understand that, blockchain becomes less mysterious and more like another tool in your dev toolbox.

In this post, we'll break blockchain down with clear analogies, developer-friendly explanations, and a simple example.

## 2. What Is a Blockchain?

A blockchain is a:

- **Ledger** (like a database or log file).
- **Distributed** (everyone has a copy).
- **Append-only** (you can only add new entries, not edit or delete).
- **Secured by consensus** (nodes agree on the truth).

**Imagine Git:**

- Every commit = a block.
- The history of commits = the chain.
- Everyone has a copy of the repo.
- Consensus = git merge rules + signing commits.

That's blockchain in developer terms.

## 3. Key Concepts

### 3.1 Blocks

- A block contains a batch of transactions.
- Each block links to the previous block using a hash (like a checksum).
- This creates the chain of blocks.

### 3.2 Nodes

- Computers on the network that maintain the blockchain.
- They store a full copy of the chain and validate new blocks.

### 3.3 Consensus

- Rules that make sure everyone agrees on the same version of the chain.
- Example: Proof of Work (Bitcoin) or Proof of Stake (Ethereum).

## 4. Example: Blockchain Transaction

Say Michael sends Avery 1 ETH:

1. **Michael signs a transaction** with his private key.
2. **The transaction goes to the network.**
3. **Nodes validate it** (does Michael have 1 ETH?).
4. **A block producer adds it** to a new block.
5. **Block is confirmed** → Avery now has 1 ETH.

Unlike a bank transfer, **no middleman is needed**.

## 5. Why It Matters for Developers

Blockchain is:

- **Immutable** – data can't be changed once added.
- **Transparent** – anyone can verify the data.
- **Programmable** – with smart contracts, you can run code on the blockchain.

This enables things that weren't possible in Web2:

- **DeFi** (banking without banks).
- **NFTs** (unique digital assets).
- **DAOs** (on-chain organizations).

## 6. Example Code: Interacting with Ethereum

With Ethers.js, you can read blockchain data just like querying an API:

```javascript
import { ethers } from "ethers";

const provider = new ethers.JsonRpcProvider("https://eth-sepolia.g.alchemy.com/v2/YOUR_API_KEY");

async function getBlockNumber() {
  const blockNumber = await provider.getBlockNumber();
  console.log("Current block:", blockNumber);
}

getBlockNumber();
```

This shows the latest block — like asking a database for the most recent record.

## 7. Key Takeaways

- **Blockchain is a distributed, append-only ledger.**
- **Blocks, nodes, and consensus keep it secure.**
- **For developers, blockchain is like a database + backend you can't shut down.**

## 8. References

- [Ethereum Whitepaper](https://ethereum.org/en/whitepaper/)
- [Bitcoin Whitepaper](https://bitcoin.org/bitcoin.pdf)
- [Blockchain Basics – IBM](https://www.ibm.com/topics/what-is-blockchain)

---

**📖 Blog Series Navigation:**

🏠 Series Home: [From Web Developer to Blockchain Engineer](https://github.com/HsinMuShen/blog/issues/11)  
→ Next: [Web2 vs Web3: What Changes for Developers?](https://github.com/HsinMuShen/blog/issues/13)
