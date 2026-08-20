### Module 3: Logic Optimization Techniques

CONTENTS
1. Combinational Logic Optimization
2. Sequential Logic Optimization
3. Sequential Optimization – Unused Outputs

---

#### 1. Combinational Logic Optimization
**What it is:** The process of simplifying purely combinational logic (no clock, no memory) to use the fewest gates possible while preserving the exact same truth table.

Synthesis tools like Yosys apply techniques such as:
- **Constant Propagation:** If an input to a gate is tied to a fixed `0` or `1`, the tool eliminates the gate entirely and propagates the constant forward.

<img width="1858" height="918" alt="Screenshot 2026-08-19 194301" src="https://github.com/user-attachments/assets/631dd07e-8260-47df-a4ca-290ae835d08a" />

  
- **Boolean Logic Minimization:** Reduces redundant logic using techniques similar to K-map simplification (e.g., merging overlapping product terms, removing don't-cares).

<img width="1790" height="949" alt="Screenshot 2026-08-19 194533" src="https://github.com/user-attachments/assets/f2687873-8c78-465b-9173-8b5adc88c14e" />


**Example:**
```verilog
assign y = a ? b : (a ? c : d);
```
Since both branches of the outer condition depend on `a` being true, the tool recognizes `c` is unreachable and simplifies this to:
```verilog
assign y = a ? b : d;
```
This directly reduces the number of gates (fewer muxes) in the final netlist, saving area and power without changing the output for any input combination.

<!-- Add screenshot/waveform here if available -->

---

#### 2. Sequential Logic Optimization
**What it is:** Optimization techniques applied specifically to circuits containing flip-flops (memory elements), where the *state* of the circuit over time matters, not just the instantaneous logic.

Common techniques include:
- **State Optimization:** Removing unreachable or redundant states in an FSM, or merging equivalent states that produce identical outputs for identical future input sequences.
- **Retiming:** Moving flip-flops across combinational logic boundaries (without changing functional behavior) to balance delay between pipeline stages and improve maximum clock frequency.
- **Sequential Cloning (Register Replication):** Duplicating a flip-flop that drives many high-fanout loads, so timing/loading is split across multiple copies instead of one flop struggling to drive everything.

**Example — Retiming:**
If a flip-flop sits *before* a long combinational path, moving it to sit *after* the path (functionally equivalent, same overall latency) can balance the delay between two pipeline stages, allowing a higher clock frequency without altering the circuit's behavior.

<img width="1914" height="1079" alt="Screenshot 2026-08-19 195536" src="https://github.com/user-attachments/assets/5fc7c450-c16f-4f9f-95ab-b93c3a0d9e81" />

<img width="1879" height="1024" alt="Screenshot 2026-08-19 200431" src="https://github.com/user-attachments/assets/05d35419-4658-4782-9f62-dd9477759159" />

---

#### 3. Sequential Optimization – Unused Outputs
**What it is:** A specific case of sequential optimization where the synthesis tool detects that certain flip-flop outputs (or bits of a register) are never actually used anywhere downstream in the design — and removes the associated hardware entirely.

**How it works:**
If you design, for example, a 4-bit counter but only ever use 2 of its output bits in your actual logic (the other 2 bits toggle internally but never connect to anything meaningful), Yosys will:
1. Trace the output bits forward through the design.
2. Determine that 2 of the 4 flip-flops have no path to any primary output or other used logic.
3. Prune those unused flip-flops and their associated feeding logic completely from the final netlist.

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
Here, `full_count[3:2]` never reaches the output port `count`. During synthesis, Yosys recognizes this and **removes those two unused flip-flops and their incrementing logic entirely**, saving area and power — the synthesized circuit behaves as if you had declared only a 2-bit counter, even though you wrote a 4-bit one.

<!-- Add screenshot/waveform here if available -->
