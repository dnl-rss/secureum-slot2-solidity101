### 97. Programming style

Programming style involved coding conventions adopted for a programming language, which are not enforced by a compiler.
- Style is about consistency.
- Consistency with style is important.
- Consistency within a project is more important.
- Consistency within one module or function is event more important.

There are two main categories of style:
1. Layout
2. Naming Conventions

Programming style affect readability and maintainability, both of which affect security.

### 98. Code layout

1. Indentation: 4 spaces per indentation level
2. Tabs or Spaces: 4 spaces are preferred. Avoid mixing tabs with spaces.
3. Blank Lines: Surround top level declarations in solidity source with 2 blank lines.
4. Max Line Length: Keep lines to a max length of 79 (or 99) characters
5. Wrapped lines should:
    1. not attach the argument to the opening parenthesis
    2. One, and only one, indent should be Used
    3. Each argument should fall on its own line
    4. The terminating element `)` should be placed on the final line by itself
6. Source file encoding: UTF-8 or ASCII encoding preferred.
7. Imports: import statements should always be placed at the top of the file
8. Order of Functions: Ordering helps readers identify which functions they can call, and to find the `constructor()` and `fallback()` definitions easily. Functions should be grouped by visibility and ordered as such:
    - constructor
    - receive
    - fallback
    - external
    - public
    - internal
    - private
9. Mutability of Functions: within each grouping above, place `view` and `pure` functions last

### 99. More code layout

1. Whitespace in expressions: Avoid extraneous whitespace in the following situations
    - immediately inside parenthesis
    - brackets or braces (with the exception of single line function declarations)
2. Control structures: braces denoting the body of a `contract`, `library`, `function`, or `struct` should:
     - Precede the opening brace with a single space
     - Open braces on the same line as the declaration
     - Close braces on their own line, at the same indentation level as the beginning of the declaration
3. Mapping declaration: do not separate any nested mapping keyword from its type by whitespace `mapping(address -> int)`
4. Array declarations: array variables should not have a space between the type and the brackets `int[]`
5. Strings: use double quotes instead of single quotes `"abc"`
6. Operators: surround operators with a single space on either side `a + b`
    - Operators with higher priority can exclude surrounding whitespace in order to denote precedence `a*b + c`
    - Use the same amount of whitespace on either side of an operator
7. Layout of contract elements:
    - pragmas
    - imports
    - interfaces
    - libraries
    - contracts
8. Inside each contract, library, or interface use this order:
    - type declarations
    - state variables
    - events
    - functions

### 100. Naming convention

1. Naming style conventions:
    - lowercase
    - lower_case_with_underscores
    - UPPERCASE
    - UPPER_CASE_WITH_UNDERSCORES
    - CapitalizedWords
    - mixedCase
    - Capitalized_Words_With_Underscores
    - single_trailing_underscore_
2. Avoid these single letter names. They are hard to distinguish from numerals `1` or `0`:
    - `l` lowercase letter "el"
    - `O` uppercase letter "oh"
    - `I` uppercase letter "eye."
3. Names of contracts and libraries should match their filenames.
    - If a contract file includes multiple contracts or libraries, then the filename should match the core contract, but avoid this if possible.

> Use these naming styles for each type

| type | style | examples |
| ---- | ----- | -------- |
| Contracts and libraries | CapsWords | SimpleToken, SmartBank, CertificateHashRepository, Player, Congress, Owned |
| Structs | CapsWords | MyCoin, Position, PositionXY |
| Events | CapsWords | Deposit, Transfer, Approval, BeforeTransfer, AfterTransfer |
| Function | mixedCase | getBalance(), transfer(), verifyOwner(), addMember(), changeOwner() |

### 101. More Naming Conventions

> Use these naming styles for each type

| type | style | examples |
| ---- | ----- | -------- |
| function arguments | mixedCase | initialSupply, account, recipientAddress, senderAddress, newOwner |
| local and state variables | mixedCase | totalSupply, remainingSupply, balanceOf, creatorAddress, isPreSale, tokenExchangeRate |
| constants | UPPERCASE_WITH_UNDERSCORES | MAX_BLOCKS, TOKEN_NAME, TOKEN_TICKER, CONTRACT_VERSION |
| modifiers | mixedCase | onlyBy, onlyAfter, onlyDuringThePreSale |
| enum | CapWords | TokenGroup, Frame, HashStyle, CharacterLocations |

To avoid naming collisions with a built-in type and reserved names, use single_trailing_underscore_
