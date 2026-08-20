### Module 3 (Class): Logic Optimization Techniques

CONTENTS
1. Combinational Logic Optimization
2. Sequential Logic Optimization
3. Sequential Optimization – Unused Outputs

This section covers the theory behind how synthesis tools simplify combinational and sequential logic to reduce area, power, and unnecessary hardware — without altering functional behavior.

---

#### 1. Combinational Logic Optimization
**What it is:** The process of simplifying purely combinational logic (no clock, no memory) to use the fewest gates possible while preserving the exact same truth table.

Synthesis tools like Yosys apply:
- **Constant Propagation:** If an input to a gate is tied to a fixed `0` or `1`, the gate is eliminated and the constant is propagated forward.
- **Boolean Logic Minimization:** Reduces redundant logic similar to K-map simplification — merging overlapping product terms, removing don't-cares.

**Example:**
```verilog
assign y = a ? b : (a ? c : d);
```
Since both branches of the outer condition depend on `a` being true, `c` is unreachable. The tool simplifies this to:
```verilog
assign y = a ? b : d;
```
Fewer gates (muxes) in the final netlist → less area, less power, identical truth table.

---

#### 2. Sequential Logic Optimization
**What it is:** Optimization applied to circuits containing flip-flops (memory elements), where the *state* of the circuit over time matters, not just instantaneous logic.

Common techniques:
- **State Optimization:** Removing unreachable states in an FSM, or merging equivalent states that produce identical outputs for identical future input sequences.
- **Retiming:** Moving flip-flops across combinational logic boundaries (without changing functional behavior) to balance delay between pipeline stages and improve max clock frequency.
- **Sequential Cloning (Register Replication):** Duplicating a flip-flop with high fanout so loading is split across multiple copies instead of one flop driving everything.

**Example — Retiming:**
A flip-flop sitting *before* a long combinational path can be moved to sit *after* it (functionally equivalent, same overall latency), balancing delay between pipeline stages and allowing a higher clock frequency.

---

#### 3. Sequential Optimization – Unused Outputs
**What it is:** A specific case where the synthesis tool detects that certain flip-flop outputs (or register bits) are never used anywhere downstream, and removes that hardware entirely.

**How it works:**
1. The tool traces each output bit forward through the design.
2. If a flip-flop has no path to any primary output or other used logic, it's marked unused.
3. That flip-flop and its feeding logic are pruned from the final netlist.

**Example:**
```verilog
module counter(input clk, input rst, output [1:0] count);
    reg [3:0] full_count;
    always @(posedge clk or posedge rst)
        if (rst) full_count <= 0;
        else full_count <= full_count + 1;

    assign count = full_count[1:0]; // only lower 2 bits used
endmodule
```
`full_count[3:2]` never reaches the output port `count`. Yosys removes those two unused flip-flops and their incrementing logic entirely — the synthesized circuit behaves as if only a 2-bit counter had been declared, saving area and power.
