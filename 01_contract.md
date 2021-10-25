## 13. Contracts

Smart contracts are fundamental to Ethereum's utility as a **smart contract platform**.

Contracts are similar to *classes* in object-oriented programming:
- *store persistent data* in state variables
- *define functions* that may modify these variables
- can *inherit from* and *interact with* other contracts

> State variables, functions, and contract inheritance/interactions are demonstrated by the contracts below:

```solidity
// SPDX-License-Identifier:any string representing the license
pragma solidity 0.8.7;

contract CounterContract {

    // store persistent data in state variables
    uint internal counter;

    // define functions that may modify state variables
    function incrementCounter() internal {
        counter += 1;
    }  
}

// can inherit from other contracts
contract MaxCapacity is CounterContract {

    uint maxCapacity;

    constructor(uint capacity) {
        maxCapacity = capacity;
    }

    function safeIncrement() public {
        require ( counter < maxCapacity , "Cannot exceed capacity!");    
        incrementCounter();
    }

    function getCounter() public view returns (uint) {
        return counter;
    }
}

// can interact with ather contracts
contract Caller {

    MaxCapacity contract_instance = new MaxCapacity(3);

    function callSafeIncrement() public {
        contract_instance.safeIncrement();
    }

    function getCounter() public view returns (uint) {
        return contract_instance.getCounter();
    }
}
```
## 14. Declarations Within Contracts

Contracts contain declarations of:
- custom data types: `struct` and `enum`
- state variables
- events
- errors
- modifiers
- constructor
- functions
- contracts: as `contract`, `library`, or `interface`

### 15. State Variables

*State variables* are permanently stored in contract `storage`.

They can be accessed and/or modified by the functions of a contract.

They persist between transactions which may or may not modify the contract state.

> The value "counter" is a state variable, which persists between transactions, and is modified by calls to function incrementCounter()

```solidity
// SPDX-License-Identifier:any string representing the license
pragma solidity 0.8.7;

contract CounterContract {

    // store persistent data in state variables in contract storage
    uint public counter;

    // define functions that may modify state variables
    function incrementCounter() public {
        counter += 1;
    }  
}
```

#### 16. State Visibility Specifiers

The *visibility* of a state variable determines where its value can be read.
1. `public`: accessible internally or via messages; A getter function is generated.
2. `internal`: only accessible from the contract they are defined in, or contracts deriving from it.
3. `private`: only accessible from the contract they are defined in.

> The effects of state variable visibility on contract inheritance is demonstrated below:

```solidity
// SPDX-License-Identifier:any string representing the license
pragma solidity 0.8.7;

contract VarVis {

    // getter function is generated for external contracts to read "aPublicVar"
    // derived contracts iherhit "aPublicVar" and may read/modify directly
    uint public aPublicVar;

    // no getter function generated
    // derived contracts inherit "anInternalVar" and may read/modify directly
    uint internal anInternalVar;

    // only accessible in this contract
    uint private aPrivateVar;
}

contract Demo is VarVis {

    // these functions attempt to modify inherited public, internal, and private variables

    function incrementPublicVar() public {
        // this works
        aPublicVar += 1;
    }

    function incrementInternalVar() public {
        // this works
        anInternalVar += 1;
    }

    function incrementPrivateVar() public {
        // does not compile: DeclarationError: undeclared identifier
        // aPrivateVar += 1;
    }
}
```

> WARNING: Marking a variable as `private` does not prevent others from reading the data stored on the EVM. It only prevents other contracts reading it.

#### 17. State Mutability

Variables are mutable by default, but immutable types `constant` or `immutable` are often more gas efficient than mutable types.

>The following table summarizes some of the differences between immutable types *constant* or *immutable*

|             | Constant             | Immutable          |
| ----------- | -------------------- | ------------------ |
| Value fixed | at compile time      | after construction |
| Assigned    | at declaration       |                    |
| Types       | `string` or values types | only values types |
| Size        | variable             | 32 bytes |
| Cannot      | access storage or blockchain data | be read during construction |
| Compiler copies | the expession | the value |

The compiler does not reserve a storage slot for these variables.

> Usage of `constant` and `immutable` variable types is demonstrated in the contract below:

```solidity
// SPDX-License-Identifier:any string representing the license
pragma solidity 0.8.7;

contract ConstantImmutable {

    // both of these assignments work
    uint constant CONSTANT_VAR = 5 + 7;
    uint immutable IMMUTABLE_VAR = 4 + 3;

    // constant must be assigned to at declaration
    // uint constant UNASSIGNED_CONSTANT;

    // constant may not read blockchain state
    // uint constant CONSTANT_VAR2 = block.number;
    // TypeError: Initial value for constant vairlable has to be compile-time constant

    // immutable may be initialized here and assigned in constructor
    uint immutable IMMUTABLE_VAR2;

    constructor() {

        // immutable may read blockchain state
        IMMUTABLE_VAR2 = block.number;
    }


    function multiplyByConstant(uint x) public pure returns (uint) {

        // the compiler reevaluates the expression for CONSTANT_VAR (5 +7) here
        return x * CONSTANT_VAR;
    }

    function multiplyByImmutable(uint x) public pure returns (uint) {

        // the compiler copy and pastes the value for IMMUTABLE_VAR (7) here
        return x * IMMUTABLE_VAR;
    }

    function multiplyByImmutable(uint x) public view returns (uint) {

        // the compiler copy and pastes the value for IMMUTABLE_VAR (current block number) here
        return x * IMMUTABLE_VAR;
    }

    function modifyConstant() public {

        // reassignment fails
        // CONSTANT_VAR = 2;
        // TypeError: Cannot assign to a constant variable
    }

    function modifyImmutable() public {

        // reassignment fails
        // IMMUTABLE_VAR = 2;
        // TypeError: cannot write to immutable here:
        // Immutable variables can only be initialized inline and assigned directly in the constructor
    }
}
```

#### 18. Gas Cost of State Variables

Compared to regular state variables, the gas costs of `constant` and `immutable` variables are much lower.
- For `constant` variables, the assigned *expression is copied* to all places where it is accessed in the code and it is *evaluated multiple times*. This allows for local optimization.
- For `immutable` variables, the assigned *expression is evaluated once* at construction time and its *value is copied* to all places where it is accessed in the code. 32 bytes are reserved for these values, even if they would fit in fewer bytes. Due to this, constant variables are sometimes cheaper than immutables ones.

> `constant` variables support both string and value type. `immutable` only supports value types.

```solidity
// SPDX-License-Identifier:any string representing the license
pragma solidity 0.8.7;

contract ConstantImmutable2 {

    // immutable does NOT allow string types
    // string immutable IMMUTABLE_VAR = "a string";
    // TypeError: Immutable variables cannot have a non-value type

    // constant does allow string types
    string constant CONSTANT_VAR = "a string";

}
```

### 19. Functions

Functions are *executable units of code* typically defined *inside a contract*, but can also be defined outside of one.

They have different levels of *visibility* toward other contracts.

```solidity
// a "free" function is defined outside of a contract
function freeFunction() {
    // executable code defined here    
}

// usually functions are defined within contracts
contract FunctionsDemo {

    function contractFunction() {
        // executable code defined here
    }
}
```

#### 20. Function parameters

Function *parameters* are declared the same way as *variables*, and the name of unused parameters can be omitted.

Function parameters can be used as any other local variable, and can be reassigned.

```Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract FunctionParametersDemo {

    uint public sum;

    function writeSum(uint param1, uint param2) public {
        // function call reverts if param1 or param2 is not uint type
        sum = param1 + param2;
    }

    function overrideString(string memory param1) public {
        // parameters can be used as any other local variables,
        // including assignments
        // however, param1 only persists for the lifetime of this function call
        param1 = "override";
    }
}
```

> TODO: "name of unused parameters can be omitted" ... include example

#### 21. Function return variables

Function *return variables* are declared with the same syntax as parameters in the function declaration, after the `returns` keyword.
- Names of return variables can be omitted.
- Return variables are zero-intialized at the start of the function and retain that value until reassigned.
- May be used as any other local variable.
- The return value may be set either by a `return` statement, or assigning to the name of the return variable.

> Here, the return parameter name is omitted. Its type is `uint`:

```solidity
function addTwoNumbers(uint param1, uint param2) returns (uint) {
  return param1 + param2;
}
```

> Alternatively, here the return parameter is named `returnVar` which is assigned to in the function body

```solidity
function addTwoNumbers(uint256 param1, uint256 param2) returns (uint256 returnVar) {
  returnVar = param1 + param2;
}
```

#### 22. Function modifiers

A function `modifier` can be used to control the behavior of a function prior to execution.  
- If the modifier prevents execution of the function body, the default values of return variables are returned
- The underscore `_;` symbol can appear multiple times in a modifier; each occurrence is replaced with the function body
- Multiple function modifiers are applied by specifying them in a whitespace separated list and are evaluated in order

> This contract restricts access of the function `transferOwnership()` to its current `owner` using the modifier `onlyOwner`

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract Ownable {

  address private owner;

  modifier onlyOwner() {
    require(owner == msg.sender, "Ownable: caller is not the owner");
    _;
  }

  // will revert if msg.sender is not owner
  function transferOwnership(address newOwner) public onlyOwner {
    owner = newOwner;
  }

}
```

#### 23. Function visibility

The *visibility* of a contract is either:
1. `public`: included in contract interface and can be called internally or via messages
2. `external`: included in contract interface but *cannot* be called internally
3. `internal`: *not* included in contract interface; only called internally or from derived contracts
4. `private`: only called internally.

> The visibility of a contract determines how it can be called:

| | current contract | derived contracts | interface messages |
|-|-|-|-|
| `public`   | ✅ | ✅ | ✅ |
| `external` | ⬜️ | ⬜️ | ✅ |
| `internal` | ✅ | ✅ | ⬜️ |
| `private`  | ✅ | ⬜️ | ⬜️ |

Free functions always have `internal` visibility, their code is included in all contracts that call them, similar to internal library functions

> To demonstrate function visibility, here is a contract can set the value of `setToHelloWorld` to "Hello World" via functions of each type.

```Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract FunctionVisibility {

    string public setToHelloWorld;

    function publicFunction() public {
        // callable by external contracts, derived contracts, and functions in this contract
        setToHelloWorld = "Hello World";
    }
    function externalFunction() external {
       // callable by external contracts
        setToHelloWorld = "Hello World";
    }
    function internalFunction() internal {
        // callable by derived contracts and functions in this contract
        setToHelloWorld = "Hello World";
    }
    function privateFunction() private {
        // callable by only functions in this contract
        setToHelloWorld = "Hello World";
    }
    function reset() public {
        setToHelloWorld = "";
    }
}
```

> A `DerivedContract` may set the value of the string by diretly calling the `public` and `internal` functions. `external` and `private` functions are not inherited.

```solidity
contract DerivedContract is FunctionVisibility {

    function callPublic() public {
        // This succeeds via inheritance of public functions
        publicFunction();
    }

    function callExternal() public {
        // This fails. External functions are not inherited by derived contracts
        // externalFunction();
        // DeclarationError: Undeclared identifier
    }

    function callInternal() public {
        // This succeeds via inheritance of internal functions
        internalFunction();
    }

    function callPrivate() public {
        // This fails. Private functions are not inherited by derived contracts
        // privateFunction();
        // DeclarationError: Undeclared identifier
    }
}
```

> An `ExternalContract` set the value of the string via `public` and `external` method calls included in the contract interface. `internal` and `private` functions are not included in the interface.

```solidity
contract ExternalContract {

    FunctionVisibility external_contract = new FunctionVisibility();

    function callPublic() public {
        // This succeeds. Public functions are included in contract interface
        external_contract.publicFunction();
    }

    function callExternal() public {
        // This succeeds. External functions are included in contract interface
        external_contract.externalFunction();
    }

    function callInternal() public {
        // This fails. Internal functions are not included in contract interface
        // external_contract.internalFunction();
        // TypeError: internalFunction not found or not visible after argument-dependent lookup in contract FunctionVisibility
    }

    function callPrivate() public {
        // This fails. Private functions are not included in contract interface
        // external_contract.privateFunction();
        // TypeError: member "privateFunction" not found or not visibile after argement-dependent lookup in contract FunctionVisibility
    }

    function callGetter() public returns (string memory) {
        return external_contract.setToHelloWorld();
    }
}
```

#### 24. Function mutability

The *mutability* of a function can *restrict read and write* access to the contract state.
- *Write restrictions* are enforced at the EVM level via `STATICCALL` opcodes
- *Read restrictions* are only enforced by the solidity compiler
- Functions are restricted by default in their *ability to send/receive Ether*; such functions must be marked `payable`.

> Solidity mutability types and privileges:

| Specifier   | Read | Modify | Send Ether |
| ----------- | ---- | ------ | ---------- |
|  `payable`  | ✅   | ✅     | ✅ |
|  none       | ✅   | ✅     | ⬜️ |
| `view`      | ✅   | ⬜️     | ⬜️ |
| `pure`      | ⬜️   | ⬜️     | ⬜️ |

`pure` functions can neither read nor write to the contract state.

Operations considered to modify the contract state include:
- Writing to state variables
- Emitting events
- Creating other contracts
- Calling `SELFDESTRUCT`
- Sending Ether via calls
- Calling any function not marked `view` or `pure`
- Using low-level calls
- Using inline assembly containing certain opcodes

Operations considered to read contract state include:
- Reading from state variables
- Accessing `address(this).balance` or another `<address>.balance`
- Accessing any of the members of a `block`, `tx`, or `msg` (excluding `msg.sig` and `msg.data`)
- Calling any function not marked `pure`
- Using inline assembly that contains certain opcodes

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract FunctionMutability {

    string public setToHelloWorld;

    function writeFunction() public {
        // this succeeds. functions with no mutability specifier may read and write contract storage
        setToHelloWorld = "Hello World";
    }

    function viewFunction() public view returns (string memory) {
       // this fails. view functions may  not modify contract storage
       // setToHelloWorld = "Hello World";
       // TypeError: function declared as view, but this expression modifies the state

       // this succeeds. view functions may read contract storage
       return setToHelloWorld;
    }

    function pureFunction() public pure returns (string memory) {
        // this fails. pure functions may neither read nor write contract storage
        // return setToHelloWorld;
        // TypeError: Function declared as pure, but this expression reads from the environment or state and thus requires "view"

        // this succeeds. does not read contract storage.
        return "Hello World"

    }

    function payableFunction() public payable {
        // functions intented to accept ether (via msg.value) must be marked payable, otherwise they will revert
        // in this case, the caller pays 1 ether to set the value of the string
        require( msg.value = 1 ether );
        setToHelloWorld = "Hello World";
    }

    function reset() public {
        setToHelloWorld = "";
    }
}
```

#### 25. Function overloading

A contract can have multiple functions of the same name but with different parameter types, this is called "overloading."

Overloaded functions are selected by matching the function *call arguments* to the *declaration arguments* in the current scope.

Return parameters are not taken into account.

> This overloaded function `getSum()` can either be called with two or three integer parameters: `getSum(1,2)` or `getSum(1,2,3)`

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

Functions that are defined outside of contracts are called *free functions* and always have implicit `internal` visibility.

Their code is included in all contracts that call them, similar to an internal library function.

### 27. Events

*Events* are an abstraction of the EVM's logging functionality.
- Emitting events stores the argements in the transaction's **log** (a special data structure in the blockchain)
- Applications can subscribe and listen to these events through the RPC interface of an Ethereum client.
- Logs are associated with the contract `address` and are accessible on the blockchain for as long as the block is accessible.
- Logs and `event` data are **not** accessible from *within contracts*

> An application can listen for `transferOwnership` events from this contract address

```solidity
contract Ownable {

  address private owner;

  event OwnershipTransferred(address previousOwner, address newOwner);

  modifier onlyOwner() {
    require(owner == _msg.sender, "Ownable: caller is not the owner");
    _;
  }

  function transferOwnership(address newOwner) public onlyOwner {
    owner = newOwner;

    // emit an event showing that ownership was transferred
    emit OwnershipTransferred(oldOwner, newOwner);
  }
}
```



### 28. Indexed event parameters

The optional attribute `indexed` for up to three `event` parameters adds them to a special data structure known as "topics" instead of the data part of the log.

Using arrays (string and bytes) as indexed arguments results in the `keccak256` hash stored as a topic instead of the array itself. This is because topics can only hold a single word (32 bytes).

All parameters without the `indexed` attribute are ABI-encoded into the `data` part of the log.

Topics allow you to search for events, i.e. filtering a sequence of blocks for certain events.

> A `Deposit` event is emitted by this contract

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.21 <0.9.0;

contract ClientReceipt {
    event Deposit(address indexed from, bytes32 indexed id, uint value);

    function deposit(bytes32 id) public payable {
        emit Deposit(msg.sender, id, msg.value);
    }
}
```

> The event can be detected in the Javascript API

```javascript
var abi = /* abi as generated by the compiler */;
var ClientReceipt = web3.eth.contract(abi);
var clientReceipt = ClientReceipt.at("0x1234...ab67" /* address */);

var depositEvent = clientReceipt.Deposit();

// watch for changes in the contract
depositEvent.watch(function(error, result){
    // result contains non-indexed arguments and topics
    // given to the `Deposit` call.
    if (!error)
        console.log(result);
});


// Or pass a callback to start watching immediately
var depositEvent = clientReceipt.Deposit(function(error, result) {
    if (!error)
        console.log(result);
});
```

> The output of the above looks like this:

```JSON
{
   "returnValues": {
       "from": "0x1111…FFFFCCCC",
       "id": "0x50…sd5adb20",
       "value": "0x420042"
   },
   "raw": {
       "data": "0x7f…91385",
       "topics": ["0xfd4…b4ead7", "0x7f…1a91385"]
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

`struct` types group several variables into a custom data structure.

Struct members are access using a period `.`.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.3;

contract TodoList {

    // declare struct type, with string containing todo item and bool indicating completion
    struct Todo {
        string text;
        bool completed;
    }

    // declare an array of todo structs
    Todo[] public todos;

    // create a struct item and add to array
    function addItem(string memory text) public {
        todos.push( Todo(text, false));
    }

    // complete an item in list by index
    function completeItem(uint index) public {
        todos[index].completed = true;
    }
}
```

### 31. Enum Types

`enum` types create custom types with a finite set of constant values to improve readability.
- May take a minimum of 1 member and a maximum of 256 members.
- Explicitly convertible to/from integers.
- Options are represented by unsigned integer values starting from 0.
- Default value is the first member

> Demonstration of emum types. Note that the values returned for `status` are actually of type `uint8`: {0:Pending, 1:Shipped, ..., 4:Canceled}

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.3;

contract EnumDemo {

    // enum representing shipping status
    enum Status { Pending, Shipped, Accepted, Rejected, Canceled }   

    // default value of an enum is its first element, in this case "Pending"
    Status public status;  

    // update status using enum Status type parameter
    function update(Status _status) public {
        status = _status;
    }

    // update with a specific enum member
    function cancel() public {
        status = Status.Canceled
    }
}
```

### 32. Constructor function

*Contract creation* can be triggered by an *external transaction* or from *within a solidity contract*.
- When a contract is created, its `constructor()` function is executed. A constructor is optional, and only one is allowed.
- After the constructor has executed, the final code of the contract is stored on the blockchain. This code includes all `public` and `external` functions and all functions that are reachable from there through function calls.
- The deployed code does **not** include the *constructor code* or *internal functions* that are only called from the constructor.

> The `purpose` string of this contract is assigned within the constructor, and must be supplied as an argument on deployment

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.3;

contract ConstructorDemo {

    // purpose initialized outside of constructor as ""
    string public purpose;

    // purpose assigned within constructor, based on deployment argment
    constructor(string memory _purpose) {
        purpose = _purpose;
    }
}
```

### 33. Receive Function

A contract may have at most one `receive()` function, which is declared without the function keyword.

This function takes no arguments, does not return anything, must have `external` visibility, and is `payable`.

```Solidity
receive() external payable {}
```

`receive()` executes on:
- calls to a contract with empty calldata.
- plain Ether transfers via `send()` or `transfer()`

`receive()` may only rely on a 2300 gas stipend forwarded from `send()` and `transfer()`, which allows the function to update the `address.balance` and leaves little room for other operations in the function body.

### 34. Fallback function

A contract can have at most one `fallback()` function, which is declared without the function keyword.

This function must have `external` visibility. It is *optionally* `payable`.

```solidity
fallback() external payable {}
```

`fallback()` is executed on:
- calls to a contract containing `calldata`, but the function selector does not match any declared contract functions
- calls with no `calldata` and `receive()` is not declared.

`fallback()` may only rely on 2300 gas available from `send()` or `transfer()`, similarly to `receive()`.
