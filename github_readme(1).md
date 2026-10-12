# FPGA Hardware Accelerator for Number Theoretic Transform (NTT)

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Hardware: FPGA](https://img.shields.io/badge/Hardware-FPGA-orange.svg)
![Language: Verilog](https://img.shields.io/badge/Language-Verilog-green.svg)

## Overview
This repository contains a high-performance FPGA hardware implementation of the **Number Theoretic Transform (NTT)**, designed to accelerate polynomial multiplication for Lattice-Based Post-Quantum Cryptography (PQC). 

Polynomial multiplication in the cyclotomic ring $\mathcal{R}_q = \mathbb{Z}_q[x] / (x^n + 1)$ is the primary computational bottleneck in NIST-standardized schemes like **ML-KEM (Kyber, FIPS 203)** and **ML-DSA (Dilithium, FIPS 204)**. This IP core reduces the $\mathcal{O}(n^2)$ complexity of schoolbook multiplication to $\mathcal{O}(n \log n)$ using a pipelined, memory-efficient hardware architecture.

## Key Features
* **Reconfigurable Butterfly Unit (BU):** Supports both Cooley-Tukey (Forward NTT) and Gentleman-Sande (Inverse NTT) algorithms using a unified DSP datapath.
* **Montgomery Reduction Datapath:** Bypasses expensive hardware division by modulo $q$. Utilizes $R=2^{16}$ in the Montgomery domain, reducing modular reduction to purely DSP multiplications and costless wire routing.
* **Conflict-Free Memory Banking:** Implements a parity-based address XOR banking scheme to supply True Dual-Port BRAMs, ensuring zero pipeline stalls during layer transitions.
* **Bit-Accurate Golden Model:** Includes a Python-based reference model for generating twiddle factors and verifying RTL output against NIST FIPS test vectors.

## Repository Structure
```text
├── rtl/                  # Verilog HDL source files
│   ├── ntt_top.v         # Top-level NTT module and FSM
│   ├── butterfly.v       # Unified CT/GS Butterfly Unit
│   ├── montgomery.v      # Montgomery reduction pipeline
│   └── dual_port_ram.v   # Memory wrappers
├── tb/                   # Testbenches
│   └── ntt_tb.v          # Self-checking Verilog testbench
├── model/                # Python Golden Model
│   ├── ntt_model.py      # Bit-accurate NTT/INTT Python implementation
│   └── twiddle_gen.py    # Script to generate ROM initialization files (.mem)
├── docs/                 # Documentation and project reports
└── README.md
```

## Prerequisites
* **Simulation:** ModelSim, Questa, or Verilator.
* **Synthesis:** Xilinx Vivado or Intel Quartus Prime (depending on target board).
* **Software:** Python 3.x (for generating ROM files and running the golden model).

## Getting Started

### 1. Generate ROM and Test Vectors
Before running the Verilog simulation, generate the twiddle factor ROM files and randomized test vectors using the Python model.
```bash
cd model
python twiddle_gen.py --scheme kyber --output ../tb/twiddles.mem
python ntt_model.py --generate_vectors > ../tb/test_vectors.txt
```

### 2. Run RTL Simulation
Launch your preferred Verilog simulator to verify the hardware against the Python test vectors.
```bash
cd tb
# Example using a generic Makefile or simulator command
vlib work
vlog ../rtl/*.v ntt_tb.v
vsim -c ntt_tb -do "run -all; quit"
```
The testbench is self-checking and will assert `TEST PASSED` if the Verilog memory output perfectly matches the Python Golden Model output.

### 3. Synthesis and Implementation
1. Create a new project in your FPGA vendor tool (e.g., Vivado).
2. Add all `.v` files from the `rtl/` directory.
3. Add the generated `.mem` files to the project for ROM initialization.
4. Constrain your target clock frequency (e.g., 200 MHz).
5. Run Synthesis and Implementation. Check the utilization report to verify that the Montgomery multipliers correctly inferred dedicated DSP blocks (e.g., DSP48E1/E2).

## Hardware Integration
The top-level module (`ntt_top.v`) acts as an independent cryptographic coprocessor. In a full System-on-Chip (SoC) design, this IP can be wrapped with an AXI-4 interface, allowing a soft-core processor (like MicroBlaze or Nios II) to write coefficients to the BRAM, assert a `start` signal, and read back the transformed coefficients once the `done` interrupt is raised.

## References
* NIST FIPS 203: Module-Lattice-Based Key-Encapsulation Mechanism Standard
* NIST FIPS 204: Module-Lattice-Based Digital Signature Standard
* *Note: See the `docs/` folder for the full mathematical breakdown and hardware architecture report.*

## License
This project is licensed under the MIT License - see the LICENSE file for details.