# Day 1: Introduction to Verilog RTL Design and Synthesis

## Introduction to Simulation

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

![Simulation Flow](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/008a514609306cd1d7bc3812fa1ff36b4c96e25a/Day1/Images/Simflow.png)  

![Example](https://github.com/navneetprasad1311/vsd-soc-pgrm-w1/blob/008a514609306cd1d7bc3812fa1ff36b4c96e25a/Day1/Images/Example.png)

---

## Lab Simulation

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

## Guide for Simulation

### Step 1: Compile the Verilog Code

Use **Icarus Verilog** to compile the design and testbench:
```bash
iverilog -o output_file.vvp design.v testbench.v  
```
> [!Note]
> Using the `-o` option lets us choose where the compiled output (`.vvp`) is stored, keeping the workflow organized.

---

### Step 2: Run the Simulation

Execute the compiled file with:
```bash
vvp output_file.vvp  
```
This generates a `.vcd` file (Value Change Dump) that records all signal transitions, required for viewing waveforms in GTKWave

---

### Step 3: View Waveforms

Open the waveform file in GTKWave:
```bash
gtkwave testbench.vcd  
```
> [!TIP]
> You can add signals to the waveform viewer by dragging them from the left panel in GTKWave.

This allows us to visualize the functional behavior of the Verilog code and verify its correctness.

---


