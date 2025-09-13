# Web2 vs Web3 — What Changes for Developers?

```
           /\_/\              🌐 Web2           →           🔗 Web3
          ( •.• )           (Centralized)               (Decentralized)
         / >🔄< \           |                           |
                            └─ Client ↔ Server         └─ Client ↔ Blockchain
                            └─ Platform owns data      └─ User owns data
                            └─ Email/password login    └─ Wallet connection
```

## Table of Contents

1. [Introduction](#1-introduction)
2. [Web2 vs Web3 at a Glance](#2-web2-vs-web3-at-a-glance)
3. [What Stays the Same](#3-what-stays-the-same)
4. [What Changes](#4-what-changes)
5. [Example: Web3 Login](#5-example-web3-login)
6. [Why It Matters](#6-why-it-matters)
7. [Key Takeaways](#7-key-takeaways)
8. [References](#8-references)

---

## 1. Introduction

As a web developer, you're used to building apps with React, APIs, and databases. That's Web2: client-server architecture.

But Web3 introduces a shift: instead of centralized servers, your app can run on a decentralized blockchain. Data isn't owned by platforms — it's owned by users.

Let's compare Web2 vs Web3, see what's similar, what's different, and walk through an example of logging in with a wallet.

## 2. Web2 vs Web3 at a Glance

| Feature | Web2 (Today's Internet) | Web3 (Decentralized) |
|---------|-------------------------|----------------------|
| **Auth** | Email/password, OAuth | Wallets (MetaMask, WalletConnect) |
| **Data storage** | Centralized DB (MySQL, MongoDB) | Blockchain state, IPFS, Arweave |
| **Servers** | AWS, GCP, APIs | Smart contracts on Ethereum / other chains |
| **Monetization** | Ads, subscriptions | Tokens, NFTs, DeFi |
| **Ownership** | Platforms own content/data | Users truly own digital assets |

## 3. What Stays the Same

- **Frontend frameworks** (React, Next.js, Vue).
- **State management, routing, UI design.**
- **Using APIs** (though now some APIs are contracts).

## 4. What Changes

- **Authentication**: users connect wallets, not email/password.
- **State**: some or all data is on-chain.
- **Payments**: built-in with tokens, no Stripe/PayPal needed.

## 5. Example: Web3 Login

Instead of a login form, Web3 apps ask users to connect their wallet.

```javascript
import { ethers } from "ethers";

async function connectWallet() {
  if (!window.ethereum) throw new Error("MetaMask not installed");
  
  const provider = new ethers.BrowserProvider(window.ethereum);
  const accounts = await provider.send("eth_requestAccounts", []);
  
  console.log("Connected wallet:", accounts[0]);
}
```

This is the Web3 equivalent of `login()`.

## 6. Why It Matters

- **User empowerment**: users own their assets (tokens, NFTs).
- **Interoperability**: apps share standards like ERC-20, ERC-721.
- **New business models**: tokens, staking, DAOs.

## 7. Key Takeaways

- **Web3 extends Web2** with decentralization, wallets, and tokens.
- **Developers don't abandon Web2 skills** — they add blockchain knowledge.
- **Logging in with a wallet** replaces traditional authentication.

## 8. References

- [Ethereum for Web Developers](https://ethereum.org/en/developers/)
- [IPFS Documentation](https://docs.ipfs.tech/)
- [The Graph Protocol](https://thegraph.com/docs/en/)

---

**📖 Blog Series Navigation:**

← Previous: [What is Blockchain? A Developer's Guide Beyond the Hype](https://github.com/HsinMuShen/blog/issues/12)  
🏠 Series Home: [From Web Developer to Blockchain Engineer](https://github.com/HsinMuShen/blog/issues/11)  
→ Next: [Setting Up Your Web3 Developer Environment with Hardhat v3 + ethers.js](https://github.com/HsinMuShen/blog/issues/14)
