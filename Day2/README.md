# Day 2 - Timing libs, Hierarchical vs Flat synthesis and Efficient Flop coding styles

This document introduces three key aspects of RTL design and synthesis.  
The first is the `.lib` timing library (`sky130_fd_sc_hd__tt_025C_1v80.lib`),  
an essential part of open-source PDKs that defines timing, power, and functionality.  
The second is synthesis methodologies, comparing hierarchical and flat approaches  
while outlining their trade-offs in scalability and optimization.  
The third is efficient flip-flop coding styles, where proper techniques enhance performance, power use, and synthesis quality.  

---

## Table of Contents

1. [**Timing Libraries**](#timing-libraries) 
   - SKY130 PDK  
   - Understanding `.lib` files (`sky130_fd_sc_hd__tt_025C_1v80.lib`)  
   - File inspection and contents  

2. [**Hierarchical vs Flat Synthesis**](#hierarchical-vs-flat-synthesis) 
   - Hierarchical Synthesis  
     - Workflow, commands, and verification  
   - Flat Synthesis  
     - Workflow, commands, and verification  
   - Comparison Table  
   - Sub-module Level Synthesis  
     - Importance, workflow, and verification  

3. [**Flip-Flop Coding Styles**](#the-various-flip-flop-coding-styles) 
   - Asynchronous Reset/Set  
   - Synchronous Reset/Set  

4. [**Simulation and Synthesis of Flip-Flops**](#synthesis-and-simulation-of-flip-flops)  
   - Simulation workflow (`iverilog` → `vvp` → `GTKWave`)  
   - Synthesis workflow (Yosys commands, netlist visualization)  

5. [**Summary**](#summary)
   - Key takeaways on timing libraries, synthesis approaches, and flip-flop coding


---

## Timing Libraries

### SKY130 PDK

The SKY130 PDK (Process Design Kit) provides all the necessary design files, standard cell libraries, and technology data required for ASIC design using the open-source SkyWater 130nm process.  
It includes `.lib` timing libraries, LEF/DEF files for layout, and GDSII data for fabrication.  
The `.lib` files, such as `sky130_fd_sc_hd__tt_025C_1v80.lib`, define the timing, power, and functional behavior of each standard cell under different conditions.  
These timing libraries are crucial for accurate synthesis, static timing analysis, and overall design optimization in digital circuits.

In `.lib` timing libraries like `sky130_fd_sc_hd__tt_025C_1v80.lib`, the filename itself encodes the PVT (Process, Voltage, Temperature) conditions used to characterize the cells. Understanding this naming convention helps correlate the library to the specific conditions:

- **sky130_fd_sc_hd** – Base library name:  
  - `sky130` → SkyWater 130nm process  
  - `fd` → Fully Depleted (process type)  
  - `sc` → Standard Cells  
  - `hd` → High Density variant  

- **tt** – Process corner:  
  - `tt` → Typical-Typical (nominal NMOS and PMOS)  
  - Other possible corners include `ff` (Fast-Fast), `ss` (Slow-Slow), `fs` (Fast-Slow), `sf` (Slow-Fast)

- **025C** – Temperature condition:  
  - 25°C (room/typical temperature)  
  - Other libraries may use `-40C` for low or `125C` for high temperature

- **1v80** – Voltage condition:  
  - 1.80 V supply voltage  
  - Variations could be `1v62` for lower voltage or `1v98` for higher voltage

So, `sky130_fd_sc_hd__tt_025C_1v80.lib` specifically refers to the High-Density standard cell library for SkyWater 130nm, characterized at **typical process**, **25°C**, and **1.8V**. This systematic naming allows tools to select the appropriate library for synthesis, timing analysis, or corner-based verification.

### Understanding the `lib` file

To open the `sky130_fd_sc_hd.lib`, navigate to the directory where the library is stored. \
That is, `~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib` 
This is done using the command,

```bash
cd ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib
ls
```

You can inspect the contents of the library using a text editor like `vim`, `nano`, or `gedit`, personally I prefer vim so:

```bash
vim sky130_fd_sc_hd__tt_025C_1v80.lib
```

The `.lib` file contains detailed definitions of each standard cell, including:  
- **Cell name and type** (e.g., AND, OR, DFF)  
- **Pin definitions** (input, output, clock)  
- **Timing information** (setup, hold, propagation delays)  
- **Power characteristics** (dynamic and leakage power)  
- **Conditions for PVT corners** (process, voltage, temperature)

By understanding the structure of the `.lib` file, you can analyze cell behavior and ensure accurate synthesis and timing analysis in your RTL design.


![Timing library](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/Timing_lib.png)

---

## Hierarchical vs Flat Synthesis

Synthesis is the process of converting RTL code into a gate-level netlist using standard cells from the library. 
There are two main approaches: **hierarchical synthesis** and **flat synthesis**, each with its own advantages and trade-offs.

### Hierarchical Synthesis
- The design is synthesized **module by module**, preserving the hierarchy of the RTL.  
- **Advantages:**
  - Easier to debug and manage large designs.
  - Reusable modules can be synthesized separately.
  - Reduces runtime and memory usage for very large designs.
- **Disadvantages:**
  - May miss some cross-module optimizations.
  - Overall area or timing may be slightly less optimal compared to flat synthesis.

### Flat Synthesis
- The entire design is synthesized as a **single flattened block**, ignoring module boundaries.  
- **Advantages:**
  - Allows the synthesis tool to optimize across module boundaries.
  - Can achieve better area, timing, and power optimization.
- **Disadvantages:**
  - Requires more memory and runtime for large designs.
  - Harder to debug or modify specific parts of the design.

In practice, designers often use a **mixed approach**, keeping critical modules hierarchical while flattening performance-critical paths to achieve a balance between manageability and optimization.

Suppose the file `multiple_modules.v` has to be synthesized. The first step is to **locate the file** in your project directory. \
This is done using,
```bash
cd ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

This design contains multiple modules, as shown below:

```verilog
module sub_module2 (input a, input b, output y);
	assign y = a | b;
endmodule

module sub_module1 (input a, input b, output y);
	assign y = a&b;
endmodule


module multiple_modules (input a, input b, input c , output y);
	wire net1;
	sub_module1 u1(.a(a),.b(b),.y(net1));  //net1 = a&b
	sub_module2 u2(.a(net1),.b(c),.y(y));  //y = net1|c ,ie y = a&b + c;
endmodule
```

### Hierarchical Synthesis Workflow

**Hierarchical synthesis of `multiple_modules.v` using Yosys:**

### 1. **Open the directory where you would want to run the synthesis**

```bash
cd ~/Documents/Verilog/Labs
```

### 2. **Run hierarchical synthesis in Yosys:**  

```bash
yosys
```

Inside yosys,

```bash
read_liberty -lib ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/verilog_files/multiple_modules.v
synth -top multiple_modules
dfflibmap -liberty ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib  
abc -liberty ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib  
show multiple_modules
show -format png multiple_modules
write_verilog ~/Documents/Verilog/Labs/multiple_modules_hier.v 
```
![Workflow](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/Workflow.png)
![yosys_show_multiple_modules](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/yosys_show_mutliple_modules.png)


**Explanation:**  
- `read_liberty` loads the standard cell timing and power characteristics.  
- `dfflibmap` ensures flip-flops are mapped to library cells.  
- `abc` performs technology mapping and logic optimization.  
- `synth -top` preserves module hierarchy while generating the gate-level netlist.  
- `write_verilog` outputs the hierarchical netlist ready for further analysis or place-and-route.

This approach keeps sub-modules separate, making it easier to **debug, reuse, and manage large designs**, while still producing a synthesized netlist compatible with the SkyWater 130nm PDK.

### 3. **Verify the Synthesis**

#### Netlist Dot File:

![Netlist Dot File](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/Dot_file.png)

#### Statistics:

<pre>

=== multiple_modules ===

        +----------Local Count, excluding submodules.
        | 
        5 wires
        5 wire bits
        5 public wires
        5 public wire bits
        4 ports
        4 port bits
        2 submodules
        1   sub_module1
        1   sub_module2

=== sub_module1 ===

        +----------Local Count, excluding submodules.
        | 
        3 wires
        3 wire bits
        3 public wires
        3 public wire bits
        3 ports
        3 port bits
        1 cells
        1   $_AND_

=== sub_module2 ===

        +----------Local Count, excluding submodules.
        | 
        3 wires
        3 wire bits
        3 public wires
        3 public wire bits
        3 ports
        3 port bits
        1 cells
        1   $_OR_

=== design hierarchy ===

        +----------Count including submodules.
        | 
        2 multiple_modules
        1 sub_module1
        1 sub_module2

        +----------Count including submodules.
        | 
       11 wires
       11 wire bits
       11 public wires
       11 public wire bits
       10 ports
       10 port bits
        - memories
        - memory bits
        - processes
        2 cells
        1   $_AND_
        1   $_OR_
        2 submodules
        1   sub_module1
        1   sub_module2
</pre>

### Flat Synthesis Workflow

**Flat synthesis of `multiple_modules.v` using Yosys:**

### 1. **Open the directory where you want to run the synthesis**

```bash
cd ~/Documents/Verilog/Labs
```

### 2. **Run flat synthesis in Yosys:**  
```bash
yosys
```

Inside yosys,

```bash
read_verilog ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/verilog_files/multiple_modules.v 
synth -top multiple_modules
abc -liberty ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
flatten
show multiple_modules
show -format png multiple_modules
write_verilog ~/Documents/Verilog/Labs/multiple_modules_flat.v 
```

![Flat Synthesis Workflow](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/Workflow_flat.png)
![yosys_show_multiple_modules_flat](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/yosys_show_multiple_modules_flat.png)

**Explanation:**  
- `synth -top` generates the gate-level netlist of the flattened design. 
- `abc` performs technology mapping and logic optimization. 
- `flatten` removes module hierarchy so the tool can optimize across the entire design.    
- `write_verilog` outputs the flat netlist ready for further analysis or place-and-route.

### 3. **Verify the Flat Synthesis**

#### Netlist Dot File:

![Netlist Dot File](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/Dot_file_flat.png)


#### Statistics

<pre>
=== multiple_modules ===

        +----------Local Count, excluding submodules.
        | 
        5 wires
        5 wire bits
        5 public wires
        5 public wire bits
        4 ports
        4 port bits
        2 submodules
        1   sub_module1
        1   sub_module2

=== sub_module1 ===

        +----------Local Count, excluding submodules.
        | 
        3 wires
        3 wire bits
        3 public wires
        3 public wire bits
        3 ports
        3 port bits
        1 cells
        1   $_AND_

=== sub_module2 ===

        +----------Local Count, excluding submodules.
        | 
        3 wires
        3 wire bits
        3 public wires
        3 public wire bits
        3 ports
        3 port bits
        1 cells
        1   $_OR_

=== design hierarchy ===

        +----------Count including submodules.
        | 
        2 multiple_modules
        1 sub_module1
        1 sub_module2

        +----------Count including submodules.
        | 
       11 wires
       11 wire bits
       11 public wires
       11 public wire bits
       10 ports
       10 port bits
        - memories
        - memory bits
        - processes
        2 cells
        1   $_AND_
        1   $_OR_
        2 submodules
        1   sub_module1
        1   sub_module2
</pre>


### Comparison 

![Comparison](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/Comparison.png)


Key Differences:


| Feature           | Hierarchical Synthesis       | Flat Synthesis         |
| ----------------- | ---------------------------- | ---------------------- |
| **Modules**       | Preserves submodules         | Flattens all into one  |
| **Optimization**  | Local per module             | Global across design   |
| **Debugging**     | Easier, hierarchy intact     | Harder, hierarchy lost |
| **Port Ordering** | Preserved                    | May change             |
| **Use Case**      | Verification, modular design | Maximum optimization   |



## Sub-module Synthesis

RTL (Register Transfer Level) designs are typically modular, consisting of multiple functional blocks or sub-modules. Sub-module level synthesis allows each sub-module to be synthesized independently.

**Why is sub-module level synthesis important?**

- **Optimization and Area Reduction:** Synthesizing sub-modules separately allows the tool to optimize each one individually. This includes logic optimization, technology mapping, and area minimization, resulting in more efficient resource usage and a smaller overall chip area.  
- **Reusability:** Each sub-module can be designed, verified, and optimized independently. They can then be reused across different designs, saving development time and improving efficiency.  
- **Parallel Processing:** Sub-modules can be synthesized concurrently, which speeds up the synthesis process. For large designs, parallel synthesis can significantly reduce turnaround time.  

**Commands to run sub-module synthesis:**

```bash
read_liberty -lib ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/verilog_files/multiple_modules.v
synth -top sub_module1
abc -liberty ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
show -format png
```
![Workflow](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/Workflow-sub.png)
![yosys_show_submodule1](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/yosys_show_multiple_submodule1.png) 

### Verification of Sub-module Synthesis

#### Netlist Dot File 

![Dot_file_sub](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/Dot_file_sub.png)

#### Statistics 

<pre>
=== sub_module1 ===

        +----------Local Count, excluding submodules.
        | 
        3 wires
        3 wire bits
        3 public wires
        3 public wire bits
        3 ports
        3 port bits
        1 cells
        1   $_AND_

</pre>

---

## The Various Flip-Flop Coding Styles  

### 1. **Asynchronous Reset Flip-Flop**

```verilog
always @(posedge clk or posedge rst) begin
    if (rst)
        q <= 0;
    else
        q <= d;
end
```
  - The output `q` is immediately reset to `0` when `rst` is asserted, regardless of the clock.  
  - Useful for global resets in FPGA or ASIC designs.


### 2. **Asynchronous Set Flip-Flop**
```verilog
always @(posedge clk or posedge set) begin
    if (set)
        q <= 1;
    else
        q <= d;
end
```

  - The output `q` is immediately set to `1` when `set` is asserted, independent of the clock.  
  - Helps initialize signals to a known state quickly.


### 3. **Synchronous Reset Flip-Flop**
```verilog
always @(posedge clk) begin
    if (rst)
        q <= 0;
    else
        q <= d;
end
```
  - The output `q` is reset to `0` **only on the active clock edge** when `rst` is asserted.  
  - Keeps all flip-flops aligned with the clock and avoids timing issues.


### 4. **Synchronous Set Flip-Flop**
```verilog
always @(posedge clk) begin
    if (set)
        q <= 1;
    else
        q <= d;
end
```
  - The output `q` is set to `1` **only on the active clock edge** when `set` is asserted.  
  - Ensures predictable and controlled behavior for sequential logic.

---


### Synthesis and Simulation of Flip-FLops

#### **Simulation**

Firstly, iverilog simulation of the flipflop is done, to verify its functionality 

Locate the file in the `verilog_files` folder

```bash
cd Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

```bash
iverilog -o ~/Documents/Verilog/Labs/dff_syncres.vvp dff_syncres.v tb_dff_syncres.v
```

Then, in the labs folder, run the compiled file

```bash
cd ~/Documents/Verilog/Labs
vvp dff_syncres.vvp
```

Now, run the `.vcd` file through GTKWave and check its waveform

```bash
gtkwave tb_dff_syncres.vcd
```

Waveform:

![Waveform](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/Waveform.png)

#### **Synthesis**

Synthesis is done through yosys by following these commands.

```bash
yosys #to run yosys
```

Then, inside yosys

```bash
read_liberty -lib ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/verilog_files/dff_syncres.v
synth -top dff_syncres
dfflibmap -liberty ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
show -format png
```

Netlist:

![Synthesis](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day2/Images/yosys_dffsyncres.png)


## Summary

- **Timing Libraries:**  
  - `.lib` files (e.g., `sky130_fd_sc_hd__tt_025C_1v80.lib`) define standard cell timing and characteristics used for accurate synthesis and timing analysis.

- **Synthesis Approaches:**  
  - **Hierarchical Synthesis:** Synthesizes sub-modules individually; allows better optimization and area control.  
  - **Flat Synthesis:** Synthesizes the entire design at once; simpler but may miss local optimization opportunities.  
  - Flattening can be done post-hierarchical synthesis if needed.

- **Efficient Flip-Flop Coding Styles:**  
  - **Asynchronous Reset/Set:** Immediate response, independent of clock; useful for global initialization.  
  - **Synchronous Reset/Set:** Changes occur only on clock edge; ensures predictable timing and avoids hazards.  

- **Simulation & Synthesis Workflow:**  
  - **Simulation:** `iverilog` → `vvp` → `GTKWave`.  
  - **Synthesis (Yosys):** Load library → read Verilog → `synth` → `dfflibmap` → `abc` → `show`.

---

> [!Note]
> For convenience and organized access, all netlist and compiled Verilog files have been stored in the `~/Documents/Verilog/Labs` folder, facilitating efficient simulation and verification.

---
