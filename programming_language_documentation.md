# Programming Language Specification

## Overview

This programming language (PL) is a statically typed, stack-only language compiled to a custom assembly language. It operates with a single stack and no heap, and function execution is limited to *n* steps per frame. This constraint enables aggressive compile-time optimizations such as constant evaluation of pure function calls.

## Key Properties

- **Stack-only model:** All data lives on a single stack. No heap or multiple stacks.
- **Compiled to custom assembly:** The underlying assembly language is register-based.
- **Execution step limit:** Functions may run for at most *n* steps each frame.
- **Static typing:** Types are resolved at compile time.

## Syntax Overview

### Variable Declarations

```
VARIABLES
  name1: type1
, name2: type2
, ...
END
```

It's also possible to declare variables like this:

```
VARIABLES
  name: expression
, ...
END
```

In that case, the type of the variable is inferred from the type of the expression.

### Type Aliases

```
TYPEDEFS
  alias1: type1
, alias2: type2
, ...
END
```

### Assignments

Assignments use `SET` blocks. Right-hand side (RHS) expressions in each block execute in parallel. Impure function calls cannot coexist in the same block.

```
SET
  lval1: rval1
, lval2: rval2
, ...
WITH
  symbol1: rval3
, symbol2: rval4
, ...
END
```

Execution order: all `WITH` blocks are evaluated from last to first, then the `SET` block is applied. `WITH` blocks are optional.

To swap `a` and `b` it's possible to write:

```
SET
  a: b
, b: a
END
```

### Tuples

Tuple type declaration:

```
(
  symbol1: type1
, symbol2: type2
, ...
)
```

Tuple value declaration:

```
(
  symbol1: rval1
, symbol2: rval2
, ...
)
```

The empty tuple is written `()`.

### Vectors

Vector type declaration:

```
type^number_of_elements
```

Vector value declaration:

```
(
  rval1
, rval2
, ...
)
```

Minimum vector size is 2.

### Functions

Functions take a single parameter and return a single value. Both can be tuples or vectors.

Similarly, function bodies are made of a single instruction, but this instruction can be an instruction block.

Function header declarations:

```
FUNCTION_HEADERS
  name1: in_type1:out_type1
, name2: in_type2:out_type2
, ...
END
```

Function bodies:

```
FUNCTION_BODIES
  name1: instruction1
, name2: instruction2
, ...
END
```

Instruction blocks:

```
BEGIN
  instruction1;
  instruction2;
  ...
END
```

Inside the function, the parameter is named `in` and the return value `out`. Example function body (increment input by 1):

```
SET out: in + 1 END
```

### Modules

Exporting functions:

```
EXPORT
  out_name1: in_name1
, out_name2: in_name2
, ...
END
```

Importing functions:

```
IMPORT
  FROM module_name1
    in_name1: out_name1
  , in_name2: out_name2
  FROM module_name2
    ...
END
```

`in_name` represents the name as seen from inside the module, `out_name` the name as seen from the outside of the module.

### Control Flow

No recursion is allowed. Loops (`for`, `while`) and conditional statements (`if`) are suppororted:

```
IF 
  condition1: instruction1
, condition2: instruction2
, ...
END
```

```
FOR 
  symbol in start1 ... end1: instruction1
, symbol in start2 ... end2: instruction2
, ...
END
```

```
WHILE 
  condition1: instruction1
, condition2: instruction2
, ...
END
```

### Sum Types

```
(
  symbol1: type1
| symbol2: type2
| ...
)
```

A sum type must have at least two entries.

### Enums

```
ENUM
  symbol1: constant1
, symbol2: constant2
, ...
END
```

### Pattern matching:

```
MATCH value WITH
  symbol1: x: instruction1
, symbol2: y: instruction2
, ...
END
```

`MATCH` can operate on sum types, enums and integers. It's possible to add a default option with `_`.

## Operator Compatibility

Operators can be applied to values if they are compatible:

- **Tuples:** Must have the same field names in the same order and compatible types.
- **Vectors:** Must have the same length and compatible element types.
- `u8`, `i8`, `u16`, `i16`, `u32`, `i32`, `f32` are compatible with each other.
- `bool` is not compatible with any other type.
- If the values are not of the same type, implicit casts are added **if possible**.

Operators also works on specific types. Examples:

- `||` works on `bool`.
- `>` works on `u8`, `i8`, `u16`, etc.

The language provides three operators to "fold" the vectors and tuples:

- `ALL`, which indicates if all the values are true.
- `NONE`, which indicates if all the values are false.
- `ANY`, which indicates if one or more values is true.

These operators only work on tuples and vectors ultimately made of bools.

## Implicit casts

The rule is that implicit casts can be added if they are lossless. For example, if you add an u8 to an i8, both are casted to i16.

There is no implicit cast between i32 and u32. Similarly, i32 and u32 cannot be casted to float.

## Pointers (References)

The language provides stack references ("pointers") that refer to variables on the stack. Pointers cannot be assigned or returned, preventing dangling pointers. They can be passed to functions and, since assignments are forbidden, no invalid references persist after a function returns.

## Annotations

The language may include annotations (TBD). For example:

- **@pure**: indicates a function is pure.
- **@nodiscard**: indicates the result of a function mustn't be discarded.

# Notes

- Error handling should be done with a combination of sum types (indicating either the result or an error) and the annotation @nodiscard.
- As is the language does not offer constants. However, a compiler should evaluate pire function calls with constant parameters at compile time. I'll maybe add an annotation **@const**.
