# RISC-V 5-Stage Pipelined Processor (Verilog)

A Verilog implementation of a simplified **32-bit RISC-V-style pipelined processor** with a **5-stage pipeline** and a testbench for functional simulation.

---

## Overview

This project implements a basic pipelined RISC processor that supports a custom instruction set.  
The design uses a **two-phase clock (`clk1` and `clk2`)** to separate pipeline stages and avoid data hazards between adjacent stages.

---

## MIPS32 Architecture Features

- **32 × 32-bit General Purpose Registers (GPRs)** — `R0` to `R31`
- **R0 hardwired to logic 0**
- **32-bit Program Counter (PC)**
- **No flag registers** (carry, zero, sign, etc.)
- **Few addressing modes**
- **Only Load and Store instructions can access memory**
- **Memory word size is 32 bits (word addressable)**

---

## Pipeline Stages

| Stage | Clock | Description |
|------|------|-------------|
| **IF – Instruction Fetch** | clk1 | Fetches instruction from memory and handles branch redirection |
| **ID – Instruction Decode** | clk2 | Decodes instruction, reads registers, sign-extends immediate |
| **EX – Execute** | clk1 | Performs ALU operations, address calculation, branch evaluation |
| **MEM – Memory Access** | clk2 | Loads/stores data from/to memory |
| **WB – Write Back** | clk1 | Writes results back to the register file |

---

## Supported Instructions

### R-Type (Register–Register ALU)

| Opcode | Instruction | Operation |
|------|-------------|-----------|
| 000000 | ADD Rd, Rs1, Rs2 | Rd = Rs1 + Rs2 |
| 000001 | SUB Rd, Rs1, Rs2 | Rd = Rs1 - Rs2 |
| 000010 | AND Rd, Rs1, Rs2 | Rd = Rs1 & Rs2 |
| 000011 | OR Rd, Rs1, Rs2 | Rd = Rs1 \| Rs2 |
| 000100 | SLT Rd, Rs1, Rs2 | Rd = (Rs1 < Rs2) ? 1 : 0 |
| 000101 | MUL Rd, Rs1, Rs2 | Rd = Rs1 × Rs2 |
| 000110 | HLT | Halt execution |

---

### I-Type (Immediate / Load / Store / Branch)

| Opcode | Instruction | Operation |
|------|-------------|-----------|
| 001000 | LW Rd, offset(Rs) | Rd = Mem[Rs + offset] |
| 001001 | SW Rs2, offset(Rs1) | Mem[Rs1 + offset] = Rs2 |
| 001010 | ADDI Rd, Rs, imm | Rd = Rs + imm |
| 001011 | SUBI Rd, Rs, imm | Rd = Rs - imm |
| 001100 | SLTI Rd, Rs, imm | Rd = (Rs < imm) ? 1 : 0 |
| 001101 | BNEQZ Rs, offset | if (Rs ≠ 0) PC = PC + offset |
| 001110 | BEQZ Rs, offset | if (Rs = 0) PC = PC + offset |

---

## Instruction Encoding
[31:26] Opcode (6 bits)
[25:21] Source Register 1 / Rs (5 bits)
[20:16] Source Register 2 / Rd (5 bits)
[15:11] Destination Register (RR-type)
[15:0] Immediate / Offset (16 bits, sign-extended)

---

## Branch Handling

Branch resolution occurs at the **EX stage**.  

When a branch is taken:
- The pipeline flushes incorrectly fetched instructions using the `TAKEN_BRANCH` flag.
- The **Program Counter (PC)** is redirected to the computed branch target address.

---

## Simulation

### Prerequisites

- **Icarus Verilog** or any compatible Verilog simulator (ModelSim, Vivado, etc.)
- **GTKWave** (optional for waveform viewing)

---

### Running with Icarus Verilog

```bash
# Compile
iverilog -o riscv_sim riscv.v

# Run simulation
vvp riscv_sim

# View waveforms
gtkwave mips.vcd

