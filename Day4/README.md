# Day 4 - GLS, Blocking vs Non-blocking and Synthesis-Simulation mismatch

This document covers Gate-Level Simulation (GLS) to verify post-synthesis behavior, and explores common Synthesis-Simulation mismatches caused by sensitivity lists and blocking vs non-blocking assignments in Verilog.

## Table of Contents

1. [Gate-Level Simulation (GLS)](#gate-level-simulation-gls)
 
2. [Synthesis-Simulation Mismatch](#synthesis-simulation-mismatch)
   2.1 Sensitivity List Issues    
   2.2 Blocking and Non-blocking Statements in Verilog  
   2.3 Caveats with Blocking Statements  

3. [Labs on Gate-Level Simulation](#labs-on-gate-level-simulation) 
   3.1 [`ternary_operator_mux.v`](#ternary_operator_muxv)
       - Verilog Code  
       - RTL Simulation  
       - Gate-level Simulation  

4. [Labs on Synthesis-Simulation Mismatch](#labs-on-synthesis-simulation-mismatch)
   4.1 [`bad_mux.v`](#bad_muxv) 
       - Verilog Code  
       - RTL Simulation  
       - Gate-level Simulation  
       - Comparison  
   4.2 [`blocking_caveat.v`](#blocking_caveatv)
       - Verilog Code  
       - RTL Simulation  
       - Gate-level Simulation  
       - Comparison  

5. [Summary](#summary)

---

## Gate-Level Simulation (GLS)

- Simulation of the synthesized netlist (gates + flops) instead of RTL.
- Used to verify:
  * No synthesis–simulation mismatches.
  * Correct reset/initialization.
  * Timing behavior (with SDF delays).

Types:
1. Zero-delay → checks logic equivalence.
2. Unit-delay → exposes races.
3. SDF back-annotated → real timing validation.

Pros:
- Catches mismatches, reset, and timing issues.

Cons:
- Slow and harder to debug than RTL.

![gls](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/gls.png)

---

## Synthesis-Simulation Mismatch 

### Sensitivity list

**Problem:**
- In Verilog, combinational always blocks require all input signals that affect
  outputs to be in the sensitivity list.
- If a signal is missing, the simulator may not re-evaluate the block when that
  input changes, producing stale or incorrect outputs.
- Example:
    ```verilog
    always @(a) begin
      y = a & b;   // 'b' missing → simulator ignores changes in 'b'
    end
    ```

**Synthesis Behavior:**
- Synthesis tools assume combinational blocks depend on all RHS signals.
- Hardware will behave correctly, causing a mismatch with simulation.

**Fix:**
- Use "always @(*)" in Verilog or "always_comb" in SystemVerilog.
- This ensures all relevant inputs are included automatically.
- Example:
    ```verilog
    always @(*) begin
      y = a & b;   // now simulator and hardware match
    end
    ```

**Summary:**
- Missing sensitivity list entries cause simulation-only errors.
- Synthesis assumes full sensitivity, so always use @(*) for combinational logic.


### Blocking And Non Blocking Statements In Verilog

**1. Blocking Assignment (=)**
- Executes statements sequentially, one after another.
- The right-hand side is evaluated and assigned immediately before moving
  to the next statement.
- Simulation depends on the order of statements, which may not reflect actual
  hardware behavior in sequential logic.
- Example:
    ```verilog
    always @(posedge clk) begin
      q = d;
      r = q;   // r gets the new value of q immediately
    end
    ```
- Best used for combinational logic where order-dependent evaluation is intended.

**2. Non-blocking Assignment (<=)**
- Executes in parallel within a time step.
- All right-hand sides are evaluated first, then all left-hand sides are updated
  together at the end of the time step.
- Correctly models flip-flops in sequential logic, avoiding simulation–hardware mismatches.
- Example:
    ```verilog
    always @(posedge clk) begin
      q <= d;
      r <= q;   // r gets old value of q, just like real hardware
    end
    ```
- Essential for clocked/sequential always blocks.

**Best Practices:**
- Blocking (=) → combinational blocks.
- Non-blocking (<=) → sequential/clocked blocks.
- Mixing blocking in sequential logic can lead to race conditions and
  simulation mismatches.

### Caveats With Blocking Statement

**Problem:**
- Using blocking (=) statements inside sequential (clocked) always blocks can cause:
  * Order-dependent simulation vs. hardware mismatches.
  * Incorrect register inference or unintended behavior.

**Example:**
    ```verilog
    always @(posedge clk) begin
       q = d;
       r = q;   // r uses the updated value of q immediately
    end
    ```
- In real hardware, both q and r would be updated simultaneously by flip-flops,
  but blocking statements simulate sequential updates, causing mismatch.

**Fix:**
- Use non-blocking (<=) assignments for all clocked/sequential logic.
- Corrected example:
    ```verilog
    always @(posedge clk) begin
       q <= d;
       r <= q;   // r gets old value of q, matching actual flip-flop behavior
    end
    ```

**Best Practice:**
- Avoid blocking (=) in sequential blocks to ensure simulation matches synthesized hardware.

![blockingvsnonblocking](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/blockingvsnonblocking.png)

---

## Labs on Gate Level Simulation

### `ternary_operator_mux.v`

**Verilog Code:**

![ternary_operator_mux](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/ternary_operator_mux.png)

```verilog
module ternary_operator_mux (input i0 , input i1 , input sel , output y);
        assign y = sel?i1:i0;
        endmodule
```
> Opened in `vim` inside the folder `verilog_files`

**RTL Simulation Results**

_Workflow_:

![workflow1](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/workflow1.png)


_Waveform_ :

![waveform1](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/waveform1.png)


**Gate-level Simulation Results**

_Workflow_ :

![workflow2](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/workflow2.png)
![synth_show](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/synth_show.png)

To perform GLS use the following command 

```bash
iverilog -o ~/Documents/Verilog/Labs/ternary_operator_mux_gls.vvp ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/primitives.v ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/sky130_fd_sc_hd.v ~/Documents/Verilog/Labs/ternary_operator_mux_net.v ~/Documents/Verilog/sky130RTLDesignAndSynthesisWorkshop/verilog_files/tb_ternary_operator_mux.v
```

Format:

```
iverilog -o <output file path> <verilog models path> <netlist file path> <testbench file path>
```

![workflow3](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/workflow3.png)

_Waveform_ :

![waveform2](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/waveform2.png)


**We see that there are no mismatches in functionality.**

---

## Labs on Synthesis-Simulation Mismatch

### `bad_mux.v`

**Verilog Code:**

![bad_mux](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/bad_mux.png)

```verilog
module bad_mux (input i0 , input i1 , input sel , output reg y);
always @ (sel)
begin
        if(sel)
                y <= i1;
        else
                y <= i0;
end
endmodule
```

This Verilog code only checks for changes in `sel` which does not account for changes in either i1 or i0, inherently behaving as a latch.

**RTL Simulation Results**

_Workflow_ :

![workflow4](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/workflow4.png)


_Waveform_ :

![waveform3](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/waveform3.png)


**Gate-level Simulation Results**

After creating the netlist through `yosys`,

![workflow5](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/workflow5.png)

Perform GLS,

![workflow6](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/workflow6.png)

_Waveform_ :

![waveform4](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/waveform4.png)


**Comparison**

![comparison](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/comparison.png)

Clearly, we see that when `sel` is low, the activity of `i0` is reflected on `y` in case of the Gate-level Simulation, whereas the output `y` has not changed with respect to `i0` in RTL Simulation.\
Thus resulting in **Synthesis-Simulation Mismatch**

---

### `blocking_caveat.v`

**Verilog Code:**

![blocking_caveat](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/blocking_caveat.png)

```verilog
module blocking_caveat (input a , input b , input  c, output reg d);
reg x;
always @ (*)
begin
        d = x & c;
        x = a | b;
end
endmodule
```

**RTL Simulation Results**

_Workflow_ :

![workflow7](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/workflow7.png)

_Waveform_ :

![waveform5](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/waveform5.png)


**GLS Simulation Results**

After creating the netlist through `yosys`,

![workflow8](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/workflow8.png)

Perform GLS,

![workflow9](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/workflow9.png)

_Waveform_ :

![waveform6](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/waveform6.png)


**Comparison**

![comparisonbc](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day4/Images/comparisonbc.png)

In RTL simulation, blocking executes sequentially,

`d = x & c` uses old `x`, then `x = a | b` updates after, so `d` lags. 

In hardware, logic is parallel,

`x = a | b` and `d = (a | b) & c` in the same cycle.

Fix: Use non-blocking (<=) or split into separate always blocks for combinational and sequential logic.

---

## Summary:

- Gate-Level Simulation (GLS): Simulates netlist with delays; ensures post-synthesis behavior matches RTL. Useful for verifying resets, DFT, and X-propagation, but slow.
- Synthesis-Simulation Mismatch: Occurs when RTL sim differs from hardware due to coding issues.
- Sensitivity Lists: Missing signals cause stale outputs in sim, though synthesis assumes full sensitivity. Fix: use always @(*).
- Blocking vs Non-blocking:
  * Blocking (=): sequential, order-dependent.
  * Non-blocking (<=): parallel, updates at timestep end (models flops).
  * Best practice: use = for combinational, <= for sequential.
- Blocking Caveats: Using = in clocked blocks causes order-dependent mismatches (e.g., register inference errors). Fix: use <= or restructure.

---

> [!Note]
> For convenience and organized access, all netlist and compiled Verilog files have been stored in the `~/Documents/Verilog/Labs` folder, facilitating efficient simulation and verification.

---