# C-to-MIPS: a Systematic Method for Procedures, Stack, Recursion, Arrays, and Multiple Calls

The difficult part is usually **not knowing individual instructions**. It is deciding:

1. Which values go into which registers.
    
2. Which values might be destroyed by a function call.
    
3. Which values must be preserved on the stack.
    
4. Where execution must return after a call.
    
5. How each recursive call keeps its own copy of variables.
    

The slides introduce the procedure convention on pages 36–45: arguments use `$a0–$a3`, results use `$v0–$v1`, temporary registers may be overwritten, saved registers must be restored, `$sp` manages the stack, and `$ra` contains the return address.

---

# 1. Your first question: could the leaf example use temporary registers?

Yes. In fact, that would be simpler.

The slide uses this C function:

```c
int leaf_example(int g, int h, int i, int j)
{
    int f;
    f = (g + h) - (i + j);
    return f;
}
```

The slide puts `f` in `$s0`. Because `$s0` is a **saved register**, the function must preserve its previous value:

```mips
leaf_example:
    addi $sp, $sp, -4
    sw   $s0, 0($sp)

    add  $t0, $a0, $a1
    add  $t1, $a2, $a3
    sub  $s0, $t0, $t1
    add  $v0, $s0, $zero

    lw   $s0, 0($sp)
    addi $sp, $sp, 4
    jr   $ra
```

That stack work is necessary **only because the function chooses to modify `$s0`**. The slide explicitly says `f` is placed in `$s0`, “hence, need to save `$s0` on stack.”

A much simpler solution is:

```mips
leaf_example:
    add  $t0, $a0, $a1       # t0 = g + h
    add  $t1, $a2, $a3       # t1 = i + j
    sub  $v0, $t0, $t1       # return value = t0 - t1
    jr   $ra
```

This version is completely valid.

## Why do we not save `$t0` and `$t1`?

Because `$t0–$t9` are temporary registers. The calling convention says a procedure is allowed to overwrite them.

The caller must assume:

```text
After jal, all $t registers may have changed.
```

By contrast, if a procedure changes `$s0–$s7`, it must return them to their original values. The slide’s register rules explicitly distinguish `$t` registers as overwriteable and `$s` registers as saved/restored.

## Important conclusion

A C local variable does **not automatically require the stack**.

This:

```c
int f;
```

does not mean:

```text
f must be stored in memory.
```

It may be kept in:

- `$t0`
    
- `$s0`
    
- `$v0`
    
- a stack location
    

depending on what the function needs.

---

# 2. The core register convention

You should know this table before solving procedure problems.

|Registers|Purpose|Preservation rule|
|---|---|---|
|`$a0–$a3`|Function arguments|Callee may overwrite|
|`$v0–$v1`|Return values|Callee places result here|
|`$t0–$t9`|Temporary values|Callee may overwrite|
|`$s0–$s7`|Saved variables|Callee must restore if modified|
|`$sp`|Stack pointer|Must return to original position|
|`$ra`|Return address|Must be saved if another `jal` will overwrite it|
|`$gp`|Global/static data pointer|Points near static data|
|`$fp`|Frame pointer|Optional stable reference for a stack frame|

Two rules explain almost everything:

> If your procedure changes an `$s` register, save and restore it.

> If your procedure executes another `jal`, save its current `$ra`.

---

# 3. What exactly does `jal` do?

The instruction:

```mips
jal function
```

performs two operations:

```text
1. Save the address of the following instruction in $ra.
2. Jump to function.
```

Returning is done using:

```mips
jr $ra
```

That copies the address in `$ra` into the program counter, causing execution to continue after the original `jal`. This is exactly the mechanism described on page 38.

Consider:

```mips
main:
    addi $a0, $zero, 5
    jal  square
    add  $s0, $v0, $zero
```

And:

```mips
square:
    mul  $v0, $a0, $a0
    jr   $ra
```

Assume the instructions have these addresses:

```text
0x00400000    addi $a0, $zero, 5
0x00400004    jal square
0x00400008    add $s0, $v0, $zero
```

When this runs:

```mips
jal square
```

MIPS performs:

```text
$ra = 0x00400008
PC  = address of square
```

At the end of `square`:

```mips
jr $ra
```

means:

```text
PC = 0x00400008
```

Therefore execution resumes here:

```mips
add $s0, $v0, $zero
```

## Think of `$ra` as a bookmark

`jal` writes a bookmark saying:

> “After this function finishes, continue from here.”

`jr $ra` follows that bookmark.

---

# 4. Why does a nested function need to save `$ra`?

Consider:

```c
int A(int x)
{
    return B(x) + 1;
}
```

Suppose `main` calls `A`:

```mips
main:
    jal A
After_A:
    ...
```

When `main` executes `jal A`:

```text
$ra = address of After_A
```

Now inside `A`:

```mips
A:
    jal B
After_B:
    ...
```

When `A` executes `jal B`, it changes `$ra` again:

```text
$ra = address of After_B
```

The original return address back to `main` has now disappeared.

Therefore `A` must save its original `$ra` before calling `B`.

```mips
A:
    addi $sp, $sp, -4
    sw   $ra, 0($sp)

    jal  B

    lw   $ra, 0($sp)
    addi $sp, $sp, 4

    addi $v0, $v0, 1
    jr   $ra
```

Without saving `$ra`, the final:

```mips
jr $ra
```

would jump to `After_B`, not back to `main`.

That is why the slide says a non-leaf procedure must save its return address and any arguments or temporary values needed after the nested call.

---

# 5. Leaf versus non-leaf procedures

## Leaf procedure

A leaf procedure calls no other function.

```c
int addTwo(int x, int y)
{
    return x + y;
}
```

```mips
addTwo:
    add $v0, $a0, $a1
    jr  $ra
```

It does not need to save `$ra`, because it never executes another `jal`.

## Non-leaf procedure

A non-leaf procedure calls another procedure.

```c
int calculate(int x)
{
    return square(x) + 5;
}
```

```mips
calculate:
    addi $sp, $sp, -4
    sw   $ra, 0($sp)

    jal  square

    addi $v0, $v0, 5

    lw   $ra, 0($sp)
    addi $sp, $sp, 4
    jr   $ra
```

Because `calculate` executes `jal square`, it must preserve the address of its own caller.

---

# 6. What exactly is the stack?

The stack is an area of memory used for temporary procedure data.

It commonly stores:

- Saved `$ra`
    
- Saved `$s` registers
    
- Arguments needed after another call
    
- Temporary values needed after another call
    
- Local arrays and structures
    
- Other local variables that cannot remain in registers
    

The slide calls one function’s stack storage a **procedure frame** or **activation record**.

## The stack grows downward

“Downward” means toward smaller addresses.

Suppose initially:

```text
$sp = 1000
```

To allocate 8 bytes:

```mips
addi $sp, $sp, -8
```

Now:

```text
$sp = 992
```

The newly allocated bytes are approximately:

```text
Address 992 to 999
```

You can use two word-sized locations:

```text
0($sp) = address 992
4($sp) = address 996
```

For example:

```mips
sw $a0, 0($sp)
sw $ra, 4($sp)
```

Stack picture:

```text
Higher address

1000        Previous stack boundary
 996        Saved $ra       4($sp)
 992        Saved $a0       0($sp)  <- $sp

Lower address
```

To free the frame:

```mips
addi $sp, $sp, 8
```

Now `$sp` returns to `1000`.

---

# 7. Adjusting `$sp` does not save anything

This is a crucial distinction.

```mips
addi $sp, $sp, -8
```

does not place any values in memory.

It only changes where `$sp` points.

You must separately store values:

```mips
sw $ra, 4($sp)
sw $a0, 0($sp)
```

Think of it as reserving two empty lockers:

```mips
addi $sp, $sp, -8
```

Then putting values into the lockers:

```mips
sw $a0, 0($sp)
sw $ra, 4($sp)
```

When returning:

```mips
lw $a0, 0($sp)
lw $ra, 4($sp)
addi $sp, $sp, 8
```

The order is:

```text
Allocate → Store → Use → Load → Deallocate
```

---

# 8. Difference between `8($s3)` and changing `$sp`

These are completely different operations.

## Memory addressing

```mips
lw $t0, 8($s3)
```

means:

```text
Read the word from memory address ($s3 + 8)
and place it into $t0.
```

It does not change `$s3`.

For example:

```text
$s3 = 1000
```

Then:

```mips
lw $t0, 8($s3)
```

reads from:

```text
1000 + 8 = 1008
```

Afterward:

```text
$s3 is still 1000.
```

## Stack allocation

```mips
addi $sp, $sp, -8
```

means:

```text
Change $sp itself.
```

For example:

```text
Before: $sp = 1000
After:  $sp = 992
```

It does not read or write memory.

## Combined stack use

```mips
addi $sp, $sp, -8
sw   $a0, 0($sp)
sw   $ra, 4($sp)
```

The first instruction creates the frame. The next instructions put values into the frame.

## Same addressing syntax, different base register

Both of these use `offset(base)`:

```mips
lw $t0, 8($s3)
lw $t1, 4($sp)
```

But their meanings depend on what the base register represents:

```text
$s3 may point to an array.
$sp points to the current stack frame.
```

---

# 9. The stack-frame method

Before writing instructions, draw the frame.

Suppose a function needs to preserve:

- `$ra`
    
- argument `$a0`
    
- argument `$a1`
    
- one intermediate result
    

That is four words:

```text
4 words × 4 bytes = 16 bytes
```

Design:

```text
12($sp)   saved $ra
 8($sp)   intermediate result
 4($sp)   saved $a1
 0($sp)   saved $a0
```

Then write:

```mips
addi $sp, $sp, -16
sw   $a0, 0($sp)
sw   $a1, 4($sp)
sw   $ra, 12($sp)
```

This is much safer than randomly modifying `$sp` throughout the function.

Use this structure:

```text
Prologue:
    Allocate the whole frame.
    Save everything.

Body:
    Perform calculations and calls.

Epilogue:
    Restore everything.
    Deallocate the whole frame.
    Return.
```

---

# 10. What should you save?

Do not save everything automatically. Ask this question:

> Will I need this value after a `jal`, and is the called procedure allowed to overwrite it?

## Save `$ra` when

Your function itself performs at least one `jal`.

```mips
jal otherFunction
```

A recursive function also performs a `jal`, even though it calls itself.

## Save an argument when

You will need the original argument after a nested call.

Example:

```c
return n * fact(n - 1);
```

After `fact(n - 1)` returns, you still need the original `n`.

Therefore save `$a0`.

## Save a temporary when

You calculate something before a call and need it after the call.

```c
int x = a + b;
int y = otherFunction();
return x + y;
```

The value `x` must survive `jal otherFunction`.

A `$t` register cannot be trusted across the call, so save `x` on the stack or keep it in a properly preserved `$s` register.

## Save an `$s` register when

Your procedure modifies it at all.

Even a leaf procedure must preserve `$s0` if it uses `$s0`.

## Do not save values that are dead

If a value is never used after a call, there is no reason to preserve it.

---

# 11. A reusable leaf-procedure template

## Leaf using only temporary registers

```mips
function:
    # Computation using $a, $t and $v registers

    add  $v0, ..., ...
    jr   $ra
```

No stack is required.

## Leaf using saved registers

```mips
function:
    addi $sp, $sp, -4
    sw   $s0, 0($sp)

    # Use $s0
    ...

    # Put result in $v0
    ...

    lw   $s0, 0($sp)
    addi $sp, $sp, 4
    jr   $ra
```

---

# 12. A reusable non-leaf template

```mips
function:
    addi $sp, $sp, -N

    sw   $ra, RA_OFFSET($sp)

    # Save arguments, temporaries or $s registers
    # that must survive nested calls
    sw   ..., ...($sp)

    # Prepare arguments
    ...
    jal  another_function

    # Restore values needed after the call
    lw   ..., ...($sp)

    # Use returned value from $v0
    ...

    lw   $ra, RA_OFFSET($sp)
    addi $sp, $sp, N
    jr   $ra
```

Every path that reaches `jr $ra` must restore the stack correctly.

If you allocate:

```mips
addi $sp, $sp, -12
```

you must eventually execute:

```mips
addi $sp, $sp, 12
```

before returning.

---

# 13. Multiple function calls

Consider:

```c
int calculate(int x, int y)
{
    int p = addTwo(x, y);
    int q = subtractTwo(x, y);
    return p + q;
}
```

Arguments:

```text
x → $a0
y → $a1
```

Problems:

- `calculate` calls functions, so save `$ra`.
    
- It needs `x` and `y` again for the second call.
    
- It needs `p` after the second call.
    
- The second call overwrites `$v0`.
    

Design a 16-byte frame:

```text
12($sp)  saved $ra
 8($sp)  p
 4($sp)  y
 0($sp)  x
```

MIPS:

```mips
calculate:
    addi $sp, $sp, -16

    sw   $a0, 0($sp)        # save x
    sw   $a1, 4($sp)        # save y
    sw   $ra, 12($sp)       # save return address

    jal  addTwo             # v0 = addTwo(x, y)

    sw   $v0, 8($sp)        # save p

    lw   $a0, 0($sp)        # restore x for second call
    lw   $a1, 4($sp)        # restore y for second call

    jal  subtractTwo        # v0 = subtractTwo(x, y)

    lw   $t0, 8($sp)        # t0 = p
    add  $v0, $t0, $v0      # return p + q

    lw   $ra, 12($sp)
    addi $sp, $sp, 16
    jr   $ra
```

Leaf functions:

```mips
addTwo:
    add $v0, $a0, $a1
    jr  $ra
```

```mips
subtractTwo:
    sub $v0, $a0, $a1
    jr  $ra
```

## Why save `p`?

After `addTwo`:

```text
$v0 = p
```

But then:

```mips
jal subtractTwo
```

causes `$v0` to hold `q`.

Therefore `p` must be stored before the second call.

This is a common exam pattern:

```text
Result of call 1 is needed after call 2
→ save result of call 1.
```

---

# 14. Recursion is just repeated normal function calls

There is no special “recursion hardware.”

This:

```c
fact(n - 1)
```

simply executes:

```mips
jal fact
```

again.

The crucial difference is that every call needs its own:

- Original argument
    
- Return address
    
- Local values
    

That is why every call creates a separate stack frame.

The slide’s factorial implementation allocates 8 bytes and saves `$ra` and `$a0` before the recursive call.

One source inconsistency should be noted: page 42 prints `if (n < 1) return f;`, but page 43 implements the base result as `1`. For factorial, the page 43 implementation is the meaningful version.

---

# 15. Factorial, line by line

C:

```c
int fact(int n)
{
    if (n < 1)
        return 1;
    else
        return n * fact(n - 1);
}
```

MIPS from the slide:

```mips
fact:
    addi $sp, $sp, -8
    sw   $ra, 4($sp)
    sw   $a0, 0($sp)

    slti $t0, $a0, 1
    beq  $t0, $zero, L1

    addi $v0, $zero, 1
    addi $sp, $sp, 8
    jr   $ra

L1:
    addi $a0, $a0, -1
    jal  fact

    lw   $a0, 0($sp)
    lw   $ra, 4($sp)
    addi $sp, $sp, 8

    mul  $v0, $a0, $v0
    jr   $ra
```

## Why save `$ra`?

Because:

```mips
jal fact
```

overwrites `$ra`.

## Why save `$a0`?

Before recursion:

```text
$a0 = n
```

Then:

```mips
addi $a0, $a0, -1
```

changes it to:

```text
n - 1
```

The deeper calls continue changing `$a0`.

But after recursion returns, we need the original `n`:

```c
n * fact(n - 1)
```

Therefore the original value is retrieved using:

```mips
lw $a0, 0($sp)
```

## Why can we not simply add 1 to `$a0`?

For one call level, it may look possible. But after deep recursion, `$a0` may have been changed many times, and a callee is generally allowed to overwrite argument registers.

More importantly, each call needs its own original value:

```text
fact(3) needs 3
fact(2) needs 2
fact(1) needs 1
```

There is only one physical `$a0` register, so all three original values cannot remain there simultaneously.

The stack provides separate storage for each call.

---

# 16. Visualizing `fact(3)`

Assume `$sp` starts at `1000`.

## Call `fact(3)`

```text
$sp = 992

996: return to main
992: n = 3
```

Then it calls `fact(2)`.

## Call `fact(2)`

```text
$sp = 984

988: return to fact(3)
984: n = 2

996: return to main
992: n = 3
```

Then it calls `fact(1)`.

## Call `fact(1)`

```text
$sp = 976

980: return to fact(2)
976: n = 1

988: return to fact(3)
984: n = 2

996: return to main
992: n = 3
```

Then it calls `fact(0)`.

## Call `fact(0)`

```text
$sp = 968

972: return to fact(1)
968: n = 0
```

Base case:

```text
fact(0) returns 1
```

Now frames unwind.

## Returning to `fact(1)`

It restores:

```text
n = 1
```

Then:

```text
$v0 = 1 × 1 = 1
```

## Returning to `fact(2)`

It restores:

```text
n = 2
```

Then:

```text
$v0 = 2 × 1 = 2
```

## Returning to `fact(3)`

It restores:

```text
n = 3
```

Then:

```text
$v0 = 3 × 2 = 6
```

Final result:

```text
fact(3) = 6
```

## The central recursion idea

Registers are shared by all calls.

Stack frames are private to individual calls.

---

# 17. Why each recursive call allocates again

This code appears once:

```mips
fact:
    addi $sp, $sp, -8
```

But it executes every time `fact` is called.

For `fact(3)`:

```text
fact(3) executes -8
fact(2) executes -8
fact(1) executes -8
fact(0) executes -8
```

Similarly, every return executes its own:

```mips
addi $sp, $sp, 8
```

Therefore:

```text
Four calls → four allocations
Four returns → four deallocations
```

The source code contains one allocation instruction, but that instruction runs multiple times.

---

# 18. Fixed array indexing

The slide explains that memory is byte addressed and a MIPS word occupies four bytes. Therefore index 8 has byte offset `8 × 4 = 32`.

Suppose:

```c
x = A[2];
```

And:

```text
Base address of A → $s3
x → $s0
```

Then:

```text
A[2] offset = 2 × 4 = 8
```

MIPS:

```mips
lw $s0, 8($s3)
```

Memory picture:

```text
A[0]    0($s3)
A[1]    4($s3)
A[2]    8($s3)
A[3]   12($s3)
```

Memory is byte addressed, and aligned words start at addresses divisible by four.

---

# 19. Variable array indexing

Suppose:

```c
x = A[i];
```

Registers:

```text
A base address → $s3
i              → $s1
x              → $s0
```

We cannot write:

```mips
lw $s0, $s1($s3)
```

because the offset field is a constant, not a register.

Calculate the address:

```mips
sll $t0, $s1, 2       # t0 = i × 4
add $t0, $s3, $t0     # t0 = address of A[i]
lw  $s0, 0($t0)       # s0 = A[i]
```

The left shift by two is used because:

```text
2² = 4
```

Therefore:

```text
i << 2 = i × 4
```

## Address versus value

After:

```mips
add $t0, $s3, $t0
```

`$t0` contains an address.

After:

```mips
lw $s0, 0($t0)
```

`$s0` contains the value stored at that address.

Do not confuse:

```text
Address of A[i]
```

with:

```text
Value of A[i]
```

---

# 20. Storing into a variable array index

C:

```c
A[i] = x;
```

Registers:

```text
A base → $s3
i      → $s1
x      → $s0
```

MIPS:

```mips
sll $t0, $s1, 2
add $t0, $s3, $t0
sw  $s0, 0($t0)
```

Remember:

```mips
lw destination, address
```

means memory to register.

```mips
sw source, address
```

means register to memory.

---

# 21. Array initialization with a loop

C:

```c
void initialize(int A[])
{
    int i;

    for (i = 0; i < 5; i++)
        A[i] = i * 2;
}
```

Assume:

```text
A base address → $a0
```

This is a leaf function, so we can use temporary registers without a stack.

```mips
initialize:
    addi $t0, $zero, 0       # i = 0

Loop:
    slti $t1, $t0, 5         # t1 = 1 if i < 5
    beq  $t1, $zero, Done

    sll  $t2, $t0, 2         # byte offset = i * 4
    add  $t2, $a0, $t2       # address of A[i]

    add  $t3, $t0, $t0       # value = i * 2
    sw   $t3, 0($t2)         # A[i] = i * 2

    addi $t0, $t0, 1         # i++
    j    Loop

Done:
    jr $ra
```

For each iteration:

```text
i = 0 → address A + 0
i = 1 → address A + 4
i = 2 → address A + 8
i = 3 → address A + 12
i = 4 → address A + 16
```

---

# 22. Summing an array

C:

```c
int sum(int A[], int n)
{
    int total = 0;
    int i;

    for (i = 0; i < n; i++)
        total = total + A[i];

    return total;
}
```

Arguments:

```text
A base → $a0
n      → $a1
```

MIPS:

```mips
sum:
    add  $t0, $zero, $zero   # i = 0
    add  $t1, $zero, $zero   # total = 0

Loop:
    slt  $t2, $t0, $a1       # i < n?
    beq  $t2, $zero, Done

    sll  $t3, $t0, 2         # offset = i * 4
    add  $t3, $a0, $t3       # address of A[i]
    lw   $t4, 0($t3)         # t4 = A[i]

    add  $t1, $t1, $t4       # total += A[i]

    addi $t0, $t0, 1
    j    Loop

Done:
    add  $v0, $t1, $zero
    jr   $ra
```

No stack is required because:

- It is a leaf.
    
- It only uses temporary registers.
    
- It has no local array requiring memory.
    

---

# 23. A local array on the stack

Consider:

```c
int example()
{
    int A[4];

    A[0] = 10;
    A[1] = 20;

    return A[0] + A[1];
}
```

The array requires:

```text
4 integers × 4 bytes = 16 bytes
```

MIPS:

```mips
example:
    addi $sp, $sp, -16

    addi $t0, $zero, 10
    sw   $t0, 0($sp)         # A[0] = 10

    addi $t0, $zero, 20
    sw   $t0, 4($sp)         # A[1] = 20

    lw   $t0, 0($sp)
    lw   $t1, 4($sp)
    add  $v0, $t0, $t1

    addi $sp, $sp, 16
    jr   $ra
```

Stack layout:

```text
12($sp)  A[3]
 8($sp)  A[2]
 4($sp)  A[1]
 0($sp)  A[0]
```

It does not save `$ra`, because the procedure does not call another procedure.

---

# 24. Recursion combined with arrays

Consider a harder problem:

```c
int recursiveSum(int A[], int n)
{
    if (n == 0)
        return 0;

    return A[n - 1] + recursiveSum(A, n - 1);
}
```

Arguments:

```text
A base → $a0
n      → $a1
```

After the recursive call, we need:

```text
A[n - 1]
```

We can calculate it before recursion and save its value.

Frame:

```text
4($sp)  saved $ra
0($sp)  saved A[n - 1]
```

MIPS:

```mips
recursiveSum:
    addi $sp, $sp, -8
    sw   $ra, 4($sp)

    beq  $a1, $zero, Base

    addi $t0, $a1, -1        # t0 = n - 1
    sll  $t0, $t0, 2         # offset = (n - 1) * 4
    add  $t0, $a0, $t0       # address of A[n - 1]
    lw   $t1, 0($t0)         # t1 = A[n - 1]

    sw   $t1, 0($sp)         # save element across call

    addi $a1, $a1, -1
    jal  recursiveSum

    lw   $t1, 0($sp)         # restore current element
    lw   $ra, 4($sp)
    addi $sp, $sp, 8

    add  $v0, $t1, $v0       # element + recursive result
    jr   $ra

Base:
    addi $v0, $zero, 0

    lw   $ra, 4($sp)
    addi $sp, $sp, 8
    jr   $ra
```

## Why save `$t1`?

Because `$t1` contains the current array element, but a recursive call may overwrite temporary registers.

Therefore:

```mips
sw $t1, 0($sp)
```

preserves it.

This example combines:

- Array address calculation
    
- Recursion
    
- `$ra` preservation
    
- Temporary-value preservation
    
- Stack-frame balancing
    

---

# 25. The frame pointer `$fp`

The stack pointer may sometimes change during a function. That can make offsets difficult to track.

The frame pointer is an optional stable reference to the current frame.

Conceptually:

```text
$fp → fixed point in current frame
$sp → current bottom of the stack
```

In simple CT problems, you usually do this:

```mips
addi $sp, $sp, -16
```

and never change `$sp` again until the epilogue. In that case, `$sp` itself is a stable base, so `$fp` is unnecessary.

The page 44 diagram shows a frame containing saved arguments, a saved return address, saved registers, and local arrays or structures, with `$fp` near the high-address side and `$sp` near the low-address side.

Use this exam rule:

> For fixed-size frames, use `$sp` with fixed offsets unless the question specifically introduces `$fp`.

---

# 26. Memory layout on page 45

The page 45 diagram divides process memory into several regions.

From low addresses toward high addresses:

```text
High addresses
────────────────────────
Stack
    grows downward
         ↓

Free space

         ↑
Heap / dynamic data
    grows upward

Static data
    global variables
    static variables
    strings
    constant arrays

Text
    machine instructions

Reserved
────────────────────────
Low addresses
```

The diagram shows representative MIPS addresses:

```text
Text begins near:         0x00400000
Static data near:         0x10000000
$gp near:                 0x10008000
Initial $sp near:         0x7FFFFFFC
```

## Text segment

Contains program instructions:

```mips
add
lw
jal
beq
...
```

The program counter `$pc` points into this area.

## Static data segment

Contains variables that exist for the entire program:

```c
int count;
static int total;
char message[] = "Hello";
```

The global pointer `$gp` is initialized near the static-data segment so global data can be accessed using positive or negative offsets.

For example, if a problem states:

```text
count is at 0($gp)
total is at 4($gp)
```

then:

```mips
lw $t0, 0($gp)
lw $t1, 4($gp)
```

## Heap

Contains dynamically allocated memory:

```c
malloc(...)
```

or in Java:

```java
new Object()
```

The heap normally grows toward higher addresses.

## Stack

Contains automatic procedure storage:

- Stack frames
    
- Saved registers
    
- Saved return addresses
    
- Local arrays
    
- Local variables stored in memory
    

The stack normally grows toward lower addresses.

## Global array versus local array

Global/static array:

```text
Stored in static-data segment
Often accessed relative to $gp
Exists throughout the program
```

Local array:

```text
Stored in the current stack frame
Accessed relative to $sp or $fp
Disappears when the function returns
```

---

# 27. A complete method for translating C to MIPS

For every question, follow this sequence.

## Step 1: Identify each function

For every function, determine:

```text
Arguments
Return value
Local variables
Nested calls
```

## Step 2: Classify the function

Ask:

```text
Does this function execute any jal?
```

If no:

```text
Leaf
```

If yes:

```text
Non-leaf
```

Recursion is non-leaf.

## Step 3: Assign registers

Use:

```text
Arguments             → $a0–$a3
Return result         → $v0
Short-lived values    → $t registers
Long-lived variables  → $s registers or stack
Array base            → argument, saved register or provided base register
```

## Step 4: Mark values needed after every call

For each `jal`, draw a line in the C code:

```c
value = firstFunction();

---------------- jal boundary ----------------

use value;
```

Anything created before the boundary and used afterward must survive the call.

Save it using:

- Stack
    
- A saved register that is correctly preserved
    

## Step 5: Design the whole stack frame

Example:

```text
Need:
    saved ra       4 bytes
    saved a0       4 bytes
    saved result   4 bytes

Total: 12 bytes
```

Map it:

```text
8($sp)  ra
4($sp)  result
0($sp)  a0
```

Only then write:

```mips
addi $sp, $sp, -12
```

## Step 6: Write the prologue

```mips
addi $sp, $sp, -N
sw   ...
sw   ...
```

## Step 7: Translate arithmetic, conditions and loops

Do this the same way as basic non-function code.

## Step 8: Prepare arguments before each call

```mips
add  $a0, ...
add  $a1, ...
jal  function
```

## Step 9: Immediately handle the result

The result is in `$v0`.

If another call will happen and the result is still needed:

```mips
sw $v0, offset($sp)
```

## Step 10: Write the epilogue

```mips
lw   ...
lw   $ra, ...
addi $sp, $sp, N
jr   $ra
```

---

# 28. Common mistakes

## Mistake 1: Saving `$ra` in every function

A leaf procedure does not need to save `$ra` unless something else will overwrite it.

## Mistake 2: Not saving `$ra` in a non-leaf function

Every `jal` replaces `$ra`.

## Mistake 3: Assuming `$t` registers survive a function call

They do not have to survive.

## Mistake 4: Using `$s0` without preserving it

If you modify `$s0`, save its old value and restore it.

## Mistake 5: Allocating stack space but not storing values

```mips
addi $sp, $sp, -8
```

only allocates. It does not save `$ra` or arguments.

## Mistake 6: Forgetting that `$v0` is overwritten by each call

If you need the first function’s result after a second call, save it.

## Mistake 7: Using array index directly as a byte offset

For integer arrays:

```text
byte offset = index × 4
```

## Mistake 8: Returning without restoring `$sp`

Every function must leave `$sp` exactly where it found it.

## Mistake 9: Restoring `$sp` before loading saved values

Wrong:

```mips
addi $sp, $sp, 8
lw   $ra, 4($sp)
```

After changing `$sp`, the offsets refer to different addresses.

Correct:

```mips
lw   $ra, 4($sp)
addi $sp, $sp, 8
```

## Mistake 10: Forgetting one return path

If a function has an `if` with two returns, both paths must clean up the stack.

---

# 29. The most useful question to ask while solving

At every line, ask:

> What values must still exist later?

Examples:

```c
return n * fact(n - 1);
```

After recursion, still needed:

```text
original n
return address
```

Therefore save both.

```c
p = f(x);
q = g(y);
return p + q;
```

After `g(y)`, still needed:

```text
p
return address
```

Therefore preserve those.

```c
return f(x);
```

After `f(x)`, nothing except the returned value is needed. This may sometimes allow a simpler structure.

---

# 30. Compact exam cheat sheet

```text
FUNCTION INPUT:
    $a0–$a3

FUNCTION OUTPUT:
    $v0–$v1

CALL:
    jal function
    → saves following address in $ra
    → jumps to function

RETURN:
    jr $ra

$t REGISTER:
    May be destroyed by a called function

$s REGISTER:
    Must be restored by the function that changes it

SAVE $ra:
    If the current function executes jal

SAVE ARGUMENT/TEMPORARY:
    If it is needed after jal

ALLOCATE N BYTES:
    addi $sp, $sp, -N

STORE:
    sw register, offset($sp)

RESTORE:
    lw register, offset($sp)

FREE N BYTES:
    addi $sp, $sp, N

INTEGER ARRAY:
    address of A[i] = base + i × 4

VARIABLE INDEX:
    sll index, index, 2
    add address, base, index

RECURSION:
    Ordinary jal to the same function
    Every call creates its own frame

STACK RULE:
    Every -N must eventually be matched by +N
```

---
