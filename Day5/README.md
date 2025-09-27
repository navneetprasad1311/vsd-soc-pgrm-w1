# Day 5 - Optimization in Synthesis

This documentation covers optimization techniques in Verilog synthesis. Focusing on proper use of conditional constructs (if-else, case), avoiding inferred latches, and using loops (for and generate-for) for scalable, clean, and synthesizable designs.

## Table of Contents  
  
1. [**Verilog Conditional Constructs**](#verilog-conditional-constructs-if-else-and-case-statements)
   - 2.1 If-Else Statements  
   - 2.2 Case Statements  
   - 2.3 If-Else with Case 
2. [**Inferred Latches**](#inferred-latches)
   - 3.1 Concept of Inferred Latches  
   - 3.2 Partial Assignments  
   - 3.3 [Labs on Incomplete If-Else](#labs-on-incomplete-if-else)
       - incomp_if.v  
       - incomp_if2.v  
   - 3.4 [Labs on Incomplete Case](#labs-on-incomplete-case) 
       - incomp_case.v  
       - bad_case.v  
3. [**Verilog Looping Constructs**](#verilog-looping-constructs-for-loop--generate-for-loop)
   - 4.1 For Loop  
   - 4.2 Generate For Loop  
4. [**Labs on For Loop**](#labs-on-for-loop)
   - mux_generate.v  
   - demux_generate.v  
5. [**Labs on Generate For Loop**](#labs-on-generate-for-loop)
   - 8-bit Ripple Carry Adder (RCA)  
6. [**Summary**](#summary)


## Verilog Conditional Constructs: If-Else and Case Statements

### 1. If-else Statements

Executes code based on conditions, allowing different actions depending on signal values.

**`if-else`**

_Syntax_ :
```verilog
always @(*) begin
    if (condition)
        statement1;
    else
        statement2;
end
```

_Example_ :
```verilog
always @(*) begin
    if (sel)
        y = a;
    else
        y = b;
end
```

**Nested `if`**

_Syntax_ :
```verilog
always @(*) begin
    if (condition1) begin
        if (condition2)
            statement1;
        else
            statement2;
    end else begin
        statement3;
    end
end
```
_Example_ :
```verilog
always @(*) begin
    if (sel1) begin
        if (sel2)
            y = a;
        else
            y = b;
    end else begin
        y = c;
    end
end
```

---

### 2. Case Statements

Multi-way selection based on a single expression.

_Syntax_ :
```verilog
always @(*) begin
    case(expression)
        value1: statement1;
        value2: statement2;
        ...
        default: statementN;
    endcase
end
```

_Example_ :
```verilog
always @(*) begin
    case(sel)
        2'b00: y = a;
        2'b01: y = b;
        2'b10: y = c;
        default: y = 0;
    endcase
end
```

---

## 3. If-Else with Case

_Syntax_ :
```verilog
always @(*) begin
    if (condition) begin
        case(expression)
            value1: statement1;
            value2: statement2;
            ...
            default: statementN;
        endcase
    end
    else
        statementX;
end
```
_Example_ :
```verilog
always @(*) begin
    if (enable) begin
        case(sel)
            2'b00: y = a;
            2'b01: y = b;
            2'b10: y = c;
            default: y = 0;
        endcase
    end else begin
        y = 0;
    end
end
```

> [!Note] 
> Outer `if` controls conditions; inner `case` handles multiple selections. Always assign outputs in all paths.


**If-else vs Case**

| Feature | If-Else | Case |
|---------|---------|------|
| **Purpose** | Conditional execution based on Boolean expressions | Multi-way selection from discrete values |
| **Usage** | Binary or nested decisions | Multiple options, like multiplexers or states |
| **Default Handling** | Optional; missing branches may infer latches | Include `default` to avoid latches |
| **Readability** | Can get complex with many conditions | Cleaner for many discrete choices |
| **Evaluation** | Conditions checked sequentially | Expression matched against all cases |

---


## Inferred Latches

In Verilog, inferred latches occur when a combinational block does not assign a value to an output in all possible branches of an `if-else` statement or a `case` block.

For example, if an `if` statement only assigns `y = a;` when `enable` is true but has no else branch, the synthesizer infers a latch to hold the previous value of `y`. 

Partial assignments occur when only some bits of a vector or some outputs of a module are assigned in a combinational block, leaving others undefined in certain conditions. Like incomplete if statements, partial assignments can lead to **inferred latches**, because the hardware needs to "remember" the previous value of unassigned bits.  

To avoid all this, all outputs should be assigned in every possible branch, such as adding an `else` clause like `y = 0;`. Similarly, in `case` statements, including a `default` ensures that outputs are fully defined and prevents unintended latches.

**Example of Inferred Latch:**
```verilog
always @(*) begin
    if (enable)
        y = a;
    // else branch missing → y retains previous value → latch inferred
end
```

**Corrected Version (No Latch):**
```verilog
always @(*) begin
    if (enable)
        y = a;
    else
        y = 0;  // output assigned in all branches
end
```

---

## Labs on Incomplete `if-else`

### `incomp_if.v`

**Verilog Code** 
```verilog
module incomp_if (input i0 , input i1 , input i2 , output reg y);
always @ (*)
begin
        if(i0)
                y <= i1;
end
endmodule
```

**RTL Simulation**

_Workflow_ :

![workflow1](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow1.png)

_Waveform_:

![waveform1](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/waveform1.png)

**Synthesis Netlist Check**

_Workflow_ :

![workflow2](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow2.png)

_Netlist Dot File_ :

![dotfile1](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/dotfile1.png)

- If `i0` is high, `y` takes the value of `i1`.
- If `i0` is low, `y` is **not assigned**, so it retains its previous value, causing a **latch to be inferred**.


---

### `incomp_if2.v`

**Verilog Code**

```verilog
module incomp_if2 (input i0 , input i1 , input i2 , input i3, output reg y);
always @ (*)
begin
        if(i0)
                y <= i1;
        else if (i2)
                y <= i3;

end
endmodule
```

**RTL Simulation**

_Workflow_ :

![workflow3](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow3.png)

_Waveform_ :

![waveform2](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/waveform2.png)


**Synthesis Netlist Check**

_Workflow_ :

![workflow4](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow4.png)

_Netlist Dot File_ :

![dotfile2](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/dotfile2.png)

- If `i0` is high, `y` follows `i1`.
- Else, if `i2` is high, `y` follows `i3`.
- If neither `i0` nor `i2` is high, `y` retains its previous value (causing a latch to be inferred).

---

## Labs on Incomplete `case`

### `incomp_case.v`

**Verilog Code**

```verilog
module incomp_case (input i0 , input i1 , input i2 , input [1:0] sel, output reg y);
always @ (*)
begin
        case(sel)
                2'b00 : y = i0;
                2'b01 : y = i1;
        endcase
end
endmodule
```

**RTL Simulation**

_Workflow_ :

![workflow5](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow5.png)

_Waveform_ :

![waveform3](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/waveform3.png)

**Synthesis Netlist Check**

_Workflow_ :

![workflow6](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow6.png)

_Netlist Dot FIle_ :

![dotfile3](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/dotfile3.png)

- If `sel` is `2'b00`, `y` takes the value of `i0`.
- If `sel` is `2'b01`, `y` takes the value of `i1`.
- For all other values of `sel` (`2'b10` or `2'b11`), `y` is **not assigned**, so it retains its previous value, causing a **latch to be inferred**.

---


### `bad_case.v`

**Verilog Code**

```verilog
module bad_case (input i0 , input i1, input i2, input i3 , input [1:0] sel, output reg y);
always @(*)
begin
        case(sel)
                2'b00: y = i0;
                2'b01: y = i1;
                2'b10: y = i2;
                2'b1?: y = i3;
                //2'b11: y = i3;
        endcase
end

endmodule
```

**RTL Simulation**

_Workflow_ :

![workflow7](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow7.png)

_Waveform_ :

![waveform4](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/waveform4.png)


**Post Synthesis Check**

_Synthesis Workflow_ :

![workflow8](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow8.png)

_Netlist Dot File_ :

![dotfile4](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/dotfile4.png)

_Simulation Workflow_:

![workflow9](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow9.png)

_GLS Waveform_ :

![waveform5](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/waveform5.png)

- For `sel = 2'b00`, `y = i0`;  
- For `sel = 2'b01`, `y = i1`;  
- For `sel = 2'b10`, `y = i2`; 
- `sel = 2'b11` is commented out, so for this value `y` is **not assigned**, causing a **latch** to be inferred in _RTL Simulation_.

But this inferred latch is corrected in simulation as seen in the **GLS Waveform**

---

## Verilog Looping Constructs: `for` loop & `generate for` loop

### 1. `for` Loop

- **Purpose:** Repeatedly executes a block of statements a fixed number of times within an `always` block.
- **Usage:** Mainly used for iterative assignments in **combinational or sequential logic**.

**Syntax:**
```verilog
always @(*) begin
    integer i;
    for (i = 0; i < N; i = i + 1) begin
        array[i] = input_signal[i];
    end
end
```

- **Notes:**  
  - Executes **during simulation**, not synthesis-time replication.  
  - Loop index must be an **integer**.  
  - Useful for operations like summing arrays, shifting, or copying signals.

---

### 2. `generate for` Loop

- **Purpose:** Used at **compile/synthesis time** to generate multiple instances of modules or repeated RTL structures.
- **Usage:** Creates hardware replication, like multiple adders, flip-flops, or gates.

**Syntax:**
```verilog
genvar i;
generate
    for (i = 0; i < N; i = i + 1) begin : loop_label
        my_module u_inst (
            .in(signal_in[i]),
            .out(signal_out[i])
        );
    end
endgenerate
```

- **Notes:**  
  - Executes **at synthesis/compile time**, producing multiple hardware instances.  
  - `genvar` is required for the loop index.  
  - Commonly used for arrays of registers, multiplexers, or replicated modules like **Ripple Carry Adders (RCA)**.

---

## Labs on `for` loop

## `mux_generate.v`

**Verilog Code**

```verilog
module mux_generate (input i0 , input i1, input i2 , input i3 , input [1:0] sel  , output reg y);
wire [3:0] i_int;
assign i_int = {i3,i2,i1,i0};
integer k;
always @ (*)
begin
for(k = 0; k < 4; k=k+1) begin
        if(k == sel)
                y = i_int[k];
end
end
endmodule
```

**Simulation**

_Workflow_ :

![workflow10](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow10.png)

_Waveform_ :

![waveform6](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/waveform6.png)


- Implements a 4-to-1 multiplexer using a `for` loop.
- Inputs `i0`–`i3` are packed into a 4-bit wire `i_int` for indexing.
- Loop iterates over indices 0 to 3 inside an `always` block.
- Assigns `y = i_int[k]` when `k` matches the select signal `sel`.
- Only the selected input is passed to the output.
- Avoids multiple nested `if-else` or `case` statements, making code concise.

---


## `demux_generate.v`

**Verilog Code**

```verilog
module demux_generate (output o0 , output o1, output o2 , output o3, output o4, output o5, output o6 , output o7 , input [2:0] sel  , input i);
reg [7:0]y_int;
assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;
integer k;
always @ (*)
begin
y_int = 8'b0;
for(k = 0; k < 8; k++) begin
        if(k == sel)
                y_int[k] = i;
end
end
endmodule
```

**Simulation**

_Workflow_ :

![workflow11](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow11.png)

_Waveform_ :

![waveform7](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/waveform7.png)  

- Implements an 8-output demultiplexer using a `for` loop.  
- Outputs `o0`–`o7` are packed into an 8-bit register `y_int` for easy indexing.  
- `y_int` is initialized to 0, then only the bit matching `sel` is set to input `i`.  
- Routes the single input `i` to the selected output while others remain 0.  
- Loop executes sequentially during simulation; no latches are inferred.  
- Provides a concise alternative to multiple if-else or case statements.

---

## Labs on `generate for` loop

## 8-bit Ripple Carry Adder (RCA)

- An RCA is built by cascading multiple **full adders**.  
- Each full adder adds two input bits along with a carry-in, producing a sum and a carry-out.  
- In an 8-bit RCA, the carry-out of each stage is passed as the carry-in to the next stage.  
- The first adder takes an external carry-in (usually 0), and the final adder produces the overall carry-out.
- The `generate for` loop instantiates one full adder per bit, automatically chaining the carry-out of stage *k* to the carry-in of stage *k+1*.  
- This makes the code **shorter, scalable, and less error-prone**, especially for larger adders (e.g., 16-bit, 32-bit).  
- Only the loop index changes for each stage, while the wiring of sum and carry is handled systematically.  
- If you change the bit-width (say from 8 to 16), only the loop limit needs updating.

**Verilog Code**

RCA (`rca.v`): _8 bit Ripple Carry Adder_

```verilog
module rca (input [7:0] num1 , input [7:0] num2 , output [8:0] sum);
wire [7:0] int_sum;
wire [7:0]int_co;

genvar i;
generate
        for (i = 1 ; i < 8; i=i+1) begin
                fa u_fa_1 (.a(num1[i]),.b(num2[i]),.c(int_co[i-1]),.co(int_co[i]),.sum(int_sum[i]));
        end

endgenerate
fa u_fa_0 (.a(num1[0]),.b(num2[0]),.c(1'b0),.co(int_co[0]),.sum(int_sum[0]));


assign sum[7:0] = int_sum;
assign sum[8] = int_co[7];
endmodule
```

FA (`fa.v`): _1 bit Full Adder_

```verilog 
module fa (input a , input b , input c, output co , output sum);
        assign {co,sum}  = a + b + c ;
endmodule
```

**RTL Simulation**

_Workflow_ :

![workflow12](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow12.png)

_Waveform_ :

![waveform8](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/waveform8.png)


**Synthesis**

_Workflow_ :

![workflow13](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow13.png)

_Netlist Dot File_ :

![dotfile5](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/dotfile5.png)

**Gate-Level Simulation**

_Workflow_ :

![workflow14](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/workflow14.png)

_Waveform_ :

![waveform9](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/main/Day5/Images/waveform9.png)

Thus, the *RTL Simulation*, *Synthesis*, and *Gate-level simulation* of the **8-bit Ripple Carry Adder** have been successfully completed and verified.

---

## Summary

- **If-Else & Case:** Control logic in Verilog; `if-else` handles conditional decisions, while `case` is better for multi-way selection. Missing branches in either can cause **inferred latches**, which hold old values unintentionally.  
- **Inferred Latches:** Occur when outputs are not assigned in all paths (incomplete `if`, `case`, or partial assignments). Always use `else` or `default` to avoid them.  
- **For Loops:** Used inside `always` blocks for simulation-time iteration, simplifying repeated assignments.  
- **Generate For Loops:** Create multiple hardware instances at synthesis time; useful for scalable designs like adders or multiplexers.  
- **Lab Highlights:**  
  - Incomplete `if`/`case` → inferred latches.  
  - `mux_generate` and `demux_generate` show how `for` loops reduce repetitive code.  
  - 8-bit RCA with `generate for` demonstrates scalable hardware replication.  

Overall, careful use of conditional constructs and loops helps write clean, latch-free, and scalable Verilog code.

---