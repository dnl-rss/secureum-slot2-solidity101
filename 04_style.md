### 97. Programming style

Coding converntions for writing solidity code. Style is about consistency. Consistency with style is important. Consistency within a project is more important. Consistency within one module or function is event more important.

There are two main categories of style:
1. Layout
2. Naming Conventions

Programming style affect readability and maintainability, both of which affect security.

### 98. Code layout

1. Indentation: 4 spaces per indentation level
2. Tabs or Spaces: 4 spaces are preferred. Avoid mixing tabs with spaces.
3. Blank Lines: Surround top level declarations in solidity source with two blank lines.
4. Max Line Length: Keep lines to a max length of 79 (or 99) characters
5. Wrapped lines should:
    1. not attach the argument to the opening parenthesis
    2. One, and only one, indent should be Used
    3. Each argument should fall on its own line
    4. The terminating element `)` should be placed on the final line by itself
6. Source file encoding: UTF-8 or ASCII encoding preferred.
7. Imports: import statements should always be placed at the top of the file
8. Order of Functions: Ordering helps readers identify which functions they can call, and to find the `constructor()` and `fallback()` definitions easily. Functions should be grouped by visibility and ordered as: constructor -> receive -> fallback -> external -> public -> internal -> private. Within a grouping, place the view and pure functions last.

### 99. More code layout

1. Whitespace in expressions: Avoid extraneous whitespace in the following situations -- immediately inside parenthesis, brackets or braces, with the exception of single line function declarations
2. Control structures: braces denoting the body of a contract, library, functions, and structs should:
     1. Be open on the same line as the declaration
     2. close on their own line at the same indentation level as the beginning of the declaration
     3. The opening brace should be preceded by a single space
3. Function declaration: for short function declarations, keep the opening brace of the function body on the same line as the function declaration. The closing brace should be at the same indentation level as the function declaration. The opening brace should be preceded by a single space.
4. Mappings: in variable declarations, do not sparate any nested mapping keyword from its type by whitespace.
5. Variable declarations: Declarations of array variables should not have a space between the type and the brackets.
6. Strings: use double quotes instead of single quotes
7. Operators: surround operators with a single space on either side. Operators with higher priority can exclude surrounding whitespace in order to denote precedence. THis is meant to improve readability for complex statements. You should always use the same amount of whitespace on either side of an operator.
8. Layout of contract elements: pragma, import, interfaces, libraries, contracts. Inside each contract, library, or interface use this order: type declarations, state variables, events, and functions.

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
2. Avoid: `l` lowercase letter "el", `O` uppercase letter "oh", and `I` uppercase letter "eye." Never use any of these for single letter variable names. They are often indistinguable from numerals `1` and `0`.
3. Names of contracts and libraries should match their filenames. If a contract file includes multiple contracts or librieis, then the filename should match the core contract, but avoid this if possible.

| type | style | examples |
| ---- | ----- | -------- |
| Contracts and libraries | CapsWords | SimpleToken, SmartBank, CertificateHashRepository, Player, Congress, Owned |
| Structs | CapsWords | MyCoin, Position, PositionXY |
| Events | CapsWords | Deposit, Transfer, Approval, BeforeTransfer, AfterTransfer |
| Function | mixedCase | getBalance(), transfer(), verifyOwner(), addMember(), changeOwner() |

### 101. More Naming Conventions

| type | style | examples |
| ---- | ----- | -------- |
| function arguments | mixedCase | initialSupply, account, recipientAddress, senderAddress, newOwner |
| local and state variables | mixedCase | totalSupply, remainingSupply, balanceOf, creatorAddress, isPreSale, tokenExchangeRate |
| constants | UPPERCASE_WITH_UNDERSCORES | MAX_BLOCKS, TOKEN_NAME, TOKEN_TICKER, CONTRACT_VERSION |
| modifiers | mixedCase | onlyBy, onlyAfter, onlyDuringThePreSale |
| enum | CapWords | TokenGroup, Frame, HashStyle, CharacterLocations |

To avoid naming collisions with a built-in type and reserved names, use single_trailing_underscore_
