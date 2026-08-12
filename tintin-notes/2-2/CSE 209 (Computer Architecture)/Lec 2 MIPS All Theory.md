
The chapter is fundamentally about one question:

> **How does a high-level program eventually become simple operations that the processor can actually execute?**

The slides use **MIPS** as the model instruction set.

---

# 1. Pages 1–3 — Instruction Set and MIPS

## What is an instruction set?

An **instruction set** or **ISA — Instruction Set Architecture** is the set of operations a processor understands.

For example, a processor may understand operations corresponding to:

```text
add
subtract
load from memory
store to memory
compare
branch
jump
```

So if you write:

```c
x = a + b;
```

the CPU does not understand C.

Eventually it receives a machine instruction representing something like:

```text
ADD register1, register2, register3
```

### Important distinction

The ISA is the interface between:

```text
Software
   ↓
Instruction Set Architecture
   ↓
Hardware
```

You can think of it as the processor's **language specification**.

Different processor families have different ISAs:

```text
MIPS
ARM
x86
RISC-V
...
```

But they all need broadly similar functionality.

---

# 2. Why does the chapter use MIPS?

The slides use MIPS because it is representative of many modern ISAs and has a relatively simple, regular design.

The important idea is not that you are expected to become a MIPS historian.

MIPS is being used because its structure makes the relationship between:

```text
C/C++
↓
Assembly
↓
Machine instructions
↓
Hardware
```

easy to see.

---

# 3. RISC idea

MIPS is commonly categorized as a **RISC** architecture:

> **Reduced Instruction Set Computer**

The basic philosophy is:

```text
Use a relatively small set of simple,
regular instructions.
```

Instead of having one enormous instruction that performs a complicated operation, combine several simple instructions.

This theme keeps reappearing throughout the slides.

---

# 4. Pages 4–5 — Arithmetic operations

MIPS arithmetic generally follows:

```mips
add destination, source1, source2
```

So:

```mips
add $t0, $s1, $s2
```

means:

$$
t0=s1+s2  
$$

There are:

- two source operands
    
- one destination operand
    

This regularity is deliberate.

---

# 5. Design Principle 1 — Simplicity favors regularity

The first major design principle in the chapter is:

> **Simplicity favors regularity.**

Instead of designing every instruction differently, MIPS tries to keep similar operations in similar forms.

Why?

Because regular hardware is:

- easier to design,
    
- easier to decode,
    
- easier to execute quickly,
    
- easier for compilers to target.
    

So something like:

```text
ADD:  destination, source, source
SUB:  destination, source, source
AND:  destination, source, source
OR:   destination, source, source
```

is cleaner than giving every instruction its own arbitrary operand arrangement.

This is not only aesthetic. It directly affects processor implementation.

---

# 6. Pages 6–7 — Registers

The processor has a small collection of extremely fast storage locations called **registers**.

The slides describe MIPS as having:

```text
32 registers
each 32 bits wide
```

A 32-bit quantity is called a:

> **word**

The registers are numbered:

```text
0 – 31
```

but programmers normally use symbolic names such as:

```text
$t0
$s0
$a0
$v0
...
```

---

# 7. Why registers?

Suppose the CPU needs:

```text
5 + 7
```

It is much faster if both values are already inside the CPU in registers than if the CPU repeatedly accesses main memory.

Registers are therefore used for frequently accessed values.

The slides distinguish temporary and saved registers, for example `$t0–$t9` and `$s0–$s7`.

---

# 8. Design Principle 2 — Smaller is faster

Why only a small number of registers?

Why not have:

```text
1,000,000 registers?
```

Because larger hardware structures generally take longer to access.

Hence:

> **Smaller is faster.**

Registers are extremely fast partly because there are relatively few of them.

Compare:

```text
Registers:
    tiny quantity
    extremely fast

Main memory:
    enormous quantity
    slower
```

It's a classic engineering trade-off.

---

# 9. Pages 8–10 — Memory operands

Registers cannot hold everything.

Large things such as:

- arrays,
    
- structures,
    
- dynamically allocated data
    

must live in **memory**.

The important MIPS model is:

> Arithmetic instructions normally work on registers, not directly on arbitrary memory operands.

So:

```text
Memory → register
do calculation
register → memory
```

This is why we have:

```mips
lw      # load word
sw      # store word
```

---

# 10. Load versus store

## `lw`

```mips
lw $t0, 8($s3)
```

means:

> Read a word from memory address `$s3 + 8` and put that word into `$t0`.

Direction:

```text
MEMORY → REGISTER
```

---

## `sw`

```mips
sw $t0, 8($s3)
```

means:

> Take the value in `$t0` and write it to memory address `$s3 + 8`.

Direction:

```text
REGISTER → MEMORY
```

A common theory question is simply to ask you to distinguish the two.

---

# 11. Byte-addressed memory

MIPS memory in the slides is:

> **byte addressed**

That means every memory address identifies **one byte = 8 bits**.

Suppose:

```text
Address 1000
Address 1001
Address 1002
Address 1003
```

These are four individual bytes.

A MIPS word is:

```text
32 bits = 4 bytes
```

Therefore one word occupies four consecutive byte addresses.

---

# 12. Why does `A[8]` have offset 32?

For:

```c
int A[];
```

each element occupies four bytes.

Therefore:

$$  
\text{byte offset}=i\times4  
$$

So:

```text
A[0] → offset 0
A[1] → offset 4
A[2] → offset 8
...
A[8] → offset 32
```

This follows directly from byte addressing.

---

# 13. Word alignment

The slides say words are **aligned**.

A word address should be a multiple of four:

```text
0
4
8
12
16
...
```

rather than starting at arbitrary byte positions.

So an aligned 32-bit word could occupy:

```text
1000–1003
```

but not normally:

```text
1001–1004
```

in the model being taught.

Alignment simplifies hardware memory accesses.

---

# 14. Big Endian versus Little Endian

Suppose a 32-bit word is:

```text
0x12345678
```

It contains four bytes:

```text
12
34
56
78
```

### Big endian

Lowest memory address contains the **most significant byte**:

```text
Address 1000 → 12
Address 1001 → 34
Address 1002 → 56
Address 1003 → 78
```

### Little endian

Lowest memory address contains the **least significant byte**:

```text
Address 1000 → 78
Address 1001 → 56
Address 1002 → 34
Address 1003 → 12
```

The slides describe the MIPS model here as **Big Endian**.

Important:

> Endianness changes the ordering of bytes in memory. It does not reverse the bits inside each byte.

---

# 15. Page 11 — Registers versus memory

Why does a compiler try to keep variables in registers?

Because memory operations require additional load/store instructions.

Instead of:

```text
do calculation
```

the CPU may need:

```text
load
load
calculate
store
```

So register use saves instructions and typically improves performance.

The slides use the term:

> **spill**

If there aren't enough registers, some values must be temporarily placed in memory.

That is called **spilling registers/variables to memory**.

---

# 16. Page 12 — Immediate operands

An **immediate** is a constant encoded directly inside the instruction.

Example:

```mips
addi $s3, $s3, 4
```

The `4` is an immediate value.

Why is this useful?

Otherwise you might have to store `4` somewhere in memory and load it before adding it.

Constants are common, so supporting immediate operands is useful.

---

# 17. No `subi`

The slides point out there is no need for a separate subtract-immediate instruction.

Instead of:

```text
x - 5
```

do:

```mips
addi ..., ..., -5
```

Because:

$$ 
x-5=x+(-5)  
$$

Again, fewer instruction types means simpler ISA design.

---

# 18. Design Principle 3 — Make the common case fast

Small constants occur constantly:

```text
i++
i--
sp -= 4
x += 5
```

Therefore MIPS provides convenient immediate arithmetic.

The principle is:

> **Make the common case fast.**

Don't necessarily optimize every rare situation equally. Optimize what programs do frequently.

---

# 19. Page 13 — `$zero`

Register 0 in MIPS is:

```text
$zero
```

It always reads as:

```text
0
```

and cannot be changed.

If you try to write to it, it remains zero.

Why is that useful?

Because many common operations involve zero.

For example, conceptually copying a register:

```mips
add $t2, $s1, $zero
```

gives:

$$  
t2=s1+0=s1  
$$

So one fixed zero register simplifies many operations.

---

# 20. Pages 14–18 — Signed and unsigned numbers

This is an important theory block.

Bits by themselves do not inherently mean:

```text
positive
negative
integer
character
instruction
```

Their interpretation matters.

---

# 21. Unsigned integers

For an `n`-bit unsigned integer:

$$  
0\le x\le2^n-1  
$$

For 32 bits:

$$
0\text{ to }2^{32}-1  
$$

which is:

```text
0 to 4,294,967,295
```

The leftmost bit is simply another magnitude bit.

Example:

```text
11111111
```

as an unsigned 8-bit value is:

```text
255
```

---

# 22. Two's-complement signed integers

MIPS represents signed integers using **two's complement**.

For `n` bits, the range is:

$$  
-2^{n-1}  
\text{ to }  
2^{n-1}-1  
$$

For 32 bits:

```text
-2,147,483,648
to
+2,147,483,647
```

---

# 23. Why is the signed range asymmetric?

You get:

```text
2^32 total bit patterns
```

For signed representation:

```text
2^31 negative values
2^31 nonnegative values
```

But nonnegative includes zero.

Therefore:

```text
negative:
-2147483648 ... -1

nonnegative:
0 ... +2147483647
```

So there is one more negative value than positive value.

---

# 24. Sign bit

For a 32-bit signed number, bit 31 — the leftmost bit — acts as the sign bit:

```text
0 → non-negative
1 → negative
```

Special patterns:

```text
0:
00000000 ... 00000000

-1:
11111111 ... 11111111

largest positive:
01111111 ... 11111111

most negative:
10000000 ... 00000000
```

---

# 25. Same bits, different interpretation

Take:

```text
11111111...11111111
```

As signed two's complement:

```text
-1
```

As unsigned:

```text
4,294,967,295
```

**The bits did not change.**

Only their interpretation changed.

This becomes important later for:

```text
slt vs sltu
```

---

# 26. Two's-complement negation

To obtain `-x` from `x`:

1. invert every bit,
    
2. add 1.
    

Example using four bits:

```text
+2:
0010

Invert:
1101

Add 1:
1110
```

So:

```text
1110 = -2
```

The slide states the rule as **complement and add 1**.

---

# 27. Why can't the most-negative value be negated?

For 32 bits:

```text
minimum = -2^31
```

Its positive equivalent would be:

```text
+2^31
```

But the maximum representable signed positive integer is:

```text
2^31 - 1
```

Therefore:

```text
-(-2^31)
```

cannot fit in signed 32-bit representation.

---

# 28. Sign extension

Suppose you have an 8-bit signed value and want to represent it using 32 bits.

You must preserve its numerical value.

For positive values:

```text
00000010
```

extend using zeroes:

```text
00000000 00000000 00000000 00000010
```

For a negative value:

```text
11111110
```

you replicate the sign bit:

```text
11111111 11111111 11111111 11111110
```

This is:

> **sign extension**

The slides point out that MIPS uses sign extension for things such as `addi`, `lb`, `lh`, and branch displacements.

---

# 29. Zero extension

Unsigned data doesn't need sign preservation.

So unsigned smaller values are extended by placing zeroes on the left.

This distinction later explains:

```text
lb   → sign extend
lbu  → zero extend
```

---

# 30. Pages 19–23 — Instructions as binary

Assembly such as:

```mips
add $t0, $s1, $s2
```

is still designed for humans.

The CPU ultimately receives a **32-bit machine instruction**.

That binary representation is:

> **machine code**

Every MIPS instruction in these slides is represented by a 32-bit instruction word.

---

# 31. Why instruction formats?

The processor needs to know things such as:

```text
What operation?
Which registers?
Which constant?
Which address?
```

So the 32 bits are divided into fields.

MIPS uses a small number of regular instruction formats.

---

# 32. R-format

R-format is used by many register-to-register instructions.

Format:

```text
| op | rs | rt | rd | shamt | funct |
  6    5    5    5      5       6
```

Total:

$$  
6+5+5+5+5+6=32  
$$

The fields are:

### `op`

Opcode.

Identifies the broad operation/instruction class.

### `rs`

First source register.

### `rt`

Second source register.

### `rd`

Destination register.

### `shamt`

Shift amount.

For non-shift operations it is normally zero.

### `funct`

Function field.

Further specifies the exact operation for R-format instructions.

---

# 33. Why are register fields 5 bits?

MIPS has:

```text
32 registers
```

And:

$$  
2^5=32  
$$

Therefore exactly five bits are enough to specify any register:

```text
00000 → register 0
...
11111 → register 31
```

---

# 34. Opcode versus function

For many R-format operations the opcode identifies a common format/group, while `funct` selects the specific operation.

So the processor effectively uses both fields.

That is why `add` and `sub`, for example, can share the same broad R-format structure while having different function codes.

---

# 35. Why hexadecimal?

Writing:

```text
00000010001100100100000000100000
```

is unpleasant.

Hexadecimal compresses binary because:

$$  
1\text{ hex digit}=4\text{ bits}  
$$

Examples:

```text
0000 = 0
0001 = 1
...
1001 = 9
1010 = A
1011 = B
1100 = C
1101 = D
1110 = E
1111 = F
```

So eight hex digits represent a 32-bit quantity.

Hex is not a different underlying representation in the CPU. It is simply a more convenient human notation.

---

# 36. I-format

I-format:

```text
| op | rs | rt | immediate/address |
  6    5    5          16
```

It is used for things such as:

- immediate arithmetic,
    
- loads,
    
- stores,
    
- branches.
    

The 16-bit field may represent a constant or address-related offset.

---

# 37. Why have multiple instruction formats?

R-format needs:

```text
three register numbers
```

while immediate instructions need:

```text
two registers + constant
```

You cannot fit everything into one perfectly identical layout while retaining a fixed 32-bit instruction.

So MIPS uses multiple formats but tries to make them similar.

This leads to:

# Design Principle 4 — Good design demands good compromises

The slides explicitly make this point for instruction formats.

A design goal may conflict with another goal:

```text
Want regularity
BUT
Need different operand types
AND
Want every instruction to remain 32 bits
```

The solution is a compromise.

---

# 38. Pages 24–28 — Logical operations

Logical instructions manipulate individual bits.

The slides map common operations as follows:

```text
Shift left      sll
Shift right     srl
Bitwise AND     and / andi
Bitwise OR      or / ori
Bitwise NOT     implemented using nor
```

These operations are useful for:

- masks,
    
- extracting bit fields,
    
- inserting bit fields,
    
- testing flags,
    
- setting bits.
    

---

# 39. Shift left logical

```mips
sll $t0, $t1, 2
```

moves all bits left two positions and introduces zeroes on the right.

Example:

```text
00000101   = 5

shift left 1:

00001010   = 10
```

For values that fit:

$$  
x << i=x\times2^i  
$$

So shifting left two positions multiplies by four.

This is why array indexing uses:

```mips
sll index, index, 2
```

for 4-byte integers.

---

# 40. Shift right logical

`srl` shifts bits right and fills the new leftmost bits with zeros.

For unsigned values:

$$  
x >> i\approx x/2^i  
$$

The slides explicitly qualify the divide interpretation as **unsigned only**.

For signed negative numbers, merely inserting zeros on the left would not preserve the sign.

---

# 41. `shamt`

R-format contains:

```text
shamt
```

meaning:

> **shift amount**

It specifies how many positions an instruction such as `sll` or `srl` should shift.

Because it is five bits:

```text
0–31
```

can be represented for a 32-bit word.

---

# 42. AND and masking

Bitwise AND follows:

```text
0 & 0 = 0
0 & 1 = 0
1 & 0 = 0
1 & 1 = 1
```

Its useful property is:

```text
x & 1 → preserves bit
x & 0 → clears bit
```

Therefore a mask can **select** particular bits.

Example:

```text
value:  10110110
mask:   00001111
----------------
result: 00000110
```

The top four bits disappear.

That is why the slides describe AND as useful for **masking bits**.

---

# 43. OR and setting bits

OR:

```text
0 | 0 = 0
0 | 1 = 1
1 | 0 = 1
1 | 1 = 1
```

Useful property:

```text
x | 0 → preserve bit
x | 1 → force bit to 1
```

Therefore OR lets you set selected bits without disturbing others.

---

# 44. NOT using NOR

MIPS doesn't need a dedicated one-operand NOT instruction in this design.

Recall:

$$  
A\operatorname{NOR}B=\neg(A\lor B)  
$$

If:

```text
B = 0
```

then:

$$
A\lor0=A  
$$

therefore:

$$
A\operatorname{NOR}0=\neg A  
$$

Hence:

```mips
nor $t0, $t1, $zero
```

produces:

```text
NOT $t1
```

Again, the fixed `$zero` register is useful.

---

# 45. Pages 29–35 — Control flow

Normally the processor executes instructions sequentially:

```text
instruction 1
instruction 2
instruction 3
...
```

To implement:

```c
if
else
while
for
switch
```

you need to change which instruction executes next.

That's what **branches and jumps** do.

---

# 46. `beq`

```mips
beq rs, rt, Label
```

means:

> If `rs == rt`, continue execution from `Label`.

Otherwise continue with the next instruction.

---

# 47. `bne`

```mips
bne rs, rt, Label
```

means:

> If `rs != rt`, branch to the label.

---

# 48. `j`

```mips
j Label
```

means unconditional jump.

There is no condition.

The basic branch/jump semantics are given directly on page 29.

---

# 49. Why labels?

Humans don't want to manually write:

```text
jump 40 bytes forward
```

They write:

```mips
j Exit
```

The assembler determines the appropriate encoded target.

This is why labels are **assembler-level names for instruction locations**.

---

# 50. Basic block

A **basic block** is a sequence of instructions with:

- no branch into the middle,
    
- no branch out of the middle,
    
- except a branch may occur at the end,
    
- and the beginning may be a branch target.
    

Example:

```text
A:
instruction
instruction
instruction
branch B
```

is one basic block.

Why does it matter?

Because within a basic block the control flow is predictable:

```text
once you enter at the top,
all instructions execute sequentially
until the end.
```

Compilers use basic blocks for optimization, and processors can exploit their straight-line nature.

---

# 51. `slt`

`slt` means:

> **set on less than**

```mips
slt rd, rs, rt
```

If:

$$  
rs<rt  
$$

then:

```text
rd = 1
```

otherwise:

```text
rd = 0
```

It does not branch by itself.

You commonly combine it with `beq` or `bne`.

---

# 52. `slti`

Immediate version:

```mips
slti $t0, $s0, 10
```

means:

```text
if $s0 < 10:
    t0 = 1
else:
    t0 = 0

```

---

# 53. Why doesn't MIPS need `blt`, `bge`, etc. as basic hardware instructions?

The slides explicitly ask this.

The reasoning is:

- equality/inequality tests can be simpler/faster,
    
- more complex compare-and-branch operations require additional hardware work,
    
- making those instructions part of the critical execution path could slow instruction processing,
    
- `slt` + `beq/bne` can already express the behavior.
    

Therefore the architecture uses a compromise.

This is another example of:

> Don't judge an ISA by how many fancy instructions it has.

---

# 54. Signed versus unsigned comparison

Suppose a register contains all ones:

```text
11111111 ... 11111111
```

Signed interpretation:

```text
-1
```

Unsigned interpretation:

```text
4,294,967,295
```

Therefore comparison must know which interpretation you intend.

### Signed

```mips
slt
slti
```

### Unsigned

```mips
sltu
sltiu
```

The slides demonstrate the same bits producing opposite comparison results depending on signed versus unsigned interpretation.

---

# 55. Pages 36–45 — Procedures

You already covered the coding side, so here is the theory model.

A procedure/function call involves six conceptual stages in the slides:

1. Put parameters where the callee can access them.
    
2. Transfer control to the procedure.
    
3. Acquire any required storage.
    
4. Execute the procedure.
    
5. Put result somewhere the caller expects.
    
6. Return to the caller.
    

---

# 56. Register convention

The slide gives these roles:

```text
$a0–$a3  arguments

$v0–$v1  result values

$t0–$t9  temporary registers
          callee may overwrite them

$s0–$s7  saved registers
          callee must preserve them

$gp       global pointer

$sp       stack pointer

$fp       frame pointer

$ra       return address
```

This convention allows separately compiled functions to cooperate.

Without a convention, one function wouldn't know:

```text
Where are my arguments?
Where should I put the result?
Which registers am I allowed to destroy?
```

---

# 57. Caller and callee

If:

```c
main()
{
    f();
}
```

then:

```text
main = caller
f    = callee
```

Inside `f`, if it calls `g`:

```text
f becomes caller of g
g becomes callee
```

The labels are relational, not permanent properties of a function.

---

# 58. `jal`

`jal` means:

> **jump and link**

It:

1. records the address needed to return,
    
2. jumps to the procedure.
    

The saved return location goes into:

```text
$ra
```

---

# 59. `jr $ra`

At the end of the function:

```mips
jr $ra
```

puts the return address back into the program counter.

So control returns to the caller.

The slide presents `jal` and `jr $ra` as the core procedure-call mechanism.

---

# 60. Why save `$ra` in non-leaf functions?

Each new `jal` writes a new return address into `$ra`.

Therefore if function `f` was called by `main`, and then `f` executes:

```mips
jal g
```

the return address from `f → main` would be lost unless `f` preserved it.

Thus non-leaf procedures need to preserve their return address. The slide also says to preserve any arguments/temporaries that are needed after the nested call.

---

# 61. Leaf procedure

A **leaf procedure** calls no other procedures.

Hence it does not normally face `$ra` being overwritten by another `jal`.

But if it modifies an `$s` register, that `$s` register still has to be saved/restored.

Those are independent issues.

---

# 62. Non-leaf procedure

A **non-leaf procedure** calls another function.

Recursion is a special case:

```text
function calls itself
```

which still makes it non-leaf.

---

# 63. Stack

The stack is temporary procedure storage.

The stack pointer:

```text
$sp
```

indicates the current stack position.

In the slide's memory model the stack grows toward **lower addresses**.

So allocation usually means:

```mips
addi $sp, $sp, -N
```

and deallocation:

```mips
addi $sp, $sp, N
```

---

# 64. Procedure frame / activation record

Each invocation of a procedure may have its own **stack frame**, also called an:

> **activation record**

It can contain things such as:

- saved registers,
    
- saved return address,
    
- arguments that need preserving,
    
- local arrays,
    
- structures,
    
- other automatic local data.
    

The slide explicitly introduces this on page 44.

This is why recursion works: every recursive call gets another activation record.

---

# 65. Frame pointer

`$fp` may provide a stable reference to the current procedure frame.

Why might that help?

`$sp` can potentially move as more stack storage is allocated.

A frame pointer can remain fixed, allowing consistent offsets.

For the simple examples in your slides, `$sp` is often sufficient.

---

# 66. Page 45 — Memory layout

The process address space is divided conceptually into several regions.

From lower toward higher addresses, the slide shows roughly:

```text
Reserved

Text

Static data

Dynamic data / heap
       ↑ grows upward

       free space

Stack
       ↓ grows downward
```

---

# 67. Text segment

Contains:

> program machine instructions.

The **program counter** points to instructions in this region.

---

# 68. Static-data segment

Contains data that exists for the lifetime of the program, such as:

- global variables,
    
- static variables,
    
- constant arrays,
    
- strings.
    

The slide associates `$gp` with this region.

---

# 69. Heap / dynamic data

Used for dynamically allocated objects.

Examples from the slide:

```c
malloc(...)
```

and Java:

```java
new ...
```

It normally grows upward in the illustrated memory model.

---

# 70. Stack / automatic storage

Used for procedure-associated automatic storage.

Examples:

```text
local arrays
saved registers
saved $ra
procedure frames
```

It normally grows downward.

---

# 71. Pages 46–49 — Character data and strings

Page 46 introduces character encodings.

There are several ways to map numeric values to characters.

## ASCII

The slide says ASCII contains:

```text
128 characters
95 graphic
33 control
```

A character such as:

```text
'A'
```

is ultimately represented as a numeric code.

---

# 72. Latin-1

The slides describe Latin-1 as a 256-character byte-encoded character set containing ASCII plus additional graphic characters.

Since:

$$
2^8=256  
$$

one byte can encode 256 distinct values.

---

# 73. Unicode

ASCII is far too small for all human writing systems.

Unicode is designed to represent characters from many languages and symbol sets.

The slide describes Unicode as a 32-bit character set and mentions:

```text
Java
C++ wide characters
...
```

It also mentions variable-length encodings:

```text
UTF-8
UTF-16
```

The important theoretical distinction is:

> **Unicode defines characters/code points; encodings such as UTF-8 specify how those characters are stored as bytes.**

The slide introduces ASCII, Latin-1, Unicode, UTF-8 and UTF-16 together on page 46.

---

# 74. Byte and halfword operations

Not every memory value is a 32-bit word.

MIPS therefore provides smaller loads/stores.

### Byte

```text
8 bits
```

### Halfword

```text
16 bits
```

### Word

```text
32 bits
```

---

# 75. `lb` versus `lbu`

Both load one byte.

### `lb`

```mips
lb rt, offset(rs)
```

loads 8 bits and **sign extends** them to 32 bits.

Useful when the byte is interpreted as signed.

### `lbu`

```mips
lbu rt, offset(rs)
```

loads 8 bits and **zero extends** to 32 bits.

---

# 76. `lh` versus `lhu`

Same concept, but for 16-bit halfwords:

```text
lh  → load halfword + sign extension
lhu → load halfword + zero extension
```

---

# 77. `sb` and `sh`

```text
sb → store byte
sh → store halfword
```

The slide notes that these store only the rightmost byte or halfword of the source register.

So:

```mips
sb $t0, 0($t1)
```

does not write all 32 bits.

Only the lowest eight bits are stored.

---

# 78. Null-terminated strings

The slides' `strcpy` example uses a **null-terminated string**.

A C string:

```text
"CAT"
```

is conceptually:

```text
'C' 'A' 'T' '\0'
```

The final:

```text
'\0'
```

is a zero byte marking the end of the string.

So a copying loop continues until it copies zero.

---

# 79. Why string indexing has no `×4`

For:

```c
char s[];
```

each element occupies:

```text
1 byte
```

So:

$$  
\text{address}(s[i])=\text{base}+i  
$$

For:

```c
int A[];
```

each element occupies four bytes:

$$
\text{address}(A[i])=\text{base}+4i  
$$

This becomes particularly relevant to the previous-year recursive `char[]` question.

---

# 80. Page 50 — 32-bit constants

Here's an important limitation:

An I-format instruction only has a:

```text
16-bit immediate field
```

So how do we put an arbitrary 32-bit constant into a 32-bit register?

The slides use:

```mips
lui
```

followed by:

```mips
ori
```

---

# 81. `lui`

`lui` means:

> **load upper immediate**

It takes a 16-bit constant and places it in the upper 16 bits of a register.

The lower 16 bits become zero.

Conceptually:

```text
lui rt, ABCD

rt =
ABCD 0000
```

where each group represents 16 bits.

---

# 82. Completing the constant with `ori`

Then:

```mips
ori rt, rt, lower16
```

places desired bits into the lower half.

Suppose desired constant is:

```text
0x12345678
```

Conceptually:

```mips
lui  $t0, 0x1234

# t0 = 0x12340000

ori  $t0, $t0, 0x5678

# t0 = 0x12345678
```

This is a crucial theoretical consequence of fixed 32-bit instruction size:

> A single instruction cannot simultaneously contain a full 32-bit constant plus its opcode/register information.

---

# 83. Pages 51–55 — Addressing

There are several ways operands or targets are specified.

Page 55 summarizes the addressing modes, while pages 51–54 explain branch and jump targets.

You should know **five** major forms in this chapter.

---

# 84. Register addressing

Operand is inside a register.

Example:

```mips
add $t0, $t1, $t2
```

The source operands are:

```text
$t1
$t2
```

---

# 85. Immediate addressing

Operand is directly contained in the instruction.

```mips
addi $t0, $t1, 5
```

The `5` is the immediate operand.

---

# 86. Base/displacement addressing

Used for memory:

```mips
lw $t0, 12($s1)
```

Effective memory address:

$$  
\text{address}=\text{contents of }s1+12  
$$

So:

```text
register = base
constant = displacement/offset
```

---

# 87. PC-relative addressing

Branches commonly go to nearby instructions.

Instead of encoding the complete 32-bit target address, the branch stores an offset relative to the program counter.

The slide gives:

$$  
\text{target address}
=
PC+\text{offset}\times4  
$$

and notes that the PC has already advanced by four at this point.

Why multiply by four?

Because MIPS instructions are four bytes and are word aligned.

Therefore an offset of:

```text
1
```

means one instruction, i.e. four bytes.

This allows a 16-bit field to represent a larger byte range.

---

# 88. Why are branches PC-relative?

Most branches correspond to:

```text
if
while
for
```

and therefore jump to code located relatively nearby.

It would be wasteful to dedicate enough bits for an arbitrary complete 32-bit address every time.

So:

> **Use a small relative offset for the common case.**

Again: make the common case fast/economical.

---

# 89. Jump addressing

`j` and `jal` may need to reach farther than normal branches.

Their format provides a:

```text
26-bit address field
```

The slide calls this:

> **(pseudo)direct addressing**

The conceptual target is formed using:

```text
upper 4 PC bits
+
26 encoded bits
+
00
```

The final `00` exists because instructions are 4-byte aligned.

---

# 90. Why can't `j` contain the complete 32-bit address?

Again:

```text
instruction itself = 32 bits
```

You already need bits for the opcode.

Therefore the instruction cannot contain:

```text
opcode + complete arbitrary 32-bit target
```

inside only 32 bits.

Pseudo-direct addressing reconstructs much of the target using bits implied by alignment and bits inherited from the current PC.

---

# 91. Target-addressing example

Page 53 gives a loop at known instruction addresses and shows the encoded branch and jump fields.

The theoretical lesson is more important than the numeric example:

### Branch

Store relative distance:

```text
target - current PC
```

in instruction-sized units.

### Jump

Store the relevant 26 target bits, since the final two zeros are implied by alignment.

---

# 92. Branching too far

A branch's displacement is only 16 bits.

What if the target is beyond that range?

The assembler can transform:

```mips
beq $s0, $s1, L1
```

into conceptually:

```mips
bne $s0, $s1, L2
j   L1
L2:
```

Why does that work?

The condition is inverted:

```text
If not equal → skip jump
If equal     → execute long-range j
```

This is an excellent example of:

> The assembler can hide ISA limitations from the programmer.

---

# 93. Pages 56–57 — Synchronization

Now the chapter briefly introduces **parallel processors**.

Suppose two processors share memory.

```text
P1 writes X
P2 reads X
```

What if they access `X` at almost the same time?

The result may depend on ordering.

This can create a:

> **data race**

---

# 94. Data race

A data race occurs when concurrent agents access shared data without sufficient synchronization and at least one modifies it, so behavior can depend on the timing/order of accesses.

The simple intuition:

```text
P1 thinks X = old value
P2 changes X
P1 writes based on stale information
```

The final result becomes timing-dependent.

---

# 95. Why do we need atomic operations?

Consider:

```text
read lock
check lock
write lock
```

If those are ordinary separate operations, another processor might intervene between them.

We need something **atomic**.

Atomic means:

> The relevant operation behaves as an indivisible unit with respect to competing accesses.

The slides describe it as preventing another access from occurring between the critical read and write.

---

# 96. MIPS `ll` and `sc`

MIPS supplies an atomic pair:

```text
ll → load linked
sc → store conditional
```

---

# 97. `ll` — load linked

```mips
ll $t1, 0($s1)
```

loads a memory value, while the hardware keeps track of the relevant location/state.

---

# 98. `sc` — store conditional

Later:

```mips
sc $t0, 0($s1)
```

tries to store.

According to the slide:

### Success

If the location has not changed since `ll`:

```text
store occurs
$t0 = 1
```

### Failure

If it has changed:

```text
store fails
$t0 = 0
```

Then code can retry.

---

# 99. Why retry?

Imagine:

```text
P1: ll X

P2: modifies X

P1: sc X
```

P1's `sc` fails because its earlier knowledge of `X` is no longer safe.

So P1 starts over.

That gives software a way to implement locks and atomic updates.

---

# 100. Atomic swap and locks

Page 57 uses `ll/sc` to implement an atomic swap useful for testing/setting a lock variable.

You don't need deep operating-systems synchronization for this chapter.

Remember:

```text
Problem:
    two processors could interfere.

Solution:
    atomic operation.

MIPS mechanism:
    ll + sc.

sc returns:
    1 success
    0 failure.
```

---

# 101. Pages 58–65 — How a program becomes executable

This is one of the most theory-heavy sections.

The overall pipeline is approximately:

```text
High-level source
      ↓
Compiler
      ↓
Assembly / machine/object module
      ↓
Assembler
      ↓
Object module(s)
      ↓
Linker
      ↓
Executable image
      ↓
Loader
      ↓
Program in memory
      ↓
Execution
```

Page 58 calls this **Translation and Startup**.

---

# 102. Compiler

A compiler translates higher-level code toward machine-level representation.

Conceptually:

```c
x = y + z;
```

becomes machine/assembly operations.

The diagram notes that many modern compilers can produce object modules directly rather than necessarily producing a human-readable assembly file first.

---

# 103. Assembler

The assembler translates assembly language into machine instructions.

Example:

```mips
add $t0, $t1, $t2
```

becomes the corresponding 32-bit binary instruction.

But the assembler does more than pure one-to-one translation.

It also handles:

- labels,
    
- pseudoinstructions,
    
- object-module metadata,
    
- relocation information.
    

---

# 104. Pseudoinstructions

A **pseudoinstruction** looks like a MIPS instruction to the programmer but does not correspond directly to one real machine instruction.

The slides call them:

> “figments of the assembler's imagination.”

Example:

```mips
move $t0, $t1
```

assembler may translate to:

```mips
add $t0, $zero, $t1
```

The CPU does not need a physical `move` instruction.

---

# 105. `blt` as a pseudoinstruction

The slide gives:

```mips
blt $t0, $t1, L
```

translated to something like:

```mips
slt $at, $t0, $t1
bne $at, $zero, L
```

So one convenient assembly statement may become multiple real instructions.

---

# 106. `$at`

Register 1:

```text
$at
```

means:

> **assembler temporary**

The assembler can use it while expanding pseudoinstructions.

That is why ordinary programmer code should be careful about relying on `$at` when pseudoinstructions are being used.

---

# 107. Object module

After assembly/compilation, you may get an **object module**.

This is not necessarily a complete executable program yet.

It contains translated machine code plus information needed to combine it with other pieces.

The slide lists several parts.

---

# 108. Object-module header

Describes the contents of the object module.

Think:

```text
What sections exist?
How large are they?
Where is information located?
```

---

# 109. Text segment in an object file

Contains:

> translated machine instructions.

---

# 110. Static-data segment

Contains data allocated for the lifetime of the program/module.

Examples:

```text
global/static values
constants
```

---

# 111. Relocation information

This is important.

Suppose a module contains a reference to something whose final address isn't known yet.

For example:

```text
call function defined in another object file
```

The assembler may not know where that function will finally reside.

It therefore records:

> **relocation information**

which identifies places that may have to be fixed when final addresses become known.

---

# 112. Symbol table

Contains information about:

- global definitions,
    
- external references.
    

For example, module A may say:

```text
I define foo.
I need bar from elsewhere.
```

The linker uses symbol tables to connect those references.

---

# 113. Debug information

Associates machine-level information with the original source.

A debugger needs to know things such as:

```text
this instruction corresponds to source line 27
```

The object module may contain that mapping.

---

# 114. Linker

Programs often consist of several pieces:

```text
main.o
math.o
io.o
library code
...
```

The **linker** combines object modules into an executable image.

The slide lists three main tasks:

1. Merge segments.
    
2. Resolve labels/symbols and determine addresses.
    
3. Patch location-dependent and external references.
    

---

# 115. Resolving symbols

Suppose:

```text
main.o calls function foo
```

while:

```text
utils.o defines foo
```

During separate compilation, `main.o` may not know `foo`'s final address.

The linker eventually determines:

```text
foo is located here
```

and patches the call/reference.

---

# 116. Static linking

With **static linking**, the necessary library code is incorporated into the executable image during linking.

Advantage:

```text
everything needed is already included
```

Possible disadvantage:

```text
larger executable
duplicate library code in many programs
```

The translation/startup diagram specifically labels static linking.

---

# 117. Loader

The **loader** takes the executable image and prepares it to run.

Page 62 gives a specific sequence.

Conceptually:

1. Read the executable header.
    
2. Determine segment sizes.
    
3. Create the process's virtual address space.
    
4. Place/map text and initialized data.
    
5. Set up arguments on the stack.
    
6. Initialize registers such as `$sp`, `$fp`, `$gp`.
    
7. Transfer execution to startup code.
    

---

# 118. Startup routine and `main`

You do not generally start directly at the first line of `main`.

Runtime startup code may first:

- initialize process state,
    
- prepare arguments,
    
- prepare runtime environment.
    

Then it calls:

```text
main
```

The slide says the startup routine copies arguments into argument registers and calls `main`; when `main` returns, it performs an exit syscall.

This explains why:

> `main` is itself essentially called by startup/runtime machinery.

---

# 119. Dynamic linking

Instead of including every library routine permanently in the executable, **dynamic linking** allows a library procedure to be linked/loaded when required.

The slides give these advantages:

- avoid bloating the executable with all transitively referenced libraries,
    
- automatically use newer library versions,
    
- only link/load a procedure when called.
    

This requires the procedure/library code to be relocatable.

---

# 120. Static versus dynamic linking

A useful exam comparison:

| Static linking                      | Dynamic linking                    |
| ----------------------------------- | ---------------------------------- |
| Library code linked into executable | Library resolved/loaded later      |
| Larger executable possible          | Smaller executable possible        |
| Code already included               | Requires runtime linkage support   |
| Fixed linked copy                   | Can pick up newer library versions |

Don't confuse either with **compilation**.

Compiler and linker are different stages.

---

# 121. Lazy linkage

Page 64 shows **lazy linkage**.

The essential idea:

> Don't fully resolve a dynamically linked routine until it is actually called.

The diagram contains:

```text
indirection table
stub
linker/loader code
dynamically mapped code
```

The first call may go through a small **stub**.

The stub invokes linker/loader functionality to find/load the actual routine.

Afterward, an indirection entry can point to the resolved function so subsequent calls can proceed more directly.

You do not need to memorize implementation internals beyond the high-level mechanism unless your teacher emphasized the diagram.

---

# 122. Why “lazy”?

Because work is postponed:

```text
Not used?
    Don't resolve it.

Used?
    Resolve when first needed.
```

This follows a common systems principle:

> Avoid paying a cost until it is necessary.

---

# 123. Page 65 — Starting Java applications

This slide contrasts Java execution with straightforward native compilation.

Java source can be compiled to:

> **bytecode**

for the Java Virtual Machine — JVM.

The slide describes JVM bytecode as a relatively simple, portable instruction set.

Why portable?

Because instead of compiling directly to:

```text
MIPS
ARM
x86
```

you target:

```text
JVM bytecode
```

Then each platform has its own JVM implementation.

---

# 124. Interpretation

The JVM can:

> interpret bytecodes.

Meaning roughly:

```text
read next bytecode
determine what it means
perform operation
repeat
```

This supports portability but interpretation itself has runtime overhead.

---

# 125. JIT compilation

The slide also shows compiling **“hot” methods** into native machine code.

This is the idea of:

> **Just-In-Time compilation (JIT)**

Frequently executed methods can be translated from bytecode into native instructions for the host machine.

Then those operations need not keep being interpreted one bytecode at a time.

So the model is:

```text
Java source
   ↓
bytecode
   ↓
JVM

less-used code → interpretation

hot code      → native compilation
```

That is the main point of page 65.

---

# 126. Pages 66–67 — Fallacies

The textbook uses “fallacy” for ideas that sound reasonable but are incorrect or oversimplified.

---

# 127. Fallacy: powerful instruction ⇒ higher performance

You might think:

```text
Instruction A does 10 things.

Therefore it must be faster than
10 simple instructions.
```

Not necessarily.

The slide's argument is:

- powerful instructions may reduce instruction count,
    
- but complex instructions are harder to implement,
    
- they can make processor execution/control more complicated,
    
- complexity may hurt performance of common/simple cases.
    

So:

> **Instruction count alone is not performance.**

---

# 128. Why simple instructions can be fast

A regular/simple ISA may make:

- decoding easier,
    
- pipelining easier,
    
- implementation faster,
    
- compiler optimization easier.
    

A sequence of simple instructions may outperform one very complex instruction depending on the architecture.

This reinforces MIPS/RISC philosophy.

---

# 129. Fallacy: assembly automatically gives higher performance

Historically programmers sometimes hand-wrote assembly for maximum speed.

The slide warns against assuming:

```text
human-written assembly
>
compiler-generated machine code
```

Modern optimizing compilers understand complex processor behavior and can perform many optimizations automatically.

Additionally, assembly has disadvantages:

- far more code,
    
- lower programmer productivity,
    
- increased error risk,
    
- architecture dependence.
    

The slide therefore rejects “use assembly for high performance” as a universal rule.

---

# 130. Fallacy: backward compatibility means an ISA never changes

Page 67 states:

> Backward compatibility does **not** imply the instruction set never grows.

Instead, instruction sets can **accrete** additional instructions while retaining older ones.

The slide uses x86 as the example.

So backwards compatibility often means:

```text
old instructions continue working
+
new instructions are added
```

rather than:

```text
instruction set permanently frozen
```

---

# 131. Page 68 — Pitfalls

Unlike a fallacy, a **pitfall** is an easy mistake you can make while applying otherwise correct ideas.

The slide gives two important ones.

---

# 132. Pitfall 1 — Sequential words are not at sequential addresses

This is extremely relevant to your coding test.

Because memory is byte addressed:

```text
int A[0] → address 1000
int A[1] → address 1004
int A[2] → address 1008
```

NOT:

```text
1000
1001
1002
```

Therefore when advancing an `int*` manually:

```text
increase byte address by 4
```

not one.

The slide explicitly says:

> Increment by 4, not 1.

---

# 133. Why `char` is different

A character occupies one byte:

```text
char[0] → 1000
char[1] → 1001
char[2] → 1002
```

So whether pointer movement is `1` or `4` depends on element size.

This is precisely why array type matters.

---

# 134. Pitfall 2 — pointer to an automatic variable after function return

Consider:

```c
int *f() {
    int x = 5;
    return &x;
}
```

`x` is an automatic local variable.

Its storage belongs to `f`'s stack frame.

When `f` returns:

```text
frame is popped
```

Therefore that memory is no longer valid as `x`'s live storage.

A returned pointer to it becomes invalid/dangling.

The slide warns specifically against keeping a pointer to an automatic variable after the procedure returns.

This is a beautiful connection between:

```text
C pointer lifetime
```

and:

```text
MIPS stack frames.
```

---

# 135. Page 69 — Final design principles

The final slide summarizes four architecture design principles:

### 1. Simplicity favors regularity

Uniform designs are easier to implement efficiently.

### 2. Smaller is faster

Small structures such as register files can be accessed quickly.

### 3. Make the common case fast

Optimize common operations such as small constants and nearby branches.

### 4. Good design demands good compromises

Architecture design involves conflicting goals:

```text
simplicity
flexibility
instruction size
performance
hardware cost
```

No single design maximizes everything simultaneously.

---

# 136. Compiler → assembler → hardware

The final slide also emphasizes **layers**.

Think:

```text
C/C++ program
      ↓
Compiler
      ↓
Assembly / object code
      ↓
Assembler
      ↓
Machine instructions
      ↓
Processor hardware
```

Each layer hides details from the layer above it.

This abstraction is one of the central ideas of computer organization.

---

# 137. MIPS versus x86

The final slide says MIPS is typical of **RISC ISAs**, contrasting it with x86.

For this chapter, the takeaway is:

### MIPS/RISC style

```text
simple
regular
fixed-length instructions
load/store architecture
small number of instruction formats
```

### x86 comparison

Historically more complex and more irregular, partly due to its long backwards-compatible evolution.

The chapter is not asking you to master x86; the contrast reinforces the MIPS design philosophy.

---

# 138. The entire chapter as one mental picture

If you understand this diagram, you understand what Chapter 2 is trying to teach:

```text
                HIGH-LEVEL PROGRAM
                       |
                       v
                    Compiler
                       |
                       v
                 MIPS Assembly
                       |
                       v
                    Assembler
                       |
                       v
                  Object Module
                       |
                       v
                     Linker
                       |
                       v
                  Executable
                       |
                       v
                     Loader
                       |
                       v
              --------------------
              | Process Memory   |
              |                  |
              | Stack            |
              | Heap             |
              | Static Data      |
              | Text             |
              --------------------
                       |
                       v
                 MIPS PROCESSOR
                       |
         +-------------+-------------+
         |                           |
      Registers                    Memory
         |
         +-- arithmetic
         +-- logic
         +-- comparison
         +-- branches
         +-- procedure calls
```

Everything in the 69 slides fits somewhere in that diagram.

---

# 139. Theory questions I would expect

For a written/quiz component, make sure you can answer these without looking:

- What is an ISA?
    
- Why is MIPS considered regular/simple?
    
- Explain all four design principles.
    
- Why are registers faster but limited in number?
    
- Register versus memory.
    
- What does byte-addressed memory mean?
    
- What is word alignment?
    
- Big endian versus little endian.
    
- Immediate operand and why it is useful.
    
- Why does `$zero` exist?
    
- Signed versus unsigned range.
    
- Two's-complement representation and negation.
    
- Sign extension versus zero extension.
    
- R-format fields.
    
- I-format fields.
    
- Why multiple formats are needed.
    
- Why hex is useful.
    
- AND masking and OR bit-setting.
    
- Logical shifts and relationship to powers of two.
    
- Basic block.
    
- `slt` versus `sltu`.
    
- Why MIPS primarily uses `beq/bne` rather than every possible compare-and-branch.
    
- Caller versus callee.
    
- Leaf versus non-leaf.
    
- Role of `$a`, `$v`, `$t`, `$s`, `$ra`, `$sp`, `$fp`, `$gp`.
    
- Why `$ra` needs saving.
    
- Stack frame / activation record.
    
- Text/static/heap/stack.
    
- ASCII, Unicode, UTF-8/UTF-16 at the level shown in the slides.
    
- `lb` versus `lbu`.
    
- `lh` versus `lhu`.
    
- Why `lui` is needed.
    
- PC-relative addressing.
    
- Pseudodirect addressing.
    
- Why branch offset is multiplied by four.
    
- What happens when a branch target is too far.
    
- Main addressing modes.
    
- Data race and atomic operation.
    
- `ll/sc`.
    
- Compiler versus assembler versus linker versus loader.
    
- Pseudoinstruction.
    
- Object module components.
    
- Relocation information and symbol table.
    
- Static versus dynamic linking.
    
- Lazy linkage.
    
- JVM bytecode, interpretation and JIT.
    
- The fallacies and pitfalls on pages 66–68.
    

Those are effectively the **complete theory syllabus represented by this deck**.

The most theory-to-code connections to have completely internalized are: **byte addressing → array offsets; signedness → `slt/sltu`; sign/zero extension → `lb/lbu`; addressing modes → branches/jumps/load-store; procedure convention → stack/recursion; and character size → `lbu/sb` rather than `lw/sw`.** That is where the theory directly changes whether your MIPS translation is correct.