## Static Typing

Solidity is a statically typed language, meaning that the type of each variable must be specified at compile-time. Other examples include C, C++, Java, Rust, Go, and Scala.
In contrast, dynamically typed languages only require types on runtime values.

## Value and Reference Types

Solidity has two categories of types `value` types and `reference` types.  

### Value Types

`value` types are called so because variables of these types will always be passed by value (they are copied when used as function arguments or in assignments).

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

### Reference Types

`reference` types assign a pointer to a value in memory. They can be accessed and modified through multiple names, yet reference the same underlying variable.

> Includes:

| specifier | type | description |
| --------- | ---- | ----------- |
| `bytes` or `string` | dynamically sized arrays | |
| `struct` | custom grouping of other variables | |
| `mapping` | key-value pairs | key may be any value type |

## Default Values

A declared variable will have an initial default value whose byte representation is all zeros. This is typically the "zero state" of whatever the type is.

> Default values:

| specifier | default value |
| --------- | ------------- |
| `bool` | `false` |
| `int` or `uint`| `0` |
| `string` or `bytes` | `""` |
| `enum` | first member |

## Scoping

Scoping in solidity follows the widespread scoping rules of C99

1. Variables are visible from the point right after declaration until the closure of the smallest bracket set `{}` that contains that declaration. An exeception to this rule includes variables declared in the initialization part of a for-loop, which are only visible until the end of the for-loop
2. Varibles that are parameter-like (function parameters, modifier parameters, catch parameters, ...) are visible inside the code block that follows -- the body of the function or modifier or catch blocks
3. Variables and other items declared outside of a code block are visible even before they were declared. This means you can use state variables and call functions before they are declared in code.

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

## Type operations

### Boolean

- `!`: logical negation
- `&&`: logical conjunction
- `||`: logical disjunction
- `==`: equality
- `!=`: inequality

### Integers

- `<=`, `<`, `==`, `!=`, `>=`, `>`: comparisons, evaluate to `bool`
- `&`, `|`, `^`, `~`: bit operators
- `<<`, `>>`: shift operators
- `+`, `-`, `*`, `/`, `%`, **: arithmatic operators

Todo: bitwise exclusive or, bitwise negation, left shift, right shift, unary -

Integers are restricted to the range of their declared byte sized, which ranges from 8 to 256 in increments of 8 bytes.

| type     | lower bound | upper bound   |
| -------- | ----------- | ------------- |
| `int8`   | 0           | (2**8) - 1    |      
| `int16`  | 0           | (2**16) - 1   |   
| `int24`  | 0           | (2**24) - 1   |
| ...      | ...         | ...           |
| `int256` | 0           | (2**256) - 1  |

TODO: table for uint

Arithmetic on integers is performed in "checked" mode by default since `v0.8.0`. This integrates Open Zepplin's [SafeMath](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeMath.sol) standard into arithmatic operations and will revert transactions that cause integer values to overflow or underflow. The "unchecked" mode can be used via `unchecked{ ... }.`

### Address

The address type comes in two flavors:
1. `address`: holds a 20-byte value for an Ethereum address
2. `address payable`: same as `address`, with additional members `.transfer()` and `.send()`

You can only send Ether to an `address payable`, but `address` can be converted via `payable(address)`.

- `<=`, `<`, `==`, `!=`, `>=`, `>`: comparisons, evaluate to `bool`

> Members of address types

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

### Send Ether via Transfer, Send, and Call

The are three methods available to send Ether within a contract: `transfer()`, `send()`, and `call{value:}()`.

> The key differences in these methods involve their return types, gas forwarding, behavior on failure, and where in the receiving contract they are processed.

| | gas forwarded | return types | on failure | processed in |
|-|-|-|-|-|
| `transfer()` | 2300 | none | transaction reverts | `receive()` |
| `send()` | 2300 | `(bool, bytes memory)` | returns `false` | `receive()` |
| `call{value:}("")` | all, or specified | `(bool, bytes memory)` | returns `false` | `fallback()` |

Note that `transfer()` and `send()` are both processed in the `receive()` function of the recipient account. If not is defined, they are processed in `fallback()`.

Both `transfer()` and `send()` forward limited amounts of gas. This is just enough to update `address.balance` of the recipient account, but any additional execution in `receive()` will cause the transfer to fail.

While `transfer()` and `send()` were favored in the past because they are resistant to *reentrancy attacks*, `call{value:}()` is now the favored method and the developer must implement other safeguards against reentrancy.

TODO: Why was this recommendation made? What shortcomings of tranfer() and send() surfaced the change?

### 49. Call, Delegatecall, and STATICCALL

The low-level address methods `call()`, `delegatecall()`, and `staticcall()` can be used to interface with contracts that do not adhere to the ABI or gain more control over the encoding.

Each method takes a single `bytes memory` message data parameter encoded with `abi.encode()`, `abi.encodePacked()`, `abi.encodeWithSelector`, or `abi.encodeWithSignature`. They return `(bool, bytes memory)`: a success condition and return data.

Modifiers for `gas` and `value` are used to specify the amount of gas and Ether passed to the callee. Note that `value` is not supported by `delegatecall()`.

The purpose of `delegatecall()` is to use library or logic code stored in the callee contract, but to operate on the state of the caller contract.

| | gas modifier | value modifier | reverts if |
|-|-|-|-|
| `call()` | ✅ | ✅ | none |
| `delegatecall()` | ✅ | ⬜️ | none |
| `staticcall()` | ✅ | ✅ | called function modifies state |

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

The type `byte[]` is an array of bytes, but wastes 31 bytes of space fore each element due to padding rules (except in storage). It is recommended to use `bytes` instead.

### 52. literals

Literals can be of 5 types:
1. Address literals: hexadecimal literals between 39-41 digits that pass the address checksum test. values that do not pass the checksum test produce an error.
2. Rational and integer literals: integer literals are formed from a sequence of numbers in the range 0-9. Decimal fraction literals are formed by a `.` with at least one number on one side. Scientific notation is supported, where the base can have fractions and the exponent cannot. Underscores can be used to separate digits for readability.
3. String literals: written with either double or single quotes (`"foo"` or `'bar'`). Only contain printable ASCII characters and a set of escape characters.
4. Unicode literals: Unicode literals prefixed with the keyword `unicode` can contain any valid UTF-8 sequence. They also upport the same escape sequences as string literals.
5. Hexadecimal literals: hexadecimal digits prefixed with keyword `hex` enclosed by double or single quotes.

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

Enums are a user defined solidity type. They must have between 1-256 members. The default value is the first member

```solidity
enum Fruits { APPLE, BANANA, PEACH }; // enum fruit restricting types of fruits
```

### 54. Function Types

Function types are the types for functions. Variables of function type can be assigned from functions. Function parameters of `function` type can be used to pass functions into another function call. Return variables may be of `function` type.

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

1. `memory`: lifetime limited to external function call
2. `storage`: lifetime limited to contract lifetime and location of state variables
3. `calldata`: non-modifiable, non-persistent area where function arguments are stored. behaves life memory. required for parameters of external functions but can be used for other variables.

### 56. Data Location & Assignement

Data location are relevant for both the *persistence of data* and *semantics of assignments*.

1. Assignments between `storage` and `memory` (or from `calldata`) always create an independent copy.
2. Assignments from `memory` to `memory` only create references. This means that changes to one `memory` variable are also visible in all other memory variables that refer to the same data.
3. Assignments from `storage` to a local storage variable also only assign a reference.
4. All other assignments to `storage` always copy. Examples include assignments to state variables or to members of local variables of storage `struct` type, even if the local variable itself is just a reference.

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

### 59. Variables of type `bytes` and `string` are special arrays

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

1. Unlike `storage` arrays, `memory` arrays cannot be resized. It does not have a `.push()` member function.
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

An array literal is a comma separated list of one or more expresions, enclosed in square brackets.

1. It is always a statically sized memory array whose length is the number of expressions
2. The base type of the array is the type of the first expression on the list such that all other expressions can be implicitly converted to it. Otherwise, a `TypeError` is thrown.
3. Fixed size memory arrays cannot be assigned to dynamically sized memory arrays

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

### 62. Gas cost of push and pop

Increasing the length of a storage array by calling `push()` has constant gas cost because storage is zero initialized, while decreasing the length by `pop()` depends on the size of the element being removed.

If the element is an array, it can be very costly, because it includes explicitly clearing the removed elements.

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

Array slices are a view on a contiguous portion of an array. They are written as `x[start:end]`, where start and end are expression resulting in `uint256` type (or implicitly convertivle to it). The first element of the slice is `x[start]` and the last element is `x[end-1]`

1. If `start > end` or `end > array.length`, an exception is thrown
2. Both start and end are optional. start defaults to 0 and end defaults to the length of the array.
3. Array slices do not have any Members
4. They are implicitly convertible to arrays of their underlying type and support index access. Index access is not absolute in the underlying array, but relative to the start of the slice.
5. Array slices do not have a type name which means no variable can have an array slices as type and the only exist in intermediate expressions
6. Array slices are only implemented for calldata Arrays
7. Array slices are useful to ABI decode secondary data passed in function parameters

> Slices are only implemented in solidity for `calldata`. This is useful for parsing `calldata`, for example the function selector is always the first four bytes.

```solidity
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

Structs help define new aggregate types by combining other value/reference types into one unit. Struct types can be used inside mappings and arrays. They may contain mappings and arrays. It is not possible for a struct to contain a member of its own type.

### 65. Mapping types

Mapping define key-value pairs and are declared using the syntax `mapping(_KeyType => _ValueType) _VariableName`.

1. The `_KeyType` can be any built-in value type, bytes, string, or any contract or enum type. OTher user defined or complex types (mappings, structs, or arrays) are not allowed. `_ValueType` can be any type (including mappings, structs, or arrays).
2. Key data is not stored in a mapping, only its keccack256 hash is used to look up the value.
3. Mappings do not have a length or a concept of a key or value being set
4. They can only have a data location of storage and thus allowed for state variables, storage reference types in functions, or parameters for library Functions
5. They cannot be used as parameters or return parameters of contract functions that are publicly visible. These restructions are also true for arrays and structs that contain mappings.
6. You cannot iterate over mappings, i.e. you cannot enumerate their keys. It is possible, though, tho implement a data structure on top of them and iterate over that.

### 66. Operators Involving LValues

An "LValue" is a variable, or something that can be assigned to

1. `a += e` is equivalent to `a = a + e`. The operators `-=`, `*=`, `%=`, `|=`, `&=`, `^=` act similarly.
2. `a++` and `a--` are equivalent to `a+=1/a-=1` but the expression itself still has the previous value of `a`.
3. In contrast, `--a` and `++a` have the same effect on `a`, but return the value after the change.

### 67. delete

1. `delete a` assignes the initial value for the type to `a`
2. For integers it is equivalent to `a=0`
3. FOr arrays, it assigns a dynamic array of length zero or a static array of the same length with all elements set to their initial value
4. `delete a[x]` deletes the item at index `x` of the array and leaves all other elements and the length of the array untouched
5. for `struct`, it assigns a strcut with all members resets
6. delete has no effect on mappings. If you delete a strcut, it will reset all members that are not mappings and also recurs into the members unless they are mappings
7. For mappings, individual keys and what they map to can be deleted. If `a` is a mapping, then `delete a[x]` will delete the value stored at `x`.

### 68. Implicit Conversions

An implicit type conversion is automatically applied by the compiler in some cases when making assignments, passing arguments, or applying operations.

Implicit conversion between value-types is possible if it makes sense semantically and no information is lost. For example, `uint8` is convertible to `uint16`, and `int128` to `int256`, but `int8` is not convertible to `uint256` because `uint256` cannot hold values such as `-1`.

### 69. Explicit Conversion

IF the compiler does not allow implicit conversion but a developer is reasonably confident that a conversion will work, an explicit type conversion is possible. This may result in unexpected behavior and bypasses some security features of the complier (e.g. `int` to `uint`)

1. If an integer is explicitly converted to a smaller type, higher order bits are cut off
2. If an integer is explicitly converted to a larger type, it is padded on the left (the higher order end)
3. Fixed size bytes types explicitly converted to a smaller type will cut of the bytes to the right
4. Fixed size bytes types explicitly converted to a larger type will pad bytes to the right.

### 70. Conversion between literals and elementary types

1. Decimal and hexadecimal number literals can be implicitly converted to any integer type that is large enough to represent it without trucation
2. Decimal number literals cannot be implicitly converted to fixed size byte arrays
3. Hexadecimal number literals can be, but only if the number of hex digits exactly fits the size of the bytes type. As an exception both decimal and hexadecimal literals which have a value of zero can be converted to any fixed size bytes type
4. String literals and hex string literals can be implicitly converted to fixed size byte arrays, if their number of characters matches the size of the bytes type

### 71. A literal number can take a suffix of wei, gwei, or ether to specify a sub-denomination of Ether

```solidity
uint amount1 = 1 ether;
uint amount2 = 1e9 gwei;
uint amount3 = 1e18 wei;
assert( amount1 == amount2 & amount2 == amount3 );
```

### 72. Suffixes like seconds, minutes, hourse, days, and weeks after literal numbers can be used to specify units of time where seconds are the base unit

Be careful when applying these calendar calculations, because not every year equals 365 days (leap years) and not every day has 24 hours (leap seconds).

The suffixes cannot be applied directly to variables but can be applied by multiplication.

```solidity
assert( 1 == 1 seconds );
assert( 1 minutes == 60 seconds);
assert( 1 hours == 60 minutes );
assert( 1 days == 24 hours );
assert( 1 weeks == 7 days );
```

### 73. Block and Transaction Properties

> members of block object

| member | return type | description |
| ------ | ------- | --------- |
| `block.chainid` | `uint` | current chain id |
| `block.coinbase(address payable)` | `address payable` | current block miners address |
| `block.difficulty` | `uint` | current block difficulty |
| `block.gaslimit` | `uint` | current block gaslimit |
| `block.number` | `uint` | current block number |
| `block.timestamp` | `uint` | current block timestamp as seconds since unix epoch |
| `blockhash(uint blockNumber)` | `bytes32` | hash of given block (only 256 most recent) |

 > Members of msg object

| member | return type | description |
| ------ | ------- | --------- |
| `msg.data` | `bytes calldata` | complete calldata |
| `msg.sender` | `address` | sender of the message (current call) |
| `msg.sig` | `bytes4` | first four bytes of calldata (function identifier) |
| `msg.value` | `uint` | number of wei sent with message |

> Members of tx object

| member | return type | description |
| ------ | ------- | --------- |
| `tx.gasprice` | `uint` | gas price of the transaction |
| `tx.origin` | `address` | sender of the transaction (full call chain) |
| `gasleft()` | `uint256` | remaining gas in transaction |

### 74. The values of all members of `msg` can change for every external function call, including library functions.

### 75. Avoid randomness via block timestamps or hashes

Do not rely on `block.timestamp` or `blockhash` as a source of randomness. Both can be influenced by miners. The current block timestamp must be strictly larger than the timestamp of the last block, but the only guarantee is that is will be somewhere between the timestamps of two consecutive blocks in the canonical chain

### 76. Non-archival clients only store 256 most recent blocks

This is for scalability reasons. Thus, solidity can only access `blockhash(uint blockNumber)` for the 256 most recent blocks. Otherwise it returns zero.

### 77. ABI encoding and decoding functions

| member | return type | description |
| `abi.decode(bytes memory encodedData, (...))` | `(...)` | decodes given data, while the types are given in parentheses as second argument |
| `abi.encode(...)` | `bytes memory` | encodes given arguments |
| `abi.encodePacked(...)` | `bytes memory` | packed encoding of given arguments, can be ambiguous! |
| `abi.encodeWithSelector(bytes4 selector, ...)` | `bytes memory` | encodes given arguments starting from the second and prepends given four byte selector |
| `abi.encodeWithSignature(string memory signature, ...)` | `bytes memory` | equivalent to `abi.encodeWithSelector(bytes4(keccack256(bytes(signature))), ...)` |

### 78. Error handling functions

> Solidity functions useful for handling errors

| function | on failure |
| -------- | ---------- |
| `assert(bool condition)` | throws panic error, reverts state changes - used for internal errors |
| `require(bool condition)` | reverts if condition not met - used for input errors or external components |
| `require(bool condition, string memory message)` | also provides an error message |
| `revert()` | aborts execution and reverts state changes |
| `revert(string memory reason)` | also provides an explanatory string |

### 79. Mathematical and cryptographic functions

> Several solidity functions useful for cryptographic applications

| function | return type | description |
|-|-|-|
| `addmod(uint x, uint y, uint k)` | `uint` | computes `(x+y)%k` where addition is performed with arbitrary precision and does not wrap arround `2**256`. Assert(k!=0) enforces from version 0.5.0 |
| `mulmod(uint x, uint y, uint k)` | `uint` | computer `(x*y)%k` where the multiplication is performed with arbitrary precision and does not wrap around `2**256`. assert(k!=0). |
| `keccak256(bytes memory)` | `bytes32` | computes keccak256 hash of input |
| `sha256(bytes memory)` | `bytes32` | computes SHA256 hash of input |
| `ripemd160(bytes memory)` | `bytes20` | computes RIPEMD-160 hash of input |
| `ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s)` | `address` | recovers address associated with the public key from elliptic curve signature or return zero on error. does not return `address payable`. |

### 80. Ecrecover

When using `ecrecover`, a valid signature can be turned into a different valid signature without requiring knowledge of the corresponding private key. This is usually not a problem unless you require signature to be unique or use them to identify items. OpenZepplin has an ECDSA helper library to wrap `ecrecover` without this issue.

### 81. Contract related objects

1. `this` is the current contract, explicitly convertible to `address`
2. `selfdestruct(address payable recipient)` destroys the current contract, sending funds to `recipient` and ending execution

### 82. `selfdestruct()` has some peculiarities

The receiving contract's `receive()` function is not executed and the contract is only destroyed at the end of the transaction. A revert may undo the destruction.

### 83. Type information

The expression `type(X)` can be used to retrieve information about the type `X`, where `X` can either be a contract or an integer type.

#### Contract type information

For a contract type `C` the following is available:
1. `type(C).name`: the name of the contract
2. `type(C).creationCode`: memory byte array containing the creation bytecode of the contract. Used for inline assembly to build custom creation routines, especially using the `CREATE2` opcode. This property cannot be accessed in the contract itself or any derived contract. It causes the bytecode to be included in the bytecode of the call site and thus circular references like that are not possible.
3. `type(C).runtimeCode`: memory byte array containing runtime bytecode of the contract. This code is usually deployed by the constructor of C. If the constructor of C uses inline assembly, this might be different from the actual deployed bytecode. Note that libraries modify their runtime bytecode at the time of deployment to guard against regular calls. Same restrictions as with `.creationCode` also apply here.

For an interface type `I` the following is available:
- `type(I).interfaceId`: bytes4 value containing EIP-165 interface identifier of a given interface I. The identifier is defind as the XOR of all function selectors defined within the interface itself, excluding inherited functions.

#### 84. Integer type information

For an integer type T, the following is available:
1. `type(T).min`: smallest value representable by type T
2. `type(T).max`: largest value representable by type T

### 85. Control structures

Solidity has `if`, `else`, `while`, `do`, `for`, `break`, `continue`, `return` with usual semantics known for C or Javascript.

1. Parentheses can not be ommitted for conditionals, but curly braces can be ommitted around single-statement bodies
2. Note that there is no type conversion from non-boolean to boolean types as there is in C and JavaScript, so `if (1){...}` is not valid in Solidity.
