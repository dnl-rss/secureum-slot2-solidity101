## 13. Contracts

Contracts are similar to classes in OOP languages: they contain persistent data in state variables, functions that can modifiy these variables, and contracts can inherit from other contracts.

## 14. Declarations Within Contracts

Contracts contain declarations of:
- state variables
- functions
- function modifiers
- events
- errors
- struct types
- enum types

### 15. State Variables

State Variables are permanently stored in contract storage and can be accessed by all functions of a contract.

#### 16. State Visibility Specifiers

Visibility of state variables is either:
1. `public`: accessible internally or via messages; A getter function is generated.
2. `internal`: only accessible from the contract they are defined in, or contracts deriving from it.
3. `private`: only accessible from the contract they are defined in.

> Note: Marking a variable as private does not prevent others from reading the data stored on the EVM. It only prevents other contracts from programatically reading it.

#### 17. State Mutability

Variables are mutable by default, but immutable types `constant` or `immutable` are often more gas efficient than mutable types.

>The following table summarizes some of the differences between immutable types *constant* or *immutable*

|             | Constant             | Immutable          |
| ----------- | -------------------- | ------------------ |
| Value fixed | at compile time      | after construction |
| Assigned    | at declaration       |                    |
| Types       | `string` or `values` | only `values` |
| Size        | variable             | 32 bytes |
| Cannot      | access storage or blockchain data | cannot be read during construction |

The compiler does not reserve a storage slot for these variables.

#### 18. Gas Cost of State Variables

Compared to regular state variables, the gas cost of `constant` and `immutable` variables are much lower.

For constant variables, the expression assigned to it is copied to all places where it is accessed in the code and it is re-evaluated each time. This allows for local optimization.

Immutable variables are evaluated once at construction time and their value is copied to all places where it is accessed in the code. 32 bytes are reserved for these values, even if they would fit in fewer bytes. Due to this, constant variables are sometimes cheaper than immutables ones.

The only supported types for immutable variables are strings and value types. Only strings for constants.

### 19. Functions

Functions are executable units of code typically defined inside a contract, but can also be defined outside of one. They have different levels of visibility toward other contracts.

#### 20. Function parameters

Function `parameters` are declared the same way as variables, and the name of unused parameters can be omitted. Function parameters can be used as any other local variable, and can be reassigned.

#### 21. Function return variables

Function `return` variables are declared with the same syntax after the `returns` keyword. Names of return variables can be omitted. Return variables can be used as any other local variable. They are initialized with their default value and retain that value until they are reassigned. A function with return values must receive those in a return statement and assignment.

> This example shows a simple function with two input parameters and one return variable. Note that in older version of solidity, this function would be subject to arithmatic overflow.

```solidity
function addTwoNumbers(uint256 param1, uint256 param2) returns (uint256) {
  return param1 + param2;
}
```

> Alternatively, one could name the return variable and assign it in the function body

```solidity
function addTwoNumbers(uint256 param1, uint256 param2) returns (uint256 returnVar) {
  returnVar = param1 + param2;
}
```

#### 22. Function modifiers

Function `modifiers` can be used to control the behavior of a function prior to execution.  

> This simplified version of Open Zepplin's [ownable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol) standard demonstrates the use of `onlyOwner()` modifier to restrict access to the function `transferOwnership()`

```solidity
contract Ownable {

  address private _owner;

  modifier onlyOwner() {
    require(owner() == _msg.sender(), "Ownable: caller is not the owner");
    _;
  }

  // will revert if msg.sender is not this._owner
  function transferOwnership(address newOwner) public onlyOwner {
    _owner = newOwner;
  }

}
```

Multiple function modifiers are applied by specifying them in a whitespace-separated list and are evaluated in the order presented.

If the modifier prevents execution of the function body, the default values of return variables are returned.

The underscore `_;` symbol can appear in a modifier multiple times, each occurrence is replaced with the function body.

#### 23. Function visibility

The `visibility` of a contract is either:
1. `public`: included in contract interface and can be called internally or via messages
2. `external`: included in contract interface but *cannot* be called internally
3. `internal`: *not* included in contract interface; only called internally or from derived contracts
4. `private`: only called internally.

Free functions always have `internal` visibility, their code is included in all contracts that call them, similar to internal library functions

#### 24. Function mutability

The `mutability` of a function can restrict read and write access to the contract state. Modify restrictions are enforced at the EVM level via `STATICCALL` opcodes, but read restrictions are only enforced by the solidity compiler. Additionally, functions are restricted by default in their ability to send Ether; such functions must be marked `payable`.

> Solidity mutability types and privileges:

| Specifier   | Read | Modify | Send Ether |
| ----------- | ---- | ------ | ---------- |
|  `payable`  | ✅   | ✅     | ✅ |
|  none       | ✅   | ✅     | ⬜️ |
| `view`      | ✅   | ⬜️     | ⬜️ |
| `pure`      | ⬜️   | ⬜️     | ⬜️ |

Operations considered to modify the contract state include:
- Writing to state Variables
- Emitting events
- Creating other contracts
- `SELFDESTRUCT`
- Sending Ether via calls
- Calling any function not marked view or pure
- Using low-level calls
- Using inline assembly containing certain opcodes

Operations considered to read contract state include:
- Reading from state variables
- Accessing `address(this).balance` or `<address>.balance`
- Accessing any of the members of a block, tx, or msg (excluding msg.sig and msg.data)
- Calling any function not marked pure
- Using inline assembly that contains certain opcodes

#### 25. Function overloading

A contract can have multiple functions of the same name but with different parameter types, this is called "overloading." Overloaded functions are selected by matching the function declarations in the current scope to the arguments supplied in the function call. Return parameters are not taken into account.

> This overloaded function `getSum()` can either be called with two integer parameters, `getSum(1,2)` returns 3, or with three integers, `getSum(1,2,3)` returns 6 `

```solidity
// called if function receives two integer parameters
function getSum(uint a, uint b) public pure returns (uint) {
  return a + b;
}

// called if function receives three integer parameters
function getSum(uint a, uint b, uint c) public pure returns (uint) {
  return a + b + c;
}
```

#### 26. Free Functions

Functions that are defined outside of contracts are called *free functions* and always have implicit internal visibility. Their code is include in all contracts that call them, similar to an internal library function.

### 27. Events

`Events` are an abstraction of the EVM's logging functionality. Emitting events causes the arguments to be stored in the transaction's log -- a special data structure in the blockchain. These logs are associated with the contract `address` and are accessible on the blockchain as long as the block is accessible.

The `Log` and its `event` data is not accessible from within contracts (even those that create them). Applications can subscribe and listen to the events through the RPC interface of an Ethereum client.

### 28. Indexed event parameters

The optional attribute `indexed` for up two three event parameters adds them to a special data structure known as "topics" instead of the data part of the log.

Using arrays (string and bytes) as indexed arguments results in the keccak-256 hash stored as a topic instead of the array itself. This is because topics can only hold a single word (32 bytes).

All parameters without the `indexed` attribute are ABI-encoded into the `data` part of the log.

Topics allow you to search for events, i.e. filtering a sequence of blocks for certain events.

> Emitting an `event` in our `Ownable` allows applications to listen for events showing that ownership of the contract was changed. Because the event parameters are `indexed`, an application could more efficiently query the blockchain log for transfers to or from a specific address.

```solidity
contract Ownable {

  address private _owner;

  event OwnershipTransferred(address indexed previousOwner, address indexed new Owner);

  modifier onlyOwner() {
    require(owner() == _msg.sender(), "Ownable: caller is not the owner");
    _;
  }

  function transferOwnership(address newOwner) public onlyOwner {
    _owner = newOwner;

    // emit an event showing that ownership was transferred
    emit OwnershipTransferred(oldOwner, newOwner);
  }
}
```

### 29. Emit

Events are emmitted using `emit` followed by the name of the event and the arguments

> This contract will `emit` an `event` whenever a client deposits ether into the bank.

```Solidity
contract EtherBank {

  mapping(address -> uint256) balances;

  event Deposit(address sender, address amount);

  function Deposit() {
    balances[msg.sender] += msg.value
    emit Deposit(msg.sender, msg.value);
  }
}
```

### 30. Struct Types

`struct` types group several variables into a custom data structure. Struct members are access using a period `.`,

### 31. Enum Types

`enum` types create custom types with a finit set of constant values to improve readability. They may take a minimum of 1 member and a maximum of 256 members. They can be explicitly converted to/from integers. Options are represented by unsigned integer values starting from 0. Default value is the first member

> An `enum` object restricting types of fruit to `APPLE`, `BANANA`, or `PEACH`

```solidity
enum Fruit {APPLE, BANANA, PEACH}
```

> A ``struct`` defining a basket containing apples, bananas, and peaches. The basket has a maximum capacity of fruit that it can hold, and may be covered or uncovered.

``` solidity
struct FruitBasket {
  uint apples;
  uint bananas;
  uint peaches;
  uint capacity;
  bool covered;
}
```

> A `contract` utilizing the `Fruit` enum type and `FruitBasket` struct.

``` solidity
contract FruitShopping {

  FruitBasket public basket;

  function addFruit(Fruit _fruit) notFull notCovered {
    if _fruit == APPLE { basket.apples += 1 }
    else if _fruit == BANANA { basket.bananas += 1 }
    else if _fruit == PEACHES { basket.peaches += 1 }
  }

  function changeCover() {
    if (basket.cover == true) { basket.cover = false }
    else if (basket.cover == false) { basket.cover = true }
  }

  modifier notFull {
    require(basket.capacity > basket.apples + basket.bananas + basket.peaches, "Basket is full");
    _;
  }

  modifier notCovered {
    require(basket.cover == true, "Basket is covered");
    _;
  }
}
```

### 32. Constructor function

Contract creation can be triggered by an external transaction or from within a solidity contract. When a contract is created, its `constructor()` function is executed. A constructor is optional, and only one is allowed.

After the constructor has executed, the final code of the contract is stored on the blockchain. This code includes all public and external functions and all functions that are reachable from there through function calls.

The deployed code does not include the constrcutor code or internal functions that are only called from the constructor.

### 33. Receive Function

A contract may have at most one `receive()` function, which is declared without the function keyword. This function takes no arguments, does not return anything, must have `external` visibility, and is `payable`.

```Solidity
receive() external payable {}
```

`receive()` executes on calls to a contract with empty calldata. This function is executed on plain Ether transfers via `.send()` or `.transfer()`. Both `.send()` and `.transfer()` only forward `2300 gas` to `receive()`, allowing `receive()` to update the `address.balance` and leaving little room for other operations in the function body.

### 35. Fallback function

A contract can have at most one `fallback()` function, which is declared without the function keyword. This function must have external visibility. It may or may not be payable.

```solidity
fallback() external payable {}
```

`fallback()` is executed on calls to a contract containing calldata, but none of the other functions match the given function signature. It is also called when there is no `msg.data` and `receive()` is not declared.

Like `receive()`,`fallback()` may only rely on 2300 gas available from `send()` or `transfer()`.
