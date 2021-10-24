### 86. Exceptions

Solidity uses *state reverting execptions* to handle errors.

Such an exception *undoes all changes made to the state* in the current call (and sub-calls), and flags an `Error` to the caller.

1. Exceptions in a sub-call usually *bubble up* (the exception is rethrown in the calling function) automatically.
    - The exceptions to this rule are `send()` and low-level functions `call()`, `delegatecall()`, and `staticcall()`: they return `false` instead.
2. Exceptions in external calls can be caught with the `try/catch` statement.
3. Exceptions can contain data that is passed back to the caller.
    - This data consists of a 4-byte selector and subsequent ABI encoded data.
    - The selector is computed in the same way as a function selector (first four bytes of `keccak256` hash of function signature, in this case an error signature)
4. Solidity supports two error signatures: `Error(string)` and `Panic(uint256)`.
    - `Error(string)` is used for *regular error conditions*
    - `Panic(uinnt256)` is used for *errors that should not be present in bug-free code*.

### 87. Account existence in call, delegatecall, and staticcall

The low level functions `call()`, `delegatecall()`, and `staticcall()` return `true` as their first return value if the account called is non-existent, per the design of the EVM. Account existence must be checked *prior to calling* if needed.

### 88. assert

- The `assert()` function creates an error of type `Panic(uint256)`.
- `assert()` should only be used to test for *internal errors*, and to check *invariants*.
- Properly functioning code should *never* create a `Panic`
- Use `require()`, not `assert()`, to check invalid *external input*.

### 89. Panic

A `panic` exception is generated in several situations.

> The error code supplied with the error data indicated the kind of panic:

1. `0x01`: `assert` called with argument that evaluates to `false`
2. `0x01`: arithmetic operation results in underflow or overflow (except in `unchecked` blocks)
3. `0x12`: division (or modulo) by `0`
4. `0x21`: conversion of a value too big or negative into an `enum`
5. `0x22`: accessing `storage` byte array that is incorrectly encoded
6. `0x31`: `pop` on empty array
7. `0x32`: accessing array, `bytesN`, or slice at an out-of-bounds or negative index
8. `0x41`: too much `memory` allocated, or array created is too large
9. `0x51`: calling zero-initialized variable of `internal` function type

### 90. require

- The `require()` function either creates an error of type either `Error()` or `Error(string)`.
- `require()` should be used to *detect valid conditions* during runtime.
- This includes checks on *inputs* or *return values* from calls to external contracts.
- A message `string` is optionally provided to `require()`, but not for `assert()`, this encourages the use of `require()` during runtime.

### 91. Error

An `Error(string)` exception (an exception without data) is generated in the following situations:

1. Calling `require(condition)` with a `condition` that evaluates to `false`
2. Performing an *external function call* where the called contract *contains no code*
3. Receiving Ether via a `public` function **not marked** `payable` (including `constructor()` and `fallback()`)
4. Receiving Ether via a `public` getter function

### 92. revert

There are two methods to trigger a direct state reversion without a check condition: `revert([string])` or `revert CustomError(art1, arg2)`
- `revert CustomError(arg1, arg2)` statement takes a custom error argument without parentheses.
- `revert([string])` function takes an optional error `string` passed back to the caller as an `Error(string)` exception.
-  Using  `revert CustomError()` will usually be more gas efficient than `revert(string)`, because the name of the `ErrorType` can describe the error with only four bytes. A longer description can be supplied via `NatSpec`.

### 93. try/catch

Try/catch statements are used to handle errors triggered by external calls.

- The `try` keyword must be followed by an `Expression` representing either an *external function call* or a *contract creation* (`new ContractName()`).
- The optional `returns()` statement declares return variables matching the types returned by `Expression`.
- If `Expression` triggers no error, the return variables are assigned, the success block is executed, and the catch blocks are ignored.
- If `Expression` triggers an error, one of the `catch` blocks is executed based on `ErrorType` received.

> Generalized syntax of try/catch

```solidity
try Expression [returns()] {
  // success block
  // ...
}
catch ErrorType1 {
  // failure block1
  // ...
}
catch ErrorType2 {
  // failure block2
  // ...
}
```

### 94. Catch blocks

Solidity supports different kinds of `catch` blocks depending on the type of error:
1. `catch Error(string memory reason) {...}`: This catch clause is executed if the error was caused by `revert("reasonString")` or `require(false, "reasonString")` (or an internal error that causes such an exception).
2. `catch Panic(uint errorCode) {...}`: If error was caused by a panic (failing assert, division by zero, invalid array access, arithmetic overflow, etc), this catch clause will be executed
3. `catch (bytes memory lowLevelData) {...}`: This clause is executed if the error signature does not match any other clause, an error during error message decoding, or no error data was provided with the exception. The declared variable provides access to the low-level error data in that case
4. `catch {...}`: This clause returns no data with the exception

### 95. try/catch state changes

- If a catch block is executed, then the state changing effects of the external call have been reverted.
- If a the success block is executed, the effects were not reverted.

If the effects of the call have been reverted, then execution either:
- Continues in a catch block
- Execution of the `try/catch` statement itself reverts (e.g. due to an error decoding failure as noted above or due to not providing a low level catch clause).

### 96. Reasons for failed calculations

The reason behind a failed call can be manifold:
- Do not assume that the error message is coming directly from the called contract.
- The error might have happened *deeper down in the call chain*, forwarded by the called contract.
- It could also be due to an *out-of-gas* situation and not a deliberate error condition. The caller always retains *63/64th* of the gas in a call so that even if the called contract runs out of gas, the caller still has some.
