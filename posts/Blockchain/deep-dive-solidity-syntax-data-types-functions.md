# Deep Dive into Solidity: Syntax, Data Types & Functions

```
          /\_/\              💻 Solidity Code    →    🔍 Data Types    →    ⚙️ Functions
         ( •.• )           (Smart Contract)          (uint, string)         (view, pure)
        / >💡< \           |                        |                      |
                           └─ State Variables       └─ address, mapping    └─ payable
                           └─ Contract { }          └─ Storage vs Memory   └─ Modifiers
```

## Table of Contents

1. [Introduction](#1-introduction)
2. [Solidity Syntax Basics](#2-solidity-syntax-basics)
3. [State Variables and Visibility](#3-state-variables-and-visibility)
4. [Data Types in Solidity](#4-data-types-in-solidity)
   - [4.1 Integers](#41-integers)
   - [4.2 Boolean](#42-boolean)
   - [4.3 String](#43-string)
   - [4.4 Address](#44-address)
   - [4.5 Arrays](#45-arrays)
   - [4.6 Mapping](#46-mapping)
5. [Functions in Solidity](#5-functions-in-solidity)
   - [5.1 Basic Function](#51-basic-function)
   - [5.2 View vs Pure vs Payable](#52-view-vs-pure-vs-payable)
6. [The msg Global Variable](#6-the-msg-global-variable)
   - [6.1 Common Properties](#61-common-properties)
7. [Memory vs Storage](#7-memory-vs-storage)
8. [Example: Bank Contract](#8-example-bank-contract)
9. [Key Takeaways](#9-key-takeaways)
10. [References](#10-references)

---

## 1. Introduction

In the last post, we deployed our first smart contract — a simple HelloWorld program. Now it's time to go deeper into Solidity, the primary language for writing Ethereum smart contracts.

Just like JavaScript, Python, or Rust, Solidity has:

- **Variables** (store data on-chain)
- **Data types** (integers, strings, addresses, arrays, mappings)
- **Functions** (to read or update state)
- **Visibility & modifiers** (control who can access functions)

But because Solidity runs on the Ethereum Virtual Machine (EVM), it also comes with blockchain-specific concepts:

- **Storage vs memory**
- **Gas costs** for different operations
- **Special types** like `address` and `mapping`

By the end of this post, you'll be comfortable with the building blocks of Solidity programs.

## 2. Solidity Syntax Basics

Every Solidity file starts with:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;
```

- **SPDX license** → declares how your code can be reused.
- **pragma solidity** → compiler version (always lock your version).

Then comes the `contract` keyword:

```solidity
contract MyContract {
    // code here
}
```

This is like a class in other languages — it groups state variables and functions together.

## 3. State Variables and Visibility

A state variable is data stored on-chain. Example:

```solidity
contract Example {
    uint public count = 0;   // public state variable
    string private name;     // private state variable
}
```

- **`public`** → generates a free getter (`count()` is callable).
- **`private`** → only functions inside this contract can use it.

Other options:
- **`internal`** (like protected in OOP).
- **`external`** (only callable from outside).

## 4. Data Types in Solidity

### 4.1 Integers

```solidity
uint public a = 10;   // unsigned integer (0 or positive)
int public b = -5;    // signed integer
```

You can also choose sizes: `uint8`, `uint256` (default).

### 4.2 Boolean

```solidity
bool public isActive = true;
```

### 4.3 String

```solidity
string public greeting = "Hello Solidity!";
```

### 4.4 Address

Special type for Ethereum addresses:

```solidity
address public owner = msg.sender;  // deployer's address
```

Useful for sending/receiving ETH and access control.

### 4.5 Arrays

```solidity
uint[] public numbers;       // dynamic array
uint[3] public fixedNums;    // fixed-size array
```

### 4.6 Mapping

Like a dictionary or hash map:

```solidity
mapping(address => uint) public balances;
```

## 5. Functions in Solidity

### 5.1 Basic Function

```solidity
function add(uint x, uint y) public pure returns (uint) {
    return x + y;
}
```

- **`public`** → callable by anyone.
- **`pure`** → doesn't read or modify state.
- **`returns (uint)`** → specifies return type.

### 5.2 View vs Pure vs Payable

**`view`** → can read state, but not modify it.

```solidity
function getBalance() public view returns (uint) {
    return balances[msg.sender];
}
```

**`pure`** → can't read or write state. Pure math.

```solidity
function double(uint x) public pure returns (uint) {
    return x * 2;
}
```

**`payable`** → allows the function to receive ETH.

```solidity
function deposit() public payable {
    balances[msg.sender] += msg.value;
}
```

## 6. The msg Global Variable

In Solidity, `msg` is a special global variable automatically provided by the Ethereum Virtual Machine (EVM) for every transaction or call.

It carries metadata about the current function call.

### 6.1 Common Properties

**`msg.sender`**

The address that called the function.

Example:

```solidity
address public owner;
constructor() {
    owner = msg.sender; // whoever deploys the contract becomes the owner
}
```

**`msg.value`**

The amount of ETH (in wei) sent with the transaction.

Only works inside `payable` functions.

Example:

```solidity
function deposit() public payable {
    balances[msg.sender] += msg.value;
}
```

**`msg.data`**

The raw input data for the function call.

Rarely used directly, but important for low-level programming.

**`msg.sig`**

The first 4 bytes of `msg.data` — used to identify which function is being called.

📬 **Analogy**

Imagine you're mailing a letter to a smart contract:

- `msg.sender` = your return address.
- `msg.value` = the amount of money you included.
- `msg.data` = the content of the letter.
- `msg.sig` = the code on the envelope that says "open with function X."

✅ **Without `msg`, contracts wouldn't know who is calling them or how much ETH was sent.**

## 7. Memory vs Storage

Solidity has two main "locations" for variables:

- **Storage** → permanent, written to blockchain (costs gas).
- **Memory** → temporary, only exists during function execution.

Example:

```solidity
function setName(string memory _name) public {
    name = _name;   // writes to storage
}
```

You must declare whether a string/array is in `memory` or `storage`.

## 8. Example: Bank Contract

Let's combine everything into a simple Bank contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

contract Bank {
    mapping(address => uint) public balances;

    // Deposit ETH
    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    // Check balance
    function getBalance() public view returns (uint) {
        return balances[msg.sender];
    }

    // Withdraw ETH
    function withdraw(uint amount) public {
        require(balances[msg.sender] >= amount, "Not enough balance");
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }
}
```

## 9. Key Takeaways

- **Solidity contracts are like classes**, with state variables and functions.
- **Data types** include `uint`, `bool`, `string`, `address`, arrays, and mappings.
- **Function types:**
  - `view` (read only)
  - `pure` (math only)
  - `payable` (can receive ETH)
- **The `msg` global variable** provides transaction metadata like `msg.sender` and `msg.value`.
- **Storage vs memory matters**: storage is permanent (gas-heavy), memory is temporary.

## 10. References

- [Solidity Documentation: Types](https://docs.soliditylang.org/en/stable/types.html)
- [Solidity by Example](https://solidity-by-example.org/)
- [Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/)

---

**📖 Blog Series Navigation:**

← Previous: [Intro to Smart Contracts: The "Hello World" of Blockchain](https://github.com/HsinMuShen/blog/issues/16)  
🏠 Series Home: [From Web Developer to Blockchain Engineer](https://github.com/HsinMuShen/blog/issues/11)  
→ Next: Building a Token: ERC-20 and ERC-721 Standards
