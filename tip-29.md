```
tip: 29
title: Bitwise shifting instructions in Tron
author: @ithinker1991
type: Standards Track
discussions to: https://github.com/tronprotocol/TIPs/issues/29
category: TRC
status: Final
created: 2019-03-07
```
## Simple Summary

To provide native bitwise shifting with cost on par with other arithmetic operations. Just like EIP145 in Ethereum.

## Abstract

Native bitwise shifting instructions are introduced, which are more efficient processing wise on the host and are cheaper to use by a contract.

## Motivation

TVM is lacking bitwise shifting operators, but supports other logical and arithmetic operators. Shift operations can be implemented via arithmetic operators, but that has a higher cost and requires more processing time from the host. Implementing `SHL` and `SHR` using arithmetics cost each 35 engine, while the proposed instructions take 3 engine.

## Specification

The following instructions are introduced:
**`0x1b`: `SHL` (shift left)**
The `SHL` instruction (shift left) pops 2 values from the stack, first `arg1` and then `arg2`, and pushes on the stack `arg2` shifted to the left by arg1 number of bits. The result is equal to
`(arg2 * 2^arg1) mod 2^256`
Notes:
- The value (`arg2`) is interpreted as an unsigned number.
- The shift amount (`arg1`) is interpreted as an unsigned number.
- If the shift amount (`arg1`) is greater or equal 256 the result is 0.
- This is equivalent to `PUSH1 2 EXP MUL`.

**`0x1c`: `SHR`(logical shift right)**
The `SHR` instruction (logical shift right) pops 2 values from the stack, first `arg1` and then `arg2`, and pushes on the stack `arg2` shifted to the right by `arg1` number of bits with zero fill. The result is equal to
`floor(arg2 / 2^arg1)`
Notes:

- The value (`arg2`) is interpreted as an unsigned number.
- The shift amount (`arg1`) is interpreted as an unsigned number.
- If the shift amount (`arg1`) is greater or equal 256 the result is 0.
- This is equivalent to `PUSH1 2 EXP DIV`.

**`0x1d`: `SAR`(arithmetic shift right)**
The `SAR` instruction (arithmetic shift right) pops 2 values from the stack, first `arg1` and then `arg2`, and pushes on the stack `arg2` shifted to the right by `arg1` number of bits with sign extension. The result is equal to
`floor(arg2 / 2^arg1)`
Notes:

- The value (`arg2`) is interpreted as a signed number.
- The shift amount (`arg1`) is interpreted as an unsigned number.
- If the shift amount (`arg1`) is greater or equal 256 the result is 0 if `arg2` is non-negative or -1 if `arg2` is negative.
- This is not equivalent to `PUSH1 2 EXP SDIV`, since it rounds differently. See `SDIV(-1, 2) == 0`, while `SAR(-1, 1) == -1`.
- The cost of the shift instructions is set at `verylow` tier (3 engine).

## Rationale

Instruction operands were chosen to fit the more natural use case of shifting a value already on the stack. This means the operand order is swapped compared to most arithmetic insturctions.

## Backwards Compatibility

The newly introduced instructions have no effect on bytecode created in the past.

## Test Cases

**`SHL` (shift left)**

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000001
PUSH 0x00
SHL
--
0x0000000000000000000000000000000000000000000000000000000000000001
`

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000001
PUSH 0x01
SHL
--
0x0000000000000000000000000000000000000000000000000000000000000002
`

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000001
PUSH 0xff
SHL
--
0x8000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000001
PUSH 0x0100
SHL
--
0x0000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000001
PUSH 0x0101
SHL
--
0x0000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0x00
SHL
--
0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0x01
SHL
--
0xfffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0xff
SHL
--
0x8000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0x0100
SHL
--
0x0000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000000
PUSH 0x01
SHL
--
0x0000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0x7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0x01
SHL
--
0xfffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe
`

**`SHR` (logical shift right)**

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000001
PUSH 0x00
SHR
--
0x0000000000000000000000000000000000000000000000000000000000000001
`

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000001
PUSH 0x01
SHR
--
0x0000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0x8000000000000000000000000000000000000000000000000000000000000000
PUSH 0x01
SHR
--
0x4000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0x8000000000000000000000000000000000000000000000000000000000000000
PUSH 0xff
SHR
--
0x0000000000000000000000000000000000000000000000000000000000000001
`

`PUSH 0x8000000000000000000000000000000000000000000000000000000000000000
PUSH 0x0100
SHR
--
0x0000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0x8000000000000000000000000000000000000000000000000000000000000000
PUSH 0x0101
SHR
--
0x0000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0x00
SHR
--
0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0x01
SHR
--
0x7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0xff
SHR
--
0x0000000000000000000000000000000000000000000000000000000000000001
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0x0100
SHR
--
0x0000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000000
PUSH 0x01
SHR
--
0x0000000000000000000000000000000000000000000000000000000000000000
`

**`SAR` (arithmetic shift right)**

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000001
PUSH 0x00
SAR
--
0x0000000000000000000000000000000000000000000000000000000000000001
`

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000001
PUSH 0x01
SAR
--
0x0000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0x8000000000000000000000000000000000000000000000000000000000000000
PUSH 0x01
SAR
--
0xc000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0x8000000000000000000000000000000000000000000000000000000000000000
PUSH 0xff
SAR
--
0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
`

`PUSH 0x8000000000000000000000000000000000000000000000000000000000000000
PUSH 0x0100
SAR
--
0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
`

`PUSH 0x8000000000000000000000000000000000000000000000000000000000000000
PUSH 0x0101
SAR
--
0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0x00
SAR
--
0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0x01
SAR
--
0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0xff
SAR
--
0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
`

`PUSH 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0x0100
SAR
--
0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
`

`PUSH 0x0000000000000000000000000000000000000000000000000000000000000000
PUSH 0x01
SAR
--
0x0000000000000000000000000000000000000000000000000000000000000000
`

`PUSH 0x4000000000000000000000000000000000000000000000000000000000000000
PUSH 0xfe
SAR
--
0x0000000000000000000000000000000000000000000000000000000000000001
`

`PUSH 0x7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0xf8
SAR
--
0x000000000000000000000000000000000000000000000000000000000000007f
`
`
PUSH 0x7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0xfe
SAR
--
0x0000000000000000000000000000000000000000000000000000000000000001`


`PUSH 0x7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0xff
SAR
--
0x0000000000000000000000000000000000000000000000000000000000000000`


`PUSH 0x7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
PUSH 0x0100
SAR
--
0x0000000000000000000000000000000000000000000000000000000000000000`


## Copyright

Copyright and related rights waived via [CC0](LICENSE.md).
