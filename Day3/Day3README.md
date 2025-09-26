# Day 3: Combinational and Sequential Optimizations

Optimisations are widely used by synthesis tools to simplify logic, improve timing,
and reduce resource usage while preserving the functional behavior of the design.

---

## Table of Contents


1. [**Combinational Logic Optimization**](#combinational-logic-optimization)
   - Constant Propagation  
   - Boolean Logic Optimization  
   - [Labs: `opt_check.v`, `opt_check2.v`, `opt_check3.v`](#labs-on-combinational-logic-optimization)

2. [**Sequential Logic Optimization**](#sequential-logic-optimization)
   - State Optimization  
   - Cloning  
   - Retiming  
   - [Labs: `dff_const1`, `dff_const2`, `dff_const3`](#labs-on-sequential-logic-optimization)

3. [**Summary**](#summary)


---

## Combinational Logic Optimization:

## 1. Constant Propagation
**Definition:**  
When input signals or internal signals are assigned constant values, the synthesis tool
replaces those signals with their constant equivalents and simplifies the logic around them.

**Example in Verilog:**
<pre>
    assign y = a & 1'b1;  // Simplifies to: y = a  
    assign z = b & 1'b0;  // Simplifies to: z = 0  
</pre>
**How it works:**
- Detects nets or variables that always hold a constant (0, 1, or fixed value).
- Propagates that constant through the circuit.
- Removes gates or logic that become redundant.

**Benefits:**
- Reduces gate count and power usage.
- Simplifies design, enabling further optimizations like dead code elimination.

![ConstantPropagation](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day3/Images/Constant%20Propagation.png)

---

## 2. Boolean Logic Optimization

Boolean logic optimization simplifies digital circuits to reduce hardware, increase speed, and lower power consumption.

## Purpose
- Minimize gates
- Reduce delay
- Save power
- Improve readability

## Methods
- **Algebraic Simplification**: Apply Boolean laws (e.g., `A + A' = 1`, `A·1 = A`)
- **Karnaugh Map (K-Map)**: Visual grouping of 1s/0s to simplify expressions
- **Quine-McCluskey**: Tabular method for systematic simplification
- **EDA Tools**: Automatic optimization (constant propagation, dead code elimination, subexpression reuse)

## Example
<pre>
    Original: `F = A·B + A·B' + A'·B`  
    Simplified: `F = A + B`
</pre>

Optimized logic reduces gates and improves performance.

---

## Labs on Combinational Logic Optimization 

### `opt_check.v`

Locate the file inside the `verilog_files` folder and view it using your preferred text editor.

```bash
cd Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/verilog_files/
vim opt_check.v
```
**Verilog Code**

![opt_check](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day3/Images/opt_check.png)

**Functionality**

We see that the logic is 

<pre>
    y =  a ? b : 0
</pre>

This is simply an AND gate made with complicated logic, let us optimise it. \
Synthesise it through yosys by following the steps given in Day 2 \
Make sure you add the below line between `synth -top` and `abc -liberty`

**Logic Optimization in Yosys**

Synthesise the design through Yosys as outlined in Day 2. Make sure to add the following line between `synth -top` and `abc -liberty`:

```bash
opt_clean -purge
```

> [!Note]
> This performs logic optimization and cleanup with complete removal of unused elements.

> - **opt_clean**:  
>  - Optimizes the design by removing **unused wires, cells, and trivial logic** (like buffers or redundant gates).

> - **-purge**:  
>  - Completely removes anything **not connected to outputs**, including dead wires, cells, or modules.

> **Effect:**  
> The netlist becomes **smaller, cleaner, and free of unused/redundant elements**, reducing area and improving synthesis results.

Running synthesis would get you this optimised Netlist, represented by this dot file.

![opt_check_show](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day3/Images/opt_check_show.png)


### `opt_check2`

**Verilog Code:**

```verilog
module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule
```

This essentially is just `OR` function of a and b.

**Logic Optimisation in Yosys:**

![opt_check2_show](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day3/Images/opt_check2_show.png)


### `opt_check3`

**Verilog Code:**

```verilog
module opt_check3 (input a , input b, input c , output y);
        assign y = a?(c?b:0):0;
endmodule
```

This performs `AND` operation between inputs a, b and c. We can verify this through our optimisation.

**Logic Optimisation in Yosys:**

![opt_check3_show](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day3/Images/opt_check3_show.png)

---

## Sequential Logic Optimization

## 1. State Optimization
**Definition:**  
In Finite State Machines (FSMs), many states may be redundant, unreachable, or suboptimal.
State optimization involves minimizing and re-encoding states to achieve simpler and more efficient logic.

**Techniques:**
- **Unreachable state removal:** Eliminates states that cannot be reached due to logic constraints.
- **Equivalent state merging:** Combines states that behave identically for all inputs.
- **State re-encoding:** Changes encoding scheme (binary, one-hot, gray, etc.) to balance area vs. speed.

**Example:**  
An FSM with 8 states but only 5 are reachable → synthesis removes 3 unreachable states.

**Benefits:**
- Reduces number of flip-flops required.
- Decreases next-state logic complexity.
- Improves area, power, and possibly timing.

---

## 2. Cloning
**Definition:**  
Cloning refers to duplicating logic elements (gates or small logic cones) to reduce fanout load
or shorten critical paths.

**How it works:**
- If one logic gate drives too many loads, the capacitance increases and slows down timing.  
- Synthesis tools duplicate that gate, creating multiple identical drivers.  
- Each duplicated instance drives a subset of the loads.

**Example:**  
A single AND gate output driving 20 flip-flops might be cloned into 2 AND gates,  
each driving 10 flip-flops.

**Benefits:**
- Improves timing closure by reducing load per driver.
- Balances critical paths.
- Sacrifices some area (extra gates) for speed.

---

## 3. Retiming
**Definition:**  
Retiming is the process of moving flip-flops across combinational logic without altering the functional behavior.  
The goal is to balance logic delays and shorten the critical path.

**How it works:**
- Registers can be shifted forward or backward through logic gates.
- The number of registers in a loop is preserved, so overall pipeline depth is unchanged.
- Ensures functional equivalence while redistributing logic between clock stages.

**Example:**  
If one pipeline stage has heavy logic and the next has very light logic,  
retiming moves some registers so the logic is balanced more evenly.

**Benefits:**
- Increases maximum achievable clock frequency.
- Reduces critical path delay.
- Improves performance without changing overall latency.

---
 
## Labs on Sequential Logic Optimization 

## `dff_const1`

**Verilog Code**
```verilog
module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end
endmodule
```

**Functionality**

This D flip-flop has:  
- Asynchronous reset to 0  
- Loads constant 1 when not in reset

**Logic Optimization in Yosys**

![dff_const1_show](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day3/Images/dff_const1_show.png)


## `dff_const2`

**Verilog Code**
```verilog
module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
        if(reset)
                q <= 1'b1;
        else
                q <= 1'b1;
end

endmodule
```

**Functionality**

- **Asynchronous reset to 1**: When `reset` is high, `q` is set to `1`.  
- **Constant output 1**: On every positive clock edge (when not in reset), `q` is also set to `1`.  

Effectively, **`q` is always `1`**, regardless of the clock or reset.  

In short, this flip-flop behaves like a **constant logic 1** generator.

**Logic Optimisation in Yosys**

Running synthesis will generate the optimized netlist, which can be represented in a dot file:

![dff_const2_show](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day3/Images/dff_const2_show.png)

## `dff_const3`

**Verilog Code**
```verilog
module dff_const3(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
        if(reset)
        begin
                q <= 1'b1;
                q1 <= 1'b0;
        end
        else
        begin
                q1 <= 1'b1;
                q <= q1;
        end
end

endmodule
```

**Functionality**

- **Asynchronous reset**:  
  - When `reset` is high, `q` is set to `1` and `q1` is set to `0`.

- **On each positive clock edge (when not in reset)**:  
  - `q1` is set to `1`.  
  - `q` takes the **previous value of `q1`** (because of non-blocking assignment).

**Effectively:**  
- After reset, `q` will output `0` on the first clock edge (because `q1` was `0`), and then `1` on all subsequent clock edges.  
- `q1` serves as a one-cycle delayed signal to drive `q`.

In short, `q` produces a **one-cycle delayed transition from 1 to 1**, starting effectively from `0` after reset.

**Logic Optimisation in Yosys**

Running synthesis will generate the optimized netlist, which can be represented in a dot file:

![dff_const3_show](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day3/Images/dff_const3_show.png)

---

## Summary

### Combinational
- **Constant Propagation**: Replace constants → reduce gates/power (`y = a & 1'b1 → y = a`)
- **Boolean Optimization**: Simplify logic expressions (`F = A·B + A·B' + A'·B → F = A + B`)
- **Labs**: opt_check.v, opt_check2.v, opt_check3.v

### Sequential
- **State Optimization**: Remove unreachable/equivalent FSM states
- **Cloning**: Duplicate gates to reduce fanout and balance paths
- **Retiming**: Shift flip-flops to balance delays
- **Labs**: dff_const1, dff_const2, dff_const3

### Tools
- `opt_clean -purge` in Yosys
- Reduces gates, power, critical path; preserves functionality

