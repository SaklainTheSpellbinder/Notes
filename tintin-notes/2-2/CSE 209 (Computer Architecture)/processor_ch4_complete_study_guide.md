# Chapter 4 — The Processor
## Complete Intuitive Study Guide for Part 1 + Part 2
### Single-cycle datapath → control → pipelining → forwarding → stalls → branch hazards → ILP

> **Primary source:** the two uploaded lecture decks, **Chapter 4 (Part 1)** and **Chapter 4 (Part 2)**.
>
> **Your immediate goal:** understand Part 2 well enough for a CT, while learning Part 1 deeply enough that the Part 2 datapath does not feel like a collection of mysterious boxes and wires.
>
> **Longer-term bridge:** understand the concepts well enough to later design a small processor in Logisim and write a simple assembler/compiler-like translator for its ISA. The sections explicitly marked **Project bridge** are conceptual extrapolations from the lecture material, not claims made by the slides.

---

# 0. The one mental model to keep for the entire chapter

A processor is fundamentally doing this:

```text
instruction bits
     ↓
 CONTROL UNIT ───────→ control signals
     ↓                     ↓
 DATAPATH ←──────────── decides which datapath routes/actions are active
     ↓
new data + new processor state
```

The **datapath** contains the things that *carry, store, and transform values*:

- PC
- instruction memory
- register file
- ALU
- data memory
- adders
- sign-extension logic
- shift logic
- multiplexers
- pipeline registers in a pipelined processor

The **control unit** looks at the instruction and answers questions such as:

- Which register is the destination?
- Should the ALU's second input be a register or an immediate?
- What operation should the ALU perform?
- Should data memory be read?
- Should data memory be written?
- Should a register be written?
- If a register is written, does its new value come from the ALU or memory?
- Should the PC continue normally, branch, or jump?

So a CPU diagram becomes much less frightening if you read it as:

> **Where can the data go, and which control bit decides each choice?**

A **mux** is almost always a visible answer to the question:

> “There are two possible values that could go here. Which one should this instruction use?”

A **control signal** is almost always the answer to:

> “Should this hardware action happen for this instruction?”

A **pipeline register** is the answer to:

> “This instruction cannot finish everything this cycle. What information must survive until the next stage?”

A **hazard unit** is the answer to:

> “Is the normal next-cycle flow unsafe because an instruction needs information that is not ready yet?”

A **forwarding unit** is the answer to:

> “The correct value already exists somewhere inside the pipeline. Can I route it directly to where it is needed instead of waiting for normal write-back?”

Keep returning to those questions throughout the chapter.

---

# PART I — FOUNDATION: THE SINGLE-CYCLE PROCESSOR

# 1. Why Chapter 4 starts from instruction execution

The lecture begins by separating processor performance into three broad factors:

1. **Instruction count** — mainly influenced by the ISA and compiler.
2. **CPI (cycles per instruction)** — influenced by CPU organization.
3. **Clock cycle time** — influenced by CPU hardware and critical path.

The chapter first builds a simplified MIPS implementation and then improves it with pipelining.

The instruction subset used in the slides is:

- memory: `lw`, `sw`
- arithmetic/logical: `add`, `sub`, `and`, `or`, `slt`
- control transfer: `beq`, `j`

This subset is deliberately small, but it forces the datapath to support nearly every important category of processor behavior:

```text
register → ALU → register          R-type
register + immediate → memory      load/store address calculation
memory → register                  load
register → memory                  store
compare → choose next PC           branch
instruction target → next PC       jump
```

If you understand why the hardware supports those flows, designing a smaller custom CPU later becomes much easier.

---

# 2. The lifecycle of one instruction

The Part 1 slides give the generic instruction execution sequence:

```text
1. PC → instruction memory
2. Fetch instruction
3. Instruction register numbers → register file
4. Read needed register values
5. Use ALU for whatever this instruction needs
6. Possibly access data memory
7. Possibly write a register
8. Select next PC
```

The ALU is reused conceptually for different jobs:

- arithmetic: `add`, `sub`, etc.
- effective address calculation: base + offset for `lw/sw`
- comparison: subtraction for `beq`

The PC may become:

```text
PC + 4                       normal next instruction
branch target                taken branch
jump target                  jump
```

This is why the complete datapath has several alternate routes near the PC.

---

# 3. CPU overview: read the diagram from left to right

A very useful simplified view is:

```text
        ┌──────────────┐
PC ───→ │ Instruction  │ ───→ instruction fields
        │   Memory     │
        └──────────────┘
                 │
                 v
          ┌─────────────┐
          │ Register    │
          │    File     │
          └─────────────┘
            │       │
            │       └──────────────┐
            v                      │
          ┌─────┐                  │
          │ ALU │                  │
          └─────┘                  │
            │                      │
            v                      v
       effective addr          store data
            │
            v
       ┌───────────┐
       │ Data Mem  │
       └───────────┘
            │
            └────────→ possible register write-back
```

But the actual full datapath has many muxes because different instructions use different pieces of the path.

---

# 4. Why you cannot simply connect alternate wires together

This is the purpose of the Part 1 multiplexer slide.

Suppose the ALU's second input sometimes needs:

- register value `ReadData2`
- sign-extended immediate

You cannot electrically say:

```text
ReadData2 -----┐
               ├---- ALU input B    ← WRONG
Immediate -----┘
```

Both sources would be trying to drive the same wire.

Instead:

```text
ReadData2 --------0\
                   >--- ALU input B
Immediate --------1/
                  ↑
               ALUSrc
```

The mux guarantees that only one source is selected.

### Mux rule

For a 2-to-1 mux:

```text
S = 0 → output = I0
S = 1 → output = I1
```

This simple rule explains a huge portion of processor control.

---

# 5. Datapath versus control

The Part 1 “Control” slide overlays control lines on the datapath.

Think of it this way:

```text
DATAPATH = roads
CONTROL  = traffic signals + switches
```

The instruction itself contains encoded bits. The control unit decodes those bits and produces control signals.

For example:

```text
opcode says lw
     ↓
control knows:
- destination is rt
- use immediate in ALU
- ALU must add
- read memory
- write register
- write memory data to register
```

The same physical ALU and register file can therefore implement several instructions simply by changing control values.

That reuse is one of the core ideas of processor design.

---

# 6. Logic design basics: combinational versus sequential hardware

This distinction is essential for both single-cycle and pipelined CPUs.

## 6.1 Binary representation

The slides assume:

- low voltage → `0`
- high voltage → `1`
- one wire carries one bit
- a multi-bit value uses a **bus**, i.e. multiple parallel wires

Example:

```text
4-bit value 1011

bit3 ─ 1
bit2 ─ 0
bit1 ─ 1
bit0 ─ 1
```

A “32-bit wire” on a CPU drawing therefore really means a 32-wire bus.

For a future 4-bit datapath, the same concept applies with 4 data wires in a bus.

---

## 6.2 Combinational logic

A combinational circuit has no memory of the past.

Its current output is determined by its current inputs.

Examples from the slides:

```text
AND gate: Y = A & B
Adder:    Y = A + B
Mux:      Y = S ? I1 : I0
ALU:      Y = F(A,B)
```

If the inputs change, after propagation delay the output changes.

No clock is inherently needed to “perform” an AND or addition.

---

## 6.3 Sequential/state elements

A state element stores information over time.

The primary example is a register.

```text
        ┌──────────┐
D ─────→│ Register │────→ Q
Clock ─→│          │
        └──────────┘
```

For an edge-triggered register, the stored value changes at the active clock edge, shown in the slides as the rising edge (`0 → 1`).

Between edges, Q remains stable even if combinational logic is changing around it.

This creates the rhythm of a synchronous processor:

```text
clock edge
   ↓
stored state becomes new values
   ↓
combinational logic reacts and computes
   ↓
result settles before next edge
   ↓
next clock edge stores the result
```

---

# 7. Register write-enable

A normal edge-triggered register updates every active clock edge.

Often we do **not** want that.

Therefore we add a write control:

```text
if rising_clock_edge:
    if Write == 1:
        Q ← D
    else:
        Q remains unchanged
```

This concept later becomes critical for hazards.

When stalling a pipeline, one thing we do is disable updates to:

- PC
- IF/ID pipeline register

So the current instruction is held rather than overwritten.

---

# 8. Clocking methodology and critical path

The lecture's clocking methodology is:

```text
state element
   ↓
combinational logic
   ↓
state element
```

The combinational logic must finish before the next active clock edge.

Therefore:

> **The slowest state-to-state combinational path determines the minimum legal clock period.**

This is the **critical path** idea.

If one instruction path requires 800 ps, a single-cycle implementation cannot safely use a 500 ps clock just because some other instruction would finish in 500 ps.

That becomes the motivation for pipelining later.

---

# 9. Building the datapath incrementally

The lecture does not throw the full CPU at you immediately. It adds hardware according to what each instruction requires.

This is also the best way to think when later designing your own processor:

```text
What does instruction X need?
→ add the minimal necessary route/component
→ if two instruction types need alternate values at one destination, add a mux
→ generate a control bit to select that mux
```

---

# 10. Instruction fetch datapath

Every instruction begins with fetch.

The basic fetch unit is:

```text
                ┌──────────────────┐
PC ────────────→│ Instruction Mem  │────→ Instruction
│               └──────────────────┘
│
│     ┌─────┐
└────→│  +  │────→ PC + 4
      └─────┘
         ↑
         4
```

Why `+4`?

MIPS instructions in this design are 32 bits = 4 bytes and are word-aligned.

So sequential addresses are:

```text
0, 4, 8, 12, 16, ...
```

The PC is a register because the processor must remember which instruction address it is currently using.

---

# 11. MIPS instruction formats you must understand before reading the wires

The slides recall three formats.

## 11.1 R-format

```text
31      26 25   21 20   16 15   11 10    6 5       0
+---------+-------+-------+-------+--------+----------+
| opcode  |  rs   |  rt   |  rd   | shamt  | funct   |
+---------+-------+-------+-------+--------+----------+
   6 bits   5 bits  5 bits  5 bits  5 bits    6 bits
```

Important meaning:

- `rs` = first source register
- `rt` = second source register
- `rd` = destination register
- `funct` = selects the specific R-type ALU operation

Example:

```text
add $t0, $t1, $t2

rs = $t1
rt = $t2
rd = $t0
```

---

## 11.2 Load/store I-format

```text
31      26 25   21 20   16 15                       0
+---------+-------+-------+---------------------------+
| opcode  |  rs   |  rt   |      address/immediate    |
+---------+-------+-------+---------------------------+
   6 bits   5 bits  5 bits             16 bits
```

For:

```text
lw $t0, 12($s1)
```

- `rs` = base register `$s1`
- `rt` = destination `$t0`
- immediate = `12`

For:

```text
sw $t0, 12($s1)
```

- `rs` = base register `$s1`
- `rt` = source data register `$t0`
- immediate = `12`

This is important: **the same field `rt` has different semantic roles depending on the instruction.**

That is exactly why control logic exists.

---

## 11.3 Branch format in this subset

`beq` uses:

- `rs`
- `rt`
- signed 16-bit displacement

It compares the two registers and, if equal, changes the PC to a PC-relative target.

---

## 11.4 Jump format

The jump instruction uses:

```text
opcode: 6 bits
address: 26 bits
```

The target is reconstructed using the jump field, two zero bits, and the upper PC bits.

---

# 12. Register file: why it has two read ports and one write port

An R-type instruction often needs two operands simultaneously:

```text
add rd, rs, rt
```

Therefore the register file has:

```text
Read register 1 address ──→ [ Register File ] ──→ Read data 1
Read register 2 address ──→ [               ] ──→ Read data 2
Write register address ───→ [               ]
Write data ────────────────→ [               ]
RegWrite ─────────────────→ [               ]
```

Two read ports let the CPU obtain both operands during the same cycle/stage.

The one write port is enough in this simple scalar processor because at most one instruction completes a register write per cycle.

---

# 13. R-type datapath

For an R-type ALU operation:

```text
instruction rs ─→ Register File ─→ operand A ─┐
                                               │
instruction rt ─→ Register File ─→ operand B ─┤→ ALU → result
                                               │
instruction rd ────────────────────────────────┘       │
                                                       ↓
                                                  Register File write
```

Conceptually:

```text
Reg[rd] ← ALU(Reg[rs], Reg[rt])
```

The exact ALU operation comes from the `funct` field after ALU-control decoding.

---

# 14. Load/store address calculation

Both `lw` and `sw` calculate an effective address:

```text
address = Reg[rs] + sign_extend(immediate)
```

This means the ALU needs two different possible B operands:

```text
for R-type: second register value
for lw/sw:  sign-extended immediate
```

Hence the **ALUSrc mux**.

```text
ReadData2 ----------------0\
                           >---- ALU input B
SignExtendedImmediate ----1/
                          ↑
                       ALUSrc
```

For loads/stores:

```text
ALUSrc = 1
ALU does ADD
```

---

# 15. Why sign extension exists

The immediate is 16 bits, but the ALU inputs are 32 bits in this MIPS datapath.

So the immediate must become 32 bits.

For signed offsets, the sign bit is replicated:

```text
16-bit +5:
0000 0000 0000 0101
→
0000000000000000 0000000000000101

16-bit -1:
1111 1111 1111 1111
→
1111111111111111 1111111111111111
```

The sign-extension box on the slide is therefore mostly wiring: copy the sign bit into the newly created upper positions.

---

# 16. `lw`: complete data movement

Example:

```text
lw $t0, 12($s1)
```

Think of it as:

```text
Reg[$t0] ← Memory[Reg[$s1] + 12]
```

Hardware flow:

```text
$s1 → register file → base value ─────────┐
                                          │
12 → sign extend ─────────────────────────┤→ ALU(add) → address
                                          │              │
                                          │              v
                                          │         Data Memory read
                                          │              │
                                          │              v
                                          └──────────→ write data
                                                        │
                                                        v
                                                     Reg[$t0]
```

Important controls:

- ALU uses immediate
- ALU adds
- memory is read
- destination is `rt`
- register write enabled
- register receives **memory data**, not ALU result

---

# 17. `sw`: complete data movement

Example:

```text
sw $t0, 12($s1)
```

Conceptually:

```text
Memory[Reg[$s1] + 12] ← Reg[$t0]
```

Two different register values are needed:

- `$s1` for address base
- `$t0` for data to store

Flow:

```text
Reg[$s1] ────────┐
                  ├→ ALU(add) → memory address
Immediate ───────┘

Reg[$t0] ───────────────────→ memory write data
```

`sw` does **not** write the register file.

This becomes important in hazard detection and forwarding: a register field appearing in an instruction does not automatically mean that the instruction writes that register.

---

# 18. Branch (`beq`) datapath

For:

```text
beq rs, rt, offset
```

there are two parallel questions:

1. Are the values equal?
2. If equal, where should the PC go?

## 18.1 Equality test

The slides reuse the ALU:

```text
Reg[rs] - Reg[rt]
```

If result = 0:

```text
Zero = 1
```

Therefore equality is detected without a separate full comparator in the initial single-cycle design.

---

## 18.2 Branch target calculation

The branch target is:

```text
PC + 4 + (sign_extended_offset << 2)
```

Why shift left 2?

The displacement counts words, while the PC is a byte address.

Shifting left by 2 multiplies by 4:

```text
offset 1 → +4 bytes
offset 2 → +8 bytes
offset -1 → -4 bytes
```

The slide notes that this shift is essentially a rerouting of wires, because shifting by a constant does not require a general-purpose shifter.

---

# 19. Branch PC selection

There are now two possible next sequential/branch PC values:

```text
PC + 4 -------------------0\
                            >---- next PC
Branch target ------------1/
                           ↑
                         PCSrc
```

A common logic relation is:

```text
PCSrc = Branch AND Zero
```

So:

```text
Branch=0 → normal PC+4 regardless of Zero
Branch=1, Zero=0 → branch not taken → PC+4
Branch=1, Zero=1 → branch taken → branch target
```

This is a good example of how **datapath status** (`Zero`) and **decoded control** (`Branch`) combine.

---

# 20. Why the first-cut single-cycle datapath needs separate instruction and data memory

A single-cycle instruction may need:

- instruction memory at the beginning of its cycle
- data memory later in the same cycle for `lw/sw`

If there were one single-ported memory resource, the instruction would be trying to use the same resource for two roles in one cycle.

Therefore the simple design uses separate:

```text
Instruction Memory
Data Memory
```

This same resource conflict later appears again as a **structural hazard** in pipelining.

---

# 21. The important muxes in the full single-cycle datapath

Memorize them by purpose, not by location.

## 21.1 RegDst

Question:

> Which instruction field names the destination register?

```text
rt ----------------0\
                    >---- Write register address
rd ----------------1/
                   ↑
                 RegDst
```

- `lw` → `rt` → `RegDst=0`
- R-type → `rd` → `RegDst=1`

---

## 21.2 ALUSrc

Question:

> Where does ALU input B come from?

```text
ReadData2 --------------0\
                          >---- ALU B
SignExtendedImmediate --1/
                         ↑
                       ALUSrc
```

- R-type / branch → register → `0`
- load/store → immediate → `1`

---

## 21.3 MemtoReg

Question:

> What value should be written into the register file?

```text
ALU result --------0\
                    >---- register WriteData
Memory ReadData ---1/
                   ↑
                 MemtoReg
```

- R-type → ALU result → `0`
- `lw` → memory result → `1`

---

## 21.4 PCSrc / branch mux

Question:

> Continue sequentially or go to branch target?

```text
PC+4 ----------0\
                 >---- next PC
BranchTarget --1/
                ↑
              PCSrc
```

---

## 21.5 Jump mux

After branch selection, jump adds another possible next-PC source.

Conceptually:

```text
normal-or-branch-PC ----0\
                         >---- final next PC
jump target -----------1/
                        ↑
                      Jump
```

This “mux after mux” arrangement is normal. Each mux answers a distinct decision.

---

# 22. ALU control: why there are two levels of decoding

The main control unit does not need to directly output a full ALU operation for every instruction/funct combination.

The lecture uses:

```text
opcode → Main Control → ALUOp
funct + ALUOp → ALU Control → exact ALU command
```

The ALU control values in the slides are:

| ALU control | Function |
|---|---|
| `0000` | AND |
| `0001` | OR |
| `0010` | add |
| `0110` | subtract |
| `0111` | set-on-less-than |
| `1100` | NOR |

Why split control in two?

Because some instruction categories have an obvious ALU function:

```text
lw/sw → always ADD address
beq   → always SUBTRACT for compare
```

Only R-type needs the `funct` field to distinguish the exact ALU operation.

The slide's two-bit `ALUOp` scheme is:

```text
ALUOp = 00 → ADD       (lw/sw)
ALUOp = 01 → SUBTRACT  (beq)
ALUOp = 10 → inspect funct field (R-type)
```

Examples:

```text
opcode=lw → ALUOp 00 → ALU control 0010 (ADD)

opcode=beq → ALUOp 01 → ALU control 0110 (SUB)

opcode=R-type, funct=100100
→ ALUOp 10 + funct
→ ALU control 0000 (AND)
```

This is hierarchical decoding.

---

# 23. Main control unit: what it actually sees

The main control uses the **opcode**.

The instruction's remaining fields route to the datapath:

```text
opcode → control
rs     → read register 1
rt     → read register 2, and sometimes destination candidate
rd     → R-type destination candidate
immediate → sign extender
funct → ALU-control logic for R-type
```

So the control unit is not “executing” the instruction. It is generating the configuration that makes the datapath execute it.

---

# 24. The core control signals

These are the signals you should be able to explain from memory.

| Signal | If `0` | If `1` |
|---|---|---|
| `RegDst` | destination = `rt` | destination = `rd` |
| `RegWrite` | do not change register file | write selected register |
| `ALUSrc` | ALU B = register data | ALU B = sign-extended immediate |
| `MemRead` | no memory read | read data memory |
| `MemWrite` | no memory write | write data memory |
| `MemtoReg` | write ALU result to register | write memory data to register |
| `Branch` | not a branch request | branch instruction |
| `ALUOp` | category-dependent | tells ALU-control how to choose operation |
| `Jump` | no jump | next PC uses jump target |

And:

```text
PCSrc = Branch AND Zero
```

in the single-cycle branch scheme shown.

---

# 25. Control truth table for the important instruction classes

For the simplified lecture datapath:

| Instruction | RegDst | ALUSrc | MemtoReg | RegWrite | MemRead | MemWrite | Branch | ALUOp |
|---|---:|---:|---:|---:|---:|---:|---:|---|
| R-type | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 10 |
| `lw` | 0 | 1 | 1 | 1 | 1 | 0 | 0 | 00 |
| `sw` | X | 1 | X | 0 | 0 | 1 | 0 | 00 |
| `beq` | X | 0 | X | 0 | 0 | 0 | 1 | 01 |

`X` means **don't care**: that control value cannot affect architectural state for that instruction because some later write/action is disabled.

Example for `sw`:

- `RegWrite=0`
- therefore it does not matter which destination register the RegDst mux points toward.

This “don't care because the output is unused” concept is common in control design.

---

# 26. Walkthrough: R-type instruction

Take:

```text
add $t0, $t1, $t2
```

Step by step:

1. PC fetches instruction.
2. `rs=$t1`, `rt=$t2`, `rd=$t0` fields fan out.
3. Register file reads `$t1` and `$t2`.
4. `ALUSrc=0` selects the second register value.
5. `ALUOp=10`, so funct decides exact operation.
6. funct says add, so ALU adds.
7. `RegDst=1` selects `rd=$t0`.
8. `MemtoReg=0` selects ALU result.
9. `RegWrite=1` writes `$t0`.
10. No branch/jump, so next PC is PC+4.

Data memory exists physically but is irrelevant for this instruction.

---

# 27. Walkthrough: load

Take:

```text
lw $t0, 12($s1)
```

1. Fetch instruction.
2. Register file reads `rs=$s1`.
3. Immediate 12 is sign-extended.
4. `ALUSrc=1` chooses immediate.
5. `ALUOp=00` causes addition.
6. ALU calculates effective address.
7. `MemRead=1` reads memory at that address.
8. `RegDst=0` chooses `rt=$t0` as destination.
9. `MemtoReg=1` chooses memory output.
10. `RegWrite=1` stores data into `$t0`.
11. PC normally advances by 4.

This path is long because it passes through instruction memory, register file, ALU, data memory, then register write-back.

That becomes the critical path in the Part 1 performance discussion.

---

# 28. Walkthrough: branch-on-equal

Take:

```text
beq $t0, $t1, offset
```

Two computations happen conceptually:

```text
comparison:
Reg[$t0] - Reg[$t1] → Zero

branch target:
PC+4 + (sign_extend(offset)<<2)
```

Then:

```text
PCSrc = Branch AND Zero
```

If equal, branch target is selected; otherwise PC+4.

No register file write and no data memory write occur.

---

# 29. Jump implementation

For a MIPS jump, the next address is constructed as:

```text
{ upper 4 bits of PC+4, instruction[25:0], 00 }
```

The slide summarizes it as concatenating:

- top 4 PC bits
- 26-bit jump address
- `00`

The low two zero bits again reflect word alignment.

A new `Jump` control signal is decoded from the opcode and selects the jump target through an added mux.

---

# 30. Why the single-cycle processor is slow

In a single-cycle processor, **every instruction must fit inside one clock cycle**.

The slides identify the load as the critical path:

```text
Instruction memory
→ Register file
→ ALU
→ Data memory
→ Register file
```

Even a simpler instruction such as `beq` must wait for the same global clock period.

This violates the performance principle “make the common case fast” because the clock is dictated by the longest instruction.

The solution introduced in Part 2 is to cut the work into stages and overlap different instructions.

---

# PART II — PIPELINING

# 31. Pipelining intuition: the laundry analogy

Suppose one laundry load needs:

```text
wash → dry → fold
```

Without pipelining, you finish all three steps for load 1 before starting load 2.

With pipelining:

```text
while load 1 dries,
load 2 can wash.

while load 1 folds,
load 2 dries,
load 3 washes.
```

No individual load necessarily finishes sooner.

The improvement is that **more loads finish per unit time**.

This is exactly the distinction in the slides:

- **latency** of one instruction does not decrease
- **throughput** increases

---

# 32. The five MIPS pipeline stages

These are absolutely fundamental.

```text
IF  →  ID  →  EX  →  MEM  →  WB
```

## IF — Instruction Fetch

```text
instruction ← InstructionMemory[PC]
PC+4 calculated
```

## ID — Instruction Decode / Register Read

```text
decode instruction
read source registers
generate control signals
```

## EX — Execute / Address Calculation

Depending on instruction:

```text
R-type → ALU operation
lw/sw  → calculate effective address
branch → arithmetic/comparison in the basic pipeline organization
```

## MEM — Memory Access

```text
lw → read data memory
sw → write data memory
other instructions mostly pass result through
```

## WB — Write Back

```text
R-type → ALU result to register
lw     → memory data to register
```

A useful mnemonic is:

```text
Fetch → Decode → Execute → Memory → Write
```

---

# 33. The most important pipeline timing picture

For instructions I1, I2, I3:

```text
cycle:   1    2    3    4    5    6    7
I1      IF   ID   EX   MEM  WB
I2           IF   ID   EX   MEM  WB
I3                IF   ID   EX   MEM  WB
```

Several instructions are alive simultaneously, but each one is in a different stage.

After the pipeline fills, ideally one instruction finishes every cycle.

That is why pipelining improves throughput.

---

# 34. Pipeline performance example from the slides

Given stage component delays:

- register read/write: 100 ps
- other major stages: 200 ps

The nonpipelined instruction times shown are:

| Instruction | Total |
|---|---:|
| `lw` | 800 ps |
| `sw` | 700 ps |
| R-format | 600 ps |
| `beq` | 500 ps |

A single-cycle CPU must choose:

```text
Tc = 800 ps
```

because the load is longest.

The pipelined example uses:

```text
Tc = 200 ps
```

because the longest pipeline stage is 200 ps in the simplified timing model.

Ideal balanced five-stage thinking would suggest about a 5× throughput improvement, but imperfect stage balance and pipeline fill/drain overhead reduce the actual gain.

---

# 35. Throughput versus latency

This is a common CT trap.

## Latency

Time from when one instruction starts until that same instruction finishes.

## Throughput

How frequently completed instructions emerge.

Pipelining primarily improves **throughput**.

Example:

```text
first instruction still travels IF→ID→EX→MEM→WB
```

But after filling:

```text
one instruction can complete on each cycle
```

The slides explicitly say:

> speedup is due to increased throughput; latency for each instruction does not decrease.

---

# 36. Pipeline speedup formulas and intuition

If all stages were perfectly balanced:

```text
pipeline interval ≈ nonpipeline instruction time / number of stages
```

For a long sequence of `N` instructions and `k` ideal pipeline stages:

```text
ideal cycles ≈ N + k - 1
```

For the five-stage pipeline:

```text
N + 4 cycles
```

before considering stalls and flushes.

The slide's three-instruction example gives:

```text
Tpipelined = 1400 ps
Tnonpipelined = 2400 ps
speedup = 2400/1400 ≈ 1.714
```

For very many instructions, pipeline fill/drain overhead becomes negligible, so speedup approaches the stage-count/slowest-stage limit. The slide's million-instruction example approaches 4× with its 800 ps versus 200 ps timing.

---

# 37. Why MIPS is convenient for pipelining

The slides list ISA properties that simplify pipeline implementation.

## Fixed 32-bit instruction size

Fetching boundaries are predictable.

Contrast given by slide: x86 instructions can vary in length.

## Few, regular instruction formats

Fields appear in predictable positions, which simplifies decode and register reading.

## Load/store architecture

Memory operands are accessed by explicit load/store instructions.

This lets address calculation occur in EX and actual memory access occur in MEM.

## Aligned memory operands

Memory access is simpler and can be treated as one pipeline stage in the model.

The broader lesson:

> ISA design and hardware organization influence each other.

---

# 38. What is a hazard?

A **hazard** is a situation where simply starting the next instruction in the next cycle would cause a problem.

The slides divide hazards into three types:

```text
1. Structural hazard → hardware resource conflict
2. Data hazard       → needed data is not ready
3. Control hazard    → correct next instruction is not known yet
```

A good classification test:

```text
same hardware wanted by two stages?       → structural
value dependency between instructions?    → data
uncertain PC due to branch/jump?           → control
```

---

# 39. Structural hazards

Suppose instruction and data access share one single-ported memory.

At some cycle:

```text
Instruction A in MEM wants data memory
Instruction B in IF wants instruction memory
```

If they are physically the same memory port, both cannot use it at once.

Result:

```text
one must wait → stall/bubble
```

The MIPS pipeline in the lecture avoids this particular hazard by using separate:

- instruction memory/cache
- data memory/cache

This is a design choice that removes a structural conflict.

---

# 40. Data hazards: the central timing problem

Consider:

```asm
add $s0, $t0, $t1
sub $t2, $s0, $t3
```

The `sub` needs the new `$s0` produced by `add`.

Naively:

```text
add: IF ID EX MEM WB
sub:    IF ID EX MEM WB
```

The `sub` reads registers in ID before `add` has reached normal WB.

So if the CPU only relies on the register file, the `sub` may see the **old `$s0`**.

But notice something important:

The ALU result of `add` already exists at the **end of add's EX stage**.

It does not need to wait until WB merely to become mathematically valid.

This observation leads to forwarding.

---

# 41. Forwarding / bypassing

Forwarding means:

> Route a result directly from a later pipeline location to an earlier consumer input instead of waiting for it to be written to and re-read from the register file.

Without forwarding:

```text
producer result → eventually WB → register file → later consumer
```

With forwarding:

```text
producer result → EX/MEM or MEM/WB pipeline register
                         ↓
                    mux / bypass path
                         ↓
                  consumer ALU input
```

This is why the forwarding datapath has extra wires returning from the right side toward ALU input muxes on the left.

The Part 2 slide labels this right-to-left flow as a source of hazards, but these added right-to-left bypass paths are also how data hazards are resolved.

---

# 42. The timing question that solves most forwarding problems

For every dependency, ask only two questions:

```text
1. When is the producer's value READY?
2. When does the consumer NEED the value?
```

If:

```text
ready time ≤ need time
```

and there is a hardware path, forwarding can solve it.

If:

```text
ready time > need time
```

forwarding would require “time travel,” so you must stall.

This is exactly the logic behind the lecture's load-use slides.

---

# 43. Why ordinary ALU-to-ALU dependency can be forwarded

Example:

```asm
add $s0, $t0, $t1
sub $t2, $s0, $t3
```

Timeline:

```text
cycle:  1    2    3    4    5    6
add    IF   ID   EX   MEM  WB
sub         IF   ID   EX   MEM  WB
```

`add` produces its ALU result at the end of cycle 3.

`sub` needs that value for its ALU in cycle 4.

Therefore:

```text
add result is ready before sub EX consumes it
```

So forward from the producer's pipeline register into the consumer ALU input.

No stall is required for this dependency if the forwarding hardware exists.

---

# 44. Load-use hazard: why forwarding alone fails

Consider:

```asm
lw  $s0, 20($t1)
sub $t2, $s0, $t3
```

Timeline without stall:

```text
cycle:  1    2    3    4    5    6
lw     IF   ID   EX   MEM  WB
sub         IF   ID   EX   MEM  WB
```

The load does not obtain the loaded value from data memory until the **end of MEM**, cycle 4.

But the `sub` needs `$s0` as an ALU input **during its EX stage in cycle 4**.

So:

```text
value ready: end of cycle 4
value needed: during cycle 4
```

You cannot forward a value backward to an earlier point in the same cycle's computation.

Hence:

```text
one-cycle stall is required
```

Then the consumer EX moves to the next cycle, where forwarding from the load result is possible.

This is the classic load-use hazard.

---

# 45. Code scheduling to avoid stalls

The slides show that the compiler can sometimes reorder independent instructions.

Original structure:

```asm
lw   $t1, 0($t0)
lw   $t2, 4($t0)
add  $t3, $t1, $t2   # immediate load-use dependency on $t2
sw   $t3, 12($t0)
lw   $t4, 8($t0)
add  $t5, $t1, $t4   # immediate load-use dependency on $t4
sw   $t5, 16($t0)
```

The lecture marks two stalls, giving 13 cycles.

A reordered version moves the independent third load upward:

```asm
lw   $t1, 0($t0)
lw   $t2, 4($t0)
lw   $t4, 8($t0)
add  $t3, $t1, $t2
sw   $t3, 12($t0)
add  $t5, $t1, $t4
sw   $t5, 16($t0)
```

Now useful work occupies the delay gap and the shown schedule takes 11 cycles.

This connects hardware design to compiler knowledge:

> A compiler can schedule instructions better if it knows pipeline latency constraints.

---

# 46. Control hazards

A branch changes which instruction should execute next.

The problem is that while the branch is still being processed, the IF stage wants to fetch something immediately.

Example:

```asm
beq $1, $2, target
next_instruction
...
target:
```

Before branch outcome is known, should IF fetch:

```text
PC+4 ?
```

or:

```text
branch target ?
```

That uncertainty is a control hazard.

---

# 47. Why moving branch decision earlier helps

If branch outcome is determined late, many younger instructions may already have entered the pipeline.

The slides propose adding hardware so the branch can be handled in the **ID stage**:

- target-address adder
- register comparator
- PC update capability

Earlier decision means fewer incorrectly fetched instructions need to be discarded.

This introduces another key design principle:

> Extra hardware can reduce pipeline penalties.

---

# 48. Branch prediction

Instead of always waiting for certainty, the processor can guess.

The basic strategy:

```text
predict branch outcome
↓
fetch according to prediction
↓
when actual outcome is known:
    if correct → continue
    if wrong   → discard wrong-path work and fetch correct path
```

The slides introduce **predict not taken** for simple MIPS:

```text
assume PC+4
```

If the branch really is not taken, there is no prediction penalty.

If the branch is taken, the speculative sequential instruction(s) must be discarded and the target fetched.

---

# 49. Static branch prediction

Static prediction does not maintain a per-branch runtime history.

The slides give the common heuristic:

```text
backward branch → predict taken
forward branch  → predict not taken
```

Why can that work?

A backward branch often represents a loop back-edge, which is taken repeatedly until the final iteration.

A forward branch often behaves more like skipping a block conditionally.

It is only a heuristic, not a guarantee.

---

# 50. Dynamic branch prediction

Dynamic prediction uses actual observed branch behavior.

The slides describe a branch prediction buffer / branch history table:

```text
branch PC → index predictor entry
predict using stored history
execute branch
if wrong:
    flush wrong-path instructions
    update predictor state
```

The central idea is:

> Past behavior of this branch is used as evidence for its next behavior.

---

# 51. Delayed branch

The lecture also presents the historical MIPS delayed-branch technique.

Rule:

> The instruction immediately after the branch always executes.

So software/compiler tries to place a useful instruction in that **delay slot**.

Conceptually:

```asm
beq  ... target
useful_instruction     # runs whether branch is taken or not
```

The branch transfer is delayed by one instruction.

This can hide a branch delay if the compiler can find a safe useful instruction for the slot.

---

# 52. Pipeline summary before building the hardware

At this point the lecture has established:

```text
Pipelining
→ overlaps instructions
→ increases throughput
→ does not inherently reduce one-instruction latency

But overlapping creates hazards:
→ structural
→ data
→ control
```

Now the slides move from timing diagrams into the actual pipelined datapath.

---

# 53. Why pipeline registers are necessary

A pipelined datapath is divided into stages:

```text
IF | ID | EX | MEM | WB
```

There must be state elements at the boundaries:

```text
IF → [IF/ID] → ID → [ID/EX] → EX → [EX/MEM] → MEM → [MEM/WB] → WB
```

Without these registers, the combinational outputs of one stage would simply continue changing through later stages during the same clock interval. You would not truly have separated pipeline stages.

A pipeline register does two things:

1. captures all information this instruction needs for later stages
2. keeps that information associated with the correct instruction while newer instructions enter earlier stages

A useful analogy is an envelope traveling with the instruction:

```text
instruction enters stage
stage computes new information
pipeline register seals everything needed into envelope
next clock → envelope handed to next stage
```

---

# 54. What information goes inside each pipeline register?

Do **not** think of a pipeline register as one 32-bit register.

It is really a bundle of several registers/buses.

The lecture initially illustrates approximate widths such as:

```text
IF/ID   ≈ 64 bits
ID/EX   ≈ 128 bits initially shown
EX/MEM  ≈ 97 bits initially shown
MEM/WB  ≈ 64 bits initially shown
```

and explicitly warns that sizes may need updating as the design proceeds.

Why do sizes differ?

Because different boundaries need to preserve different information.

For example, IF/ID might need both:

```text
instruction (32)
PC+4        (32)
```

so:

```text
32 + 32 = 64 bits
```

The ID/EX boundary must carry multiple register values, immediate, register identifiers, and later the control bits needed by downstream stages.

The exact final width depends on which signals the implementation chooses to carry.

---

# 55. The rule for deciding whether a signal must be pipelined

This is one of the most useful design rules in the whole chapter:

> If information is created/available now but will be needed by this same instruction in a later stage, it must travel through the intervening pipeline registers.

Examples:

- `lw` knows its destination register number in ID.
- actual write-back happens in WB several cycles later.
- therefore the destination register number must be carried through ID/EX → EX/MEM → MEM/WB.

Similarly:

- `RegWrite` is decoded in ID.
- the actual register write happens in WB.
- therefore the `RegWrite` bit must travel with the instruction all the way to WB.

This directly explains the “wrong register number” correction later in the slides.

---

# 56. Pipeline operation for `lw`, stage by stage

Take:

```asm
lw $t0, 12($s1)
```

## IF

Active resources:

```text
PC
instruction memory
PC+4 adder
IF/ID register
```

At end of stage, preserve instruction and PC-related information.

---

## ID

Active resources:

```text
instruction decode
register file read
sign extension
control generation
ID/EX register
```

The processor now knows:

- base register value
- immediate
- destination field `rt`
- downstream control requirements

All needed later information must be latched into ID/EX.

---

## EX

For load:

```text
ALU = base + immediate
```

This computes the effective memory address.

The resulting address, destination register identifier, and relevant control information move into EX/MEM.

---

## MEM

Data memory reads from the calculated address.

The loaded data moves into MEM/WB along with the destination register and WB controls.

---

## WB

The memory value is selected and written into the correct register.

The key insight is that the original instruction itself is long gone from the front of the pipeline, so every fact needed for WB must have been carried forward.

---

# 57. Pipeline operation for `sw`

A store has:

```asm
sw rt, offset(rs)
```

## ID

Read:

- `rs` = base
- `rt` = value to store

## EX

Calculate:

```text
address = Reg[rs] + sign_extended_offset
```

## MEM

Use:

```text
memory[address] ← stored register value
```

## WB

No architectural register write is needed.

The slides still show the pipeline reaching the WB position because pipeline timing remains regular, but the WB control for the store is inactive.

This is an important principle:

> An instruction can pass through a pipeline stage without doing a meaningful architectural action there.

---

# 58. Why the load datapath initially writes the wrong register

This is one of the most educational slides.

Suppose a load is currently in WB.

At the same clock cycle, a completely different younger instruction is in ID.

If the WB write-register address were taken directly from the instruction currently sitting in ID, the register file would receive the **wrong instruction's register number**.

Therefore:

```text
load's destination register number
must travel with the load through the pipeline
```

The corrected datapath adds a route that carries the destination register identifier forward to MEM/WB.

This illustrates the general pipeline principle:

```text
DATA must be pipelined
CONTROL must be pipelined
IDENTIFIERS/metadata must be pipelined
```

if they are needed later.

---

# 59. Multi-cycle pipeline diagrams versus single-cycle pipeline diagrams

The lecture uses two different visualization styles.

## Multi-cycle diagram

Shows how several instructions progress over time:

```text
          C1   C2   C3   C4   C5   C6
I1        IF   ID   EX   MEM  WB
I2             IF   ID   EX   MEM  WB
I3                  IF   ID   EX   MEM  WB
```

Best for:

- hazards
- stalls
- timing
- instruction overlap

## Single-cycle pipeline diagram

Takes **one clock cycle** and shows which datapath regions are simultaneously occupied by different instructions.

Best for:

- seeing hardware sharing
- seeing which instruction is in which stage
- tracing simultaneous control/data movement

Do not confuse “single-cycle pipeline diagram” with “single-cycle CPU.”

It means a snapshot of one cycle of a pipelined CPU.

---

# 60. Pipelined control: the key idea

The control signals are still derived from the instruction, just as in the single-cycle processor.

But now the instruction will need those signals at different future stages.

Therefore the control bits are grouped by where they are eventually used.

A standard grouping matching the slides is:

```text
EX controls:
- RegDst
- ALUOp
- ALUSrc

MEM controls:
- Branch
- MemRead
- MemWrite

WB controls:
- RegWrite
- MemtoReg
```

The control unit generates them in ID, but they are carried through the pipeline.

This is why the simplified pipelined-control slide warns that seeing `RegWrite` near decode can be confusing:

> `RegWrite` is generated based on the instruction in ID, but the actual action controlled by it occurs when that instruction reaches WB.

---

# 61. Control bits travel with their instruction

Visualize each instruction as carrying a little packet:

```text
instruction data packet:
{
    operands,
    immediate,
    destination register,
    EX control bits,
    MEM control bits,
    WB control bits
}
```

As it moves:

```text
ID/EX:
keep EX + MEM + WB controls

EX/MEM:
EX work is finished,
so only MEM + WB controls still matter

MEM/WB:
MEM work is finished,
so only WB controls still matter
```

This is why the control bundle becomes smaller as the instruction moves right.

---

# 62. Register-file read/write in the same clock cycle

One forwarding slide states the convention:

> A read of a register during a clock cycle returns the value written at the end of the first half of that cycle when such a write occurs.

Conceptually the cycle is arranged so that:

```text
first part of cycle  → WB writes register file
later part of cycle  → ID reads register file
```

Therefore an instruction in ID can see a value being written by an older instruction in WB during the same overall cycle.

This eliminates some would-be hazards without extra forwarding.

When solving timing problems from the slides, use this register-file timing convention.

---

# 63. Data-hazard example used for forwarding

The slides use:

```asm
sub $2,  $1, $3
and $12, $2, $5
or  $13, $6, $2
add $14, $2, $2
sw  $15, 100($2)
```

The first instruction produces `$2`.

Several later instructions consume `$2` at different distances.

The example demonstrates:

- some dependencies need EX/MEM forwarding
- some need MEM/WB forwarding
- sufficiently distant consumers can read the new value directly from the register file
- register identifiers must be carried down the pipeline so the hardware can compare producer destination with consumer source

---

# 64. Why register numbers must be passed through the pipeline

The forwarding unit must answer:

> Does the instruction producing a value write the same register that the current EX-stage instruction is trying to use?

For the current EX-stage instruction, source register numbers are in:

```text
ID/EX.RegisterRs
ID/EX.RegisterRt
```

Potential producing destinations are in:

```text
EX/MEM.RegisterRd
MEM/WB.RegisterRd
```

The slide writes potential matches as:

```text
1a. EX/MEM.RegisterRd = ID/EX.RegisterRs
1b. EX/MEM.RegisterRd = ID/EX.RegisterRt

2a. MEM/WB.RegisterRd = ID/EX.RegisterRs
2b. MEM/WB.RegisterRd = ID/EX.RegisterRt
```

These equalities are the core comparison tests.

But equality alone is not sufficient.

---

# 65. Why forwarding also checks `RegWrite`

Imagine an older `sw` has a field whose bits happen to equal the consumer's source register number.

That does not mean `sw` is producing a new register value.

Therefore the forwarding unit must only treat an instruction as a producer when:

```text
EX/MEM.RegWrite = 1
or
MEM/WB.RegWrite = 1
```

This expresses the semantic fact:

> Only an instruction that will actually write a register can be a register-value producer.

---

# 66. Why forwarding checks destination ≠ `$zero`

In MIPS, register `$zero` always reads as zero and should not behave like a normal writable destination.

Therefore the slide adds:

```text
EX/MEM.RegisterRd != 0
MEM/WB.RegisterRd != 0
```

Otherwise a pseudo-write aimed at register 0 might incorrectly be treated as producing a value that should be forwarded.

---

# 67. Forwarding muxes

The normal ALU inputs are no longer simple two-source choices.

Each operand may come from:

```text
00 → value stored in ID/EX (normal register value)
10 → newer result in EX/MEM
01 → older result in MEM/WB
```

Conceptually:

```text
                         ┌──────────── ID/EX operand
                         │
ALU input A ← 3-way mux ←┼──────────── MEM/WB result
                         │
                         └──────────── EX/MEM result
                              ↑
                           ForwardA
```

and similarly for `ForwardB`.

Why is `10` used for EX/MEM and `01` for MEM/WB?

That is simply the encoding selected by this design. The important concept is which source each encoding selects.

---

# 68. Forwarding conditions from the slides

## EX hazard

For operand A:

```text
if EX/MEM.RegWrite
and EX/MEM.RegisterRd != 0
and EX/MEM.RegisterRd == ID/EX.RegisterRs
then ForwardA = 10
```

For operand B:

```text
if EX/MEM.RegWrite
and EX/MEM.RegisterRd != 0
and EX/MEM.RegisterRd == ID/EX.RegisterRt
then ForwardB = 10
```

## MEM hazard

For A:

```text
if MEM/WB.RegWrite
and MEM/WB.RegisterRd != 0
and MEM/WB.RegisterRd == ID/EX.RegisterRs
then ForwardA = 01
```

For B:

```text
if MEM/WB.RegWrite
and MEM/WB.RegisterRd != 0
and MEM/WB.RegisterRd == ID/EX.RegisterRt
then ForwardB = 01
```

The labels “EX hazard” and “MEM hazard” refer to where the supplying result currently resides, not where the consumer is.

The consumer ALU is in EX.

---

# 69. Double data hazard: why forwarding needs priority

Consider:

```asm
add $1, $1, $2
add $1, $1, $3
add $1, $1, $4
```

When the third instruction executes, two older instructions may both appear to be possible producers for `$1`:

```text
EX/MEM has result from instruction 2  ← newer
MEM/WB has result from instruction 1  ← older
```

You must use the most recent program value: instruction 2's result.

Therefore EX/MEM forwarding has priority over MEM/WB forwarding.

The lecture's numerical illustration shows why forwarding the older 40 instead of the newer 60 would be wrong.

This gives the revised MEM condition:

> Only use the MEM/WB source if the EX/MEM source is **not already the matching newer producer**.

---

# 70. Revised MEM forwarding condition

For source `Rs`, conceptually:

```text
if MEM/WB is a valid writer to Rs
AND there is NOT a newer valid EX/MEM writer to Rs
then ForwardA = 01
```

Likewise for `Rt`.

Expanded in the slide style:

```text
if MEM/WB.RegWrite
and MEM/WB.RegisterRd != 0
and not(
    EX/MEM.RegWrite
    and EX/MEM.RegisterRd != 0
    and EX/MEM.RegisterRd == ID/EX.RegisterRs
)
and MEM/WB.RegisterRd == ID/EX.RegisterRs
then ForwardA = 01
```

The same structure applies to `ForwardB` with `RegisterRt`.

The principle to remember is more important than memorizing the parentheses:

```text
nearest/newest valid producer wins
```

---

# 71. Load → store memory-to-memory copy

The slide considers:

```asm
lw $2, 20($1)
sw $2, 15($3)
```

This is worth analyzing separately from classic ALU load-use.

Ask:

```text
When is loaded data ready?
When is store data actually needed?
```

For `lw`, the value becomes available after its MEM access.

For `sw`, the value being stored is ultimately consumed by **data memory in the store's MEM stage**, which is later than an ALU operand would be consumed.

Therefore a datapath with an appropriate memory-to-memory/store-data forwarding path can potentially route the load result directly to the store's memory-write input.

The important lesson from this slide is:

> A dependency involving a loaded value is not automatically “one stall.” You must ask **where and when the consumer uses the value**.

---

# 72. Load-use hazard detection logic

The lecture detects a classic immediate load-use hazard while the consumer is in ID.

The load is one stage ahead in ID/EX.

For a load:

```text
ID/EX.MemRead = 1
```

In this MIPS format, the load destination is `rt`:

```text
ID/EX.RegisterRt
```

The instruction currently in IF/ID may use source registers:

```text
IF/ID.RegisterRs
IF/ID.RegisterRt
```

So the slide's simplified condition is:

```text
ID/EX.MemRead
AND
(
    ID/EX.RegisterRt == IF/ID.RegisterRs
    OR
    ID/EX.RegisterRt == IF/ID.RegisterRt
)
```

If true:

```text
stall and insert bubble
```

For the lecture's simplified instruction subset, this is the condition to learn.

---

# 73. Why hazard detection occurs in ID

At the moment:

```text
load        → ID/EX
consumer    → IF/ID
```

we still have time to stop the consumer before it enters EX incorrectly.

Therefore the hazard detection unit compares:

- the load's destination information in ID/EX
- the consumer's source fields in IF/ID

If there is a load-use dependency, the consumer must remain in ID one extra cycle.

---

# 74. What does “stall the pipeline” actually mean electrically?

The slide gives three actions.

## Action 1 — Prevent PC update

```text
PCWrite = 0
```

PC retains its current value.

Therefore the next fetch address does not advance.

## Action 2 — Prevent IF/ID update

```text
IF/ID.Write = 0
```

The consumer instruction remains in IF/ID instead of being overwritten.

## Action 3 — Put zero control fields into ID/EX

Instead of sending the consumer's real controls into EX, force all relevant controls to zero.

That creates a **bubble / nop**.

So one conceptual stall cycle is:

```text
freeze front of pipeline
+ inject harmless no-op into the stage ahead
```

---

# 75. Why making all control fields zero creates a bubble

A bubble should not change architectural state.

If its controls are zero:

```text
RegWrite = 0
MemWrite = 0
MemRead  = 0
Branch   = 0
...
```

then it can physically flow through EX/MEM/WB without causing meaningful writes.

So a bubble is not literally “nothing existing.”

It is a pipeline slot carrying control values that make it behave like a no-operation.

---

# 76. What repeats during a stall?

The slides make this more precise.

When a load-use stall occurs:

- the dependent instruction in ID is decoded again next cycle
- the following instruction in IF is fetched again

because PC and IF/ID were frozen.

Meanwhile a zero-control bubble is inserted into EX.

This is a better mental picture than saying “the whole CPU stops.”

The older instructions to the right continue moving normally.

```text
older instructions: continue
bubble: inserted
consumer: held
following fetch: held/repeated
```

---

# 77. Datapath with hazard detection: understand every added connection

The hazard-detection datapath adds logic around the front of the pipeline.

Inputs to the hazard unit include:

```text
ID/EX.MemRead
ID/EX.rt
IF/ID.rs
IF/ID.rt
```

Outputs affect:

```text
PC write enable
IF/ID write enable
control values entering ID/EX
```

So hazard detection has two jobs:

```text
DETECT:
Is the next instruction unsafe?

ACT:
freeze PC + IF/ID
inject bubble by zeroing controls
```

This is exactly the kind of control structure you will eventually implement in hardware if your custom project includes pipelining.

---

# 78. Stalls reduce performance but preserve correctness

The slides emphasize:

- stalls reduce performance
- sometimes they are required for correct execution
- compiler scheduling can reduce them
- compiler must know pipeline structure to schedule effectively

So a processor design question is always a tradeoff:

```text
more forwarding/control hardware
↔ fewer stalls
↔ more complexity
```

---

# 79. Branch hazards when branch outcome is determined in MEM

If a branch is not resolved until MEM, then several younger instructions may already be in the pipeline on the assumed path.

For a taken branch, those instructions are wrong-path instructions.

The slide shows that they must be **flushed**.

Flushing means making them harmless, usually by clearing their control signals to zero so that they cannot update register/memory state.

Compare the vocabulary:

```text
STALL → preserve an instruction and delay progress
FLUSH → discard an instruction because it should never have executed
BUBBLE → harmless empty/no-op slot traveling through pipeline
```

These are related but not identical.

---

# 80. Reducing branch delay by moving branch resolution to ID

The slides then move branch work earlier.

Add hardware in ID for:

```text
branch target address calculation
register comparison
```

Now a branch can redirect the PC earlier.

The earlier the correct next PC becomes known, the fewer wrong-path instructions need to be fetched/decoded.

But there is a consequence:

> If branch comparison happens in ID, branch operands are needed **earlier** than normal ALU operands.

That creates special branch data-hazard timing rules.

---

# 81. Data hazards for branches: why the rules differ

Ordinary ALU instruction consumer:

```text
needs operands in EX
```

Early branch comparator:

```text
needs operands in ID
```

So a value that would be available in time for an EX consumer may **not** be available in time for a branch's ID comparator.

This is why the branch-hazard slides show additional stalls.

Always ask:

```text
producer ready where/when?
branch comparator needs value where/when?
```

---

# 82. Branch case: 2nd or 3rd preceding ALU instruction

The slide says:

> If a comparison register is a destination of the 2nd or 3rd preceding ALU instruction, the dependency can be resolved using forwarding.

Example structure:

```asm
add $1, $2, $3
add $4, $5, $6
...
beq $1, $4, target
```

Because the producers are sufficiently far ahead, their results have progressed far enough through the pipeline to be forwarded to the branch comparator in ID.

No extra stall is required for those specific distances in the shown design.

---

# 83. Branch case: immediately preceding ALU instruction

If the branch depends on the ALU instruction immediately before it, the producer result is too new to reach the ID-stage comparator on time.

The slides group this with the case of a 2nd-preceding load and state:

```text
need 1 stall cycle
```

After one cycle of delay, the value has advanced far enough to be forwarded/observed for the branch comparison.

---

# 84. Branch case: 2nd preceding load

The lecture example:

```asm
lw  $1, addr
add $4, $5, $6
beq $1, $4, target
```

The load data is produced late, at memory access.

With this distance, the branch still reaches ID too early, so:

```text
1 stall
```

is required in the shown early-branch design.

---

# 85. Branch case: immediately preceding load

Example:

```asm
lw  $1, addr
beq $1, $0, target
```

Now the branch wants the loaded value almost immediately, in its ID comparator.

The slide states:

```text
need 2 stall cycles
```

Why more than classic load-use?

Because classic load-use ALU consumer needs the value in EX.

This branch consumer needs the value one stage earlier, in ID.

So the timing gap is larger.

This is a very likely conceptual CT question.

---

# 86. A compact branch-dependency table

For the early-ID branch scheme shown in the slides:

| Producer relative to branch | Producer type | Remedy |
|---|---|---|
| 2nd/3rd preceding | ALU | forwarding |
| immediately preceding | ALU | 1 stall |
| 2nd preceding | load | 1 stall |
| immediately preceding | load | 2 stalls |

Do not memorize this table blindly. Derive it by comparing **ready time** to the branch's **ID-stage need time**.

---

# 87. Dynamic branch prediction in more detail

The slides introduce a branch history table/prediction buffer indexed by branch instruction addresses.

Basic behavior:

```text
fetch branch
↓
look up predictor entry using branch PC
↓
predict taken/not taken
↓
fetch predicted path
↓
resolve actual branch
↓
if wrong:
    flush
    redirect PC
    update prediction history
```

Deeper and superscalar pipelines benefit more from accurate prediction because more work is in flight behind each unresolved branch.

---

# 88. Why a one-bit predictor has a loop problem

Imagine an inner loop branch that behaves:

```text
T T T T T N
```

for each complete execution of the loop.

A one-bit predictor remembers only the last outcome.

At loop exit:

```text
predict T, actual N → mispredict
state becomes N
```

Next time the inner loop starts:

```text
predict N, actual T → mispredict again
```

Thus the slides note that the inner-loop branch may be mispredicted twice: once on exit, and once on the first iteration next time.

---

# 89. Two-bit predictor intuition

A two-bit predictor adds **hysteresis**: one surprising outcome does not immediately reverse the prediction direction.

The slide summarizes this as:

> Only change prediction on two successive mispredictions.

A common state interpretation is:

```text
Strongly Taken
Weakly Taken
Weakly Not Taken
Strongly Not Taken
```

A single opposite outcome generally weakens confidence; repeated opposite behavior is needed to cross from predicting one direction to the other.

This handles loop exits better than a one-bit memory.

---

# 90. Branch target buffer

Prediction answers:

```text
Taken or not taken?
```

But a taken prediction also needs:

```text
Where is the target?
```

If target calculation itself takes a cycle, even a correct “taken” prediction can incur delay.

The slides therefore introduce a **branch target buffer (BTB)**:

```text
PC → lookup cached target
```

If:

- BTB hits
- branch is predicted taken

then the target can be fetched immediately.

So:

```text
branch predictor → direction
BTB              → target address
```

They solve different parts of the branch problem.

---

# PART III — ADVANCED INSTRUCTION-LEVEL PARALLELISM

# 91. Instruction-Level Parallelism (ILP)

Pipelining already overlaps instructions, so it is a form of instruction-level parallelism.

The slides describe two ways to push ILP further:

## Deeper pipeline

Split work into more stages.

Potential benefit:

```text
less combinational work per stage
→ shorter clock cycle
```

But deeper pipelines can make hazards and branch penalties more costly.

## Multiple issue

Start more than one instruction in a single cycle.

If a CPU can issue four instructions per cycle, peak:

```text
IPC = 4
CPI = 1/4 = 0.25
```

The lecture example states a 4 GHz four-way issue machine has a theoretical peak of 16 billion instructions per second.

Real dependencies prevent perfect peak behavior.

---

# 92. Static versus dynamic multiple issue

## Static multiple issue

Compiler decides which instructions can be grouped together.

It packages them into **issue slots/packets**.

The compiler is responsible for avoiding many hazards.

## Dynamic multiple issue

The CPU inspects the instruction stream at runtime and decides how many instructions to issue.

The hardware detects dependencies and handles hazards dynamically.

Compiler scheduling may still help, but correctness is enforced by the CPU.

---

# 93. Speculation

Speculation means:

```text
guess what will be safe/useful
→ begin work early
→ later verify
→ if correct, keep result
→ if incorrect, roll back/discard
```

Examples from the slides:

- speculate on branch outcome
- speculate on a load

Speculation is useful because waiting for certainty can waste parallel execution opportunities.

But it requires recovery mechanisms when guesses are wrong.

---

# 94. Compiler versus hardware speculation

The slides distinguish two places speculation can happen.

## Compiler speculation

Compiler may move an instruction earlier, such as a load before a branch.

It can add fix-up code for exceptional/wrong cases.

## Hardware speculation

Hardware can execute instructions before knowing they definitely belong on the final path.

It may buffer results and only make them architecturally visible once safe.

If speculation was wrong:

```text
flush buffered speculative work
```

---

# 95. Static multiple issue and VLIW

In static multiple issue, the compiler groups independent operations that can execute concurrently.

An **issue packet** can be thought of as a very long instruction containing multiple operation slots.

This idea leads to:

```text
VLIW = Very Long Instruction Word
```

The compiler must understand the available hardware resources and dependency constraints.

If a slot cannot safely be filled, it may be padded with `nop`.

---

# 96. Scheduling static multiple issue

The compiler must:

```text
find independent instructions
reorder them where semantics permit
place compatible operations in the same packet
avoid intra-packet dependencies
manage some inter-packet dependencies
insert nop when no safe instruction exists
```

The exact rules depend on the ISA and implementation.

This is another direct connection between compiler design and microarchitecture.

---

# 97. Static dual-issue MIPS in the slides

The lecture's example issues two instructions per packet:

```text
slot 1: one ALU/branch instruction
slot 2: one load/store instruction
```

A packet is 64-bit aligned because it contains two 32-bit instructions.

Layout:

```text
address n:     ALU/branch
address n+4:   load/store
address n+8:   next ALU/branch
address n+12:  next load/store
...
```

If one slot has no usable instruction:

```text
insert nop
```

Peak IPC is 2, but dependencies and slot restrictions reduce actual IPC.

---

# 98. Hazards become harder in dual issue

With two instructions starting in the same cycle, some dependencies occur with effectively zero scheduling distance.

The slides give:

```asm
add  $t0, $s0, $s1
load $s2, 0($t0)
```

If both are in the same issue packet, the load wants an address based on the add result essentially immediately.

Single-issue forwarding that helped one-cycle-separated instructions cannot magically satisfy same-packet timing.

Therefore they must be split into different packets, effectively creating lost issue capacity.

The classic load-use latency also remains.

Hence the compiler needs more aggressive scheduling.

---

# 99. Dual-issue scheduling example

The slides schedule this loop:

```asm
Loop:
    lw   $t0, 0($s1)
    addu $t0, $t0, $s2
    sw   $t0, 0($s1)
    addi $s1, $s1, -4
    bne  $s1, $zero, Loop
```

Into packets approximately:

| Cycle | ALU/branch slot | load/store slot |
|---:|---|---|
| 1 | `nop` | `lw $t0,0($s1)` |
| 2 | `addi $s1,$s1,-4` | `nop` |
| 3 | `addu $t0,$t0,$s2` | `nop` |
| 4 | `bne $s1,$zero,Loop` | `sw $t0,4($s1)` |

Why did the store offset become `4($s1)`?

Because the pointer decrement was moved earlier. To preserve the original memory address, the compiler adjusts the offset.

The slide computes:

```text
IPC = 5 useful instructions / 4 cycles = 1.25
```

which is below the peak IPC of 2.

---

# 100. Loop unrolling

Loop unrolling replicates the loop body so more independent work is visible at once.

Benefits from the slides:

- more parallel operations exposed
- less loop-control overhead per useful operation

But it costs:

- more registers
- larger code size

Different registers are used for replicated iterations, which the slides call **register renaming**.

This avoids false/name dependencies caused simply by reusing the same register name.

---

# 101. Anti-dependencies / name dependencies

The lecture points out that some apparent dependencies are caused by reuse of a register name rather than a true data flow requirement.

Register renaming gives logically separate values different physical/register names so they do not unnecessarily block parallel execution.

The conceptual distinction is:

```text
true dependency:
B needs the actual value produced by A

name dependency:
A and B happen to mention the same storage name,
but one value does not logically depend on the other
```

Renaming can remove the second kind, not the first.

---

# 102. Loop-unrolling performance example

The slide's unrolled schedule achieves:

```text
IPC = 14 / 8 = 1.75
```

which is closer to the dual-issue peak of 2 than the earlier 1.25.

The point is not the exact schedule alone; it demonstrates:

```text
more independent work exposed
→ compiler can fill more issue slots
→ higher IPC
```

at the expense of code size and register usage.

---

# 103. Dynamic multiple issue / superscalar processors

A superscalar CPU dynamically decides whether to issue:

```text
0, 1, 2, ... instructions per cycle
```

depending on:

- resource availability
- dependencies
- hazards

The CPU therefore performs more runtime scheduling work than a static multiple-issue design.

The compiler can still help, but the hardware is responsible for maintaining program semantics.

---

# 104. Dynamic pipeline scheduling / out-of-order execution

The slides give:

```asm
lw    $t0, 20($s2)
addu  $t1, $t0, $t2
sub   $s4, $s4, $t3
slti  $t5, $s4, 20
```

The `addu` must wait for the load result.

But `sub` does not depend on that loaded value.

An out-of-order processor can therefore do:

```text
load starts
addu waits
sub executes meanwhile
slti may follow when its dependency is ready
```

This avoids leaving execution units idle merely because an earlier instruction is stalled.

The slides emphasize:

> execution may occur out of order, but results are committed to architectural state in order.

---

# 105. Dynamically scheduled CPU blocks

The diagram introduces ideas such as:

- instruction fetch/decode
- reservation stations
- functional units
- pending operands
- result forwarding to waiting stations
- reorder buffer for register writes
- commit unit

Intuitively:

## Reservation station

Holds an instruction near a functional unit until its operands become available.

Instead of blocking the whole pipeline, that one instruction waits locally.

## Result broadcast/forwarding

When an execution result becomes available, waiting dependent instructions can receive it.

## Reorder buffer

Keeps speculative/out-of-order results until they can safely become architecturally visible in program order.

## Commit

Makes results official in program order.

This preserves the appearance that the program executed according to its defined semantics even if internal execution order differed.

---

# 106. Why dynamic scheduling cannot simply be replaced by compiler scheduling

The slides give several reasons.

## Not all delays are predictable

Example: cache misses.

A compiler cannot know exactly which memory access will miss at runtime in every execution.

## Branch outcomes are dynamic

Compiler cannot always know which path will execute.

## Different implementations have different timing

The same ISA can be implemented with different pipelines, latencies, and hazards.

A schedule perfect for one implementation may be poor for another.

Therefore runtime hardware scheduling can adapt to actual conditions.

---

# 107. Limits of multiple issue

The slides ask “Does multiple issue work?” and answer yes, but less than ideal peak rates suggest.

Limitations include:

- real data dependencies
- difficult aliasing questions, especially pointers
- limited instruction window size
- memory latency
- limited memory bandwidth
- difficulty keeping all pipelines busy

Speculation can expose more parallelism, but adds complexity.

So peak IPC is not the same as sustained application IPC.

---

# 108. Power efficiency and the complexity wall

Advanced features such as:

- dynamic scheduling
- speculation
- wide issue
- large prediction structures

consume transistors and power.

The slides contrast generations of processors and note that multiple simpler cores may sometimes be more power-efficient than making one core increasingly complex.

This is part of the broader “power wall” motivation for multicore processors.

---

# 109. Cortex-A8 and Core i7 comparison

The lecture includes a comparison table between an ARM Cortex-A8-class design and an Intel Core i7 920.

The important conceptual contrasts shown are:

- different target markets and power budgets
- both use deep pipelines
- both use multiple issue
- Core i7 uses more aggressive dynamic out-of-order/speculative techniques
- cache hierarchy size and organization differ
- peak instructions per cycle differ

The point is that the concepts taught in the simplified MIPS pipeline scale into real processor design, but commercial processors add far more machinery.

---

# 110. ARM and Core i7 pipeline/performance slides

The following slides show real pipeline block structures and benchmark performance graphs.

For study purposes, extract these lessons:

```text
real pipelines contain many more substages than IF/ID/EX/MEM/WB
real performance differs by workload
memory hierarchy stalls matter
branch behavior matters
multiple-issue utilization is imperfect
```

The simplified five-stage MIPS model is a conceptual framework, not a claim that modern processors literally contain only five physical stages.

---

# 111. Fallacies and pitfalls from the final slides

## “Pipelining is easy”

The basic overlap idea is simple.

The difficult parts are the details:

- dependency detection
- forwarding priority
- stall control
- flush control
- branch handling

This is exactly why Part 2 spends so many slides on hazards.

## “Pipelining is independent of technology”

The feasibility and benefit of advanced techniques depend on available transistor budgets and technology trends.

## Poor ISA design can make pipelining harder

The slides mention complex instruction sets, complex addressing modes, side effects, indirection, and delayed branches as examples of complications.

Thus:

```text
ISA design ↔ datapath/control design
```

influence each other.

---

# 112. Final chapter conclusion

The lecture closes with these central ideas:

1. ISA affects datapath and control design.
2. Datapath/control considerations can also influence ISA design.
3. Pipelining increases instruction throughput through parallelism.
4. It does not inherently reduce individual instruction latency.
5. Hazards are structural, data, and control.
6. Multiple issue and dynamic scheduling increase ILP further.
7. Dependencies limit achievable parallelism.
8. Greater complexity creates power/implementation costs.

---

# PART IV — HOW TO READ THE DATAPATH DIAGRAM WITHOUT GETTING LOST

# 113. Start with state-holding boxes

Whenever you see a processor diagram, first mark:

```text
PC
register file
pipeline registers
memories
```

These are places where information can persist.

Then mark the combinational blocks:

```text
ALU
adder
mux
sign extender
comparators
control logic
```

This immediately tells you what changes continuously and what only updates at clock boundaries.

---

# 114. Then identify the main forward data path

For the five-stage pipeline, visually divide the page:

```text
             IF              ID               EX             MEM           WB

PC → InstrMem → |IF/ID| → Registers → |ID/EX| → ALU → |EX/MEM| → DataMem → |MEM/WB| → RegFile
```

Ignore backward wires at first.

Once the basic left-to-right path is clear, add:

- forwarding wires
- branch PC feedback
- hazard control wires

This reduces visual overload.

---

# 115. Every backward wire should trigger a question

In the slides, right-to-left flow is especially important because it often indicates dependency/control behavior.

Examples:

```text
WB result → register file
EX/MEM result → ALU forwarding mux
MEM/WB result → ALU forwarding mux
branch decision/target → PC selection
hazard unit → PC/IF-ID write controls
```

Ask:

> Why must information generated later influence something earlier?

Usually the answer is:

- write-back
- bypassing
- branch redirection
- stall control

---

# 116. A “mux inventory” method

For each mux, create this four-column note:

| Mux | Input 0 | Input 1/other | Control |
|---|---|---|---|
| destination register | `rt` | `rd` | RegDst |
| ALU B | register data | immediate | ALUSrc |
| register write data | ALU result | memory data | MemtoReg |
| next PC branch mux | PC+4 | branch target | PCSrc |
| jump PC mux | branch/normal result | jump target | Jump |
| ForwardA | normal A | forwarded sources | ForwardA |
| ForwardB | normal B | forwarded sources | ForwardB |

If you can explain each mux in one sentence, you understand most of the datapath.

---

# 117. A “control signal inventory” method

For each control signal ask:

```text
1. Who generates it?
2. In which stage is it used?
3. What hardware does it affect?
4. Does it have to cross pipeline registers?
5. What happens if it is 0?
6. What happens if it is 1?
```

Example:

```text
RegWrite
generated: ID control decode
used: WB
controls: register file write enable
must be carried: yes, through later pipeline regs
0: no register write
1: write selected destination
```

This is much more useful than memorizing a blue wire on a picture.

---

# 118. A “producer-consumer” method for every hazard question

Write the two instructions:

```text
producer: which register does it generate?
consumer: which source needs that register?
```

Then write:

```text
producer ready stage = ?
consumer need stage  = ?
```

Typical lecture cases:

| Producer | Value becomes usable after | Consumer type | Value needed in |
|---|---|---|---|
| ALU instruction | EX | ALU instruction | EX |
| load | MEM | ALU instruction | EX |
| ALU instruction | EX | early branch | ID |
| load | MEM | early branch | ID |
| load | MEM | store data | MEM |

Now place the instructions one cycle apart and decide whether a bypass path can arrive in time.

This method is more reliable than memorizing isolated stall numbers.

---

# PART V — CT-ORIENTED QUESTIONS YOU SHOULD BE ABLE TO ANSWER

# 119. “Why is a pipeline register needed?”

A strong answer:

> A pipeline register separates adjacent stages and stores all data, control, and metadata produced by one stage that the same instruction will need in later stages. It allows each stage to work on a different instruction in the same cycle without the values being overwritten by newer instructions.

---

# 120. “Why must control signals be pipelined?”

> Control is decoded while an instruction is in ID, but many actions occur later—for example MemRead in MEM and RegWrite in WB. Therefore the control bits must travel with the instruction through the pipeline registers until the stage where they are used.

---

# 121. “Why can ALU hazards often be solved with forwarding?”

> An ALU instruction produces its result at the end of EX. A following ALU instruction needs that value in its own EX stage one cycle later. The value therefore already exists in a pipeline register and can be bypassed directly to the consumer ALU input.

---

# 122. “Why can’t immediate load-use be solved only by forwarding?”

> A load obtains its actual data only at the end of MEM. The immediately following ALU instruction needs the value during EX in that same cycle. The value is not yet available at the time it is needed, so no physical forwarding path can send it backward in time. One stall delays the consumer until forwarding is possible.

---

# 123. “How is a one-cycle load-use stall implemented?”

> Detect `ID/EX.MemRead` and a match between the load destination `ID/EX.rt` and a source field of the IF/ID instruction. Then disable PC update, disable IF/ID update, and force the controls entering ID/EX to zero. The dependent instruction remains in ID, the next fetch is repeated, and a bubble moves into EX.

---

# 124. “What is the difference between stall and flush?”

```text
STALL:
Instruction is valid, but must wait.
Hold it and delay progression.

FLUSH:
Instruction is invalid/wrong-path.
Destroy its effect, usually by zeroing controls.
```

A bubble may be created in either mechanism, but the reason differs.

---

# 125. “Why does double forwarding prefer EX/MEM over MEM/WB?”

> Both may refer to the same destination register, but EX/MEM contains the result of the more recent producer instruction. Program semantics require the most recently produced value, so EX/MEM has priority and MEM/WB forwarding is suppressed for that operand when an EX/MEM match exists.

---

# 126. “Why can a branch need more stalls than an ALU instruction?”

> In the improved branch datapath, comparison is moved to ID to reduce control-hazard penalty. That means branch operands are needed one stage earlier than normal ALU operands, so some producer values are not ready early enough and additional stalls are required.

---

# 127. “Why separate instruction and data memory?”

Two related answers:

**Single-cycle composition:** a load/store instruction requires the instruction and data memories for distinct purposes during one instruction execution.

**Pipeline structural-hazard answer:** one instruction can be in IF while another is in MEM during the same cycle. Separate memories prevent those simultaneous accesses from competing for one resource.

---

# 128. “Does pipelining make one instruction finish faster?”

No, not necessarily.

It increases throughput by overlapping stages of different instructions. The slides explicitly say individual instruction latency does not decrease.

---

# 129. “Why is load the critical path in the single-cycle design?”

Because it passes through:

```text
instruction memory
→ register read
→ ALU address calculation
→ data memory read
→ register write-back
```

which is longer than the paths needed by R-type, store, or branch in the slide's timing model.

---

# 130. “What is a structural/data/control hazard?”

```text
Structural:
two active stages need the same unavailable hardware resource.

Data:
a consumer instruction needs a value before the producer makes it available.

Control:
the processor does not yet know which instruction address should be fetched because control flow is unresolved.
```

---

# PART VI — PROJECT BRIDGE: HOW THESE IDEAS TRANSFER TO A SMALL 4-BIT PROCESSOR

> This section is conceptual extrapolation for your possible future project. The lecture itself uses a 32-bit MIPS example.

# 131. First important clarification: “4-bit processor” does not necessarily mean “4-bit instruction”

Usually “4-bit processor” refers to the primary **data width**:

```text
ALU operands: 4 bits
register contents: 4 bits
main data bus: 4 bits
```

But an instruction still needs enough bits to encode things such as:

```text
opcode
source register number
destination register number
immediate/address bits
```

So your project specification may choose an instruction width larger than four bits.

Do not assume the instruction format until the project specifies the ISA.

---

# 132. The first design question is the ISA, not the wires

Before building hardware, you need to know what instructions the CPU promises to execute.

For example, a tiny ISA might contain some subset of:

```text
ADD
SUB
AND
OR
LOAD
STORE
MOV/LOADI
BEQ/JZ
JUMP
HALT
```

Each instruction creates hardware requirements.

Example:

```text
If ISA has LOAD:
→ need some data memory access mechanism
→ need address formation
→ need memory-to-register write-back route

If ISA has branch:
→ need comparison/condition mechanism
→ need alternate PC source

If ISA has immediate instructions:
→ need immediate extraction/extension route
```

Therefore:

```text
ISA → required datapath → required control signals
```

---

# 133. What your compiler/assembler-like program would conceptually do

If the custom hardware accepts machine instructions such as:

```text
opcode | register fields | immediate
```

then software must translate programmer-readable code into those bit patterns.

Conceptually:

```text
ADD R1, R2
↓ parse
opcode(ADD) + encoded R1 + encoded R2
↓
binary instruction
↓
program/instruction ROM
```

For labels:

```asm
loop:
    ...
    JNZ loop
```

your translator may need to calculate the numeric target/displacement corresponding to `loop`.

This is why understanding PC behavior, instruction formats, immediate fields, branch targets, and opcodes matters even before you touch Logisim.

---

# 134. A minimal conceptual non-pipelined custom processor

Do not build this yet; use it only as a mental checklist.

```text
                  ┌──────────────┐
          ┌──────→│ Program ROM  │──── instruction bits ──────┐
          │       └──────────────┘                            │
          │                                                   v
       ┌────┐                                            ┌─────────┐
       │ PC │                                            │ Control │
       └────┘                                            └─────────┘
          │                                                   │
          │                                                   ├── mux selects
          │                                                   ├── reg write
          │                                                   ├── mem write
          │                                                   └── ALU op
          │
          │     ┌───────────────┐
          └────→│ next-PC logic │
                └───────────────┘

instruction register fields
          ↓
    ┌───────────┐
    │ Registers │── operand A ──┐
    └───────────┘── operand B ──┼→ mux → ALU → result ─┐
                                │                       │
                       immediate ┘                      ├→ writeback mux → registers
                                                        │
                                                    Data RAM
```

If you can explain why each box/wire exists, you are ready to design from a specification rather than copy a diagram.

---

# 135. If the custom CPU is pipelined

Then the same fundamental logic expands to:

```text
IF → [reg] → ID → [reg] → EX → [reg] → MEM → [reg] → WB
```

For every instruction field/control bit, ask:

```text
Where is it created?
Where is it finally consumed?
Through which pipeline registers must it travel?
```

Then separately design:

```text
forwarding detection
forwarding muxes
stall detection
PC/IF register freezing
bubble injection
branch flushing/prediction policy
```

That is exactly the progression of Part 2.

---

# PART VII — MASTER CHEAT SHEET

# 136. Single-cycle datapath controls

```text
RegDst   : destination register field selector
ALUSrc   : ALU second-input selector
MemtoReg : register write-data selector
RegWrite : register file write enable
MemRead  : data memory read enable
MemWrite : data memory write enable
Branch   : identifies branch behavior
Zero     : ALU/comparator equality status
PCSrc    : usually Branch & Zero
ALUOp    : broad ALU category from main control
ALUCtrl  : exact ALU function
Jump     : jump-target PC selector
```

---

# 137. Five pipeline stages

```text
IF  = instruction fetch
ID  = decode + register read
EX  = ALU / effective-address calculation
MEM = data-memory access
WB  = register write-back
```

---

# 138. Pipeline registers

```text
IF/ID
ID/EX
EX/MEM
MEM/WB
```

Purpose:

```text
hold an instruction's required information between stages
```

---

# 139. Pipeline control grouping

```text
EX:
RegDst, ALUOp, ALUSrc

MEM:
Branch, MemRead, MemWrite

WB:
RegWrite, MemtoReg
```

Generated in ID; carried until used.

---

# 140. Hazard types

```text
structural = resource conflict
data       = dependency timing
control    = unresolved next PC
```

---

# 141. Forwarding controls

```text
ForwardA/B = 00 → normal ID/EX operand
ForwardA/B = 10 → EX/MEM result
ForwardA/B = 01 → MEM/WB result
```

Validity checks:

```text
producer.RegWrite == 1
producer.destination != 0
producer.destination == consumer.source
```

Priority:

```text
EX/MEM match beats MEM/WB match
```

---

# 142. Classic load-use detection

For the simplified slide design:

```text
if ID/EX.MemRead
and (
    ID/EX.rt == IF/ID.rs
    or
    ID/EX.rt == IF/ID.rt
):
    stall
```

Stall action:

```text
PCWrite = 0
IF/IDWrite = 0
controls into ID/EX = 0
```

Result:

```text
front-end freezes + bubble enters EX
```

---

# 143. Branch handling vocabulary

```text
branch decision late → more wrong-path instructions may exist
move branch decision earlier → lower control penalty, but harder branch data hazards
predict not taken → keep fetching PC+4
mispredict → flush and redirect
1-bit predictor → remembers last direction
2-bit predictor → resists changing prediction after one anomaly
BTB → remembers branch target address
```

---

# 144. The one timing rule worth memorizing

```text
ALU result ready after EX
load data ready after MEM
normal ALU consumer needs operand in EX
early branch consumer needs operand in ID
store memory data is finally consumed in MEM
```

From that table, most forwarding/stall answers can be derived.

---

# PART VIII — RAPID SELF-TEST

Try answering these without looking back.

1. Why is `ALUSrc` needed?
2. Why is `MemtoReg` needed?
3. Why does `lw` use `rt` as a destination but an R-type instruction uses `rd`?
4. Why does `beq` subtract its register operands?
5. Why is branch displacement shifted left two bits?
6. Why does the single-cycle design need a long clock period?
7. What is the difference between combinational and sequential hardware?
8. Why are pipeline registers required?
9. Why must the load's destination register number travel all the way to WB?
10. Why must `RegWrite` travel through pipeline registers?
11. Why can `add → sub` usually be fixed by forwarding?
12. Why can `lw → sub` immediately afterward not be fixed by forwarding alone?
13. What exactly gets frozen during a load-use stall?
14. Why are controls zeroed during bubble insertion?
15. What is the difference between a stall and a flush?
16. Why does forwarding check `RegWrite`?
17. Why does forwarding reject destination register 0?
18. Why does EX/MEM forwarding have priority over MEM/WB forwarding?
19. Why can an early-resolved branch require extra data stalls?
20. Why is an immediate previous load before a branch a two-stall case in the shown design?
21. What is the difference between direction prediction and a branch target buffer?
22. Why does a one-bit predictor perform poorly at repeated loop exits/re-entries?
23. Why does multiple issue make dependency scheduling harder?
24. Why can out-of-order execution improve performance?
25. Why must out-of-order processors still preserve architectural program semantics?

If you can explain all 25 in your own words, you understand the core of both decks rather than merely recognizing the diagrams.

---

# PART IX — SLIDE COVERAGE MAP

This section exists so you can verify that the guide did not intentionally skip lecture topics.

## Part 1 slides

```text
1      Chapter title
2      Introduction / performance factors
3      Instruction execution
4      CPU overview
5      Multiplexers
6      Control
7      Logic design basics
8      Combinational elements
9-10   Sequential elements / register write control
11     Clocking methodology
12     Building a datapath
13     Instruction fetch
14     Instruction formats
15     R-format
16-17  Load/store
18-19  Branches
20     Composing elements
21     R-type/load/store datapath
22     Full datapath
23-24  ALU control
25     Main control unit
26     Datapath with control
27     All control signals
28     R-type walkthrough
29     Load walkthrough
30     Branch walkthrough
31     Jump construction
32     Datapath with jump
33     Performance / critical path / move to pipeline
```

## Part 2 slides

```text
1       Chapter title
2       Pipelining analogy
3-4     Five-stage MIPS pipeline
5-8     Pipeline performance and speedup
9       Pipelining and ISA design
10      Hazard categories
11      Structural hazard
12-16   Data hazard and forwarding intuition
17-21   Load-use hazard / forwarding limitation
22-24   Code scheduling
25-31   Control hazard, branch stall/prediction/delayed branch
32      Pipeline summary
33      Pipelined datapath overview
34      Pipeline registers and preliminary widths
35      Pipeline diagram styles
36-40   Load through IF/ID/EX/MEM/WB
41-43   Store through EX/MEM/WB
44-45   Wrong destination register and corrected load datapath
46-49   Multi-cycle and single-cycle pipeline diagrams
50-52   Pipelined control
53-55   ALU dependency example and forwarding
56-57   Forwarding detection requirements
58-60   Forwarding paths
61      Forwarding conditions
62-64   Double hazard and revised priority
65      Datapath with forwarding
66      Load-to-store forwarding timing
67-70   Load-use stall timing/detection
71-75   Stall mechanism and hazard-detection datapath
76      Stall performance
77-81   Branch hazards and branch-data stalls
82-85   Dynamic branch prediction, 1-bit/2-bit, BTB
86      ILP
87      Multiple issue
88-89   Speculation
90-91   Static multiple issue / scheduling
92-95   Dual-issue MIPS and scheduling example
96-97   Loop unrolling and IPC improvement
98-100  Dynamic multiple issue / dynamic scheduling / CPU structures
101     Why dynamic scheduling
102     Limits of multiple issue
103     Power efficiency
104-108 Real processor examples and performance
109     Fallacies
110     Pitfalls
111     Concluding remarks
```

---

# 145. Final mental picture

If you remember only one diagram, remember this conceptual stack:

```text
                INSTRUCTION SET
                      │
                      v
              instruction fields
                      │
         ┌────────────┴────────────┐
         v                         v
     CONTROL                    DATAPATH
  decode what to do       move/transform values
         │                         │
         └────────────┬────────────┘
                      v
               architectural state
             PC / registers / memory
```

Then pipelining changes the datapath into:

```text
IF → [IF/ID] → ID → [ID/EX] → EX → [EX/MEM] → MEM → [MEM/WB] → WB
```

and overlapping instructions creates three questions:

```text
Do two instructions need the same hardware?      → structural hazard
Does one need a value another hasn't supplied?   → data hazard
Do we know which instruction comes next?         → control hazard
```

The corresponding families of solutions are:

```text
resource duplication / scheduling
forwarding / stalls / code scheduling
branch resolution / prediction / flushing
```

That is the architecture of the chapter.
