
---

# PART I — The actual C → MIPS translation mindset

When you see:

```c
int f(int a, int b) {
    ...
}
```

do **not** immediately start writing MIPS.

First rewrite the problem mentally into six categories:

```text
1. Variables
2. Arithmetic / bitwise expressions
3. Memory accesses
4. Conditions
5. Loops
6. Function calls
```

Then translate those independently.

For example:

```c
int foo(int A[], int n) {
    int sum = 0;

    for (int i = 0; i < n; i++) {
        if (A[i] % 2 == 0)
            sum += A[i];
    }

    return sum;
}
```

Don't think:

> “How the hell do I translate this whole thing?”

Break it into:

```text
A       = $a0
n       = $a1
sum     = register
i       = register

for:
    condition i < n
    body
    i++
    repeat

A[i]:
    address = A + i*4

A[i] % 2 == 0:
    test lowest bit

return:
    $v0 = sum
```

At that point it stops being one difficult problem.

---

# PART II — Your complete instruction toolbox

You already know basic arithmetic, so I'll focus on **how C constructs map to MIPS**.

---

# 1. Assignment

C:

```c
x = y;
```

If:

```text
x = $s0
y = $s1
```

without using the `move` pseudoinstruction:

```mips
add $s0, $s1, $zero
```

Because:

```text
x = y + 0
```

---

# 2. Constants

C:

```c
x = 7;
```

```mips
addi $s0, $zero, 7
```

C:

```c
x = x + 4;
```

```mips
addi $s0, $s0, 4
```

C:

```c
x = x - 4;
```

There is no `subi`.

Use:

```mips
addi $s0, $s0, -4
```

The slides explicitly introduce immediates this way.

---

# 3. Arithmetic expressions

C:

```c
x = a + b - c;
```

Suppose:

```text
x = $s0
a = $s1
b = $s2
c = $s3
```

Then:

```mips
add $t0, $s1, $s2
sub $s0, $t0, $s3
```

---

# 4. Expression ordering matters

Consider:

```c
x += y;
y += x;
```

This is **not**:

```c
oldX + oldY
oldY + oldX
```

The second statement uses the **new value of `x`**.

Correct:

```mips
add $s0, $s0, $s1      # x = x + y
add $s1, $s1, $s0      # y = y + NEW x
```

This exact trap appears in your previous-year assessment.

---

# PART III — Bitwise operations: pages 24–28

This is especially important because last year's question uses them.

The slides map:

```text
C/C++             MIPS

x << n            sll
x >> n            srl
x & y             and
x | y             or
~x                nor with $zero
```

The slide specifically describes AND as useful for masking bits and OR as useful for setting bits.

---

# 5. `&` is NOT `&&`

Extremely important.

```c
x & 1
```

means **bitwise AND**.

```c
x && 1
```

means logical AND.

They are not interchangeable.

---

# 6. Checking whether a number is even

C:

```c
if ((x & 1) == 0)
```

Binary fact:

```text
even number → least significant bit = 0
odd number  → least significant bit = 1
```

So:

```mips
andi $t0, $s0, 1

beq $t0, $zero, Even
```

Example:

```text
6 = 0110
1 = 0001
---------
    0000
```

So even.

For 7:

```text
7 = 0111
1 = 0001
---------
    0001
```

So odd.

This is much cleaner than actually calculating `% 2`.

---

# 7. OR

C:

```c
z = x | y;
```

```mips
or $s0, $s1, $s2
```

Constant:

```c
z = x | 1;
```

```mips
ori $s0, $s1, 1
```

---

# 8. What does `(y | 1) > y` actually test?

This is in your previous-year question.

Consider the lowest bit.

### If `y` is even

Example:

```text
y       = 6 = 0110
y | 1       = 0111 = 7
```

Therefore:

```text
(y | 1) > y
```

is true.

### If `y` is odd

```text
y       = 7 = 0111
y | 1       = 0111 = 7
```

So:

```text
7 > 7
```

false.

Therefore this bizarre-looking condition is effectively:

```c
y is even
```

But in an exam, translate the expression as written unless you're deliberately optimizing.

```mips
ori $t0, $s1, 1       # t0 = y | 1
slt $t1, $s1, $t0     # y < t0 ?
bne $t1, $zero, CONDITION_TRUE
```

Notice comparison reversal:

```c
(y | 1) > y
```

is equivalent to:

```text
y < (y | 1)
```

because `slt` checks `<`.

---

# 9. Shift operations

Page 25 establishes:

```text
sll by n → multiply by 2^n
srl by n → unsigned division by 2^n
```

So:

```c
x = y * 4;
```

can be:

```mips
sll $s0, $s1, 2
```

because:

```text
4 = 2²
```

Likewise:

```c
x = y * 8;
```

```mips
sll $s0, $s1, 3
```

The most important use in your lab:

```text
integer array index × 4
```

which becomes:

```mips
sll $t0, $index, 2
```

---

# PART IV — Conditions

Pages 29 onward introduce:

```mips
beq rs, rt, Label
bne rs, rt, Label
j Label
```

with `slt/slti` used to construct inequalities.

This is one of the largest sources of mistakes.

---

# 10. Equality

C:

```c
if (x == y)
    ...
```

Can directly branch to the true block:

```mips
beq $s0, $s1, True
```

or usually branch around it:

```mips
bne $s0, $s1, EndIf

# body

EndIf:
```

Both are valid.

---

# 11. Not equal

```c
if (x != y)
```

```mips
bne $s0, $s1, True
```

---

# 12. Less than

```c
if (x < y)
```

```mips
slt $t0, $s0, $s1

beq $t0, $zero, EndIf

# body

EndIf:
```

Because:

```text
slt t0,x,y

t0 = 1 if x < y
t0 = 0 otherwise
```

The slide defines exactly this behavior.

---

# 13. Greater than

There is no need for a new concept.

```c
x > y
```

means:

```text
y < x
```

Therefore:

```mips
slt $t0, $s1, $s0
```

if:

```text
x = $s0
y = $s1
```

This reversal is extremely important.

---

# 14. `<=`

C:

```c
x <= y
```

Instead of testing it directly, test the opposite:

```text
x <= y
⇔ NOT(x > y)
⇔ NOT(y < x)
```

Example:

```mips
slt $t0, $s1, $s0       # y < x ?

bne $t0, $zero, False
```

If `y < x`, then `x <= y` is false.

---

# 15. `>=`

Similarly:

```text
x >= y
⇔ NOT(x < y)
```

```mips
slt $t0, $s0, $s1

bne $t0, $zero, False
```

---

# 16. Comparison cheat sheet

Suppose:

```text
x = $s0
y = $s1
```

Then:

```text
x < y:
    slt $t0,$s0,$s1

x > y:
    slt $t0,$s1,$s0

x <= y:
    test y < x and negate logic

x >= y:
    test x < y and negate logic

x == y:
    beq

x != y:
    bne
```

Do not try memorizing six independent cases.

Memorize only:

> `slt A,B,C` asks **B < C?**

Then rearrange the C condition.

---

# PART V — Signed versus unsigned comparison

Page 35 distinguishes:

```text
slt, slti    → signed
sltu, sltiu  → unsigned
```

For instance, the same bit pattern can mean `-1` as a signed integer or `4,294,967,295` unsigned, giving different comparison results.

For normal:

```c
int x;
```

use:

```mips
slt
slti
```

For:

```c
unsigned int x;
```

use:

```mips
sltu
sltiu
```

Don't randomly use `sltu` merely because the values happen to be positive in your example.

---

# PART VI — if / else

The slide's page 30 example is:

```c
if (i == j)
    f = g + h;
else
    f = g - h;
```

The standard structure is:

```mips
bne $s3, $s4, Else

# IF body
add $s0, $s1, $s2

j EndIf

Else:
sub $s0, $s1, $s2

EndIf:
```

This gives you an important template:

```text
if (condition)
    A;
else
    B;
```

becomes:

```text
if condition is FALSE → jump Else

A
jump End

Else:
B

End:
```

---

# PART VII — if / else-if / else

C:

```c
if (x < 0)
    a = 1;
else if (x < 10)
    a = 2;
else
    a = 3;
```

Think in blocks:

```text
TEST1
    false → TEST2
    true  → BODY1 → END

TEST2
    false → ELSE
    true  → BODY2 → END

ELSE
    BODY3

END
```

MIPS:

```mips
# if (x < 0)
slt $t0, $s0, $zero
beq $t0, $zero, ElseIf

addi $s1, $zero, 1
j EndIf

ElseIf:

# if (x < 10)
slti $t0, $s0, 10
beq $t0, $zero, Else

addi $s1, $zero, 2
j EndIf

Else:
addi $s1, $zero, 3

EndIf:
```

This scales to five or ten `else if`s.

---

# PART VIII — Compound conditions

This is where “tricky” lab questions usually come from.

---

# 17. Logical AND: `&&`

C:

```c
if (x > 0 && y < 10)
```

Don't try making one gigantic MIPS condition.

Use **short-circuit logic**.

For AND:

> If ANY condition is false → skip the body.

```mips
# x > 0 ?
slt $t0, $zero, $s0
beq $t0, $zero, EndIf

# y < 10 ?
slti $t0, $s1, 10
beq $t0, $zero, EndIf

# body

EndIf:
```

This closely mirrors C semantics.

---

# 18. Logical OR: `||`

C:

```c
if (x == 0 || y == 0)
```

For OR:

> If ANY condition is true → execute body.

```mips
beq $s0, $zero, Body
beq $s1, $zero, Body

j EndIf

Body:
    # statements

EndIf:
```

---

# 19. Nested combination

C:

```c
if ((a < b && c != d) || x == 5)
```

Do not panic.

Break it structurally:

```text
(a < b AND c != d) OR x == 5
```

For example:

```mips
# Try first AND group

slt $t0, $s0, $s1       # a < b
beq $t0, $zero, CheckX

bne $s2, $s3, Body       # c != d

CheckX:
addi $t0, $zero, 5
beq  $s4, $t0, Body

j EndIf

Body:
    ...

EndIf:
```

Lab trickiness is usually **control-flow decomposition**, not exotic MIPS.

---

# PART IX — Loops

Page 31 gives the classic:

```c
while (save[i] == k)
    i++;
```

and performs:

```text
calculate address
load save[i]
test
body
jump back
```

You should know four loop templates.

---

# 20. `while`

C:

```c
while (i < n) {
    body;
    i++;
}
```

Structure:

```text
Loop:
    test condition
    false → Exit

    body

    update
    jump Loop

Exit:
```

Example:

```mips
Loop:
    slt $t0, $s0, $s1
    beq $t0, $zero, Exit

    # body

    addi $s0, $s0, 1
    j Loop

Exit:
```

---

# 21. `for`

C:

```c
for (i = 0; i < n; i++) {
    body;
}
```

Mentally rewrite it:

```c
i = 0;

while (i < n) {
    body;
    i++;
}
```

Then translate exactly like a while loop.

```mips
addi $s0, $zero, 0

Loop:
    slt $t0, $s0, $s1
    beq $t0, $zero, Exit

    # body

    addi $s0, $s0, 1
    j Loop

Exit:
```

Do not treat `for` as a separate MIPS concept.

---

# 22. `do while`

This appears in your previous-year question.

C:

```c
do {
    body;
} while (x < y);
```

Difference:

> body executes before checking the condition.

```mips
Loop:

    # body

    slt $t0, $s0, $s1
    bne $t0, $zero, Loop
```

Notice the absence of an initial condition check.

---

# 23. Nested loops

C:

```c
for (i = 0; i < n; i++) {
    for (j = 0; j < m; j++) {
        ...
    }
}
```

Just use separate labels:

```mips
OuterLoop:
    # outer condition

    addi $s1, $zero, 0       # j = 0

InnerLoop:
    # inner condition

    # body

    addi $s1, $s1, 1
    j InnerLoop

InnerExit:
    addi $s0, $s0, 1
    j OuterLoop

OuterExit:
```

The crucial error to avoid:

> Reset `j` every time a new outer iteration begins.

---

# PART X — Arrays: the most important memory concept

The slides state that memory is byte-addressed and words are 4 bytes. Integer-array indexing therefore requires `index × 4`.

---

# 24. `int A[]`

For:

```c
A[i]
```

address formula:

$$  
\text{address}(A[i])
=
\text{base}(A)+4i  
$$

Therefore:

```mips
sll $t0, $s1, 2       # 4*i
add $t0, $s0, $t0     # &A[i]
lw  $t1, 0($t0)       # A[i]
```

where:

```text
$s0 = base A
$s1 = i
```

---

# 25. Constant index

```c
x = A[7];
```

Offset:

```text
7 × 4 = 28
```

So:

```mips
lw $s0, 28($s1)
```

if `$s1` contains `A`'s base address.

---

# 26. Assignment to array

C:

```c
A[i] = x;
```

```mips
sll $t0, $s1, 2
add $t0, $s0, $t0
sw  $s2, 0($t0)
```

---

# 27. `A[i] = B[j] + 5`

Suppose:

```text
A base = $s0
B base = $s1
i      = $s2
j      = $s3
```

First `B[j]`:

```mips
sll $t0, $s3, 2
add $t0, $s1, $t0
lw  $t1, 0($t0)
```

Calculate:

```mips
addi $t1, $t1, 5
```

Address `A[i]`:

```mips
sll $t0, $s2, 2
add $t0, $s0, $t0
```

Store:

```mips
sw $t1, 0($t0)
```

Again:

> Break the statement into tiny operations.

---

# PART XI — Characters are different from integers

This matters directly for the previous-year recursion question.

```c
char src[];
char dest[];
```

A `char` is **one byte**, not four.

Therefore:

```text
src[i] address = src + i
```

NOT:

```text
src + 4*i
```

So for a character:

```mips
add $t0, $a1, $a2
lbu $t1, 0($t0)
```

where:

```text
$a1 = src
$a2 = i
```

To store a char:

```mips
sb $t1, 0($t0)
```

---

# 28. Integer vs char arrays — memorize this

```text
int A[i]:
    offset = i * 4
    lw / sw

char A[i]:
    offset = i
    lb/lbu / sb
```

This distinction alone can decide whether your recursiveCopy solution works.

---

# 29. Null character

C:

```c
src[i] == '\0'
```

`\0` has numerical value:

```text
0
```

Therefore:

```mips
lbu $t0, 0($address)
beq $t0, $zero, BaseCase
```

No special “string instruction” is involved.

Strings are just arrays of bytes terminated by zero.

---

# PART XII — Pointer increments

Suppose:

```c
int *p;
p++;
```

Since an `int` is four bytes:

```mips
addi $s0, $s0, 4
```

But:

```c
char *p;
p++;
```

is:

```mips
addi $s0, $s0, 1
```

This is the same principle as array indexing.

---

# PART XIII — Structures of array loops

A very common lab problem:

```c
for (i = 0; i < n; i++)
    A[i] = A[i] + 1;
```

Assume:

```text
A = $a0
n = $a1
```

```mips
addi $t0, $zero, 0         # i = 0

Loop:
    slt  $t1, $t0, $a1
    beq  $t1, $zero, Exit

    sll  $t2, $t0, 2
    add  $t2, $a0, $t2

    lw   $t3, 0($t2)
    addi $t3, $t3, 1
    sw   $t3, 0($t2)

    addi $t0, $t0, 1
    j Loop

Exit:
```

This contains almost every basic lab skill:

```text
loop
comparison
array address
load
arithmetic
store
```

---

# PART XIV — Searching an array

C:

```c
int find(int A[], int n, int x) {
    for (int i = 0; i < n; i++) {
        if (A[i] == x)
            return i;
    }

    return -1;
}
```

Arguments:

```text
A = $a0
n = $a1
x = $a2
```

This is a **leaf function**, so temporary registers are enough.

```mips
find:
    addi $t0, $zero, 0          # i = 0

Loop:
    slt  $t1, $t0, $a1
    beq  $t1, $zero, NotFound

    sll  $t2, $t0, 2
    add  $t2, $a0, $t2
    lw   $t3, 0($t2)

    beq  $t3, $a2, Found

    addi $t0, $t0, 1
    j Loop

Found:
    add $v0, $t0, $zero
    jr  $ra

NotFound:
    addi $v0, $zero, -1
    jr   $ra
```

Notice something important:

> A function can have multiple `jr $ra` paths if there is no cleanup to perform.

If there **is** stack cleanup, every return path must perform it.

---

# PART XV — `break`

C:

```c
while (...) {
    ...
    if (x == 5)
        break;
    ...
}
```

`break` simply jumps to the loop exit.

```mips
beq $s0, $t0, LoopExit
```

There is no special MIPS `break` needed for translating C's control flow.

---

# PART XVI — `continue`

C:

```c
for (...) {
    if (...)
        continue;

    body;
}
```

Jump to the update portion:

```text
Loop:
    condition

    if (...) → Continue

    body

Continue:
    i++
    jump Loop
```

Important for `for` loops:

> `continue` must still perform the update expression.

---

# PART XVII — Procedures: condensed rules for the lab

From the slides:

```text
$a0-$a3 = arguments
$v0-$v1 = results
$t0-$t9 = temporaries, callee may destroy them
$s0-$s7 = saved, callee must restore them
$sp     = stack pointer
$ra     = return address
```

And:

```mips
jal function
```

stores the following return address in `$ra` and jumps to the function, while:

```mips
jr $ra
```

returns.

---

# PART XVIII — The single most important call rule

Before every:

```mips
jal something
```

ask:

> **What values do I still need AFTER this call?**

Anything in:

```text
$a registers
$t registers
$v registers
$ra
```

that must survive should be protected appropriately.

This one question solves most stack problems.

---

# PART XIX — Example with two function calls

C:

```c
int f(int x) {
    int a = square(x);
    int b = cube(x);
    return a + b;
}
```

What needs to survive?

Before calling `square`:

```text
x is needed later for cube()
```

After `square`:

```text
square result is needed after cube()
```

And because `f` itself calls functions:

```text
its original $ra must survive
```

So design:

```text
0($sp)  x
4($sp)  square result
8($sp)  ra
```

MIPS:

```mips
f:
    addi $sp, $sp, -12
    sw   $a0, 0($sp)
    sw   $ra, 8($sp)

    jal square

    sw   $v0, 4($sp)

    lw   $a0, 0($sp)
    jal  cube

    lw   $t0, 4($sp)
    add  $v0, $t0, $v0

    lw   $ra, 8($sp)
    addi $sp, $sp, 12
    jr   $ra
```

That should now feel mechanical.

---

# PART XX — Recursive functions: the method that works for almost everything

For recursion, determine three things:

### 1. Base case

```c
if (...)
    return ...
```

Translate it normally.

### 2. What changes before recursion?

Example:

```c
f(n - 1)
```

### 3. What does the current call still need after recursion?

Example:

```c
return n + f(n - 1);
```

After recursion:

```text
original n is needed
original return address is needed
```

Therefore save both.

---

# PART XXI — A harder recursion example

C:

```c
int sumTo(int n) {
    if (n == 0)
        return 0;

    return n + sumTo(n - 1);
}
```

Frame:

```text
0($sp) = original n
4($sp) = original ra
```

```mips
sumTo:
    addi $sp, $sp, -8
    sw   $a0, 0($sp)
    sw   $ra, 4($sp)

    beq  $a0, $zero, Base

    addi $a0, $a0, -1
    jal  sumTo

    lw   $a0, 0($sp)
    lw   $ra, 4($sp)
    addi $sp, $sp, 8

    add  $v0, $a0, $v0
    jr   $ra

Base:
    add $v0, $zero, $zero

    lw   $ra, 4($sp)
    addi $sp, $sp, 8
    jr   $ra
```

Here we do not really need the saved `$a0` in the base path, but allocating the same frame for every call keeps the solution simple.

---

# PART XXII — Your previous year's recursive question

The assessment asks you to translate approximately:

```c
void recursiveCopy(char dest[], char src[], int i, int j) {

    if (src[i] == '\0') {
        dest[j] = '\0';
        return;
    }

    recursiveCopy(dest, src, i + 1, j - 1);

    dest[j] = src[i];
}
```

This is an excellent test question because it combines:

```text
char arrays
recursion
four arguments
values needed after recursion
$ra preservation
byte loads/stores
```

Let's reason before coding.

Arguments:

```text
$a0 = dest
$a1 = src
$a2 = i
$a3 = j
```

---

## What happens before recursion?

We need:

```c
src[i]
```

and test it.

Because `src` is `char[]`:

```text
address = src + i
```

No `×4`.

---

## What must survive the recursive call?

After:

```c
recursiveCopy(dest, src, i + 1, j - 1);
```

we execute:

```c
dest[j] = src[i];
```

Therefore the current call still needs:

```text
dest
src
original i
original j
return address
```

All five must survive recursion.

This is the important reasoning the question is testing.

---

# 23. Stack design for recursiveCopy

Five words:

```text
0($sp)   dest
4($sp)   src
8($sp)   i
12($sp)  j
16($sp)  ra
```

Total:

```text
20 bytes
```

One valid translation:

```mips
recursiveCopy:

    addi $sp, $sp, -20

    sw   $a0, 0($sp)
    sw   $a1, 4($sp)
    sw   $a2, 8($sp)
    sw   $a3, 12($sp)
    sw   $ra, 16($sp)

    # t0 = address of src[i]
    add  $t0, $a1, $a2

    # t1 = src[i]
    lbu  $t1, 0($t0)

    # if src[i] != '\0', recurse
    bne  $t1, $zero, Recurse
```

Base case:

```mips
    # dest[j] = '\0'

    add  $t0, $a0, $a3
    sb   $zero, 0($t0)

    lw   $ra, 16($sp)
    addi $sp, $sp, 20
    jr   $ra
```

Recursive case:

```mips
Recurse:

    addi $a2, $a2, 1        # i + 1
    addi $a3, $a3, -1       # j - 1

    jal recursiveCopy
```

Now the recursive call has destroyed/changed registers.

Restore current call's values:

```mips
    lw $a0, 0($sp)
    lw $a1, 4($sp)
    lw $a2, 8($sp)
    lw $a3, 12($sp)
    lw $ra, 16($sp)
```

Now:

```c
dest[j] = src[i];
```

Read current `src[i]`:

```mips
    add $t0, $a1, $a2
    lbu $t1, 0($t0)
```

Address current `dest[j]`:

```mips
    add $t0, $a0, $a3
```

Store byte:

```mips
    sb $t1, 0($t0)
```

Finish:

```mips
    addi $sp, $sp, 20
    jr   $ra
```

This question is actually more useful than factorial because it forces you to understand **what belongs to each recursion level**.

---

# PART XXIII — Can recursiveCopy save less?

Yes.

For example, since `dest` and `src` don't conceptually change, you could build a cleverer register allocation.

But for a 25-minute handwritten/lab translation, don't optimize prematurely.

A predictable frame is safer:

```text
save everything needed after recursion
perform recursion
restore
finish current operation
```

Correct and explainable beats overly clever.

---

# PART XXIV — Previous year's do-while question

The other previous-year question is essentially:

```c
int main() {

    int x = 3, y = 2;
    int limit = 50;

    do {

        if ((x & 1) == 0) {
            x += y;
            y += x;
        }

        else if ((y | 1) > y) {
            y += x;
            x += y;
        }

        else {
            x += 1;
            y += 1;
        }

    } while (x + y < limit);

    return 0;
}
```

with:

```text
x     = $s0
y     = $s1
limit = $s2
```

---

# 25. First initialize

```mips
addi $s0, $zero, 3
addi $s1, $zero, 2
addi $s2, $zero, 50
```

Because `do-while`, body starts immediately:

```mips
Loop:
```

---

# 26. First if

C:

```c
if ((x & 1) == 0)
```

```mips
andi $t0, $s0, 1

bne $t0, $zero, ElseIf
```

If result is zero, fall through into first body.

```mips
add $s0, $s0, $s1       # x += y
add $s1, $s1, $s0       # y += NEW x

j Condition
```

That jump is necessary; otherwise execution would fall into `ElseIf`.

---

# 27. Else-if

C:

```c
else if ((y | 1) > y)
```

First:

```mips
ori $t0, $s1, 1
```

Need:

```text
t0 > y
```

Equivalent:

```text
y < t0
```

Therefore:

```mips
slt $t1, $s1, $t0

beq $t1, $zero, Else
```

Body:

```mips
add $s1, $s1, $s0       # y += x
add $s0, $s0, $s1       # x += NEW y

j Condition
```

---

# 28. Else

```mips
Else:
    addi $s0, $s0, 1
    addi $s1, $s1, 1
```

---

# 29. do-while condition

C:

```c
while (x + y < limit);
```

Calculate sum:

```mips
Condition:
    add $t0, $s0, $s1
```

Compare:

```mips
    slt $t1, $t0, $s2
```

Repeat if true:

```mips
    bne $t1, $zero, Loop
```

Return:

```mips
    add $v0, $zero, $zero
```

For a pure code-translation question, that is sufficient unless your lab specifically requires OS exit/syscall conventions.

---

# PART XXV — Why last year's questions matter

They reveal something about the likely assessment style.

They are not testing:

```text
"What is the opcode of add?"
```

They are testing whether you can combine simple concepts.

The recursive question combines:

```text
recursion
char arrays
stack
parameters
return address
load/store byte
```

The iterative question combines:

```text
do while
if / else-if / else
bitwise AND
bitwise OR
slt
sequential updates
```

So **combination problems** are what you need to practice.

---

# PART XXVI — Local variables versus arrays

There are two different cases.

### Simple local variable

```c
int x;
```

Often just keep it in a register.

There is no reason to put everything on the stack.

### Local array

```c
int A[10];
```

You cannot reasonably keep ten array elements in one register.

So allocate memory:

```text
10 × 4 = 40 bytes
```

```mips
addi $sp, $sp, -40
```

Then:

```text
A base = $sp
A[i]   = $sp + 4*i
```

This is exactly the kind of local automatic data represented by the procedure-frame discussion on page 44.

---

# PART XXVII — Example: local array initialization

C:

```c
void f() {

    int A[5];

    for (int i = 0; i < 5; i++)
        A[i] = i * 3;
}
```

Allocate:

```mips
f:
    addi $sp, $sp, -20
```

Initialize `i`:

```mips
    addi $t0, $zero, 0
```

Loop:

```mips
Loop:
    slti $t1, $t0, 5
    beq  $t1, $zero, Exit
```

Calculate `A[i]`:

```mips
    sll $t2, $t0, 2
    add $t2, $sp, $t2
```

Calculate `3*i`.

Without requiring `mul`, you could do:

```mips
    add $t3, $t0, $t0
    add $t3, $t3, $t0
```

Store:

```mips
    sw $t3, 0($t2)
```

Increment:

```mips
    addi $t0, $t0, 1
    j Loop
```

Return:

```mips
Exit:
    addi $sp, $sp, 20
    jr $ra
```

No `$ra` saving required because `f` is still a leaf.

This is an excellent illustration of:

> **Using the stack doesn't automatically mean you must save `$ra`.**

Stack memory and return-address preservation are separate issues.

---

# PART XXVIII — When you have a local array AND function calls

Now:

```c
int f() {
    int A[5];

    initialize(A);

    return A[2];
}
```

Now we need:

```text
local array = 20 bytes
saved ra    = 4 bytes
```

Possible frame:

```text
0-19($sp)  A[0..4]
20($sp)    saved ra
```

Total:

```text
24 bytes
```

```mips
f:
    addi $sp, $sp, -24
    sw   $ra, 20($sp)
```

Pass address of local array:

```mips
    add $a0, $sp, $zero
```

Call:

```mips
    jal initialize
```

Then `A[2]`:

```text
2 × 4 = 8
```

```mips
    lw $v0, 8($sp)
```

Restore:

```mips
    lw   $ra, 20($sp)
    addi $sp, $sp, 24
    jr   $ra
```

That's a plausible “tricky” lab question.

---

# PART XXIX — Array as function argument

C:

```c
int sum(int A[], int n);
```

The function does **not** receive every array element in `$a0`.

It receives the **address of the first element**:

```text
$a0 = &A[0]
$a1 = n
```

Therefore inside the function:

```c
A[i]
```

means:

```text
memory at ($a0 + 4*i)
```

This is essential.

---

# PART XXX — Passing a local array

Suppose:

```c
int A[10];
foo(A);
```

If local `A` begins at `$sp`:

```mips
add $a0, $sp, $zero
jal foo
```

You are passing the **address**, not `A[0]`.

Compare:

```mips
add $a0, $sp, $zero
```

means:

```text
$a0 = address of A
```

while:

```mips
lw $a0, 0($sp)
```

means:

```text
$a0 = value of A[0]
```

Huge distinction.

---

# PART XXXI — 2D arrays

This isn't explicitly developed in pages 1–45, but if your lab instructor builds on ordinary C array addressing, the same address principle extends.

For:

```c
int A[ROWS][COLS];

A[i][j]
```

row-major C storage gives:

# [  
\text{offset}

(i \times COLS+j)\times4  
]

Then:

```text
base + offset
```

So for 5 columns:

```c
A[i][j]
```

conceptually:

```text
index = i*5 + j
offset = index*4
```

If your test stays strictly within the uploaded slides, one-dimensional arrays are the directly demonstrated case; treat this as an extension rather than a slide requirement.

---

# PART XXXII — Strings

A string:

```c
char s[] = "cat";
```

looks in memory like:

```text
address+0    'c'
address+1    'a'
address+2    't'
address+3    '\0'
```

So iteration:

```c
while (s[i] != '\0')
```

becomes:

```mips
Loop:
    add $t0, $a0, $t1
    lbu $t2, 0($t0)

    beq $t2, $zero, Exit

    ...

    addi $t1, $t1, 1
    j Loop

Exit:
```

Notice again:

```text
char index → no shift-left-by-2
```

---

# PART XXXIII — The slide's instruction-format material

Pages 19–23 discuss R-format and I-format machine encoding. For a **C→MIPS coding lab**, this is lower priority unless they explicitly ask you to encode an instruction.

Still know the categories.

### R-format

Typical:

```mips
add
sub
and
or
slt
```

Conceptually fields include:

```text
op | rs | rt | rd | shamt | funct
```

The slides define those fields on page 20.

### I-format

Used for things such as:

```mips
addi
lw
sw
beq
bne
```

and contains:

```text
op | rs | rt | 16-bit immediate/address
```

For the lab, the practical implication is more important than the binary layout:

> Immediate constants and load/store offsets are encoded in the instruction itself.

---

# PART XXXIV — Signed numbers/sign extension

Again, this is more theory than translation, but there are practical consequences.

A negative immediate:

```mips
addi $t0, $t0, -1
```

works because the immediate is sign-extended.

This is why:

```c
i--;
```

is naturally:

```mips
addi $t0, $t0, -1
```

The slides discuss two's-complement and sign extension before instruction encoding.

---

# PART XXXV — Memory layout: connect it to coding

Page 45 divides memory into:

```text
Text
Static data
Heap
Stack
```

and identifies `$gp` with static data and `$sp` with stack storage.

For translation questions, think:

```text
Code/instructions             → text

global/static variables       → static data
global arrays / strings       → static data

malloc/new                    → heap

ordinary function locals,
saved ra,
saved registers,
local arrays                  → stack
```

So:

```c
int globalA[100];

void f() {
    int localA[10];
}
```

conceptually:

```text
globalA → static-data area
localA  → f's stack frame
```

---

# PART XXXVI — `static` local variable versus normal local

C:

```c
void f() {
    static int count = 0;
}
```

Despite appearing inside the function, `count` has static lifetime.

So conceptually:

```text
static data segment
```

not:

```text
stack
```

Whereas:

```c
void f() {
    int count = 0;
}
```

is automatic/local storage and can be a register or stack location.

---

# PART XXXVII — A difficult mixed example

Suppose your lab gives:

```c
int process(int A[], int n) {

    int total = 0;

    for (int i = 0; i < n; i++) {

        if ((A[i] & 1) == 0)
            total += A[i];

        else
            total -= A[i];
    }

    return total;
}
```

Do not write MIPS immediately.

### Registers

```text
$a0 = A
$a1 = n

$t0 = i
$t1 = total
$t2 = address
$t3 = A[i]
$t4 = condition
```

Leaf:

```text
no jal
→ no need to save $ra
```

Then:

```mips
process:
    add  $t0, $zero, $zero      # i = 0
    add  $t1, $zero, $zero      # total = 0

Loop:
    slt  $t4, $t0, $a1
    beq  $t4, $zero, Done

    sll  $t2, $t0, 2
    add  $t2, $a0, $t2
    lw   $t3, 0($t2)

    andi $t4, $t3, 1
    bne  $t4, $zero, Odd

    add  $t1, $t1, $t3
    j Update

Odd:
    sub  $t1, $t1, $t3

Update:
    addi $t0, $t0, 1
    j Loop

Done:
    add $v0, $t1, $zero
    jr  $ra
```

This is exactly the level you need to become comfortable constructing without copying a template blindly.

---

# PART XXXVIII — Harder: loop + function call

C:

```c
int sumSquares(int A[], int n) {

    int sum = 0;

    for (int i = 0; i < n; i++)
        sum += square(A[i]);

    return sum;
}
```

Now the difficulty increases dramatically because:

```text
loop state must survive jal
```

Values needed across every `square()` call:

```text
A
n
i
sum
return address
```

You could make a stack-based solution.

Possible frame:

```text
0($sp)   A
4($sp)   n
8($sp)   i
12($sp)  sum
16($sp)  ra
```

Initialize:

```mips
sumSquares:
    addi $sp, $sp, -20
    sw   $a0, 0($sp)
    sw   $a1, 4($sp)
    sw   $ra, 16($sp)

    addi $t0, $zero, 0
    sw   $t0, 8($sp)       # i = 0
    sw   $zero, 12($sp)    # sum = 0
```

Loop:

```mips
Loop:
    lw $t0, 8($sp)
    lw $t1, 4($sp)

    slt $t2, $t0, $t1
    beq $t2, $zero, Done
```

A[i]:

```mips
    lw  $t3, 0($sp)
    sll $t2, $t0, 2
    add $t2, $t3, $t2
    lw  $a0, 0($t2)
```

Call:

```mips
    jal square
```

Update sum:

```mips
    lw  $t0, 12($sp)
    add $t0, $t0, $v0
    sw  $t0, 12($sp)
```

Update `i`:

```mips
    lw   $t0, 8($sp)
    addi $t0, $t0, 1
    sw   $t0, 8($sp)

    j Loop
```

Finish:

```mips
Done:
    lw   $v0, 12($sp)
    lw   $ra, 16($sp)

    addi $sp, $sp, 20
    jr   $ra
```

This is the kind of problem that initially looks awful but becomes mechanical when you identify:

> Which values must survive the nested call?

---

# PART XXXIX — A better exam strategy for `$s` registers

You can often make the previous solution cleaner by putting long-lived loop variables in:

```text
$s0, $s1, ...
```

For example:

```text
$s0 = A
$s1 = n
$s2 = i
$s3 = sum
```

Then because your function modifies these `$s` registers, **your function saves their old contents once** at entry and restores them at exit.

This can be much cleaner when there are many `jal`s.

Frame might contain:

```text
saved $s0
saved $s1
saved $s2
saved $s3
saved $ra
```

Then loop variables remain in registers across calls because a proper callee must preserve `$s` registers.

This is the real practical reason `$s` registers exist.

---

# PART XL — `$t` versus `$s`: now the distinction should make sense

Use `$t` when:

```text
temporary value
short life
doesn't need to survive a function call
```

Use `$s` when:

```text
long-lived value
especially useful across nested calls
```

But if your procedure uses `$s0`, it must do:

```mips
sw $s0, ...
```

on entry and:

```mips
lw $s0, ...
```

before returning.

So the tradeoff is:

```text
$t:
    cheap now
    unreliable across jal

$s:
    survives calls
    but your function must preserve caller's original value
```

---

# PART XLI — Stack frame strategy I recommend in the lab

Do **not** keep doing:

```mips
addi $sp,$sp,-4
sw ...
...
addi $sp,$sp,-4
sw ...
...
```

unless there is a compelling reason.

Instead calculate the entire frame first.

Example:

```text
Need:

$s0    4
$s1    4
$ra    4
local array[4] = 16

Total = 28 bytes
```

Then:

```mips
addi $sp, $sp, -28
```

Design exact offsets on scratch paper.

That almost eliminates stack bugs.

---

# PART XLII — A stack diagram should take 10 seconds

Write:

```text
24 ra
20 s1
16 s0
12 A[3]
 8 A[2]
 4 A[1]
 0 A[0]
```

Then code from the diagram.

Never try remembering arbitrary offsets mentally.

---

# PART XLIII — One very important refinement about `$ra`

Suppose:

```c
int f(int x) {
    return g(x);
}
```

The conventional easy solution is:

```mips
f:
    addi $sp,$sp,-4
    sw   $ra,0($sp)

    jal g

    lw   $ra,0($sp)
    addi $sp,$sp,4
    jr   $ra
```

For your lab, this is safe and easy.

There are tail-call optimizations that can avoid this, but **do not bother** unless specifically taught.

---

# PART XLIV — How to translate `return`

C:

```c
return x;
```

Usually:

```mips
add $v0, $s0, $zero
jr  $ra
```

But if stack cleanup is required:

```mips
add $v0, $s0, $zero

lw   ...
addi $sp, $sp, N

jr $ra
```

Think:

```text
Set return value
→ restore frame
→ return
```

---

# PART XLV — Multiple return statements

C:

```c
int abs(int x) {

    if (x >= 0)
        return x;

    return -x;
}
```

You can use multiple `jr $ra`s in a leaf procedure.

But when stack cleanup exists, a cleaner approach is often:

```text
all return branches calculate $v0
then jump to common Return label
```

Example:

```mips
...
j Return

...
j Return

Return:
    lw ...
    addi $sp,...
    jr $ra
```

This reduces mistakes.

---

# PART XLVI — An extremely useful “common epilogue” template

```mips
Function:
    # prologue

    ...

Case1:
    # set $v0
    j Return

Case2:
    # set $v0
    j Return

Return:
    # restore saved values
    # restore $ra
    # pop stack

    jr $ra
```

In recursive functions with several base cases, this can save you from forgetting stack cleanup.

---

# PART XLVII — What pages 32 and 34 mean for your lab

Page 32 defines a **basic block** as a straight-line instruction sequence with no branch inside except possibly at the end, and no branch target except at the beginning.

You probably won't “use” this directly while translating C.

But it's useful mentally.

Your MIPS program naturally becomes:

```text
Condition block
Then block
Else block
Loop update block
Exit block
```

Each label often starts another basic block.

Page 34 explains why MIPS primarily gives you simple branches such as equality/inequality rather than a giant family of rich branch instructions: more complex comparisons are built from operations such as `slt` plus branches.

For coding:

> Don't search for a magical instruction for every C condition. Build it.

---

# PART XLVIII — What you should NOT over-focus on for this lab

For a C/C++ → MIPS translation test from these pages, spend less time on:

```text
historical MIPS background
design principles
manually encoding every R-format instruction
hex conversion
detailed two's-complement derivations
basic-block compiler optimization
```

Know them for written/theory questions, but they are not what will make you good at translating unfamiliar source code.

Your highest-value topics are:

```text
1. Conditions
2. Loops
3. Arrays
4. char/string arrays
5. Function arguments/results
6. jal/$ra
7. Stack frames
8. Multiple calls
9. Recursion
10. Bitwise expressions
```

---

# PART XLIX — Your universal translation algorithm

This is what I want you to actually use during the lab.

Given any function:

```c
returnType F(parameters) {
    ...
}
```

## Phase A — analyze

Write:

```text
Arguments:
$a0 =
$a1 =
$a2 =
$a3 =

Return:
$v0 =
```

Then identify:

```text
Leaf or non-leaf?

Arrays?
    type?
    element size?

Loops?

Conditions?

Values needed across jal?

Local array / stack variables?
```

---

## Phase B — allocate registers

Example:

```text
i     → $t0
sum   → $t1
temp  → $t2
```

or for a non-leaf function:

```text
i     → $s0
sum   → $s1
```

---

## Phase C — design stack if needed

Write:

```text
0(sp)  =
4(sp)  =
8(sp)  =
...
```

Then determine total frame size.

---

## Phase D — create labels before instructions

For complicated C:

```text
Loop:
ElseIf:
Else:
Update:
Exit:
Return:
```

Writing the label skeleton first dramatically reduces control-flow mistakes.

---

## Phase E — translate one source statement at a time

Never try translating the entire function mentally.

---

## Phase F — run four checks

### Stack check

```text
Every -N matched by +N?
```

### Call check

```text
Every non-leaf function protected its return address?
```

### Memory check

```text
int → ×4 + lw/sw
char → ×1 + byte load/store
```

### Control-flow check

```text
Does each branch land where the C code would go?
```

---

# PART L — The “survival analysis” trick

This is probably the most important advanced skill.

Draw every `jal` as a wall:

```c
a = x + y;

        | JAL WALL |

b = foo(z);

return a + b;
```

Ask:

```text
What crosses the wall?
```

`a` does.

Therefore `a` cannot merely live in an unprotected `$t` register.

Another:

```c
return n * fact(n - 1);

          | JAL WALL |

Need original n on this side.
```

Save `n`.

RecursiveCopy:

```c
recursiveCopy(dest, src, i + 1, j - 1);

                  | JAL WALL |

dest[j] = src[i];
```

Crossing the wall:

```text
dest
src
i
j
$ra
```

This technique lets you derive the stack frame instead of memorizing one.

---

# PART LI — What I would expect a “hard” lab question to look like

Based on the syllabus and your previous-year assessment, a genuinely good harder question would be something like:

```c
int process(int A[], int n) {

    if (n == 0)
        return 0;

    int x = A[n - 1];

    if ((x & 1) == 0)
        return x + process(A, n - 1);

    return process(A, n - 1) - x;
}
```

This tests:

```text
array access
variable index
bitwise AND
if/else
recursion
saving x
saving ra
changing n
return arithmetic
```

If you can write that comfortably, you are in good shape.

---

# PART LII — Another difficult style

```c
int compute(int A[], int n) {

    int total = 0;

    for (int i = 0; i < n; i++) {

        if (A[i] > 0)
            total += transform(A[i]);
        else
            total -= A[i];
    }

    return total;
}
```

This tests:

```text
loop
array
comparison
function call within only one branch
long-lived loop state
return value
stack/$s registers
```

This is more difficult than either of the previous-year questions because the loop has state that must survive repeated function calls.

---

# PART LIII — One final MIPS/C cheat sheet

Keep this mental translation table:

|C/C++|MIPS idea|
|---|---|
|`x = y + z`|`add`|
|`x = y - z`|`sub`|
|`x++`|`addi x,x,1`|
|`x--`|`addi x,x,-1`|
|`x & y`|`and`|
|`x & constant`|`andi`|
|`x \| y`|`or`|
|`x \| constant`|`ori`|
|`x << n`|`sll`|
|`x == y`|`beq`|
|`x != y`|`bne`|
|`x < y`|`slt x,y`|
|`x > y`|reverse operands to `slt`|
|`if`|branch around body|
|`if-else`|branch + unconditional jump|
|`while`|test before body|
|`do-while`|test after body|
|`for`|initialization + while + update|
|`break`|jump loop exit|
|`continue`|jump loop update/test|
|`int A[i]`|`base + 4*i`, `lw/sw`|
|`char A[i]`|`base + i`, byte load/store|
|function arguments|`$a0-$a3`|
|return value|`$v0`|
|function call|`jal`|
|return from function|`jr $ra`|
|local temporary|`$t`|
|long-lived/call-surviving value|`$s` or stack|
|recursive/nested call|preserve `$ra`|
|local array|stack memory|
|global/static|static-data region|

---

# What I want you to practice now

Don't spend the remaining preparation time rereading all 45 pages linearly. The slide material is finite; the challenge is composing it.

Work through problems in this exact progression:

1. **Condition-heavy:** nested `if / else-if`, `&&`, `||`, bitwise tests.
    
2. **Array-heavy:** search, update, reverse, two arrays, char arrays.
    
3. **Loop-heavy:** `for`, `while`, `do-while`, nested loops.
    
4. **Functions:** one call → two calls → call inside loop.
    
5. **Recursion:** factorial → recursive sum → recursive array → recursive char/string.
    
6. **Mixed:** recursion + arrays + conditions, or loop + function call + arrays.
    

And for **every** solution, force yourself to write these three mini-plans first:

```text
REGISTER MAP
-----------
$a0 =
$a1 =
$t0 =
$s0 =


STACK FRAME
-----------
0($sp) =
4($sp) =
...


LABEL PLAN
----------
Loop:
Else:
Update:
Exit:
Return:
```

If you make that a habit, “tricky” C→MIPS problems become much less improvisational. The machine instructions themselves are simple; the real task is preserving the C program's **data flow, memory addressing, and control flow**.