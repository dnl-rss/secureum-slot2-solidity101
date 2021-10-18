## 35. Static Typing

Solidity is a *statically typed language*, meaning that the *type* of each variable must be *specified explicitly* at *compile-time*.

Other statically typed languages include C, C++, Java, Rust, Go, and Scala.

In contrast, *dynamically typed languages* can assign types during runtime.

## 36. Value and Reference Types

Solidity has two categories of types: `value` types and `reference` types.  

`value` types are called so because variables of these types will always be passed by value (they are copied when used as function arguments or in assignments).

`reference` types assign a pointer to a value in memory. They can be accessed and modified through multiple names, yet reference the same underlying variable.

### 37. Value Types

Value types can be considered safer because their value can only be modified through direct reference, but their persistence should be considerd

> Includes:

| specifier | type | description |
| --------- | ---- | ----------- |
| `bool` | boolean | may be `true` or `false` |
| `int` | signed integer | |
| `uint` | unsigned integer | |
| `address` | Ethereum account | 20-byte value |
| `contract` | | |
| `bytes1`, `bytes2`, ... `bytes32` | fixed size character set | length from 1 to 32 |
| `string` | dynamically sized character set | |
| `rational`, `string`, `uni`, `hex` | literals | |
| `enum` | | |
| `function` | | |

### 38. Reference Types

Reference types may be risky because their value may be changed via multiple variable names.

> Includes:

| specifier | type | description |
| --------- | ---- | ----------- |
| `bytes` or `string` | dynamically sized arrays | |
| `struct` | custom grouping of other variables | |
| `mapping` | key-value pairs | key may be any value type |

## 39. Default Values

A *declared variable* that is *not yet initialized* will have a *default zero-state value* whose byte representation is all zeros.

> Default values:

| specifier | default value |
| --------- | ------------- |
| `bool` | `false` |
| `int` or `uint`| `0` |
| `string` | `""` |
| `bytes` | `[]` |
| `enum` | first member |

## 40. Scoping

The *scope* of a variable affects its visibility within contracts and functions.

Solidity follows the widespread scoping rules of the *C99 standard*:
1. Variables are visible from the point *right after declaration* until the *closure of the smallest bracket set* `{}` that contains that declaration.
    - An *exception* to this rule includes variables declared in the *initialization part of a for-loop*, which are only visible until the end of the for-loop
2. Variables that are *parameter-like* (function parameters, modifier parameters, catch parameters, ...) are visible *inside the code block that follows* -- the body of the function or modifier or catch blocks
3. Variables and other *items declared outside of a code block* are visible *before they were even declared*. This means state variables and functions can be called before they are declared in code.

```solidity
// global_var is visible here (before declaration), and everywhere else in this code

const global_var = 2;

function scopingDemo(uint param1) {
  // param1 is visible starting here

  local_var = 3;
  // local_var is visible starting here

  for (uint var = 0, var < 10, var++) {
    // var is visible inside the for-loop
  }
  // var is no longer visible

}
// parameter1 and local_variable are no longer visible
```


## 41. Boolean Type

Declared with `bool` keyowrd. Possible values are constrants `true` and `false`

Operations on booleans:
- `!`: logical negation
- `&&`: logical conjunction
- `||`: logical disjunction
- `==`: equality
- `!=`: inequality

The operators ``||`` and ``&&`` apply the common *short-circuiting* rules: the second condition is not evaluated if the first is `false`.

Booleans are often used to control the logic of a contract. Proper usage is essential to security.

> Accidental use of logical negation in this modifier mean that restricted functions will be callable by virtually anyone except the rightful owner

```solidity
modifier onlyOwner {
    require( msg.sender != owner );
    _;
}
```

### 42. Integer Types

*Signed and unsigned integers* may be specified in various fixed sizes.
- Keywords `uint8` to `uint256` specify *unsigned integers* in steps of 8 bits
- Keywords `int8` to `int256` specify *signed integers*
- `uint` and `int` are aliases for `uint256` and `int256`, respectively.

Operators on integers:
- `<=`, `<`, `==`, `!=`, `>=`, `>`: comparisons, evaluate to `bool`
- `&`, `|`, `^`, `~`: bit operators
- `<<`, `>>`: shift operators
- `+`, `-`, `*`, `/`, `%`, **: arithmatic operators

Todo: explain bitwise exclusive or, bitwise negation, left shift, right shift, unary -

Integers are restricted to the range of their declared byte sized, which ranges from 8 to 256 in increments of 8 bytes.

| type     | lower bound | upper bound   |
| -------- | ----------- | ------------- |
| `int8`   | 0           | (2**8) - 1    |      
| `int16`  | 0           | (2**16) - 1   |   
| `int24`  | 0           | (2**24) - 1   |
| ...      | ...         | ...           |
| `int256` | 0           | (2**256) - 1  |

TODO: table for uint

### 43. Check and Unchecked Arithmetic

If an arithmetic operation causes the value of a fixed size integer to go above or below its possible range, this results in an *overflow* or *underflow* error, respectively.

Arithmetic on integers is performed in "checked" mode by default since `v0.8.0`. This integrates Open Zepplin's [SafeMath](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeMath.sol) standard into arithmetic operations and will revert transactions that cause integer values to overflow or underflow.

The "unchecked" mode can be used via `unchecked{ ... }`. Operations resulting in overflow or underflow will "wrap" around to the upper or lower bound of that variable.

### 44. Fixed Point Types

Fixed point numbers using keywords `fixed` / `ufixed` are not fully supported by Solidity yet. They can be declared, but cannot be assigned to or from. There are fixed-point libraries that are widely used for this such as DSMath, PRBMath, ABDKMath64x64 etc.

### 45. Address Types

The `address` type is a *20 byte value* refers to an Ethereum account address

`address` comes in two flavors:
1. `address`: cannot receive Ether
2. `address payable`: can receive Ether (additional members `.transfer()` and `.send()`)

`address` can be converted to `address payable` via `payable(address)`.

- `<=`, `<`, `==`, `!=`, `>=`, `>`: comparisons, evaluate to `bool`

### 46. Members of address types

| member | return type | description |
| ------ | ---- | ----------- |
| `address.balance` | `uint256` | balance of `address` in `wei` |
| `address.code` | `bytes memory` | bytecode at the `address` (can be empty) |
| `address.codehash` | `bytes32` | codehash of the `address` |  
| `address payable.transfer(uint256 amount)` | none | <ul><li>sends given amount of `wei` to `address`</li><li>forwards 2300 gas</li><li>reverts on failure</li></ul> |
| `address payable.send(uint256 amount)` | `(bool, bytes memory)` | <ul><li>sends given amount of `wei` to `address`</li><li>forwards 2300 gas</li><li>returns `false` on failure (revert with `require()`) </li></ul> |
| `address.call(bytes memory)` | `(bool, bytes memory)` | <ul><li>issues low-level `CALL` with given payload</li><li>forwards all available gas or a specified amount</li><li>returns `false` on failure (revert with `require()`) </li></ul> |
| `address.delegatecall(bytes memory)` | `(bool, bytes memory)` | <ul><li>issues low-level `DELEGATECALL` with given payload</li><li>forwards all available gas or a specified amount</li><li>returns `false` on failure (revert with `require()`) </li></ul> |
| `address.staticcall(bytes memory)` | `(bool, bytes memory)` | <ul><li>issues low-level `STATICCALL` with given payload</li><li>forwards all available gas or a specified amount</li><li>returns `false` on failure (revert with `require()`) </li></ul> |

### 47, 48 Send Ether via Transfer, Send, and Call

> Significant alteration from the secureum substack blog post contained in this section...

The are three methods available to send Ether within a contract: `transfer()`, `send()`, and `call{value:}()`.

> The key differences in these methods involve their return types, gas forwarding, behavior on failure, and where in the receiving contract they are processed.

| | gas forwarded | return types | on failure | processed in |
|-|-|-|-|-|
| `transfer()` | 2300 | none | transaction reverts | `receive()` |
| `send()` | 2300 | `(bool, bytes memory)` | returns `false` | `receive()` |
| `call{value:}("")` | all, or specified | `(bool, bytes memory)` | returns `false` | `fallback()` |

Note that `transfer()` and `send()` are both processed in the `receive()` function of the recipient account. If not is defined, they are processed in `fallback()`.

Both `transfer()` and `send()` forward only 2300 gas. This is just enough to update `address.balance` of the recipient account, but any additional execution in `receive()` will cause the transfer to fail.

`send()` does not revert the transaction, but returns false. Use `require()` to check the success condition and revert if needed.

While `transfer()` and `send()` were favored in the past because they are resistant to *reentrancy attacks*, `call{value:}()` is now the favored method and the developer must implement other safeguards against reentrancy.

TODO: Why was this recommendation made? What shortcomings of transfer() and send() surfaced the change?

> This implmentation of a smart contract bank can send ether by either `send()`, `transfer()`, or `call{value:}("")`. Note that this contract is vulnerable to reentracy via the call method.

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "hardhat/console.sol";

contract Bank {

  mapping( address => uint256 ) balances;

  constructor() public {}

  function deposit() public payable {
      balances[msg.sender] += msg.value;
  }

  function withdraw_via_transfer(uint256 amount) public {
      // forwards 2300 gas, not adjustable
      require(balances[msg.sender] >= amount, "Invalid withdraw request");
      payable(msg.sender).transfer(amount);
      balances[msg.sender] -= amount;
  }

  function withdraw_via_send(uint256 amount) public {
      // forwards 2300 gas, not adjustable
      // returns success condition
      // fails if stack depth is at 1024
      require(balances[msg.sender] >= amount, "Invalid withdraw request");
      bool sent = payable(msg.sender).send(amount);
      require(sent, "Failed to send Ether");
      balances[msg.sender] -= amount;
  }

  function withdraw_via_call(uint256 amount) public {
      // forwards all available gas
      // returns success condition and data
      require(balances[msg.sender] >= amount, "Invalid withdraw request");
      // (bool sent, bytes memory _data) = msg.sender.call.value(amount}("");
      (bool sent, bytes memory _data) = msg.sender.call{value: amount}("");
      require(sent, "Failed to send Ether");
      balances[msg.sender] -= amount;
  }
}
```

### 49. call, delegatecall, and staticcall

The low-level address methods `call()`, `delegatecall()`, and `staticcall()` can be used to interface with contracts that do not adhere to the ABI or gain more control over the encoding.

Each method takes a single `bytes memory` message data parameter encoded with `abi.encode()`, `abi.encodePacked()`, `abi.encodeWithSelector`, or `abi.encodeWithSignature`. They return `(bool, bytes memory)`: a success condition and return data.

Modifiers for `gas` and `value` are used to specify the amount of gas and Ether passed to the callee. Note that `value` is not supported by `delegatecall()`.

The purpose of `delegatecall()` is to use library or logic code stored in the callee contract, but to operate on the state of the caller contract.

| | gas modifier | value modifier | reverts if | use case |
|-|-|-|-|-|
| `call()` | ✅ | ✅ | none ||
| `delegatecall()` | ✅ | ⬜️ | none | state/modify |
| `staticcall()` | ✅ | ✅ | called function modifies state | state/use |

> TODO: Revise this section

### 50. Contract Type

Every `contract` defines its own type. Contracts can be explicitly converted to `address` type. Contract types do not support any operators. The members of contract types are the external functions, including getter functions for any public state variables.

```solidity
// explicitly convert the current contract "this" to address type with "address()"
// retrieve the balance (member of address types)
address(this).balance
```

> Todo

```solidity
// declare an external contract object at a specific address
// interact with the contract using its methods
```

### 51. Fixed Size Byte Arrays

Values of types `bytes1`, `bytes2`, ... `bytes32` hold a sequence of bytes from 1 up to 32.  

The type `byte[]` is an array of bytes, but wastes 31 bytes of space fore each element due to padding rules (except in storage).

It is recommended to use `bytes` instead.

### 52. literals

Literals can be of 5 types:
1. Address literals: *hexadecimal literals* between 39-41 digits that pass the *address checksum test*. values that do not pass the checksum test produce an error.
2. Rational and integer literals:
    - *Integer literals* are formed from a *sequence of digits* in the range 0-9.
    - *Decimal literals* are formed by a `.` with at least one number on one side.
    - *Scientific notation* is supported, where the base can have fractions and the exponent cannot.
    - *Underscores* can be used to separate digits for readability.
3. String literals: *ASCII characters* written with either double or single quotes (`"foo"` or `'bar'`). May also contain a set of escape characters.
4. Unicode literals: *UTF8 characters* prefixed with the keyword `unicode`. They also support the same escape sequences as string literals.
5. Hexadecimal literals: *hexadecimal digits* prefixed with keyword `hex` enclosed by double or single quotes.

```solidity
address myAddress = 0xE7A54673f2FfE41cf38dbA2014326064A958b709; // an address literal
int myInt = 123_456_789_000; // int literal
int myInt = 123_456_789_000; // int literal with underscores
int myInt = 1.23456789e11; // int literal with scientific notation
fixed myRational = 2.5 / .3 // rational literal. this assignment will fail because fixed point numbers are not fully supported by solidity.
string myString = "foo"; // string literal with double quotes
string myString = 'bar'; // string literal with single quotes
string myUTF8 = unicode"Hello 😃"; // unicode literal
string myHex = hex"001122FF" // hex literal with double quotes
string myHex = hex'00_11_22_FF' // hex literal with underscores and single quotes
```

### 53. Enum Types

Enums are a *user defined type*. They must have *between 1-256 members*. The *default value* is the *first member*.

```solidity
enum Fruits { APPLE, BANANA, PEACH }; // enum fruit restricting types of fruits
```

### 54. Function Types

Function types are the types for functions.
- Variables of `function` type can be assigned from functions.
- Function parameters of `function` type can be used to pass functions into another function call.
- Return variables may be of `function` type.

Functions come in two types: internal and external. Internal functions can be called inside the current contract. External functions consist of an address and a function signature. They can be passed via and returned from external function calls.

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.22 <0.9.0;


contract Oracle {
    struct Request {
        bytes data;
        function(uint) external callback; // external function type as struct member
    }

    Request[] private requests;
    event NewRequest(uint);

    // external function type as function parameter
    function query(bytes memory data, function(uint) external callback) public {
        requests.push(Request(data, callback));
        emit NewRequest(requests.length - 1);
    }

    function reply(uint requestID, uint response) public {
        // Here goes the check that the reply comes from a trusted source
        requests[requestID].callback(response);
    }
}


contract OracleUser {
    Oracle constant private ORACLE_CONST = Oracle(address(0x00000000219ab540356cBB839Cbe05303d7705Fa)); // known contract
    uint private exchangeRate;

    function buySomething() public {
        // internal function this.oracleResponse is external to contract Oracle
        ORACLE_CONST.query("USD", this.oracleResponse);
    }

    function oracleResponse(uint response) public {
        require(
            msg.sender == address(ORACLE_CONST),
            "Only oracle can call this."
        );
        exchangeRate = response;
    }
}
```

### 55. Reference Types & Data Location

Reference type refer to a value by pointing to its location in the EVM, so every reference type must have an additional annotation for the data location.  There are three data locations: `memory`, `storage`, and `calldata`

1. `memory`: lifetime limited to *external function call*
2. `storage`: lifetime limited to *contract lifetime* and location of state variables
3. `calldata`: lifetime limited to *call lifetime*. non-modifiable, non-persistent area where function arguments are stored. behaves like memory. required for parameters of external functions but can be used for other variables.

### 56. Data Location & Assignement

Data location are relevant for both the *persistence of data* and *semantics of assignments*.

Assignments of variables behave differently depending on their data location:
1. Assignments between `storage` and `memory` (or from `calldata`) always create an *independent copy*.
2. Assignments from `memory` to `memory` create *references*.
3. Assignments from `storage` to `storage` create *references*.
4. All other assignments to `storage` always *copy*.
    - Examples include assignments to state variables or to members of local variables of storage `struct` type, even if the local variable itself is just a reference.

```solidity
contract Example {
  // initialize a dynamically size int array (reference type)
  // location in storage by default
  uint[] x;

  // memoryArray is located in memory
  function f(uint[] memory memoryArray) public {

    // copy memoryArray from memory into storage
    x = memoryArray;

    // assign a pointer in y to the location of x
    uint[] storage y = x

    // modify x through y
    y.pop(); // modifies x through y

    // modify y through x
    delete x;

    // fails because y was not declared in storage (as x was)
    // y = memoryArray;

    // fails because y is contains a pointer. delete y resets the pointer, but there is not sensible location to point to.
    // delete y;

    // call g, using a reference to x
    g(x);

    // call h, using a copy in memory of x
    h(x);
  }

  function g(uint[] storage) internal pure {};
  function h(uint[] memory) public pure {};
}
```

### 57. Arrays

Arrays can have a compile time fixed size, or they can have a dynamic size

1. The type of an array of fixed size `k` and element type `T` is written as `T[k]` and an array of dynamic size as `T[]`.
2. Indicies are zero-based
3. Array elements can be of any type, including `mapping` or `struct`
4. Accessing an array past its end causes a failing assertion

```solidity
contract Example {

  // initialize a fixed integer array with 8 elements
  int[8] fixedIntArray;

  // initialize a dynamic array of booleans
  bool[] dynamicBoolArray;

  // assign to the first and last element of the fixed int array
  fixedIntArray[0] = 3;
  fixedIntArray[7] = 4;

  // this command will fail
  // fixedIntArray[8] = 5;
}
```
### 58. Array members

1. `length`: returns number of elements in array
2. `push`: appends a zero-initialized element at the end of the array. returns a reference to the element.
3. `push(x)`: appends a given element at the end of the array. returns nothing.
4. `pop()`: remove element from the end of the array. implicitly calls delete on the removed element. returns the element.


```solidity
struct UsedCar {
  string make;
  string model;
  int mileage;
}

contract Example {

  // initalize a dynamic array of structs
  UsedCar[] myCars;

  // add an empty car to myCars
  // returns UsedCar("","",0)
  myCars.push();

  // add a car to myCars
  myCars.push(UsedCar("toyota","prius",100_000));

  // returns 2
  uint numberOfCars = myCars.length;

  // remove car from myCars
  UsedCar myPrius = myCars.pop()

  // returns 1
  uint numberOfCars = myCars.length;
}
```

### 59. Bytes & String

Variables of type `bytes` and `string` are special arrays

1. `bytes` is similar to `byte[]`, but it is packed tightly in `calldata` and `memory`
2. `string` is equal to `bytes`, but does not allow length or index access
3. Solidity does not have sting manipulation functions, but there are third party string libraries
4. Use `bytes` for arbitrary length raw byte data and `string` for arbitrary length UTF8 string data
5. Use `bytes` over `byte[]` because it is cheaper. byte[] adds 32 padding bytes between the elements
6. If you can limit the length the a certain number of bytes, always use a fixed byte array `bytes1` to `bytes32` because the are much cheaper

| | bytes | byte[] | string |
|-|-|-|-|
| packing | tight | loose | tight |
| sizing | dynamic | dynamic | dynamic |
| `length` | X | X | O |
| access element by index | X | X | O |

```Solidity
//SPDX-License-Identifier: MIT
pragma solidity >= 0.8.7;

contract Example {

  // string array assignment
  string public myStringArray = "abcdefg";

  // a string literal is converted to bytes upon assignment
  // stored value is 0x61626364656667
  bytes public myBytesArray = "abcdefg";

  // assign fixed byte arrays
  bytes1 public a = 0xb5;
  bytes4 public b = 0x56_b5_ff_00;

}
```
### 60. Memory arrays

Memory arrays with dynamic length can be created using the `new` operator

1. Unlike `storage` arrays, `memory` arrays cannot be resized. It does not have a `push()` member function.
2. The required sized must be known in advance, or a new `memory` array must be created with every element copied.

```Solidity
//SPDX-License-Identifier: MIT
pragma solidity >= 0.8.7;

contract Example {
  // initialize a dynamically sized int array (reference type)
  // location in storage by default
  uint[] public storageArray;

  // memoryArray is located in memory and persists only in function f
  function f(uint[] memory memoryArray) public {

    // Results in TypeError: member push() not available
    // memoryArray.push(1);

    // copy memoryArray into storage
    storageArray = memoryArray;

    // push two elements into storageArray
    assert( storageArray.length == memoryArray.length );
    storageArray.push(1);
    storageArray.push(2);
    assert( storageArray.length == memoryArray.length + 2 );

    // copy storageArray into memoryArray
    uint[] memory memoryArray = storageArray;
    assert( storageArray.length == memoryArray.length );
  }
}
```

### 61. Array literals

An array literal is a *comma separated list* of one or more expressions, *enclosed in square brackets*: `[1,2,3]`.

1. It is always a *statically-sized memory array* whose length is the number of expressions
2. The base type of the array is the *type of the first expression* of the list. All other expressions must be *implicitly converted* to it. Otherwise, a `TypeError` is thrown.
3. *Fixed size memory arrays* **cannot** be assigned to *dynamically-sized memory arrays*

```Solidity
//SPDX-License-Identifier: MIT
pragma solidity >= 0.8.7;

contract Example {

  // the literal ["hello","world"] is type string[2] memory
  string[] public storageArray = ["hello","world"];

  function f(string[] memory memoryArray) public {

    // Fixed size memory arrays cannot be assigned to dynamically assigned memory arrays

    // Results in TypeError: string[2] memory not implicitly convertible to type uin256[] memory
    // memoryArray = ["hello","world"];

    // Results in TypeError: string[2] memory not implicitly convertible to type uin256[] memory
    // string[] memory  newMemoryArray = ["hello", "world"]

    // This works
    string[2] memory newMemoryArray = ["hello","world"];
  }
}
```

### 62. Array gas costs

Storage arrays have operations `push()` and `pop()`.

*Extending* the array with `push()` has *constant gas cost* because storage is zero initialized.

*Shortening* the array by `pop()` have *variable gas cost*, depending on the size of the element being removed.

If the removed element is an array, `pop()` can be very costly, because it includes explicitly clearing all the removed elements.

> Pushing an empty element has constant gas cost. Popping an element have variable gas cost, depending on the size of the element:  

```solidity
//SPDX-License-Identifier: MIT
pragma solidity >= 0.8.7;

contract Example {

  string[] public storageArray = ["hello","world"];

  function extendByPush() public returns (uint) {
    // storageArray.push() always costs 5099 gas
    uint gasbefore = gasleft();
    storageArray.push();
    return gasbefore - gasleft();
  }

  function shortenByPop() public returns (uint) {
    // storageArray.pop() can cost 7512 (removing "") or 10312 (removing "hello" or "world")
    uint gasbefore = gasleft();
    storageArray.pop();
    return gasbefore - gasleft() ;
  }  
}
```

### 63. Array Slices

Array slices are a *view* on a *contiguous portion of an array*.

They are written as `x[start:end]`. The first element of the returned slice is `x[start]` and the last element is `x[end-1]`.

Start and end must be expressions resulting in `uint256` type, or implicitly convertible to it.

1. An *exception* is thrown if `start > end` or `end > array.length`
2. Both `start` and `end` are *optional*
    - `start` defaults to `0`
    - `end` defaults to `array.length`
3. Slices have **no members**
4. Slices are *implicitly convertible* to arrays of their underlying type
5. Slices support *index access*, which is *relative to the slice* not to the parent array
5. Slices have **no type name** which means *no variable can be of slice type* (they only exist in intermediate expressions)
6. Slices are only implemented for `calldata` arrays
7. Slices are useful to *ABI decode* secondary data passed in function parameters

> Slices are only implemented in solidity for `calldata`. This is useful for parsing `calldata`. The function selector is always the first four bytes of `calldata`.

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Proxy {
    /// Address of the client contract managed by proxy i.e., this contract
    address client;

    constructor(address _client) public {
        client = _client;
    }

    /// Forward call to "setOwner(address)" that is implemented by client
    /// after doing basic validation on the address argument.
    function forward(bytes calldata _payload) external {
        bytes4 sig = abi.decode(_payload[:4], (bytes4));
        if (sig == bytes4(keccak256("setOwner(address)"))) {
            address owner = abi.decode(_payload[4:], (address));
            require(owner != address(0), "Address of owner cannot be zero.");
        }
        (bool status,) = client.delegatecall(_payload);
        require(status, "Forwarded call failed.");
    }
}
```

### 64. Struct Types

`struct` types define new *aggregate types* combining other value/reference types into one data structure.

Mappings and arrays may contain structs, and vice versa.

A `struct` to may **not** contain a member of its own type.

> TODO: ^ Does that mean that a `struct` cannot contain any other `struct`? Or only that it may not contain itself?

### 65. Mapping types

`mapping` types define key-value pairs.

They are declared as such: `mapping(_KeyType => _ValueType) _VariableName;`.

1. Allowed types for `_KeyType` and `_ValueType`:
    - `_KeyType` can be *any built-in value type*: `bytes`, `string`, or any `contract` or `enum` type
    - `_KeyType` may **not** be any other user defined or complex type (`mapping`, `struct`, or arrays)
    - `_ValueType` can be *any type* (including `mapping`, `struct`, or arrays).
2. `mapping` does **not store** *key data* (the `keccak256` key hash is used to loop up the value)
3. `mapping` has **no length**, or a concept of a key or value being set
4. `mapping` data can only be located in `storage`, and thus allowed for *state variables*, *storage reference* types in functions, or *parameters for library* functions
5. `mapping` **cannot** be used as a `public` *function parameter* or *return variable*. This restriction is also true for arrays and structs containing mappings.
6. `mapping` is **not iterable**. However, one can implement a data structure on top of `mapping` and iterate over that.

### 66. Shorthand Operators

An "LValue" is a variable, or something that can be assigned to

- `a += e` is equivalent to `a = a + e`.
- `-=`, `*=`, `%=`, `|=`, `&=`, and `^=` act similarly as above.
- `a++` is equivalent to `a += 1`, but the expression itself still has the *previous value* of `a` (likewise for `a--`)
- `++a` have the same effect as above, but returns the *value after the change*.

> TODO: elaborate on `a++` and `++a` notes

### 67. `delete`

The command `delete a` assigns the *initial zero value* for the type to `a`

1. For an `int`, it is equivalent to `a = 0`
2. For a *dynamic array*, it assigns a dynamic array of *length zero*
3. For a *static array*, it assigns a static array of the same length with *all elements set to their initial value*
4. For an *array element*, `delete a[x]` deletes only the element at index `x` (other elements are unchanged)
5. For a `struct`, it assigns a `struct` with all members reset
6. For a `mapping`, `delete` has **no effect**  
7. For a `struct` containing a `mapping`, `delete` will reset all members that are not `mapping` and also recurs into the members, unless they are `mapping`
8. For a `mapping` *key*,`delete a[x]` will delete the value stored at `x`

### 68. Implicit Conversions

An implicit type conversion is automatically applied by the compiler in some cases when making assignments, passing arguments, or applying operations.

*Implicit conversion* between value-types is possible if it *makes sense semantically* and *no information is lost*.

For example:
- `uint8` is convertible to `uint16`,
- `int128` is convertible to `int256`,
- `int8` is **not** convertible to `uint256` (`uint256` cannot hold negative values)

### 69. Explicit Conversion

*Explicit conversions* may be applied by a developer even when the compiler does not allow *implicit conversion*.

The developer should have reasonable confidence that the conversion will be valid, as this bypasses some security features of the complier (e.g. `int` to `uint`) and may result in unexpected behavior.

For example:
1. `int` converted to a *smaller* type, cuts off higher order bits
2. `int` converted to a *larger* type, pads on the left (the higher order end)
3. `bytesN` converted to a *smaller* type, cuts off the bytes to the right
4. `bytesN` converted to a *larger* type, pads bytes to the right.

### 70. Conversion of Literals

1. *Decimal/Hex number literals* can be *implicitly converted* to any `int` type, provided it is large enough to represent the number without truncation
2. *Decimal number literals* **cannot** be *implicitly converted* to `bytesN`
3. *Hex number literals* can be *implicitly converted* to `bytesN`, but only if the number of hex digits exactly fits the size of the bytes type
4. *Decimal/hex literals* can be *implicitly converted* to any `bytesN` size, if their value is zero can be converted (exception to previous)
5. *String/hex literals* can be *implicitly converted* to `bytesN`, if the number of characters matches the size of the bytes array

### 71. Ether Units

A literal number can take a suffix of `wei`, `gwei`, or `ether` to specify a *sub-denomination of Ether*, effectively multiplying the value by the appropriate power of 10.

> 1 ether == 1e9 qwei == 1e18 wei
```solidity
uint amount1 = 1 ether;
uint amount2 = 1e9 gwei;
uint amount3 = 1e18 wei;
assert( amount1 == amount2 & amount2 == amount3 );
```

### 72. Time Units

Suffixes like `seconds`, `minutes`, `hours`, `days`, and `weeks` after literal numbers specify *units of time* where `seconds` are the base unit

Be careful when applying these calendar calculations, because not every year has 365 days (leap years) and not every day has 24 hours (leap seconds).

```solidity
assert( 1 == 1 seconds );
assert( 1 minutes == 60 seconds);
assert( 1 hours == 60 minutes );
assert( 1 days == 24 hours );
assert( 1 weeks == 7 days );
```

> The suffixes cannot be applied directly to variables but can be applied by multiplication.

```solidity
// daysAfter is a regular uint variable
uint daysAfter = 3;

// this fails
// uint secondsAfter = daysAfter days;

// this works
uint secondsAfter = daysAfter * 1 days;
```

### 73. Block and Transaction Properties

Solidity supports `block`, `msg`, and `tx` objects to retrieve relevant information about the state of the blockchain and transaction.

> Members of `block` object

| member | return type | description |
| ------ | ------- | --------- |
| `block.chainid` | `uint` | current chain id |
| `block.coinbase(address payable)` | `address payable` | current block miners address |
| `block.difficulty` | `uint` | current block difficulty |
| `block.gaslimit` | `uint` | current block gaslimit |
| `block.number` | `uint` | current block number |
| `block.timestamp` | `uint` | current block timestamp as seconds since unix epoch |
| `blockhash(uint blockNumber)` | `bytes32` | hash of given block (only 256 most recent) |

 > Members of `msg` object

| member | return type | description |
| ------ | ------- | --------- |
| `msg.data` | `bytes calldata` | complete calldata |
| `msg.sender` | `address` | sender of the message (current call) |
| `msg.sig` | `bytes4` | first four bytes of calldata (function identifier) |
| `msg.value` | `uint` | number of wei sent with message |

> Members of `tx` object

| member | return type | description |
| ------ | ------- | --------- |
| `tx.gasprice` | `uint` | gas price of the transaction |
| `tx.origin` | `address` | sender of the transaction (full call chain) |
| `gasleft()` | `uint256` | remaining gas in transaction |

### 74. `msg` can change

The values of all members of `msg` (`msg.sender`,`msg.value`, etc) can change for every external function call, including library functions.

### 75. Randomness Source

**Do not rely** on `block.timestamp` or `blockhash` as a *source of randomness*. Both can be influenced by miners. The current block timestamp must be strictly larger than the timestamp of the last block, but the only guarantee is that is will be somewhere between the timestamps of two consecutive blocks in the canonical chain

### 76. Blockhash

`blockhash(uint blockNumber)` returns the hash of the specified block. However, for scalability reasons solidity can only access this data for the 256 most recent blocks, excluding the current block. Otherwise `blockhash()` returns `0`.

### 77. ABI Encoding/Decoding

> Solidity supports multiple functions for encoding and decoding data with respect to the contract ABI.

| member | return type | description |
| `abi.decode(bytes memory encodedData, (...))` | `(...)` | decodes given data, while the types are given in parentheses as second argument |
| `abi.encode(...)` | `bytes memory` | encodes given arguments |
| `abi.encodePacked(...)` | `bytes memory` | packed encoding of given arguments, can be ambiguous! |
| `abi.encodeWithSelector(bytes4 selector, ...)` | `bytes memory` | encodes given arguments starting from the second and prepends given four byte selector |
| `abi.encodeWithSignature(string memory signature, ...)` | `bytes memory` | equivalent to `abi.encodeWithSelector(bytes4(keccack256(bytes(signature))), ...)` |

### 78. Error handling functions

> Solidity functions useful for handling errors

| function | on failure | used for |
| -------- | ---------- | -------- |
| `assert(bool condition)` | panics if condition not met, reverts state changes | internal errors |
| `require(bool condition)` | reverts if condition not met | input errors or external components |
| `require(bool condition, string memory message)` | also provides an error message | |
| `revert()` | aborts execution and reverts state changes | force revert without a condition |
| `revert(string memory reason)` | also provides an explanatory string | |

### 79. Math/Crypto functions

> Several solidity functions useful for cryptographic applications

| function | return type | description |
|-|-|-|
| `addmod(uint x, uint y, uint k)` | `uint` | computes `(x+y)%k` where addition is performed with arbitrary precision and does not wrap arround `2**256`. Assert(k!=0) enforces from version 0.5.0 |
| `mulmod(uint x, uint y, uint k)` | `uint` | computer `(x*y)%k` where the multiplication is performed with arbitrary precision and does not wrap around `2**256`. assert(k!=0). |
| `keccak256(bytes memory)` | `bytes32` | computes keccak256 hash of input |
| `sha256(bytes memory)` | `bytes32` | computes SHA256 hash of input |
| `ripemd160(bytes memory)` | `bytes20` | computes RIPEMD-160 hash of input |
| `ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s)` | `address` | recovers `address` associated with the public key from elliptic curve signature or return zero on error. does not return `address payable`. |

> TODO: Elaborate on this section

### 80. Ecrecover Malleability (non-uniqueness)

A *valid signature* can be turned into a *different valid signature* requiring **no knowledge** of the corresponding *private key*.

This can result in a vulnerability to *replay attacks*, in which the second signature may bypass some contract logic.

This usually not a problem unless you require signature to be unique or use them to identify items.

The reason for this malleability is that the `s` value can be in either the higher order range or the lower order range.

OpenZepplin has an ECDSA helper library to wrap `ecrecover` without this issue by forcing `s` to be in the lower order range.

### 81. Context related

1. `this` is the current contract, explicitly convertible to `address`
2. `selfdestruct(address payable recipient)` destroys the current contract, sending funds to `recipient` and ending execution

### 82. selfdestruct

`selfdestruct()` has some pecularities:
- The receiving contract's `receive()` function is **not** executed
- The contract is only destroyed at the *end of the transaction*. A revert may undo the destruction.

### 83. Contract Type

The expression `type(X)` can be used to retrieve information about the type `X`, where `X` can either be a contract or an interface type.

For a `contract` type `C` the following is available:
1. `type(C).name`: the contract *name*
2. `type(C).creationCode`: `memory bytes` array containing the contract *creation bytecode*.
    - Used for inline assembly to build custom creation routines, especially using the `CREATE2` opcode.
    - This property cannot be accessed in the contract itself or any derived contract.
    - It causes the bytecode to be included in the bytecode of the call site and thus circular references like above are not possible.
3. `type(C).runtimeCode`: `memory bytes` array containing contract *runtime bytecode*.
    - This code is usually deployed by the `constructor()` of `C`.
    - If the `constructor()` of `C` uses inline assembly, this might be different from the actual deployed bytecode.
    - Libraries modify their runtime bytecode at the time of deployment to guard against regular calls.
    - Same restrictions as with `creationCode` also apply here.

For an `interface` type `I` the following is available:
1. `type(I).interfaceId`: `bytes4` value containing EIP-165 interface identifier of a given `interface I`.
    - The identifier is defined as the `XOR` of all function selectors defined within the interface itself, excluding inherited functions.

#### 84. Integer type information

For an integer type T, the following is available:
1. `type(T).min`: smallest value possible
2. `type(T).max`: largest value possible

### 85. Control structures

Solidity has `if`, `else`, `while`, `do`, `for`, `break`, `continue`, `return` with usual semantics known for C or Javascript.

1. Parentheses `(..)`can **not** be *ommitted* for conditionals, but curly braces `{...}` may be *ommitted* around *single-statement bodies*
2. There is **no type conversion** from non-boolean to `bool` types as there is in C and JavaScript, so `if (1){...}` is **not valid** in Solidity
