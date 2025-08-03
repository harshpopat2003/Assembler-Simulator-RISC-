# 16-bit ISA Toolkit — Assembler & Simulator (Python)

A Python toolchain that **assembles and simulates** a custom 16-bit ISA with six encodings (A–F), **R0–R6 + FLAGS**, and **8-bit, double-byte–addressable memory** (256 words = 512 bytes). Supports a minimal **8-bit floating-point** format for `movf`, `addf`, and `subf`. Reads from **stdin** and writes to **stdout**.

---

## Features

* **Assembler**

  * Parses **labels** (`label:`) and **top-of-file variables** (`var name`).
  * Encodes all instruction types **A–F** (including `mov` reg/imm vs reg/reg).
  * Validates: opcode/register names, **immediates 0–255**, label/variable use, **FLAGS restrictions**, and **`hlt` must be last**.
  * On errors: prints **line-numbered diagnostics** and exits; otherwise prints **one 16-bit word per line** (max 256 lines).
  * Side effects: writes `input.txt` (sanitized source) and `output.txt` (machine code/errors).

* **Simulator**

  * Components: **Memory**, **ProgramCounter (PC)**, **RegisterFile (RF)**, **ExecutionEngine (EE)**.
  * Implements: `add sub mul div rs ls xor or and not cmp jmp jlt jgt je ld st hlt` + **float**: `movf addf subf`.
  * Per instruction: prints trace
    `PC(8b) R0 R1 R2 R3 R4 R5 R6 FLAGS (each 16b)`.
  * On halt: prints **256-line memory dump** (16-bit words).
  * Bonus: saves **cycle vs. memory-address** scatter plot as `Memory.png`.

* **Floating-Point (8-bit)**

  * No sign bit; **3-bit exponent, 5-bit mantissa** (only **LSB 8 bits** of registers used for FP ops).
  * Assembler accepts `movf Rn $<float>`; simulator provides basic encode/decode helpers.

---

## Requirements

* Python 3.8+
* `matplotlib` (for the bonus plot)

```bash
pip install matplotlib
```

---

## Project Layout (suggested)

```
assembler.py        # assembler (uses stdin/stdout; writes input.txt/output.txt)
simulator.py        # main loop: MEM, RF, EE, PC
Components.py       # ProgramCounter, RegisterFile, ExecutionEngine, Memory (if split)
```

> If `Components` content is inlined in `simulator.py`, remove the `from Components import *` line.

---

## Usage

### Assemble

```bash
python3 assembler.py < program.asm > program.bin
# machine code also written to output.txt; parsed source to input.txt
```

### Simulate

```bash
python3 simulator.py < program.bin > trace.txt
# produces Memory.png (cycle vs. address)
```

---

## Input / Output Contracts

* **Assembly input (stdin → assembler):**

  * Variables must appear **before** any instruction.
  * `mem_addr` in `ld/st` must be a variable; in jumps it must be a **label**.
  * `mov Rn $Imm` where `Imm ∈ [0,255]`.
  * `movf Rn $<float>` uses 8-bit float representation (3e/5m).

* **Binary input (stdin → simulator):**

  * Each line is a **16-bit** binary string (0/1).

* **Assembler output (stdout):**

  * One **16-bit** instruction per line **or** a line-numbered error message.

* **Simulator trace (stdout, per cycle):**

  * `PC(8b) R0(16b) … R6(16b) FLAGS(16b)`
  * After `hlt`: **256 lines** of memory words (16b) as a dump.

---

## Example

**program.asm**

```asm
var X
mov R1 $10
mov R2 $100
mul R3 R1 R2
st R3 X
hlt
```

**Assemble & simulate**

```bash
python3 assembler.py < program.asm > program.bin
python3 simulator.py < program.bin > trace.txt
```

---

## Error Messages (examples)

* `Error: Invalid Instruction In Line Number N: …`
* `Error: Invalid Register/Variable Used In Line Number N: …`
* `Error: Invalid Use of Flag Register In Line Number N: …`
* `Error: Immutable Variable Out Of Range …`
* `Error: Variable Declared After Instruction …`
* `Error: hlt Instruction Missing` / `Error: hlt Present Inbetween Instructions`

> Assembler **collects and prints all detected errors** with line numbers.
