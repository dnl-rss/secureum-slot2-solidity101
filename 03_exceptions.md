### 86. Exceptions

Solidity uses state reverting execptions to handle errors. Such anexception undoes all changes made to the state in the current call (and sub-calls) and flags an error to the caller.

1. When exceptions occur in a sub-call, they "bubble up" (the exception is thrown in the calling function) automatically. Exceptions to this rule are `send()` and low-level functions `call()`, `delegatecall`, and `staticcall`: they return `false` instead.
2. Exceptions in external calls can be caught with the `try/catch` statement
3. Exceptions can contain data that is passed back to the caller. This data consists of a 4-byte selector and subsequent ABI encoded data. The selector is computed in the same way as a function selector (first four bytes of keccack256 hash of function signature, in this case an error signature)
4. Solidity supports two error signatures: `Error(string)` and `Panic(uint256)`. The prior is used for regular error conditions while the latter is used for errors that should not be present in bug-free code.

### 87. Account existence in `call()`, `delegatecall()`, and `staticcall()`

These low level functions return `true` as their first return value if the account called is non-existent, per the design of the EVM. Account existence must be checked prior to calling if needed.

### 88. `assert()`

The `assert()` function creates an error of type `Panic(uint256)`. Assert should only be used to test for internal errors, and to check for invariants. Properly functioning code should *never* create a Panic, not even on invalid external input.

### 89. `Panic`

A `panic` exception is generated in the following situations. The error code supplied with the error data indicated the kind of panic.

1. `0x01`: assert called with argument that evaluates to false
2. `0x01`: arithmetic operation results in underflow or overflow (except in unchecked blocks)
3. `0x12`: division (or modulo) by zero
4. `0x21`: conversion of a value too big or negative into an enum
5. `0x22`: accessing storage byte array that is incorrectly encoded
6. `0x31`: `.pop()` on empty array
7. `0x32`: accessing array, bytesN, or slice at an out-of-bounds or negative index
8. `0x41`: too much memory allocated, or array created is too large
9. `0x51`: calling zero-initialized variable of internal function type

### 90. `require`

The `require()` function either creates an error of type `Error(string)` or an error without any error data. It should be used to ensure valid conditions that cannot be detected until execution time. This includes condition on inputs or return values from calls to external contracts. You can optionally provide a message string from `require`, but not for `assert`

### 91. `Error`

An `Error(string)` exception (an exception without data) is generated in the following situations

1. Calling `require` with an argument that evaluates to `false`
2. performing an external function call, targeting a contract containing no code
3. Receiving Ether via a public function without a `payable` modifier (including `constructor()` and `fallback()`)
4. Receiving Ether via a public getter function

### 92. `revert`

A direct revert can be triggered using the `revert()` statement or function.

The revert statement takes a custom error as a direct argument without parentheses: `revert CustomError(arg1, arg2)`.

The revert function is another way to trigger exceptions from within other code blocks, flagging an error and reverting the current call. The function takes an optional string message containing details about the error that is passed back to the caller. It will create an `Error(string)` exception. Using a custom error instance will usually be much cheaper than a string description, because you can use the name of the error to describe it, encoded in only four bytes. A longer description can be supplied via NatSpec

### 93. `try/catch`

The `try` keyword must be followed by an expression representing either an external function call or a contract creation (`new ContractName()`). Errors inside the expression are not caught (for example if it is a complex expression that also involves internal function calls), only a revert happening inside the external call itself. The returns part (which is optional) that follows declares return variables matching the types returned by the external call. In case there was no error, these variables are assigned and the contract’s execution continues inside the first success block. If the end of the success block is reached, execution continues after the catch blocks.

### 94. Catch blocks

Solidity supports different kinds of catch blocks depending on the type of error
1. `catch Error(string memory reason) {...}`: This catch clause is executed if the error was caused by `revert("reasonString")` or `require(false, "reasonString")` (or an internal error that causes such an exception).
2. `catch Panic(uint errorCode) {...}`: If error was caused by a panic (failing assert, division by zero, invalid array access, arithmetic overflow, etc), this catch clause will be executed
3. `catch (bytes memory lowLevelData) {...}`: This clause is executed if the error signature does not match any other clause, an error during error message decoding, or no error data was provided with the exception. The declared variable provides access to the low-level error data in that case
4. `catch {...}`: This clause returns no data with the exception

### 95. Reversion with catch blocks

If exeution reaches a catch block, the the state changing effects of the external call have been reverted. If execution reaches the success block, the effects were not reverted.

If the effects have been reverted, then execution either continues in a catch block or the execution of the `try/catch` statement itself reverts (e.g. due to a decoding failure as noted above or due to not providing a low level catch clause).

### 96. Reasons for failed calculations

The reason behind a failed call can be manifold. Do not assume that the error message is coming directly from the called contract. The error might have happened deeper down in the call chain, forwarded by the called contract.

Also, it could be due to an out-of-gas situation and not a deliberate error condition. The caller always retains 63/64th of the gas in a call so that even if the called contract runs out of gas, the caller still has some.
