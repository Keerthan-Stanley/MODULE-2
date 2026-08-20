### Module 3 (Lab): Logic Optimization Techniques

CONTENTS
1. Lab: Combinational Logic Optimization
2. Lab: Sequential Logic Optimization
3. Lab: Sequential Optimization – Unused Outputs

This section walks through the practical Yosys flow to observe each optimization technique in action on the SKY130 PDK.

---

#### 1. Lab: Combinational Logic Optimization

**Goal:** Observe how Yosys simplifies redundant combinational logic (e.g. a nested ternary/mux chain) into a minimal gate count.

**Steps:**
```bash
# Launch Yosys
yosys

# Read the liberty file (standard cell library)
read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib

# Read your RTL design
read_verilog comb_opt.v

# Set the top module
synth -top comb_opt

# Run the optimization pass
opt_clean

# Map to standard cells
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib

# View the synthesized schematic
show
```
**What to check:** Compare the `stat` output (cell count) before and after `opt_clean` — you should see fewer mux/logic cells in the final netlist than in the naive RTL.

---

#### 2. Lab: Sequential Logic Optimization

**Goal:** Observe retiming/state optimization on a sequential design (e.g. an FSM or pipelined register chain).

**Steps:**
```bash
yosys

read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog seq_opt.v
synth -top seq_opt

# Sequential optimization pass
opt -full

abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
**What to check:** Use `stat` to compare flip-flop count before/after — equivalent states or redundant flops should collapse into fewer sequential elements.

---

#### 3. Lab: Sequential Optimization – Unused Outputs

**Goal:** Confirm Yosys removes flip-flops whose outputs never reach a primary output.

**Design file (`counter.v`):**
```verilog
module counter(input clk, input rst, output [1:0] count);
    reg [3:0] full_count;
    always @(posedge clk or posedge rst)
        if (rst) full_count <= 0;
        else full_count <= full_count + 1;

    assign count = full_count[1:0]; // only lower 2 bits used
endmodule
```

**Steps:**
```bash
yosys

read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog counter.v
synth -top counter

opt_clean -purge

abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib

# Check final flip-flop count
stat

show
```
**What to check:** Run `stat` and confirm only **2 flip-flops** remain in the final netlist (not 4) — proving `full_count[3:2]` and its logic were pruned since they never reach the `count` output.
