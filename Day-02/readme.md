# Day 2 – RTL Design, Hierarchical Synthesis and Sequential Logic

## 1. Objective

The objective of Day 2 was to understand multi-module RTL design, hierarchical synthesis, flattening of design hierarchy, technology mapping using the SKY130 standard cell library, and the implementation and simulation of sequential circuits.

The following concepts were covered:

- Multi-module Verilog design.
- Module instantiation and port connections.
- Hierarchical synthesis using Yosys.
- Flat synthesis and netlist generation.
- Understanding synthesized netlists.
- Liberty file exploration.
- Sequential circuit design using D Flip-Flops.
- Synchronous reset implementation.
- Functional verification using GTKWave.

---

## 2. Multi-Module RTL Design

Multi-module design allows a large digital circuit to be divided into smaller, reusable modules.

Each module performs a specific operation, and these modules can be instantiated inside a top-level module.

### Verilog Code

**File: `multiple_modules.v`**

```verilog
module sub_module2 (input a, input b, output y);
    assign y = a | b;
endmodule

module sub_module1 (input a, input b, output y);
    assign y = a & b;
endmodule

module multiple_modules (input a, input b, input c, output y);

    wire net1;

    sub_module1 u1 (.a(a), .b(b), .y(net1));
    sub_module2 u2 (.a(net1), .b(c), .y(y));

endmodule
```

### Explanation

- `sub_module1` implements an AND operation.
- `sub_module2` implements an OR operation.
- `multiple_modules` is the top-level module.
- Instance `u1` connects the first submodule.
- Instance `u2` connects the second submodule.
- `net1` carries the output of the first module to the second module.

**Overall Boolean Expression:**

`Y = (A AND B) OR C`

### Hierarchical Design

The design hierarchy contains three modules:

1. `multiple_modules` – Top-level module.
2. `sub_module1` – AND operation.
3. `sub_module2` – OR operation.

### Screenshot – RTL Design

![Multiple Modules Verilog Code](images/multiple_modules_code.png)

### Screenshot – Hierarchical Design

![Hierarchical Design](images/multiple_modules_hier.png)

---

## 3. Yosys Synthesis

Yosys is an open-source synthesis tool used to convert RTL Verilog code into a gate-level netlist.

### Step 1: Start Yosys

```bash
yosys
```

### Step 2: Read the SKY130 Liberty File

```yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

The Liberty file contains information about the standard cells, including their functionality and timing characteristics.

### Step 3: Read the Verilog File

```yosys
read_verilog multiple_modules.v
```

### Step 4: Perform Hierarchical Synthesis

```yosys
synth -top multiple_modules
```

This command performs synthesis while maintaining the module hierarchy.

### Step 5: Check the Design

```yosys
check
```

The design was checked for structural problems.

**Observed Result:**

`Found and reported 0 problems.`

### Step 6: Display the Synthesized Design

```yosys
show
```

The synthesized design can be viewed graphically.

### Screenshot – Yosys Synthesis

![Yosys Hierarchical Synthesis](images/hierarchical_synthesis.png)

---

## 4. Understanding Hierarchical Synthesis

In hierarchical synthesis, the individual modules are synthesized while maintaining their structural relationships.

The top-level module contains instances of the submodules.

**Observed synthesis statistics:**

| Parameter | Observed Value |
|---|---:|
| Number of modules | 3 |
| Number of wires | 11 |
| Number of wire bits | 11 |
| Number of ports | 10 |
| Number of port bits | 10 |
| Number of cells | 2 |

The synthesized submodules contain:

- AND gate in `sub_module1`.
- OR gate in `sub_module2`.

This demonstrates how Yosys preserves the hierarchy of a multi-module design.

---

## 5. Flat Synthesis

Flat synthesis converts the hierarchical design into a single module by removing the submodule boundaries.

### Step 1: Read the Verilog File

```yosys
read_verilog multiple_modules.v
```

### Step 2: Perform Synthesis

```yosys
synth -top multiple_modules -flatten
```

The `-flatten` option removes the module hierarchy and combines the logic into the top-level module.

### Step 3: Check the Design

```yosys
check
```

### Step 4: Generate the Flattened Netlist

```yosys
write_verilog -noattr multiple_modules_flat.v
```

### Step 5: View the Generated Netlist

```bash
gvim multiple_modules_flat.v
```

### Explanation

In flat synthesis:

- Submodule instances are removed.
- Internal connections are represented within the top-level module.
- The complete logic is available in one module.
- The resulting netlist is easier to examine as a single flattened design.

### Screenshot – Flat Synthesis

![Flat Synthesis](images/multiple_modules_flat.png)

### Screenshot – Flattened Netlist

![Flattened Netlist](images/flattened_netlist.png)

---

## 6. Technology Mapping Using SKY130 Library

Technology mapping converts the synthesized logic into standard cells available in the target technology library.

The SKY130 standard cell library provides different logic gates and sequential cells.

The Liberty file used was:

`sky130_fd_sc_hd__tt_025C_1v80.lib`

The synthesis process maps the logic into available SKY130 standard cells.

Some of the cells observed in the generated netlist include:

- `sky130_fd_sc_hd__and2_0`
- `sky130_fd_sc_hd__clkinv_1`
- `sky130_fd_sc_hd__nand2_1`
- `sky130_fd_sc_hd__dfrtp_1`

The generated netlist contains instances of standard cells with their respective input and output connections.

### Screenshot – Technology-Mapped Netlist

![SKY130 Technology Mapping](images/technology_mapped_netlist.png)

---

## 7. Arithmetic Operations Using Verilog

Arithmetic operations were also explored using simple Verilog modules.

### 7.1 Multiplication by 2

**File: `mult_2.v`**

```verilog
module mul2 (input [2:0] a, output [3:0] y);

    assign y = a * 2;

endmodule
```

This module multiplies a 3-bit input by 2 and produces a 4-bit output.

### 7.2 Multiplication by 9

**File: `mult_8.v`**

```verilog
module mult8 (input [2:0] a, output [5:0] y);

    assign y = a * 9;

endmodule
```

This module multiplies a 3-bit input by 9 and produces a 6-bit output.

### Synthesis

The modules were read and synthesized using Yosys.

```yosys
read_verilog mult_2.v
synth -top mul2
stat
check
```

For the second module:

```yosys
read_verilog mult_8.v
synth -top mult8
stat
check
```

The synthesis statistics were examined to understand how Yosys processes arithmetic expressions and optimizes the resulting logic.

### Screenshot – Multiplication by 2

![Multiplication by 2](images/mul2_synthesis.png)

### Screenshot – Multiplication by 9

![Multiplication by 9](images/mult8_synthesis.png)

---

## 8. D Flip-Flop with Synchronous Reset

A D Flip-Flop is a sequential circuit that stores one bit of data.

The output changes according to the input at the active clock edge.

A synchronous reset resets the output only when the reset signal is active at the triggering clock edge.

### Verilog Code

**File: `dff_syncres.v`**

```verilog
module dff_syncres (
    input clk,
    input async_reset,
    input sync_reset,
    input d,
    output reg q
);

always @(posedge clk)
begin
    if (sync_reset)
        q <= 1'b0;
    else
        q <= d;
end

endmodule
```

### Explanation

- `clk` – Clock input.
- `sync_reset` – Synchronous reset signal.
- `d` – Data input.
- `q` – Registered output.
- `posedge clk` – The circuit responds to the rising edge of the clock.
- When `sync_reset` is HIGH, the output becomes 0.
- Otherwise, the output follows the data input at the rising clock edge.

**Note:** Although `async_reset` is declared in the module, it is not used in the above implementation. The circuit implements synchronous reset.

---

## 9. Testbench for D Flip-Flop

A testbench was created to verify the functionality of the D Flip-Flop.

**File: `tb_dff_syncres.v`**

```verilog
`timescale 1ns / 1ps

module tb_dff_syncres;

    // Inputs
    reg clk;
    reg sync_reset;
    reg d;

    // Output
    wire q;

    // Instantiate the Unit Under Test
    dff_syncres uut (
        .clk(clk),
        .sync_reset(sync_reset),
        .d(d),
        .q(q)
    );

    initial begin

        $dumpfile("tb_dff_syncres.vcd");
        $dumpvars(0, tb_dff_syncres);

        // Initialize inputs
        clk = 0;
        sync_reset = 0;
        d = 0;

        #3000 $finish;

    end

    always #10 clk = ~clk;

    always #23 d = ~d;

    always #113 sync_reset = ~sync_reset;

endmodule
```

### Testbench Explanation

- The clock is generated using `always #10 clk = ~clk`.
- The data input changes every 23 ns.
- The synchronous reset changes every 113 ns.
- `$dumpfile` specifies the VCD output file.
- `$dumpvars` records the simulation signals.
- `$finish` terminates the simulation.

---

## 10. Functional Simulation Using Icarus Verilog

The Verilog design and testbench were simulated using Icarus Verilog.

### Step 1: Compile the Verilog Files

```bash
iverilog -o dff_sim dff_syncres.v tb_dff_syncres.v
```

### Step 2: Run the Simulation

```bash
vvp dff_sim
```

### Step 3: Open GTKWave

```bash
gtkwave tb_dff_syncres.vcd
```

### Step 4: Add Signals

The following signals were added to the GTKWave waveform viewer:

- `clk`
- `d`
- `q`
- `sync_reset`

### Simulation Result

The waveform demonstrates the operation of a D Flip-Flop with synchronous reset.

- The clock signal continuously toggles.
- The data input changes periodically.
- The output follows the data input at the rising edge of the clock when reset is LOW.
- When synchronous reset is HIGH at the rising clock edge, the output becomes LOW.

### Screenshot – GTKWave Simulation

![D Flip-Flop GTKWave Simulation](images/dff_syncres_waveform.png)

---

## 11. Key Observations

1. A multi-module Verilog design can be implemented using module instantiation.
2. Hierarchical synthesis preserves the structural organization of the design.
3. Flat synthesis removes the hierarchy and combines the logic into a single module.
4. Yosys performs RTL synthesis and generates gate-level representations.
5. Liberty files provide standard cell information for technology mapping.
6. SKY130 standard cells are used to represent the synthesized logic.
7. Arithmetic expressions can be optimized during synthesis.
8. A D Flip-Flop stores data at the active clock edge.
9. A synchronous reset takes effect only at the active clock edge.
10. GTKWave helps verify the functional behavior of sequential circuits.

---

## 12. Conclusion

Day 2 provided practical exposure to multi-module RTL design, hierarchical and flat synthesis, and technology mapping using the SKY130 standard cell library.

The session also covered arithmetic operations in Verilog and the design of a D Flip-Flop with synchronous reset.

Yosys was used to synthesize and analyze the RTL designs, while Icarus Verilog and GTKWave were used to simulate and verify sequential circuit behavior.

Overall, the exercises helped strengthen the understanding of RTL design, synthesis, standard cell mapping, and functional verification.

---

## 13. Tools and Technologies Used

| Tool / Technology | Purpose |
|---|---|
| Verilog | RTL design |
| Neovim | Verilog code editing |
| Yosys | RTL synthesis |
| SKY130 | Standard cell technology |
| Liberty File | Standard cell definitions |
| Icarus Verilog | Functional simulation |
| GTKWave | Waveform visualization |
| Linux | Development environment |

---

## 14. Repository Structure

```text
Day_02/
│
├── verilog_files/
│   ├── multiple_modules.v
│   ├── mult_2.v
│   ├── mult_8.v
│   ├── dff_syncres.v
│   └── tb_dff_syncres.v
│
├── images/
│   ├── multiple_modules_code.png
│   ├── multiple_modules_hier.png
│   ├── hierarchical_synthesis.png
│   ├── multiple_modules_flat.png
│   ├── flattened_netlist.png
│   ├── technology_mapped_netlist.png
│   ├── mul2_synthesis.png
│   ├── mult8_synthesis.png
│   └── dff_syncres_waveform.png
│
└── README.md
```

