# [EXDL] Exoteric Description Language

An attempt at making a metasyntax: notation for context-free grammar.

This project aims at improving several pain-points of Backus-Naur Form (BNF):
- Make distinction between lexing unit (token) and parsing unit (rule)
- Create syntactical constructs for easier reading and writing by humans
- Allow for code splitting (importing and exporting tokens and rules)

## [Basics]
Tokens are defined with a `token` keyword:
```
token ampersand = "&"
```
NB: Tokens can be defined with any expression with the exeption of rules.

Rules are defined with a `rule` keyword:
```
rule program = comment | function_declaration | type_declaration
```
NB: Tokens can (and should) be used to define rules.

Any token and/or rule can be exported and import:
```
// ./logical_operators.exdl
export token or   = "|"
export token xor  = "^"

// ./rules.exdl
import { xor, or } from "./logical_operators.exdl"

rule logical_operation = term ( xor | or ) term
```

## [Constructs]
To make writing language specifications easier, there are several constructs.

### [Set]
Sets (or unions) can be defined with a pipe `|` symbol, deliniating individual set members, optionally being inclosed in parentheses `()`:
```
// sets containing single character
token plus  = "+"
token minus = "-"
// a set containing two tokens
token addition = plus | minus
// a set containing another set in addition to two tokens
token arithmetic = ( addition | "*" | "/" )
```

### [Aliasing]
Set members can be aliased to access them individually later (making a set act as a map/enum):
```
// Sets allow to namespace individual tokens/rules within itself
// Nested sets still count as normal members of parent set
token whitespace = (
  | space   = " "
  | tab     = "\t"
  | newline = ( posix = "\n" | win = "\r\n" )
)

token symbol = ( slash = "/" )

rule comment = symbol.slash{2} char* whitespace.newline

// Terminal strings can be used as a shorthand: keys will be equal to their productions
token keyword = (
  | "inline"
  | "static"
  | qualifier = { "const" | "volatile" }
)
```
NB: When using a set with aliased members, it acts as a set containing all defined expression:
```
token pointer = ( "*" | "&" )
token pointer = ( star = "*" | ampersand = "*" )
// These two declarations are equivalent
rule pointer_identifier = pointer identifier
```

### [Subset]
Subset is a syntactical sugar that allows to quickly constrain a set with given set keys:
```
token keyword = (
  | "inline"
  | "static"
  | "const"
  | "volatile"
  | qualifier = keyword.( const | volatile )
)
```

### [Multipliers]
Each expression can be adorned with a regex-style multiplier symbols:
```
// ? - Zero or one
rule token_definition = keyword.export? keyword.token identifier symbol.equals expression

// * - Zero or more
rule inline_set = symbol.pipe? expression ( symbol.pipe syntax_rule )*

// + - One or more
rule identifier = char+ ( symbol.dot char+ )*

// [MIN+(,(MAX)*)?] - Minimum (and optionally maximum) number of characters
rule inline_comment = symbol.slash{2} char* whitespace.newline
```

## [📝] License

This work is licensed under
[GPLv3](https://www.gnu.org/licenses/gpl-3.0-standalone.html) (see
[NOTICE](/NOTICE)).
