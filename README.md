# Day 1 – RTL Design, Simulation and Synthesis of a 2:1 Multiplexer

---

## Overview

This report documents the theoretical concepts and practical implementation of the RTL design and synthesis flow using a **2:1 Multiplexer (MUX)** as an example.

The work includes:

- Introduction to RTL design
- RTL-to-gate-level synthesis
- Standard cell libraries and Liberty (`.lib`) files
- Faster and slower standard cells
- Setup and hold timing concepts
- Verilog RTL design of a 2:1 MUX
- Testbench creation and functional simulation
- Waveform verification using GTKWave
- RTL synthesis using Yosys
- Technology mapping using the SKY130 standard cell library
- Generation and visualization of the synthesized netlist

---

# 1. Introduction to RTL Design

RTL stands for **Register Transfer Level**. It is a level of abstraction used to describe the functionality and behavior of a digital circuit using a Hardware Description Language (HDL) such as Verilog.

At the RTL level, the designer describes:

- Inputs and outputs of the circuit
- Logical operations performed on the data
- Data flow between different parts of the design
- Sequential behavior, when required

RTL code describes **what the circuit should do**, rather than describing the physical implementation using transistors.

---

# 2. Introduction to Synthesis

Synthesis is the process of converting RTL Verilog code into a gate-level representation of a digital circuit.

During synthesis:

1. RTL code is read and analyzed.
2. Logical operations are identified.
3. RTL constructs are converted into combinational and sequential logic.
4. The logic is optimized.
5. The design is mapped to available standard cells.
6. A gate-level netlist is generated.

The output of synthesis is called a **netlist**, which describes the logic cells used in the design and their interconnections.

The general flow is:

    Verilog RTL Design
            ↓
    Functional Simulation
            ↓
    RTL Synthesis
            ↓
    Technology Mapping
            ↓
    Gate-Level Netlist

---

# 3. Introduction to Standard Cell Libraries

A standard cell library contains pre-designed digital logic cells that can be used to implement a circuit.

Examples of standard cells include:

- AND gates
- OR gates
- NAND gates
- NOR gates
- Inverters
- Multiplexers
- Flip-flops

A standard cell library provides information about:

- Logical functionality
- Timing characteristics
- Power consumption
- Area
- Input capacitance

The synthesis tool uses this information to map the RTL design to cells available in the target technology.

---

# 4. Liberty (.lib) Files

A **Liberty file**, with the extension `.lib`, contains timing and functional information about the standard cells available in a technology library.

The SKY130 standard cell library used in this project was:

    sky130_fd_sc_hd__tt_025C_1v80.lib

The Liberty file is loaded into Yosys using:

    read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

This provides the information required for technology mapping.

---

# 5. Faster and Slower Standard Cells

The same logic function can be implemented using different versions of a standard cell.

### Faster Cells

- Have lower propagation delay
- Can drive loads faster
- May use more area and power

### Slower Cells

- Have higher propagation delay
- May use less area and power
- Can be useful for satisfying hold-time requirements

Therefore, faster cells involve a trade-off between **speed, area, and power**.

---

# 6. Setup and Hold Timing

In a synchronous digital circuit, data travels from one flip-flop to another through combinational logic.

    DFF A → Combinational Logic → DFF B

The data must arrive at the destination flip-flop at the correct time.

### Setup Time

Setup time ensures that data arrives sufficiently before the active clock edge.

A simplified setup timing relationship is:

    TCLK >= TCQ + TCOMBI + TSETUP

Where:

- `TCLK` – Clock period
- `TCQ` – Clock-to-Q delay
- `TCOMBI` – Combinational logic delay
- `TSETUP` – Setup time

### Hold Time

Hold time ensures that the input data remains stable for a required duration after the clock edge.

If data arrives too early, a hold-time violation can occur. Slower cells or additional delay elements can be used to increase the delay in the data path.

Therefore:

- Faster cells can help satisfy setup timing.
- Slower cells can help satisfy hold timing.

---

# 7. Project Setup

The SKY130 RTL Design and Synthesis Workshop repository was explored to understand the available Verilog files and standard cell libraries.

### Repository Setup

![Repository Setup](screenshots/setup1.png)

The project contains important directories such as:

- `verilog_files` – Contains Verilog design and testbench files
- `lib` – Contains the SKY130 Liberty library
- `my_lib` – Contains Verilog models and primitives

### Library Contents

![Library Contents](screenshots/List_of_contents_lib.png)

### Verilog Files

![Verilog Files](screenshots/List_of_contents_verilog_files.png)

---

# 8. Introduction to the 2:1 Multiplexer

A **2:1 Multiplexer (MUX)** is a combinational circuit that selects one of two input signals and connects it to a single output.

The MUX has:

- Two data inputs: `i0` and `i1`
- One select input: `sel`
- One output: `y`

## Functional Operation

| Select (`sel`) | Output (`y`) |
|---|---|
| 0 | `i0` |
| 1 | `i1` |

The operation can be represented as:

    y = sel ? i1 : i0

Therefore:

- When `sel = 0`, the output follows `i0`.
- When `sel = 1`, the output follows `i1`.

---

# 9. RTL Design and Testbench

The 2:1 Multiplexer was implemented using Verilog.

The design uses:

- `i0` as the first input
- `i1` as the second input
- `sel` as the select signal
- `y` as the output

A testbench was used to apply different input combinations and verify the functionality of the MUX.

### RTL and Testbench Code

![MUX RTL and Testbench Code](screenshots/gvim_code_tb_and_v.png)


---

# 10. Functional Simulation

The RTL design and testbench were compiled and simulated using **Icarus Verilog**.

During simulation, a **VCD (Value Change Dump)** file was generated. The VCD file records changes in signal values over time.

The generated waveform was viewed using **GTKWave**.

---

# 11. Waveform Verification Using GTKWave

The simulation waveform was used to verify the functionality of the 2:1 Multiplexer.

### Simulation Waveform

![MUX Simulation Waveform](screenshots/GKTWave.png)

The waveform verifies that:

- When `sel = 0`, the output `y` follows input `i0`.
- When `sel = 1`, the output `y` follows input `i1`.

Thus, the RTL functionality was verified before proceeding with synthesis.

---

# 12. Introduction to Yosys

**Yosys** is an open-source tool used for Verilog synthesis.

Yosys performs operations such as:

- Reading Verilog RTL code
- Analyzing and optimizing the design
- Converting RTL into a logic representation
- Technology mapping
- Generating a gate-level netlist

The general Yosys synthesis flow is:

    Read Standard Cell Library
            ↓
    Read Verilog RTL
            ↓
    Synthesize Design
            ↓
    Technology Mapping
            ↓
    Generate Netlist

---

# 13. RTL Synthesis Using Yosys

The SKY130 Liberty library was first loaded into Yosys:

    read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

The Verilog design was then read:

    read_verilog good_mux.v

The design was synthesized by specifying the top module:

    synth -top good_mux

### Yosys Synthesis

![Yosys Synthesis](screenshots/yoysys.png)

During synthesis, Yosys analyzes the RTL design and converts it into an optimized logic representation.

---

# 14. RTL Synthesis Results

After synthesis, Yosys reports the statistics and cells used in the design.

### RTL Synthesis Statistics

![RTL Synthesis Result](screenshots/rtl_result.png)

### RTL Representation

![RTL Representation](screenshots/rtl.png)

The RTL representation shows the logical structure of the 2:1 Multiplexer.

---

# 15. Technology Mapping Using the SKY130 Library

Technology mapping converts the generic synthesized logic into actual standard cells available in the target technology library.

For this project, the target technology was the **SKY130 standard cell library**.

Yosys uses the **ABC tool** for technology mapping.

The mapping command used was:

    abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

During technology mapping, the synthesized logic is mapped to suitable cells available in the SKY130 library.

---

# 16. ABC Technology Mapping Results

The technology mapping results show that the design was successfully mapped using the SKY130 standard cell library.

### ABC Results

![ABC Technology Mapping Results](screenshots/ABC_Results.png)

The result confirms that the original RTL design was successfully mapped to a technology-specific implementation.

---

# 17. Graphical Representation of the Synthesized Design

Yosys can generate a graphical representation of the synthesized design.

The graphical representation shows:

- Input `i0`
- Input `i1`
- Select signal `sel`
- MUX cell
- Output `y`

### Synthesized MUX Diagram

![Graphical Representation](screenshots/Graphical generation.png)

This diagram provides a visual representation of the connections in the synthesized design.

---

# 18. Generated Gate-Level Netlist

After synthesis and technology mapping, a synthesized Verilog netlist was generated.

The netlist contains technology-specific standard cells and their interconnections.

### Generated Netlist

![Synthesized Netlist](screenshots/RTL_netlist.png)

The gate-level netlist represents the final synthesized implementation of the 2:1 Multiplexer.

---

# 19. Complete RTL Design and Synthesis Flow

The complete flow followed in this project is:

    1. Write Verilog RTL code
               ↓
    2. Create a testbench
               ↓
    3. Compile and simulate using Icarus Verilog
               ↓
    4. Generate a VCD waveform file
               ↓
    5. Verify the waveform using GTKWave
               ↓
    6. Load the SKY130 Liberty library into Yosys
               ↓
    7. Read the Verilog RTL design
               ↓
    8. Perform RTL synthesis
               ↓
    9. Perform technology mapping using ABC
               ↓
    10. Generate the synthesized gate-level netlist
               ↓
    11. Visualize the synthesized design

---

# 20. Tools Used

| Tool | Purpose |
|---|---|
| GVim | Writing and editing Verilog code |
| Icarus Verilog | Compiling and simulating the design |
| GTKWave | Viewing simulation waveforms |
| Yosys | RTL synthesis and netlist generation |
| ABC | Technology mapping |
| SKY130 Standard Cell Library | Technology-specific cell mapping |

---

# 21. Key Learnings

Through this work, I learned:

1. The basics of RTL design using Verilog.
2. The process of converting RTL into a gate-level netlist.
3. The purpose of standard cell libraries.
4. The role of Liberty (`.lib`) files in synthesis.
5. The difference between faster and slower standard cells.
6. Basic setup and hold timing concepts.
8. How to simulate a Verilog design using Icarus Verilog.
9. How to verify simulation results using GTKWave.
10. How to perform RTL synthesis using Yosys.
11. The basics of technology mapping using the SKY130 library.
12. How a synthesized gate-level netlist is generated.

---

# Conclusion

This work demonstrated the complete process of converting a Verilog RTL design into a technology-specific gate-level implementation.

The 2:1 Multiplexer was pre designed using Verilog and later verified through functional simulation. The verified RTL design was then synthesized using Yosys and mapped using the SKY130 standard cell library.

This exercise provided practical exposure to the RTL design flow, functional verification, synthesis, technology mapping, standard cell libraries, and gate-level netlist generation.

---
