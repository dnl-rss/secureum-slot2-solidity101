> Markdown formatted notes with example solidity code for the Secureum security bootcamp, slot 2, topic: [solidity-101](https://secureum.substack.com/p/solidity-101)

## 1. Solidity 101

Soldity is a high-level language for implementing smart contracts on Ethereum and other EVM blockchains. It was founded in 2014 by Gavin Wood and developed by Christian Reitwiessner, Alex Beregszazi, and others

> Hello World in Solidity

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.3;

contract HelloWorld {
    string public greet = "Hello World!";
}
```

### 2. Solidity influences

Solidity was influenced by:
- C++ (OOP & Syntax)
- Python (modifiers, multiple inheritance, C3 linearization, & super keywords)
- JavaScript (function level scoping & `var` keyword)

Javascript influences have been reduced since `v0.4.0`

Solidity is the most widely used smart contract language, thus it is critical for understanding contract security.

### 3. General features

Solidity is:
- A "curly bracket" language: curly brackets `{}` are used to group statements within a scoping
- object oriented (supports inheritance, libraries, and user-defined types)
- statically typed

### Sub-Chapters

1. [Precontract Elements](./00_precontract.md)
1. [Contract Elements](./01_contract.md)
2. [Typing](./02_types.md)
3. [Exceptions](./03_exceptions.md)
4. [Style](./04_style.md)
