# VSD-RTL


## 1. Introduction to RTL Design

RTL (Register Transfer Level) design is a behavioral representation of the required digital circuit specification.

It describes:
- Data flow between registers.
- Operations performed on data.
- Sequential and combinational logic.
- Clock and reset behavior.

RTL designs are generally written using Hardware Description Languages (HDLs) such as Verilog and SystemVerilog.

![RTL Design](images/rtl_design.png)

---

## 2. Introduction to Synthesis

Synthesis is the process of converting RTL Verilog code into a gate-level representation of a digital circuit.

During synthesis:

1. RTL code is read and analyzed.
2. Logic operations are identified.
3. RTL constructs are converted into logic gates and sequential cells.
4. Connections between the selected cells are established.
5. A gate-level netlist is generated.

The output of synthesis is a **netlist**, which describes the logic cells used in the design and their interconnections.

### RTL to Gate-Level Translation

The RTL design is converted into corresponding hardware elements such as:

- AND, OR and NOT gates.
- Multiplexers.
- Flip-flops.
- Other combinational and sequential cells.

![RTL to Gate-Level Translation](images/rtl_to_gate_level.png)

---

## 3. Introduction to Synthesizer

A synthesizer is a tool used to convert RTL code into a gate-level netlist.

**Yosys** is the open-source synthesis tool used in this training.

The synthesizer takes the following inputs:

- **RTL Design:** Verilog source code describing the required circuit.
- **Liberty File (.lib):** Standard-cell library containing cell information.

After processing these inputs, the synthesizer generates a netlist.

![Synthesizer Flow](images/synthesizer_flow.png)

---

## 4. Introduction to Liberty (.lib) Files

A Liberty file (`.lib`) contains information about the standard cells available in a particular technology library.

It provides information such as:

- Logic gates (AND, OR, NOT, NAND, NOR, etc.).
- Flip-flops and other sequential cells.
- Cell propagation delays.
- Setup and hold timing requirements.
- Power consumption.
- Area information.

### Different Flavours of Standard Cells

The same logic function can have multiple implementations with different performance characteristics.

For example, a 2-input AND gate may have:

- Slow version.
- Medium version.
- Fast version.

The synthesis tool selects suitable cells based on design requirements and timing constraints.

![Liberty File](images/liberty_file.png)

---

## 5. Faster Cells vs Slower Cells

The load in a digital logic circuit is mainly associated with capacitance.

To charge and discharge capacitance quickly, transistors must supply sufficient current.

### Faster Cells

- Have lower propagation delay.
- Charge and discharge capacitive loads faster.
- Generally use wider transistors.
- Require more area and potentially more power.

### Slower Cells

- Have higher propagation delay.
- Generally use narrower transistors.
- Require less area and potentially less power.
- Can help satisfy hold timing requirements.

Therefore, faster cells do not come free. They involve a trade-off between delay, area and power.

![Faster Cells vs Slower Cells](images/faster_vs_slower_cells.png)

---

## 6. Why Do We Need Slow Cells?

In a sequential circuit, data travels from one flip-flop to another through combinational logic.

**DFF A → Combinational Logic → DFF B**

Both flip-flops operate using the same clock.

The data must arrive at DFF B neither too late nor too early.

### Hold Time Constraint

Hold time ensures that the input data of the destination flip-flop remains stable for the required duration after the clock edge.

If data arrives too early, a hold-time violation can occur.

To prevent this, slower cells or additional delay elements can be used in the data path.

Thus, a digital circuit needs:

- Faster cells to satisfy performance and setup timing requirements.
- Slower cells to satisfy hold timing requirements.

![Hold Time Concept](images/hold_time_concept.png)

---

## 7. Why Are Different Flavours of Gates Required?

The combinational delay in a logic path determines the maximum operating frequency of a synchronous digital circuit.

For a register-to-register path, the simplified setup timing relationship is:

`TCLK >= TCQ + TCOMBI + TSETUP`

Where:

- `TCLK` – Clock period.
- `TCQ` – Clock-to-Q delay of the launching flip-flop.
- `TCOMBI` – Combinational logic delay.
- `TSETUP` – Setup time of the destination flip-flop.

To achieve a higher operating frequency, the clock period must be reduced.

Therefore, faster cells are required to minimize combinational delay.

However, using only faster cells may create hold-time violations. Hence, a combination of faster and slower cells is necessary.

![Setup Timing Concept](images/setup_timing_concept.png)

---

## 8. Selection of Standard Cells

The synthesis tool needs guidance to select appropriate standard cells for implementing a digital logic circuit.

If more faster cells are selected:

- Circuit delay decreases.
- Area and power may increase.
- Hold-time violations may occur.

If more slower cells are selected:

- Circuit delay increases.
- Area and power may decrease.
- Required performance may not be achieved.

This guidance is provided to the synthesis tool through **design constraints**.

Constraints help the synthesis tool balance:

- Timing.
- Power.
- Area.

---

## 9. Yosys Setup and Library Loading

Yosys requires the RTL design and the standard-cell library to perform technology mapping.

### Basic Yosys Commands

| Command | Purpose |
|---|---|
| `read_verilog` | Reads the Verilog RTL design |
| `read_liberty` | Reads the standard-cell Liberty file |
| `write_verilog` | Writes the generated Verilog netlist |

### General Synthesis Flow

1. Read the RTL design.
2. Read the Liberty library.
3. Perform synthesis and technology mapping.
4. Generate the gate-level netlist.

![Yosys Setup](images/yosys_setup.png)

---

## 10. Practical: Reading the SKY130 Liberty File

During the hands-on session, the SKY130 High Density standard-cell Liberty file was loaded into Yosys.

### Command Executed

```bash
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
