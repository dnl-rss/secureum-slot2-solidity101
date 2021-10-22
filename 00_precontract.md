### 4. Solidity File Layout

The *layout* of a solidity source file is important for *readability*.

A solidity file can contain an *arbitrary number* of `pragma`/`import` directives and `struct`/`enum`/`contract` definitions.

The best practice is to declare *elements within contracts* in this order:
- state variables
- events
- modifiers
- constructor
- functions

> A simple contract demonstrates the ordering of contract elements suggested above

```solidity
// SPDX-License-Identifier:MIT
pragma solidity 0.8.7;

contract OwnableContract {

    // state variables
    address owner;

    // events
    event TransferredOwnership(address newOwner);

    // modifiers
    modifier onlyOwner() {
        require( msg.sender == owner, "Caller is not the owner" );
        _;
    }

    // constructor
    constructor() {
        owner = msg.sender;
    }

    // functions
    function transferOwnership(address newOwner) public onlyOwner {
        owner = newOwner;
        emit TransferredOwnership(newOwner);
    }
}
```

Note the few lines **before** the `contract` declaration. These are also critical elements of a Solidity source file.

### 5. SPDX License Identifier

Solidity files should start with a *comment* indicating the **SPDX** (Software Package Data Exchange) license.
- The compiler will include the supplied string in the *bytecode metadata*
- It does **not validate** that the string is a valid [SPDX](https://spdx.org/licenses/) string

> Compile the example `OwnableContract` above in [Remix IDE](https://remix.ethereum.org/#optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.7+commit.e28d00a7.js), and note the SPDX line

```solidity
// SPDX-License-Identifier:MIT
```

> Examine `artifacts/ContractName_metadata.json` of the `contracts` directory to find the SPDX string `MIT`

```json
{
  "sources": {
    "contracts/OwnableContract.sol": {
      "keccak256": "0xc893d79b67c7ff1bbf602f425f3664aaeea280b202fae68ceb00258809d1a79f",
      "license": "MIT",
      "urls": [
        "bzz-raw://374a24f341ef6f0c92adbf46a27e515ee0cb675001e5698c9293ca9916a68b03",
        "dweb:/ipfs/QmTuVZffu8a22akPm8fuCbL334hVkfpqpsBVxg2dCuB2BC"
      ]
    }
  }
}  
```

Try compiling the example `OwnableContract` above in [Remix IDE](https://remix.ethereum.org/#optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.7+commit.e28d00a7.js), then find the SPDX string `MIT` in the metadata file.

### 6. Pragmas

The `pragma` keyword enables certain compiler features or checks.

This directive is *local to a source file*, and *does **not** apply to imports*.

> There are two main categories of pragmas:

1. **Version** Pragma:
  - Compiler Version
  - ABI Coder Version
2. **Experimental** Pragma:
  - SMT Checker

#### 7. Version Pragmas

The **version** `pragma` instructs the compiler to check whether its version matches the one required, or throw an `Error`.

It does **not**:
- change the compiler
- enable or disable features of the compiler

> This file must be compiled with **only** version `0.8.7`:

```solidity
pragma solidity 0.8.7;
```

For version `x.y.z`:
- while Solidity is still in early development, `x` is still `0`
- *major changes* increment `y`, and introduce breaking changes over previous versions
- *minor revisions* (bug-fixes) increment `z`

> A carrot prefix indicates that the file must be compiled with *at least* version 0.8.7, until the next major version (0.9.0):

```solidity
pragma solidity ^0.8.7;
```

> Complex pragmas combine multiple versions:

```solidity
pragma solidity >=0.8.0<0.8.3;
```

The solidity compiler version introduces various optimizations and security features, thus it is critical for evaluating security.

#### 8. ABI Coder pragmas

The **abicoder** `pragma` indicates the choice between two implementations of the ABI encoder and decoder:

```solidity
pragma abicoder v1;
```

> or

```solidity
pragma abicoder v2;
```

The new `abicoder v2`
- contains all the features of `v1`
- is enabled by *default*
- can encode and decode arbitrarily *nested arrays and structs*, although this may produce suboptimal code and is not as robustly tested as the `v1` encoder.
- can interact freely with `v1` or `v2` contracts.

`v1` contracts will throw an `Error` when decoding types only supported by `v2` (nested arrays and structs). Upgrading to `abicoder v2` will resolve the issue.

This `pragma` applies to all code in the file, regardless of where the code ends up.
- A contract compiled with `abicoder v1` may still contain code using the `v2` encoder by *inheriting* from another contract.
- This is okay as long as the `v2` types are used internally and not in external function signatures.

#### 9. Experimental pragmas

The **experimental** `pragma` enables compiler features that are *not robustly tested* and *not yet enabled by default*.

> The Satisfiability Modulo Theories (SMT) checker is an experimental feature:

```solidity
pragma experimental SMTChecker;
```

`SMTChecker` performs additional safety checks which are obtained by querying an SMT solver, and provide examples where the checks are violated:
- Satisfies all `require()` and `assert()` statements
- Arithmetic underflow and overflow
- Division by zero
- Trivial condition and unreachable
- Popping an empty array
- Out of bounds index access
- Insufficient funds for transfer

This module provides **formal verification** of contract security, and is important for an auditor.

### 10. Import statements

Import statements help to modularize code and work similarly to JavaScript

```Solidity
import filename;
```

### 11. Comments

Code comments improve readability and maintainability by providing in-line documentation of what code is expected to do in the implementation.

> Both single line comments `//` and multi-line comments `/*...*/` are supported

```Solidity
// a single line comment

/*
A multi-line
comment
*/
```

### 12. Natspec Comments

Ethereum **Natural Language Specification Format** (NatSpec) comments generate *JSON metadata* for developers and end users. All public interfaces should be fully annotated with NatSpec.

NatSpec tags include:
- `@title`
- `@author`
- `@notice`
- `@dev`
- `@param`
- `@return`
- `@inheritdoc`
- `@custom`

Both single line NatSpec comments `///` and multi-line comments `/**...*/` are supported

> NatSpec comments are used heavily in the contract below

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.8.2 < 0.9.0;

/// @title A simulator for trees
/// @author Larry A. Gardner
/// @notice You can use this contract for only the most basic simulation
/// @dev All function calls are currently implemented without side effects
/// @custom:experimental This is an experimental contract.
contract Tree {
    /// @notice Calculate tree age in years, rounded up, for live trees
    /// @dev The Alexandr N. Tetearing algorithm could increase precision
    /// @param rings The number of rings from dendrochronological sample
    /// @return Age in years, rounded up for partial years
    function age(uint256 rings) external virtual pure returns (uint256) {
        return rings + 1;
    }

    /// @notice Returns the amount of leaves the tree has.
    /// @dev Returns only a fixed number.
    function leaves() external virtual pure returns(uint256) {
        return 2;
    }
}

contract Plant {
    function leaves() external virtual pure returns(uint256) {
        return 3;
    }
}

contract KumquatTree is Tree, Plant {
    function age(uint256 rings) external override pure returns (uint256) {
        return rings + 2;
    }

    /// Return the amount of leaves that this specific kind of tree has
    /// @inheritdoc Tree
    function leaves() external override(Tree, Plant) pure returns(uint256) {
        return 3;
    }
}
```
