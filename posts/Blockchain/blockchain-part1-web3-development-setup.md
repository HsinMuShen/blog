# Setting Up Your Web3 Developer Environment with Hardhat v3 + ethers.js

```
           /\_/\              💻 Local Dev        →        🌐 Sepolia
          ( •.• )           (Hardhat Node)              (Live Testnet)
         / >⛏️< \           |                          |
                            └─ 20 Test Accounts       └─ Real ETH Testnet
                            └─ Instant Mining         └─ Public Blockchain
                            └─ Reset on Restart       └─ Persistent State
```

## Table of Contents

1. [Introduction](#1-introduction)
2. [Step 1: Install Prerequisites](#2-step-1-install-prerequisites)
3. [Step 2: Initialize a Hardhat Project](#3-step-2-initialize-a-hardhat-project)
4. [Step 3: Write Your First Contract](#4-step-3-write-your-first-contract)
5. [Step 4: Run a Local Blockchain](#5-step-4-run-a-local-blockchain)
6. [Step 5: Deploy the Contract](#6-step-5-deploy-the-contract)
7. [Step 6: Interact With the Contract](#7-step-6-interact-with-the-contract)
8. [Step 7: Verify on Etherscan (Sepolia Only)](#8-step-7-verify-on-etherscan-sepolia-only)
9. [Diagram: Contract Deployment & Interaction](#9-diagram-contract-deployment--interaction)
10. [Key Takeaways](#10-key-takeaways)

---

## 1. Introduction

When you first dive into blockchain, the biggest hurdle is often the developer setup.

If you're a web developer, you're used to npm, VS Code, and local servers — but Web3 adds smart contracts, wallets, and blockchains.

In this guide, we'll set up a full Web3 development environment using Hardhat v3 and ethers.js. You'll:

- **Run a local Ethereum blockchain** (Hardhat node).
- **Write and compile your first contract** (Counter).
- **Deploy it both locally and on the Sepolia testnet.**
- **Interact with it programmatically.**
- **Understand the meaning of Hardhat's output** (accounts, private keys, logs).

By the end, you'll have your first dApp workflow running like a blockchain engineer 🚀

## 2. Step 1: Install Prerequisites

- **Node.js v18+** → [Download](https://nodejs.org/)
- **npm** (comes with Node)
- **VS Code + Solidity extension**
- **MetaMask wallet** → [Download](https://metamask.io/)

## 3. Step 2: Initialize a Hardhat Project

```bash
mkdir my-dapp && cd my-dapp
npm init -y
npm install --save-dev hardhat
npx hardhat
```

**Choose:**
- "A TypeScript Hardhat project using Mocha and Ethers.js"
- Say **Yes** for ESM.

## 4. Step 3: Write Your First Contract

Create `contracts/Counter.sol`:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.28;

contract Counter {
  uint public x;

  event Increment(uint by);

  function inc() public {
    x++;
    emit Increment(1);
  }

  function incBy(uint by) public {
    require(by > 0, "incBy: increment should be positive");
    x += by;
    emit Increment(by);
  }
}
```

**Compile:**

```bash
npx hardhat compile
```

## 5. Step 4: Run a Local Blockchain

Start a local Ethereum node:

```bash
npx hardhat node
```

You'll see:

```
Accounts
========
Account #0:  0xf39fd6e51a... (10000 ETH)
Private Key: 0xac0974be...
...
```

### 🔎 What this means

- **Accounts** → 20 pre-funded test wallets.
- **Balance** → 10,000 ETH each (fake ETH).
- **Private Keys** → exposed because this blockchain is only for dev.

⚠️ **Never use these keys on mainnet.**

Think of this as your local sandbox.

## 6. Step 5: Deploy the Contract

Deployment is like publishing a REST API backend — but instead of a server, you're uploading code to a blockchain.

### 📜 scripts/deploy.ts

This script:
- Detects which network you target.
- Connects to a provider (local or Sepolia).
- Deploys the contract.
- Saves the contract address for later use.

### ▶️ Run the Deploy Script

**Local:**

```bash
npx hardhat run scripts/deploy.ts
```

**Sepolia:**

```bash
npx hardhat run --network sepolia scripts/deploy.ts
```

### ✅ Example Sepolia Output

```
Target network: sepolia
Deploying with: 0xYourWallet | target=sepolia | chainId=11155111
Counter deployed to: 0x1234abcd5678ef901234abcd5678ef901234abcd
Saved address to: deployments/sepolia.json
```

📌 **Key idea**: Contracts get their own unique address on-chain. Your wallet deployed it, but the contract now lives independently.

## 7. Step 6: Interact With the Contract

Now let's call functions on our contract.

### 📜 scripts/interact.ts

This script:
- Loads the contract address from `deployments/<network>.json`.
- Reads state (`x`).
- Calls `inc()` and `incBy(5)` to update state.

### ▶️ Run the Interaction Script

**Local:**

```bash
npx hardhat run scripts/interact.ts
```

**Sepolia:**

```bash
npx hardhat run --network sepolia scripts/interact.ts
```

### ✅ Example Output

```
Target network: sepolia
Interacting with: 0xYourWallet | target=sepolia | chainId=11155111
Connected to Counter at: 0x1234abcd5678ef901234abcd5678ef901234abcd

📊 Current value x: 0
🔄 Calling inc()...
✅ x after inc(): 1
🔄 Calling incBy(5)...
✅ x after incBy(5): 6

🎉 Successfully interacted with Counter! Total change: 0 → 6
```

### 🔎 What's Happening

- **Reads (calls) are free.**
- **Writes (transactions) cost gas.**
- The blockchain mines the transaction → updates storage → emits events.

## 8. Step 7: Verify on Etherscan (Sepolia Only)

When you deploy on Sepolia, anyone can inspect your contract.

1. Open `deployments/sepolia.json`.
2. Copy the contract address.
3. Paste into 👉 [Sepolia Etherscan](https://sepolia.etherscan.io/).

### ✅ What You'll See

- Contract Creation transaction.
- Your calls to `inc` and `incBy`.
- Gas usage under your wallet address.

This proves your contract is really live on-chain.

## 9. Diagram: Contract Deployment & Interaction

Here's a simple view of how it works:

```
+-------------------+       Deploy Tx        +---------------------+
| Your Wallet (EOA) | ---------------------> |   Ethereum Network   |
| (holds private key)|                       | (Local or Sepolia)   |
+-------------------+                        +---------------------+
        ^                                              |
        |   Contract Address Returned                  |
        +----------------------------------------------+
                                |
                                v
                     +-------------------+
                     |  Counter Contract |
                     |  (Deployed Code)  |
                     +-------------------+
                                ^
                                |
            Interact (inc, incBy, read x)
```

## 10. Key Takeaways

- **Hardhat gives you a local blockchain** with pre-funded test accounts.
- **`npx hardhat run --network sepolia`** deploys to Sepolia (chainId = 11155111).
- **Contracts live at their own addresses**, separate from wallets.
- **Reads are free, writes cost gas.**
- **Etherscan is your block explorer** to verify deployments and calls.

---

**📖 Blog Series Navigation:**

← Previous: [Web2 vs Web3: What Changes for Developers?](https://github.com/HsinMuShen/blog/issues/13)  
🏠 Series Home: [From Web Developer to Blockchain Engineer](https://github.com/HsinMuShen/blog/issues/11)  
→ [Next: Intro to Smart Contracts: The "Hello World" of Blockchain](https://github.com/HsinMuShen/blog/issues/16)
