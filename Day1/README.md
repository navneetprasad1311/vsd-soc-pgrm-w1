# Day 1: Introduction to Verilog RTL Design and Synthesis

This document covers the complete workflow of designing, simulating, and synthesizing a 2:1 multiplexer in Verilog using Yosys and the SkyWater 130nm standard cell library.


## Table of Contents

1. [Simulation](#simulation)
    - Design, Testbench, Simulator
    - Simulation Flow
    - Compiling and Running good_mux.v
    - Viewing Waveforms in GTKWave
    - Verifying Results
2. [Synthesis](#synthesis-of-good_muxv)
    - Launching Yosys
    - Reading Design and Library
    - Generic Synthesis and Technology Mapping
    - Visualizing Netlist
    - Writing Gate-Level Netlist
    - Generating Reports and Area Estimation
3. [Summary](#summary-1)

---

## Simulation

To perform simulation in Verilog, we need to understand its three fundamental components:

- **Design**: The Verilog code that implements the intended functionality.  
- **Testbench**: A simulation environment that applies various inputs to the design and verifies the outputs.  
- **Simulator**: Runs the testbench and design together, allowing observation of inputs and outputs to verify functionality.

### Simulation Flow
1. Write the **Design** (Verilog code).  
2. Write the **Testbench** (applies stimulus).  
3. Compile using **Icarus Verilog (`iverilog`)**.  
4. Run the simulation with **`vvp`**.  
5. View waveforms in **GTKWave**.

![Example](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/008a514609306cd1d7bc3812fa1ff36b4c96e25a/Day1/Images/Example.png)

---

## Lab Simulation & Synthesis

The Verilog programs are cloned from the [sky130RTLDesignAndSynthesisWorkshop](https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop/tree/main/verilog_files) repository.  
This repository also contains library files and Yosys run scripts which will help automate synthesis later.

To clone the repository, run:
```bash
cd ~/Documents/Verilog  
# The directory where you want to store the files  
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop  
```

Once downloaded, go into the files and list them:
```bash
cd sky130RTLDesignAndSynthesisWorkshop/verilog_files  
ls  
```
You should see the following files:

![Verilog_files](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/008a514609306cd1d7bc3812fa1ff36b4c96e25a/Day1/Images/Verilog_files.png)  

Once verified, we can proceed to simulation.

---

## Simulation of `good_mux.v`

### Step 1: Navigate to the Verilog files

First, go to the directory where the Verilog files are stored:

```bash
cd ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/verilog_files  
ls  
```

Make sure you can see `good_mux.v` and `tb_good_mux.v` listed.

#### Verilog Code Analysis

The code for the multiplexer (`good_mux.v`):

```verilog
module good_mux (
    input  i0, 
    input  i1, 
    input  sel, 
    output reg y
);
always @(*)
begin
    if (sel)
        y <= i1;
    else 
        y <= i0;
end
endmodule
```

#### How It Works
- **Inputs**:  
  - `i0`, `i1` → data inputs  
  - `sel` → select line  
- **Output**:  
  - `y` → output (registered)  
- **Logic**:  
  - If `sel = 1`, output `y` gets `i1`.  
  - If `sel = 0`, output `y` gets `i0`.  

This is the basic behavior of a **2:1 multiplexer**, where the select line chooses which input drives the output.


---

### Step 2: Compile the design and testbench

Use **Icarus Verilog** to compile the design (`good_mux.v`) and its testbench (`tb_good_mux.v`):

```bash
mkdir -p ~/Documents/Verilog/Labs
# Create the folder where you would like to store the compiled file
iverilog -o ~/Documents/Verilog/Labs/good_mux_sim.vvp good_mux.v tb_good_mux.v
```

> [!NOTE]  
> The `-o` option lets you specify the name of the compiled output file (`good_mux_sim.vvp`) and change where it is stored (`~/Documents/Verilog/Labs`)
> This keeps your compiled files organized and easy to reference later.

---

### Step 3: Run the simulation

Navigate to the output directory using

```bash
cd ~/Documents/Verilog/Labs
```

Execute the compiled simulation file with:

```bash
vvp good_mux_sim.vvp 
``` 

This runs the simulation and generates a `.vcd` file (Value Change Dump), which contains all signal transitions during the simulation.

---

### Step 4: Open the waveform in GTKWave

View the `.vcd` file in GTKWave:

```bash
gtkwave tb_good_mux.vcd  
```

> [!TIP]  
> You can add signals to the waveform viewer by dragging them from the left panel in GTKWave.  
> Zoom in/out to examine signal transitions and verify the MUX behavior.

---

### Step 5: Verify simulation results

- Observe the input signals and the corresponding output in GTKWave.  
- Check that the output matches the expected truth table for your multiplexer.  
- Make sure all stimuli applied in `tb_good_mux.v` produces correct outputs.

---

### Workflow

![Workflow](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/2243cb648a4f3ff4bb04f581b2e62b58f7768ae3/Day1/Images/Workflow.png)

### Waveform 

![Waveform](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/2243cb648a4f3ff4bb04f581b2e62b58f7768ae3/Day1/Images/Waveform.png)

### Summary

This section demonstrates the complete Verilog simulation workflow using `good_mux.v` and its testbench `tb_good_mux.v`. The process involves:

1. **Navigating** to the directory containing the design and testbench files to ensure all required files are accessible.  
2. **Compiling** the Verilog design and testbench using `iverilog -o` to produce a simulation file (`good_mux_sim.vvp`).  
3. **Running** the simulation with `vvp` to generate a `.vcd` file (Value Change Dump), which captures all signal transitions during the simulation.  
4. **Viewing** the `.vcd` waveform in GTKWave using `gtkwave tb_good_mux.vcd` to analyze signal behavior and verify functional correctness.  

This workflow highlights the standard **Verilog simulation cycle: compile → simulate → observe → verify**. Using these commands ensures organized management of compiled files and accurate functional verification of your RTL designs.

---

## Synthesis of `good_mux.v`

Once the functional correctness of the Verilog design is verified through simulation, the next step is **synthesis**.  
Synthesis converts the RTL (Register Transfer Level) description written in Verilog into a **gate-level netlist** mapped to a given standard cell library.

---

### Flow of Synthesis

1. Write the RTL design in Verilog.  
2. Choose a target technology library (e.g., `sky130_fd_sc_hd`).  
3. Run synthesis using Yosys, which translates the RTL into logic gates. 
4. Analyze the synthesis reports (area, timing, gate-level netlist).  

---

### Step 1: Launch Yosys

```bash
cd ~/Documents/Verilog/Labs
# Directory where you want to store your synthesis files
yosys
```  

---

### Step 2: Read the Design and Library Files

```bash
read_liberty -lib ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
# The directory where the library file is present 
read_verilog ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/verilog_files/good_mux.v
# The directory where the verilog file is present
```  

- `read_liberty`: Reads the target standard cell library.  
- `read_verilog`: Reads the Verilog design file.  

---

### Step 3: Run Generic Synthesis

```bash
synth -top good_mux  
```

- `-top good_mux`: Specifies the top-level module for synthesis.  

---

### Step 4: Map to Technology Library

```bash
dfflibmap -liberty ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib  
abc -liberty ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib  
```

- `dfflibmap`: Maps flip-flops to standard cells.  
- `abc`: Optimizes logic and maps it to the technology library.  

---

### Step 5: Visualize the Netlist

```bash
show good_mux
```

> [!TIP]  
> You can save the visualization to a png file \
> `show -format png good_mux_netlist.png good_mux.v`, saves PNG in the directory

![yosys_show](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day1/Images/yosys_show.png)

---

### Step 6: Write the Gate-Level Netlist

```bash
write_verilog ~/Documents/Verilog/Labs/good_mux_netlist.v 
```

This generates the synthesized gate-level netlist in Verilog format. 

---

### Step 7: Generate Reports

```bash
stat
```

> [!TIP]
> You can use the below command to save the reports directly in a text file instead of being shown in terminal. \
> `yosys -p "read_verilog good_mux.v; synth -top good_mux; stat" > ~/Documents/Verilog/Labs/good_mux_stats.txt` \
> Make sure the target folder exists before running the command.


This prints a summary of the synthesized design (cells used, area, etc.).

---

### Synthesis Outputs

#### Netlist Dot File:

![Dot File](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day1/Images/Dot_file.png)

#### Statistics:

```=== good_mux ===

        +----------Local Count, excluding submodules.
        | 
        8 wires
        8 wire bits
        4 public wires
        4 public wire bits
        4 ports
        4 port bits
        1 cells
        1   sky130_fd_sc_hd__mux2_1

```

#### Rough Area :

Multiply counts × Liberty cell area

1 × `sky130_fd_sc_hd__mux2_1`

1 × 4.0 µm² = 4.0 µm²

Estimated area: **4.0 µm²**

---

### Summary

In this section, we performed **RTL-to-gate-level synthesis** of `good_mux.v` using Yosys. The workflow included:  

1. **Launching Yosys** and reading the Verilog design along with the target standard cell library.  
2. **Running generic synthesis** (`synth`) to convert the RTL into a logic network.  
3. **Mapping to the technology library** (`dfflibmap` and `abc`) for gate-level optimization.  
4. **Visualizing the synthesized netlist** with `show` to inspect gates and connections.  
5. **Writing the gate-level netlist** (`write_verilog`) to a specified directory for later use.  
6. **Generating synthesis reports** (`stat`) to check cell count, area, and other metrics.  

This flow demonstrates the complete synthesis cycle: **read design -> synthesize -> map -> visualize -> save netlist -> analyze reports**, ensuring both functional correctness and readiness for further backend flows such as placement and routing.

---

> [!Note]
> For convenience and organized access, all netlist and compiled Verilog files have been stored in the `~/Documents/Verilog/Labs` folder, facilitating efficient simulation and verification.

---
