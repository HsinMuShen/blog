# Building a Token: ERC-20 and ERC-721 Standards

```
          /\_/\              🪙 ERC-20 Token    →    🖼️ ERC-721 NFT    →    🏪 Standards
         ( •.• )           (Fungible)             (Non-Fungible)         (Universal)
        / >💎< \           |                      |                      |
                           └─ MTK Currency        └─ Unique #1, #2       └─ MetaMask
                           └─ transfer()          └─ ownerOf()           └─ OpenSea
```

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is ERC?](#2-what-is-erc)
3. [ERC-20: Fungible Tokens](#3-erc-20-fungible-tokens)
   - [3.1 Key Functions](#31-key-functions)
   - [3.2 Example: MyToken Contract](#32-example-mytoken-contract)
4. [ERC-721: Non-Fungible Tokens (NFTs)](#4-erc-721-non-fungible-tokens-nfts)
   - [4.1 Key Functions](#41-key-functions)
   - [4.2 Example: MyNFT Contract](#42-example-mynft-contract)
5. [Running Locally: Step-by-Step](#5-running-locally-step-by-step)
6. [Interacting with ERC-20](#6-interacting-with-erc-20)
7. [Interacting with ERC-721](#7-interacting-with-erc-721)
8. [Why Standards Matter](#8-why-standards-matter)
9. [Key Takeaways](#9-key-takeaways)
10. [References](#10-references)

---

## 1. Introduction

So far, we've built simple contracts and explored Solidity basics. Now let's level up: building tokens.

Tokens are one of the most common use cases for Ethereum. They can represent:

- **Currencies** (like USDT, DAI, UNI)
- **Ownership shares** (DAO voting power)
- **Collectibles** (NFTs like CryptoPunks, Bored Apes)
- **Access rights** (tickets, memberships, subscriptions)

But tokens only work because of **standards** — agreed rules that ensure compatibility across wallets, marketplaces, and dApps.

The two most important ones:

- **ERC-20** → fungible tokens (currencies)
- **ERC-721** → non-fungible tokens (NFTs)

We'll build both, deploy them, and interact with them step by step.

## 2. What is ERC?

**ERC = Ethereum Request for Comments.**

Think of it like a web standard (HTML, CSS). Just as every browser knows how to render `<div>` or `<h1>`, every Ethereum wallet knows how to interact with ERC-20 and ERC-721 tokens.

## 3. ERC-20: Fungible Tokens

**"Fungible" = interchangeable.**

- 1 USDT = 1 USDT
- 1 ETH = 1 ETH

ERC-20 tokens are like currencies: each unit is identical.

### 3.1 Key Functions (Simplified)

- `totalSupply()` → total tokens created
- `balanceOf(address)` → how many tokens a wallet has
- `transfer(to, amount)` → send tokens
- `approve(spender, amount)` → allow another account/contract to spend tokens for you
- `transferFrom(from, to, amount)` → actually move tokens using that approval

### 3.2 Example: contracts/MyToken.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20 {
    constructor(uint initialSupply) ERC20("MyToken", "MTK") {
        // Mint all tokens to deployer
        _mint(msg.sender, initialSupply * (10 ** decimals()));
    }
}
```

🔍 **Explanation:**

- `ERC20("MyToken", "MTK")` → sets the name and symbol.
- `_mint(msg.sender, …)` → gives all tokens to the deployer.
- `decimals()` → defaults to 18, like ETH. So "100 tokens" = 100 * 10^18 units.

💡 **Real-world analogy:**
This is like creating a new currency MyToken (MTK) and minting the first batch to your wallet.

## 4. ERC-721: Non-Fungible Tokens (NFTs)

**"Non-fungible" = unique.**

- NFT #1 ≠ NFT #2

Each has its own identity (metadata, image, traits).

ERC-721 tokens are like collectibles.

### 4.1 Key Functions (Simplified)

- `ownerOf(tokenId)` → who owns the NFT
- `balanceOf(owner)` → how many NFTs someone has
- `transferFrom(from, to, tokenId)` → move ownership
- `approve(spender, tokenId)` → allow someone to move it

### 4.2 Example: contracts/MyNFT.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {
    uint public nextTokenId;

    constructor() ERC721("MyNFT", "MNFT") {}

    function mint() public {
        _safeMint(msg.sender, nextTokenId);
        nextTokenId++;
    }
}
```

🔍 **Explanation:**

- `ERC721("MyNFT", "MNFT")` → sets the NFT collection name and symbol.
- `nextTokenId` → counter for which NFT ID to mint next.
- `_safeMint(msg.sender, nextTokenId)` → creates a new NFT and assigns it to caller.

💡 **Real-world analogy:**
This is like printing a unique trading card each time someone mints: Card #0, Card #1, Card #2…

## 5. Running Locally: Step-by-Step

When testing locally with Hardhat, you need to run a blockchain node.

**Open two terminals:**

**Terminal A — start local blockchain**

```bash
npx hardhat node
```

This prints 20 funded test accounts (10,000 ETH each).

**Terminal B — compile & deploy**

```bash
npx hardhat compile

# ERC-20
npx hardhat run scripts/deploy-erc20.ts
npx hardhat run scripts/interact-erc20.ts

# ERC-721
npx hardhat run scripts/deploy-erc721.ts
npx hardhat run scripts/interact-erc721.ts
```

💡 **Why two terminals?**

- Terminal A = keeps blockchain running.
- Terminal B = sends commands to it.

## 6. Interacting with ERC-20

`scripts/interact-erc20.ts` demonstrates two patterns:

- **Direct transfer** → move tokens directly.
- **Approve + transferFrom** → let someone else pull tokens from your account.

Example snippets:

```javascript
// Direct transfer: owner -> user
await token.getFunction("transfer")(user, toUnits(100));

// Approve user to spend 50
await token.getFunction("approve")(user, toUnits(50));

// As user, pull 30 from owner
const tokenAsUser = token.connect(signer1);
await tokenAsUser.getFunction("transferFrom")(owner, user, toUnits(30));
```

💡 **Analogy:**

- `transfer` = handing someone cash.
- `approve + transferFrom` = giving them permission to swipe your card (up to a limit).

## 7. Interacting with ERC-721

`scripts/interact-erc721.ts` shows the NFT lifecycle:

- **Mint** → create NFT #0.
- **Approve** → let another account transfer it.
- **transferFrom** → that account actually moves it.

Example snippets:

```javascript
// Mint NFT to owner
await nft.getFunction("mint")();

// Approve user for tokenId 0
await nft.getFunction("approve")(user, 0);

// As user, transfer it
const nftAsUser = nft.connect(signer1);
await nftAsUser.getFunction("transferFrom")(owner, user, 0);
```

💡 **Analogy:**

- Mint = printing a unique card (#0).
- Approve = saying "User can take this card."
- TransferFrom = user actually claims it.

## 8. Why Standards Matter

- **Wallets** like MetaMask instantly display balances of any ERC-20/ERC-721 token.
- **Marketplaces** like Uniswap or OpenSea don't need custom code — they just rely on the standard.
- **Developers** can build composable apps (games, DeFi, DAOs) that interoperate easily.

**Without standards, each token would be an island.**

## 9. Key Takeaways

- **ERC-20** = fungible tokens (currencies, points, governance).
- **ERC-721** = non-fungible tokens (NFTs, collectibles, tickets).
- `transfer` = direct send.
- `approve + transferFrom` = delegated transfer.
- **Always use OpenZeppelin** to avoid reinventing standards (and bugs).

## 10. References

- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/)
- [ERC-20 Standard](https://eips.ethereum.org/EIPS/eip-20)
- [ERC-721 Standard](https://eips.ethereum.org/EIPS/eip-721)
- [Solidity by Example – ERC20](https://solidity-by-example.org/app/erc20/)
- [Solidity by Example – ERC721](https://solidity-by-example.org/app/erc721/)

---

**📖 Blog Series Navigation:**

← Previous: [Deep Dive into Solidity: Syntax, Data Types & Functions](https://github.com/HsinMuShen/blog/issues/17)  
🏠 Series Home: [From Web Developer to Blockchain Engineer](https://github.com/HsinMuShen/blog/issues/11)  
→ Next: Smart Contract Security: Common Vulnerabilities & How to Avoid Them
