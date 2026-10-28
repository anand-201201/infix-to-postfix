# Infix to Postfix Conversion (RPN) in C

A C implementation of the **Shunting-Yard style algorithm** that converts a human-readable arithmetic expression (infix notation) into a parenthesis-free **Postfix / Reverse Polish Notation (RPN)** string, using a Last-In, First-Out (LIFO) stack.

```
Input  (Infix)  : (A+B)*(C-D)^E
Output (Postfix): AB+CD-E^*
```

---

## Table of Contents

- [Aim](#aim)
- [Why Postfix?](#why-postfix)
- [Features](#features)
- [Project Structure](#project-structure)
- [Operator Precedence](#operator-precedence)
- [Algorithm](#algorithm)
- [Worked Example](#worked-example)
- [Getting Started](#getting-started)
- [Sample Runs](#sample-runs)
- [How the Code Works](#how-the-code-works)
- [Known Limitations](#known-limitations)
- [Possible Improvements](#possible-improvements)
- [Complexity](#complexity)
- [Conclusion](#conclusion)

---

## Aim

To design, analyze, and implement an algorithmic framework that converts standard arithmetic expressions (infix notation) into machine-efficient, parenthesis-free linear sequences (postfix notation / RPN) using a stack data structure.

## Why Postfix?

In infix notation (`A + B * C`), the evaluator must know operator precedence and associativity, and must handle parentheses. In postfix notation (`A B C * +`) the order of operations is already encoded in the sequence itself:

- No parentheses are needed.
- No precedence rules are needed at evaluation time.
- A compiler or calculator can evaluate the expression in a **single left-to-right pass** with one stack, in **O(n)** time.

## Features

- Converts infix expressions to postfix using a character stack.
- Supports the operators `+  -  *  /  %  ^`.
- Supports three bracket types: `( )`, `[ ]`, `{ }`.
- Operands are single alphabetic characters (`A-Z`, `a-z`).
- Basic detection of mismatched closing brackets (prints `invalid expression`).
- Clean modular layout: header, driver, and stack/priority operations in separate files.

## Project Structure

```
C project/
├── header.h       # Includes, extern globals, function prototypes
├── Main.c         # Driver: reads input, runs the conversion loop
└── operation.c    # push(), pop(), check_priority()
```

| File | Responsibility |
|------|----------------|
| `header.h` | Standard includes (`stdio.h`, `string.h`, `stdlib.h`), `extern` declarations for the shared globals, and prototypes for `check_priority`, `push`, `pop`. |
| `Main.c` | Reads the infix string with `fgets`, scans it left to right, applies the conversion rules, and prints the postfix result. |
| `operation.c` | Stack primitives and operator priority lookup. `pop()` also appends popped operators to the output (brackets are discarded). |

### Shared global variables

| Variable | Type | Purpose |
|----------|------|---------|
| `str[100]` | `char[]` | Input infix expression |
| `New[100]` | `char[]` | Output postfix expression |
| `stack[100]` | `char[]` | Operator stack |
| `i` | `int` | Scan index into `str` |
| `j` | `int` | Write index into `New` |
| `top` | `int` | Stack top index (starts at `-1` = empty) |
| `count` | `int` | Count of currently open brackets |

## Operator Precedence

| Priority | Operators | Associativity |
|:--------:|-----------|---------------|
| **3** (highest) | `^` (exponentiation) | Right-to-Left |
| **2** | `*`  `/`  `%` | Left-to-Right |
| **1** (lowest) | `+`  `-` | Left-to-Right |
| 0 | anything else / opening brackets | n/a |

## Algorithm

1. Initialize an empty output string and an empty operator stack.
2. Scan the infix expression token by token, left to right.
3. **Operand** (letter): append it directly to the output.
4. **Opening bracket** `(`, `[`, `{`: push it onto the stack.
5. **Closing bracket** `)`, `]`, `}`: pop operators to the output until the matching opening bracket is found; discard both brackets.
6. **Operator**: while the operator on top of the stack has higher or equal precedence, pop it to the output. Then push the scanned operator.
7. When the input is exhausted, pop all remaining operators to the output.

## Worked Example

Input: `(A + B) * (C - D) ^ E`  →  Output: `A B + C D - E ^ *`

| Token | Stack | Postfix Output | Action |
|:-----:|:-----:|----------------|--------|
| `(` | `[(]` | | Push opening parenthesis |
| `A` | `[(]` | `A` | Append operand |
| `+` | `[(, +]` | `A` | Push operator |
| `B` | `[(, +]` | `A B` | Append operand |
| `)` | `[]` | `A B +` | Pop until `(`, discard parentheses |
| `*` | `[*]` | `A B +` | Push operator onto empty stack |
| `(` | `[*, (]` | `A B +` | Push opening parenthesis |
| `C` | `[*, (]` | `A B + C` | Append operand |
| `-` | `[*, (, -]` | `A B + C` | Push operator |
| `D` | `[*, (, -]` | `A B + C D` | Append operand |
| `)` | `[*]` | `A B + C D -` | Pop until `(`, discard parentheses |
| `^` | `[*, ^]` | `A B + C D -` | `^` has higher priority than `*`, push |
| `E` | `[*, ^]` | `A B + C D - E` | Append operand |
| End | `[]` | `A B + C D - E ^ *` | Pop remaining operators |

## Getting Started

### Prerequisites

- A C compiler such as **GCC** or **Clang**
- A terminal (Linux, macOS, or Windows with MinGW / WSL)

### Important: define the globals

`header.h` declares the shared variables as `extern`, so they must be **defined exactly once** in one of the `.c` files. If they are not, the linker fails with `undefined reference to 'str'` (and similar errors for `New`, `stack`, `i`, `j`, `top`, `count`).

Add the following definitions near the top of `Main.c` (after the `#include`):

```c
#include "header.h"

char str[100], New[100], stack[100];
int  i, j, top = -1, count;   /* top = -1 means the stack is empty */
```

> `top` **must** be initialized to `-1`. Global `int`s default to `0`, which would make the stack look like it already contains one element.

### Build

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd "<your-repo>/C project"
gcc Main.c operation.c -o infix2postfix
```

### Run

```bash
./infix2postfix        # Linux / macOS
infix2postfix.exe      # Windows
```

```
enter the statement:(A+B)*(C-D)^E
AB+CD-E^*
```

> **Tip:** Enter the expression **without spaces**, e.g. `(A+B)*(C-D)^E`.

## Sample Runs

| Infix Input | Postfix Output |
|-------------|----------------|
| `(A+B)*(C-D)^E` | `AB+CD-E^*` |
| `A+B*C` | `ABC*+` |
| `A*B+C` | `AB*C+` |
| `A-B-C` | `AB-C-` |
| `A+(B*C)-D` | `ABC*+D-` |

## How the Code Works

**`check_priority(char ch)`**: returns `3` for `^`, `2` for `*`, `/`, `%`, `1` for `+`, `-`, and `0` for everything else (including brackets).

**`push(char ch)`**: increments `top` and stores the character on the stack.

**`pop()`**: reads the top element, appends it to `New[]` **unless it is a bracket**, decrements `top`, and returns the character.

**`main()`**:
1. Prompts for and reads the expression using `fgets`, stripping the trailing newline.
2. For each character:
   - Letter → write straight to `New[]`.
   - Opening bracket → `push` and increment `count`.
   - Closing bracket → `pop` until the matching opener, validate it, decrement `count`.
   - Operator → compare priority with the stack top and pop/push accordingly.
3. After the loop, pops everything left on the stack, then prints the result with `puts(New)`.

## Known Limitations

These are behaviors of the current implementation that you should be aware of:

- **Single-letter operands only.** Digits and multi-character identifiers (e.g. `12`, `sum`) are not supported.
- **Right-associativity of `^` is not handled.** `A^B^C` produces `AB^C^`; the mathematically correct result is `ABC^^`.
- **Spaces are not skipped.** Whitespace is treated as an operator character, so enter expressions without spaces.
- **Unbalanced brackets are not fully validated.** For example, `(A+B` (missing `)`) still produces output, and an extra closing bracket such as `A+B)` is not reported cleanly. The `count` variable is tracked but never checked at the end.
- **Closing brace `}` branch has a bug.** The `while(pop()!='{')` loop is missing its terminating `;`, so the validation `if` is absorbed into the loop body. Expressions containing `{ }` can therefore be wrongly reported as `invalid expression`.
- **Fixed buffer sizes (100 chars)**, with no overflow checks on `New[]` or `stack[]`.
- **Global state** is used for all data, so the code is not reusable or thread-safe as written.

## Possible Improvements

- Add the missing `;` in the `}` handler: `while(pop()!='{');`
- Handle right-associativity for `^` (pop only when the stack-top priority is strictly greater for `^`).
- Skip whitespace characters and support digits / multi-character operands.
- Check `count != 0` after the scan to report unbalanced brackets.
- Add bounds checking on all buffers.
- Add a postfix **evaluator** to compute the result of numeric expressions.
- Pass the stack and buffers as parameters or a struct instead of using globals.
- Add a `Makefile` and unit tests.


## Conclusion

Converting infix to postfix with a stack removes the need to manage operator hierarchy and parentheses at evaluation time. The resulting Postfix String Format (PSF) can be executed by a compiler or interpreter through a simple sequential stack traversal in linear time.

---

