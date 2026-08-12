# MIPS Assembly — Command Reference & Problem-Solving Guide

Think of this like a vocabulary list + grammar rules for a new language. Every instruction has a fixed "grammar":

```
instruction  destination, source1, source2
```

(Read it right-to-left: "take source1 and source2, combine them, put result in destination.")

---

## 1. REGISTERS — The "Nouns" of the Language

Registers are just labeled storage boxes. There are 32 of them, but only certain ones matter for now.

|Register|Number|What it's FOR|Rule|
|---|---|---|---|
|`$zero`|0|Always holds the value 0|Can never be changed|
|`$t0`–`$t9`|8–15, 24–25|Temporary scratch values|Callee (called function) may destroy these — don't trust them across a function call|
|`$s0`–`$s7`|16–23|"Saved" variables (like C local variables)|Must be preserved across function calls — if you use one, save/restore it|
|`$a0`–`$a3`|4–7|Arguments **into** a function|Set these before calling|
|`$v0`, `$v1`|2–3|Return **value(s)** from a function|Read this after a function returns|
|`$ra`|31|Return address|Set automatically by `jal`|
|`$sp`|29|Stack pointer|Points to top of stack; decreases when reserving space|
|`$fp`|30|Frame pointer|Stable reference point within a function|
|`$gp`|28|Global pointer|Points to static data|

**Mental model:** `$t` = scratch paper (can get erased anytime), `$s` = your notebook (protected, must give back unchanged), `$a`/`$v` = the "in tray" / "out tray" for function calls.

---

## 2. ARITHMETIC INSTRUCTIONS — The Basic Verbs

|Instruction|Syntax|What it does|Format|
|---|---|---|---|
|`add`|`add rd, rs, rt`|rd = rs + rt|R|
|`sub`|`sub rd, rs, rt`|rd = rs − rt|R|
|`addi`|`addi rt, rs, constant`|rt = rs + constant|I|
|`mul`|`mul rd, rs, rt`|rd = rs × rt|R|

**No subtract-immediate exists** — just use `addi` with a negative number: `addi $s2, $s1, -1` means "s2 = s1 − 1".

**Rule of thumb:** Every arithmetic instruction has EXACTLY one destination and one or two sources. There's no "do 3 things in one line" — that's why C expressions get broken into multiple instructions using `$t` registers as temporary holding spots.

---

## 3. MEMORY INSTRUCTIONS — Moving Data In/Out

|Instruction|Syntax|What it does|
|---|---|---|
|`lw`|`lw rt, offset(rs)`|Load a word FROM memory[rs+offset] INTO rt|
|`sw`|`sw rt, offset(rs)`|Store rt's value INTO memory[rs+offset]|
|`lb` / `lbu`|`lb rt, offset(rs)`|Load a single byte (signed/unsigned)|
|`sb`|`sb rt, offset(rs)`|Store a single byte|
|`lh` / `lhu`|`lh rt, offset(rs)`|Load a halfword (16-bit)|
|`sh`|`sh rt, offset(rs)`|Store a halfword|

**Critical rule:** You can NEVER do arithmetic directly on memory. Always: **load → compute → store.**

**The Array Access Recipe (memorize this cold):**

```
sll  $t1, $s_index, 2      # convert index → byte offset (×4, since each word = 4 bytes)
add  $t1, $t1, $s_base     # t1 = actual memory address of array[index]
lw   $t0, 0($t1)           # t0 = array[index]'s value
```

(For `sw`, same first two lines, then `sw $t0, 0($t1)` to write instead of read.)

---

## 4. LOGICAL / BITWISE INSTRUCTIONS

|Instruction|Syntax|What it does|Use it when...|
|---|---|---|---|
|`sll`|`sll rd, rt, shamt`|Shift left by shamt bits (fills with 0)|Multiplying by powers of 2; converting array index → byte offset|
|`srl`|`srl rd, rt, shamt`|Shift right by shamt bits (fills with 0)|Dividing unsigned numbers by powers of 2|
|`and` / `andi`|`and rd, rs, rt`|Bitwise AND — keeps bits where BOTH are 1|Masking (clearing unwanted bits)|
|`or` / `ori`|`or rd, rs, rt`|Bitwise OR — sets bits where EITHER is 1|Turning specific bits ON|
|`nor`|`nor rd, rs, rt`|NOT(rs OR rt)|Getting a NOT: `nor rd, rs, $zero` flips every bit of rs|

**Quick intuition:**

- AND = "keep only what's common" (masking)
- OR = "combine, forcing some bits on" (setting)
- Shift left by _n_ = multiply by 2ⁿ

---

## 5. COMPARISON INSTRUCTIONS — Since there's no direct <, >, ≤, ≥

|Instruction|Syntax|What it does|
|---|---|---|
|`slt`|`slt rd, rs, rt`|rd = 1 if (rs < rt) **signed**, else rd = 0|
|`slti`|`slti rt, rs, constant`|rd = 1 if (rs < constant) signed, else 0|
|`sltu`|`sltu rd, rs, rt`|Same as slt but **unsigned** comparison|

**Why this exists:** MIPS branches (`beq`/`bne`) can ONLY test equality. To test `<`, `>`, `≤`, `≥`, you must first compute a 0/1 result with `slt`, THEN branch on that result.

**Building every comparison from `slt` + `beq`/`bne`:**

|You want...|Recipe|
|---|---|
|`if (a < b)`|`slt $t0,$a,$b` → `beq $t0,$zero,SKIP` (skip if false)|
|`if (a >= b)`|`slt $t0,$a,$b` → `bne $t0,$zero,SKIP` (skip if a<b is true, meaning a>=b is false)|
|`if (a > b)`|`slt $t0,$b,$a` (swap operands!) → `beq $t0,$zero,SKIP`|
|`if (a <= b)`|`slt $t0,$b,$a` (swap operands!) → `bne $t0,$zero,SKIP`|

---

## 6. BRANCH / JUMP INSTRUCTIONS — Control Flow

|Instruction|Syntax|What it does|
|---|---|---|
|`beq`|`beq rs, rt, Label`|If (rs == rt), jump to Label|
|`bne`|`bne rs, rt, Label`|If (rs != rt), jump to Label|
|`j`|`j Label`|Always jump to Label, unconditionally|
|`jal`|`jal Label`|Save return address in `$ra`, then jump (function CALL)|
|`jr`|`jr $ra`|Jump to the address in `$ra` (function RETURN)|

---

## THE GOLDEN RULE: "Invert the condition, then skip"

You almost never translate an `if` condition directly. Instead: **branch on the OPPOSITE condition, to jump PAST the code you want to skip.**

### Pattern A — If/Else

```
      [inverted condition] → branch to ELSE
      [if-body instructions]
      j DONE
ELSE: [else-body instructions]
DONE: [continues normally]
```

### Pattern B — While Loop

```
LOOP: [inverted condition] → branch to DONE
      [loop body instructions]
      j LOOP
DONE: [continues normally]
```

### Pattern C — For Loop (basically a while loop with init + increment built in)

```
      [initialize counter]
LOOP: [inverted condition on counter] → branch to DONE
      [loop body instructions]
      [increment counter]
      j LOOP
DONE:
```

---

## PROBLEM-TYPE → WHICH RECIPE TO USE

|If the problem involves...|Use this recipe|
|---|---|
|A single arithmetic expression `f = (g+h)-(i+j)`|Break into steps using `$t` registers, one operation per line|
|`array[i]` appearing anywhere|The Array Access Recipe (sll → add → lw/sw)|
|`if / else`|Pattern A — invert condition, branch to ELSE, jump to DONE|
|`while` loop|Pattern B — invert condition, branch to DONE, jump back to LOOP|
|`for` loop|Pattern C — same as while, with init before and increment inside|
|`<`, `>`, `<=`, `>=` comparison|First compute with `slt`/`sltu`, then `beq`/`bne` on the 0/1 result|
|Signed vs unsigned bit pattern question|Re-read bit 31: as signed, 1=negative; as unsigned, always positive magnitude|
|A function/procedure call|Args into `$a0-$a3`, `jal FuncName`, result comes back in `$v0`; save `$ra` + any `$s`/`$a` values you need AFTER the call, before calling|
|Recursive function|Same as above, but MUST save `$ra` and current argument on the STACK before the recursive `jal`, since the nested call will overwrite them|

---

## MINI CHECKLIST BEFORE YOU SUBMIT AN ANSWER

- [ ] Did I only put ONE operation per instruction? (no `a = b+c-d` in one line)
- [ ] For every `array[i]`, did I convert index→byte offset with `sll ...,2`?
- [ ] For every `if`/`while`, did I invert the condition correctly?
- [ ] Did I remember the jump-over-else (`j DONE`) so I don't fall into the else block?
- [ ] For `<`/`>`/`<=`/`>=`, did I use `slt` first, not a nonexistent `blt`?
- [ ] If asked about specific bit patterns, did I check bit 31 for sign, and pick the RIGHT instruction (`slt` vs `sltu`) based on what's asked?
- [ ] For a function call, did I set up `$a0-$a3` before, and read `$v0` after?
- [ ] For a recursive/non-leaf call, did I save `$ra` (and anything else still needed) on the stack BEFORE calling, and restore it AFTER?

---

_Keep this open next to any practice problem — try to map the problem to a row in the tables above before writing any code._