# Smart Contract Security: Common Vulnerabilities & How to Avoid Them

```
          /\_/\              🛡️ Secure Coding    →    🐞 Common Bugs      →    🔨 Fixing Patterns
         ( •.• )           (Best Practices)         (Reentrancy)            (Checks-Effects-Interactions)
        / >🔑< \           |                        |                       |
                           └─ OpenZeppelin          └─ Overflow             └─ Pull Payments
                           └─ Hardhat plugins       └─ Access Control       └─ ReentrancyGuard
```

## Table of Contents

1. [Introduction](#1-introduction)
2. [Why Security Matters in Smart Contracts](#2-why-security-matters-in-smart-contracts)
3. [Top Vulnerabilities in Solidity](#3-top-vulnerabilities-in-solidity)
   - [3.1 Reentrancy Attacks](#31-reentrancy-attacks)
   - [3.2 Integer Overflow / Underflow](#32-integer-overflow--underflow)
   - [3.3 Access Control Issues](#33-access-control-issues)
   - [3.4 Denial of Service (DoS)](#34-denial-of-service-dos)
   - [3.5 Front-running Attacks](#35-front-running-attacks)
   - [3.6 Randomness Pitfalls](#36-randomness-pitfalls)
4. [Best Practices & Tools](#4-best-practices--tools)
5. [Example: Securing a Simple Bank Contract (Detailed Explanation)](#5-example-securing-a-simple-bank-contract-detailed-explanation)
6. [Key Takeaways](#6-key-takeaways)
7. [References](#7-references)

---

## 1. Introduction

Smart contracts are different from Web2 servers:

- They are **immutable** after deployment (you can't patch code in-place).
- They run on a **public blockchain** where money is involved.
- **Bugs are therefore costly and permanent.**

Historical lessons:

- **DAO (2016)** — reentrancy attack drained ~60M USD worth of ETH.
- **Parity multisig (2017)** — bug froze ~150M USD worth of ETH.

This post walks you through common pitfalls, explains why they matter, and shows concrete fixes you can copy into your projects.

## 2. Why Security Matters in Smart Contracts

In Web2 you fix and redeploy; in Web3 you often cannot. A mistake can cause:

- **Immediate loss of funds.**
- **Funds permanently locked.**
- **Fast exploitation by bots** scanning the mempool.

As a developer, treat security as a **first-class feature** — design, implement, and test with attacks in mind.

## 3. Top Vulnerabilities in Solidity

Each subsection below contains:

- A short description of the issue.
- A simple analogy.
- Minimal vulnerable code.
- A safer fix and explanation.

### 3.1 Reentrancy Attacks

**What it is**
When a function sends ETH to an address (a contract), that recipient's fallback/receive function can run immediately. If your contract updates critical state after sending ETH, a malicious recipient can reenter the function and act on stale state to steal funds.

**Analogy**
A vending machine dispenses a snack before marking your account as paid. A thief exploits this to get extra snacks before the ledger is updated.

**Vulnerable code** (bad ordering — interaction before effect):

```solidity
function withdraw(uint _amount) public {
    require(balances[msg.sender] >= _amount, "Not enough balance");

    // Interaction first — sends ETH
    (bool success, ) = msg.sender.call{value: _amount}("");
    require(success, "Transfer failed");

    // Effect later — too late
    balances[msg.sender] -= _amount;
}
```

**Why this fails**
If `msg.sender` is a malicious contract, its `receive()`/`fallback()` can call `withdraw()` again before `balances` is decremented, letting it withdraw repeatedly.

**Fix: Checks → Effects → Interactions (CEI):**

```solidity
function withdraw(uint _amount) public {
    require(balances[msg.sender] >= _amount, "Not enough balance");

    // Effects first — update internal state
    balances[msg.sender] -= _amount;

    // Interactions last — send ETH
    (bool success, ) = payable(msg.sender).call{value: _amount}("");
    require(success, "Transfer failed");
}
```

**Extra protection**
Use OpenZeppelin's `ReentrancyGuard` and the `nonReentrant` modifier as defense-in-depth.

### 3.2 Integer Overflow / Underflow

**What it is**
Before Solidity 0.8, arithmetic overflow/underflow wrapped silently. This allowed attackers to manipulate balances/counters.

**Analogy**
An odometer resets to 0 after reaching its maximum.

**Example** (pre-0.8 vulnerability):

```solidity
uint8 x = 255;
x += 1; // becomes 0 silently
```

**Fix**
Use Solidity `^0.8.0` or later (arithmetic reverts on overflow/underflow). For older code use SafeMath libraries. Also validate inputs with `require()`.

### 3.3 Access Control Issues

**What it is**
Leaving admin functions public or failing to correctly verify the caller lets anybody perform privileged operations.

**Analogy**
Leaving the vault key on the counter.

**Vulnerable code:**

```solidity
function destroy() public {
    selfdestruct(payable(msg.sender)); // anyone can call
}
```

**Fix**
Restrict with an `onlyOwner` modifier (or use OpenZeppelin's `Ownable`):

```solidity
address public owner;

constructor() {
    owner = msg.sender; // deployer becomes owner
}

modifier onlyOwner() {
    require(msg.sender == owner, "Not the owner");
    _;
}

function destroy() public onlyOwner {
    selfdestruct(payable(owner));
}
```

**Better:** Use `Ownable` for tested behavior, and consider multisig or DAO governance for production.

### 3.4 Denial of Service (DoS)

**What it is**
Unbounded loops or pushing funds to many addresses in a single transaction can run out of gas, blocking the function forever.

**Analogy**
A single server trying to handle every customer simultaneously until it crashes.

**Vulnerable code:**

```solidity
function payout() public {
    for (uint i = 0; i < payees.length; i++) {
        payable(payees[i]).transfer(1 ether);
    }
}
```

If `payees` is large, this may exceed block gas limit; then nobody can call `payout()`.

**Fix — Pull payments:**
Let each beneficiary withdraw their own funds:

```solidity
function withdraw() public {
    uint amount = balances[msg.sender];
    require(amount > 0, "Nothing to withdraw");
    balances[msg.sender] = 0;
    (bool success, ) = payable(msg.sender).call{value: amount}("");
    require(success, "Transfer failed");
}
```

**Advantages:** scalable, each user pays their own gas, and no long on-chain loops.

### 3.5 Front-running Attacks

**What it is**
Transactions live in the mempool before they are mined. Attackers or bots can see pending transactions, re-submit the same transaction with higher gas to get prioritized, or craft transactions that take advantage of yours.

**Analogy**
You place a bid in an auction; someone sees your bid, yells the same bid louder, and wins before you.

**Concrete examples**

- **DEX trade:** you post a swap that will change the price; an attacker sees it and performs the trade first to profit (sandwich attacks).
- **NFT mint:** you submit mint with favorable conditions; bots front-run to grab the scarce assets.

**Mitigations**

- **Commit-Reveal:** submit a hash commitment first, reveal later (prevents immediate copying).
- **Private Transactions / Flashbots:** submit to miners privately (not public mempool), preventing bots from seeing/predicting your intent.
- **Time-weighted** or sealed auctions, or on-chain mechanisms that reduce profit from front-running.

Note: every mitigation has tradeoffs in UX, complexity, and cost.

### 3.6 Randomness Pitfalls

**What it is**
Using predictable / miner-controllable on-chain values (`block.timestamp`, `blockhash`) for randomness is insecure — miners can manipulate them to bias outcomes such as lottery draws.

**Bad randomness example:**

```solidity
uint rand = uint(keccak256(abi.encodePacked(block.timestamp, msg.sender)));
```

Miners can tweak `block.timestamp`, exclude or include transactions, etc., to bias results.

**Analogy**
Letting the dealer pick the roulette number.

**Fix**
Use an external verifiable randomness source like **Chainlink VRF**. It returns cryptographically secure, verifiable randomness on-chain, preventing miner manipulation.

## 4. Best Practices & Tools

- **OpenZeppelin** — audited contracts (`Ownable`, `ReentrancyGuard`, `Pausable`, token standards).
- **Static analysis** — Slither (finds common issues).
- **Fuzzing** — Echidna (property-based testing).
- **Security services** — MythX, CertiK for deeper checks.

**Patterns:**

- **Checks → Effects → Interactions.**
- **Pull payments** over push loops.
- **Fail early** (`require`).
- Use `nonReentrant` on critical external-call functions.
- Prefer **multisig/timelock** for admin power.

## 5. Example: Securing a Simple Bank Contract (Detailed Explanation)

Below is a practical, minimal secure bank pattern. After the code I'll explain each line and the security rationale.

### The contract (concise, secure)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SecureBank is ReentrancyGuard, Ownable {
    mapping(address => uint) public balances;

    // deposit ETH
    function deposit() public payable {
        require(msg.value > 0, "Send some ETH");
        balances[msg.sender] += msg.value;
    }

    // safe withdraw
    function withdraw(uint amount) public nonReentrant {
        require(amount > 0, "Amount must be > 0");
        require(balances[msg.sender] >= amount, "Not enough balance");

        // Effects first: update internal ledger
        balances[msg.sender] -= amount;

        // Interactions last: send ETH
        (bool success, ) = payable(msg.sender).call{value: amount}("");
        require(success, "Transfer failed");
    }

    // admin emergency withdrawal
    function emergencyWithdraw() public onlyOwner {
        payable(owner()).transfer(address(this).balance);
    }
}
```

### What each part does & why it's safe

**`mapping(address => uint) public balances;`**

- Internal ledger mapping how much the contract owes each address.
- **Important:** this is the contract's bookkeeping — not the user's wallet balance.

**`deposit()`**

- `payable` so it can accept ETH.
- `require(msg.value > 0, "Send some ETH");` prevents accidental empty calls and spurious events.
- On success: `balances[msg.sender] += msg.value;` — contract vault increases by the ETH sent.
- **Gas & who pays:** the caller pays gas to run `deposit()`, and the ETH sent goes into the contract's balance.

**`withdraw(uint amount) public nonReentrant`**

- `nonReentrant` provides defense-in-depth against reentrancy.
- **Checks:**
  - `require(amount > 0)` avoids no-op calls.
  - `require(balances[msg.sender] >= amount)` ensures they have funds owed.
- **CEI order:**
  - **Effects:** `balances[msg.sender] -= amount;` — update ledger first so reentered calls fail.
  - **Interactions:** `(bool success, ) = payable(msg.sender).call{value: amount}("");` — perform transfer last.
- `require(success, "Transfer failed");` — revert if transfer fails so ledger reverts too.
- **Atomicity:** if call fails or require triggers, the entire tx reverts and the earlier ledger subtraction is rolled back — so no inconsistent state.
- **Who pays gas:** the caller (`msg.sender`) pays gas to execute `withdraw()`. The ETH transferred comes from the contract's balance.

**`emergencyWithdraw()` (owner only)**

- Sends all ETH from the contract to the owner: `payable(owner()).transfer(address(this).balance);`
- **Purpose:** rescue funds in case of critical bug or stuck funds.
- Controlled by `onlyOwner` (via `Ownable`).

**Why include it**

- **Rescue:** If the contract has a bug or becomes unusable, the owner can move funds to a safe place or to a patched contract, preventing loss from attackers or permanent lock.
- **Migration:** Allows moving funds during an upgrade.

**Risks**

- This is centralizing — the owner must be trusted. In production DeFi you usually replace single-owner with multisig or DAO governance, or timelocks to avoid immediate abuse.

### Additional notes and best practices around the contract

- **Use `call` over `transfer`:** `.transfer()` forwards only 2300 gas and may break legitimate recipient contracts. `.call{value: amount}("")` is the modern pattern; always check success.
- **Events:** In production add `Deposited` / `Withdrawn` events for observability.
- **Testing:** Write Hardhat tests that:
  - Deposit, withdraw normally.
  - Simulate a malicious receiver reentering; verify `nonReentrant` & CEI prevent draining.
  - Simulate a receiver that reverts on receive; verify tx reverts and state is safe.
- **Emergency governance:** prefer multisig + timelock over single-owner emergency withdraw in production.
- **Avoid storing too much on-chain:** keep logic minimal; move heavy ops off-chain where feasible.

## 6. Key Takeaways

- **Smart contract bugs cost real money** and are often irreversible.
- **Reentrancy:** send ETH only after updating internal state (CEI), and use `nonReentrant`.
- **Overflow/Underflow:** use Solidity `^0.8.0` or SafeMath for older versions.
- **Access control:** protect admin actions with `onlyOwner` or role-based access; prefer multisig for production admins.
- **DoS:** avoid unbounded loops and push-style payouts; use pull payments.
- **Front-running:** protect sensitive operations with commit-reveal or private submission channels.
- **Randomness:** never trust block variables for high-stakes randomness — use Chainlink VRF.
- **Users pay gas** for function execution; **contracts pay out ETH** taken from their vaults.

## 7. References

- [Ethereum Smart Contract Best Practices — ConsenSys](https://consensys.github.io/smart-contract-best-practices/)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/)
- [SWC Registry (Smart Contract Weaknesses)](https://swcregistry.io/)
- [Slither (static analysis)](https://github.com/crytic/slither)
- [Chainlink VRF Documentation](https://docs.chain.link/vrf/)
- [Echidna Property-based Testing](https://github.com/crytic/echidna)

---

**📖 Blog Series Navigation:**

← Previous: [Building a Token: ERC-20 and ERC-721 Standards](https://github.com/HsinMuShen/blog/issues/20)  
🏠 Series Home: [From Web Developer to Blockchain Engineer](https://github.com/HsinMuShen/blog/issues/11)  
→ Next: Connecting Frontend to Blockchain: Web3.js vs Ethers.js
