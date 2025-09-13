# Intro to Smart Contracts: The "Hello World" of Blockchain

```
          /\_/\              📝 Contract Code    →    🔗 Blockchain    →    ⚡ Execute
         ( •.• )           (Solidity)              (Deployment)           (Functions)
        / >⚡< \           |                       |                      |
                           └─ Hello World          └─ 0x1234...           └─ getMessage()
                           └─ Functions            └─ Immutable           └─ setMessage()
```

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is a Smart Contract?](#2-what-is-a-smart-contract)
3. [Writing Our First Smart Contract](#3-writing-our-first-smart-contract)
4. [Breaking Down the Code](#4-breaking-down-the-code)
5. [Compile, Deploy, Interact: The Lifecycle](#5-compile-deploy-interact-the-lifecycle)
   - [5.1 Compile](#51-compile)
   - [5.2 Deploy](#52-deploy)
   - [5.3 Interact](#53-interact)
6. [Step-by-Step Command Flow](#6-step-by-step-command-flow)
   - [6.1 Local Development (Hardhat Network)](#61-local-development-hardhat-network)
   - [6.2 Sepolia Testnet](#62-sepolia-testnet)
7. [Gas & Who Pays](#7-gas--who-pays)
8. [Diagram: Smart Contract Lifecycle](#8-diagram-smart-contract-lifecycle)
9. [Key Takeaways](#9-key-takeaways)
10. [References](#10-references)

---

## 1. Introduction

Every programming journey begins with a "Hello World."
In Web3, that means writing and deploying your first smart contract.

A smart contract is code that lives on the blockchain. Unlike backend code that runs on a server you control, a smart contract is stored at a public address and executes exactly as written — no one can secretly modify or shut it down.

In this post, we'll:

- Explain what smart contracts are and why they matter.
- Write a simple HelloWorld contract in Solidity.
- Compile, deploy, and interact with it using Hardhat + ethers.js.
- Break down key concepts like gas, public variables, view functions, and immutability.

By the end, you'll understand the foundations of blockchain programming.

## 2. What is a Smart Contract?

At a high level:

- A smart contract is like a backend service (API + database) but stored on-chain.
- It contains:
  - **State** → variables stored permanently on the blockchain.
  - **Functions** → methods that let you read or change that state.
- Once deployed, it is **immutable** — the code cannot be changed.
- Anyone can interact with it by sending transactions.

📌 **Analogy: A vending machine.**
- You put money in (transaction).
- The machine runs its code (logic).
- You get candy (output) — guaranteed by the machine's programming.

## 3. Writing Our First Smart Contract

Create a new file: `contracts/HelloWorld.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

contract HelloWorld {
    // State variable stored permanently on-chain
    string public message;

    // Constructor runs once at deployment
    constructor(string memory initMessage) {
        message = initMessage;
    }

    // Read function (free, no gas)
    function getMessage() public view returns (string memory) {
        return message;
    }

    // Write function (costs gas)
    function setMessage(string memory newMessage) public {
        message = newMessage;
    }
}
```

## 4. 🔎 Breaking Down the Code

**`contract HelloWorld { ... }`**
- Defines a contract. Think of it like a class in JS/TS, but deployed on-chain.

**`string public message;`**
- A state variable.
- `public` = visible to anyone. Solidity auto-generates a getter (so you can call `hello.message()` directly).
- Stored permanently on the blockchain.

**`constructor(string memory initMessage)`**
- Runs once, at deployment.
- Initializes `message` with "Hello, Blockchain!" or whatever you pass in.
- After that, it never runs again.

**`function getMessage() public view returns (string memory)`**
- `view` → read-only, doesn't change blockchain state.
- `returns (string memory)` → function gives back text.
- Free to call (no gas).

**`function setMessage(string memory newMessage) public`**
- Updates the state variable.
- This is a write → costs gas, because it modifies the blockchain.

## 5. Compile, Deploy, Interact: The Lifecycle

### 5.1 Compile

```bash
npx hardhat compile
```

This translates your Solidity contract → EVM bytecode and ABI, stored in `artifacts/contracts/HelloWorld.sol/HelloWorld.json`.

### 5.2 Deploy

💡 **About this script**
`scripts/deploy-hello.ts` connects to either a local Hardhat node or the Sepolia testnet, then:
- Builds a signer (wallet).
- Deploys HelloWorld with an initial message.
- Saves the deployed contract address in `deployments/{network}.json`.

📌 **Important**: If you're deploying locally, you must first run a local Hardhat node:

```bash
npx hardhat node
```

**Why?**
- The node simulates a local blockchain on your computer.
- It gives you pre-funded test accounts (with fake ETH).
- Without it, your scripts have no blockchain to connect to.

Then, in another terminal:

```bash
npx hardhat run scripts/deploy-hello.ts
```

### 5.3 Interact

💡 **About this script**
`scripts/interact-hello.ts` connects to the deployed contract and:
- Reads the current message.
- Updates the message with `setMessage`.
- Reads again to confirm the update.

Run it:

```bash
npx hardhat run scripts/interact-hello.ts
```

**Expected output:**

```
📖 Current message: Hello, Blockchain!
🔄 Setting message to: "Hello from Web3! 🚀"
✅ Updated message: Hello from Web3! 🚀
```

## 6. Step-by-Step Command Flow

### 6.1 Local Development (Hardhat Network)

**1. Start a local blockchain:**
```bash
npx hardhat node
```

**2. In another terminal, compile contracts:**
```bash
npx hardhat compile
```

**3. Deploy HelloWorld:**
```bash
npx hardhat run scripts/deploy-hello.ts
```

**4. Interact with HelloWorld:**
```bash
npx hardhat run scripts/interact-hello.ts
```

### 6.2 Sepolia Testnet

**1. Add your RPC URL and PRIVATE_KEY in `.env`:**
```env
SEPOLIA_RPC_URL=https://sepolia.infura.io/v3/YOUR_API_KEY
PRIVATE_KEY=0xYOUR_PRIVATE_KEY
```

**2. Compile contracts:**
```bash
npx hardhat compile
```

**3. Deploy HelloWorld to Sepolia:**
```bash
npx hardhat run scripts/deploy-hello.ts --network sepolia
```

**4. Interact with HelloWorld on Sepolia:**
```bash
npx hardhat run scripts/interact-hello.ts --network sepolia
```

## 7. Gas & Who Pays

- **Reads** (`getMessage`): free.
- **Writes** (`setMessage`): cost gas.
- Gas is always paid by the **caller's wallet**. The smart contract itself never pays.

📌 **Analogy: Sending a letter.**
- You (caller) must buy the stamp (gas).
- The post office (Ethereum network) delivers it.
- The receiver (smart contract) acts, but doesn't pay postage.

## 8. Diagram: Smart Contract Lifecycle

```
[ Solidity Code ] ---> (Compile) ---> [ Bytecode + ABI ]
                                  |
                                  v
                      (Deploy transaction with gas)
                                  |
                                  v
[ Blockchain ] -----> Contract stored at address 0x1234
                                  |
                                  v
   Reads (free)       <--->    Interact     <--->   Writes (cost gas)
```

## 9. Key Takeaways

- A smart contract is an immutable program deployed to a blockchain.
- **Compile → Deploy → Interact** is the essential lifecycle.
- For local dev, always run `npx hardhat node` first to simulate a blockchain.
- Reads are free; writes cost gas, paid by the caller.
- `deploy-hello.ts` handles deployment, `interact-hello.ts` handles reading/writing.

## 10. References

- [Solidity Documentation](https://docs.soliditylang.org/)
- [Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/)
- [Hardhat Documentation](https://hardhat.org/docs)
- [Ethers.js Documentation](https://docs.ethers.org/)

---

**📖 Blog Series Navigation:**

← Previous: [Setting Up Your Web3 Developer Environment with Hardhat v3 + ethers.js](https://github.com/HsinMuShen/blog/issues/14)  
🏠 Series Home: [From Web Developer to Blockchain Engineer](https://github.com/HsinMuShen/blog/issues/11)  
→ Next: [Deep Dive into Solidity: Syntax, Data Types & Functions](https://github.com/HsinMuShen/blog/issues/17)  
