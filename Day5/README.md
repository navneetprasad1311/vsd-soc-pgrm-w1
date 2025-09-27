# Day 5 - Optimization in Synthesis




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

![workflow1]()

_Waveform_:

![waveform1]()

**Synthesis Netlist Check**

_Workflow_ :

![workflow2]()

_Netlist Dot File_ :

![dotfile1]()

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

![workflow3]()

_Waveform_ :

![waveform2]()


**Synthesis Netlist Check**

_Workflow_ :

![workflow4]()

_Netlist Dot File_ :

![dotfile2]()

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

![workflow5]()

_Waveform_ :

![waveform3]()

**Synthesis Netlist Check**

_Workflow_ :

![workflow6]()

_Netlist Dot FIle_ :

![dotfile3]()

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

![workflow7]()

_Waveform_ :

![waveform4]()


**Post Synthesis Check**

_Synthesis Workflow_ :

![workflow8]()

_Netlist Dot File_ :

![dotfile4]()

_Simulation Workflow_:

![workflow9]()

_GLS Waveform_ :

![waveform5]()

- For `sel = 2'b00`, `y = i0`;  
- For `sel = 2'b01`, `y = i1`;  
- For `sel = 2'b10`, `y = i2`; 
- `sel = 2'b11` is commented out, so for this value `y` is **not assigned**, causing a **latch** to be inferred in _RTL Simulation_.

But this inferred latch is corrected in simulation as seen in the **GLS Waveform**

---