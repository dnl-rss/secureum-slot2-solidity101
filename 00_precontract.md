### 4. Layout of a solidity file

A solidity source file can contain an arbitrary number of pragma directions, import directives, and struct/enum/contract definitions.

Best practices is to declare within contracts in this order:
- state variables
- events
- modifiers
- constructor
- functions

However, a few items must be addressed before any contract is declared:

### 5. SPDX License Identifier

>Solidity files should start with a comment indicating the license. The compiler includes the supplied string in bytecode metadata.

```solidity
// SPDX-License-Identifier:MIT
```
### 6. Pragmas

The `pragma` keyword enables certain compiler features or checks. This directive is local to a source file, and does not apply to imports.

> There are two types of pragmas:

1. Version Pragma:
  - Compiler Version
  - ABI Coder Version
2. Experimental Pragma:
  - SMT Checker

#### 7. Version Pragmas

The version `pragma` does not change the solidity compiler, nor does it enable or disable features of the compiler. It instructs the compiler to check whether its version matches the one required by the pragma, or throw an error.

> This statement requires that the file be compiled with a specific version:

```solidity
pragma solidity x.y.z;
```

The latest compiler versions are in the `0.8.z` range. A different `y` in `x.y.z` indicates a breaking change in the compiler version, whereas a different `z` indicates bug fixes.

> A carrot prefix indicates that the file must be compiled with *at least* version `x.y.z`, until the next `y` version:

```solidity
pragma solidity ^x.y.z;
```

> Complex pragmas combine multiple versions:

```solidity
pragma solidity >=0.8.0<0.8.3;
```

#### 8. ABI Coder pragmas

The `abicoder` pragma indicates the choice between two implementations of the ABI encoder and decoder

> Either

```solidity
pragma abicoder v1;
```

> or

```solidity
pragma abicoder v2;
```

The new `abicoder v2` is enabled by default. It can encode and decode arbitrarily nested arrays and structs, although this may produce suboptimal code and is not as robustly tested as the v1 encoder.

An `abicoder v2` contract can interact freely with `abicoder v1` or `v2` contracts.

An `abicoder v1` contract will throw an error if it attempts to decode types only supported by the new compiler. Upgrading the abicoder version will resolve the issue.

This `pragma` applies to all code in the file, regardless of where the code ends up. A contract compiled with abicoder v1 can still contain code using the new encoder by inheriting from another contract. This is okay as long as the new types are used internally and not in external fucntion signatures.

#### 9. Experimental pragmas

This `pragma` enables experimental compiler features that are not robustly tested and not yet enabled by default.

> The only experimental pragma at the time of writing is that of a Satisfiability Modulo Theories (SMT) checker:

```solidity
pragma experimental SMTChecker;
```

`SMTChecker` performs additional safety checks which are obtained by querying an SMT solver, and provide examples where the checks are violated:
- Satisfies all require and assert statements
- Arithmetic underflow and overflow
- Division by zero
- Trivial condition and unreachable
- Popping an empty array
- Out of bounds index access
- Insufficient funds for transfer

### 10. Import statements

Import statements help to modularize code and work similarly to JavaScript

```Solidity
import filename;
```

### 11. Comments

Code comments improve readability and maintainability by providing in-line documentation of what contracts, functions , variables, expressions, control, and data flow are expected to do in the implementation.

> Both single line comments (//) and multi-line comments (/*...*/) are supported

```Solidity
// a single line comment

/*
A
multi-
line
comment
*/
```

### 12. Natspec Comments

Ethereum Natural Language Speficication Format (NatSpec) comments generate JSON format metadata for developers and end users. All public interfaces should be fully annotated

> Both single line NatSpec comments (///) and multi-line comments (/**...*/) are supported, though only single line comments are shown below:

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
