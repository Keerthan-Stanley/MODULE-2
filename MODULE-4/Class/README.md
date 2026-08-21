### Module 4 (Class): GLS, Synthesis-Simulation Mismatch & Verilog Statements

CONTENTS
1. What is GLS? Why GLS?
2. Synthesis-Simulation Mismatch
3. Blocking and Non-Blocking Statements in Verilog
4. Caveats with Blocking Statements

This section covers why we simulate the gate-level netlist (not just the RTL), why RTL and gate-level behavior can sometimes disagree, and the Verilog assignment semantics that are often the root cause of that disagreement.

---

#### 1. What is GLS? Why GLS?
**What it is:** GLS (Gate-Level Simulation) is the process of running the same testbench used for RTL simulation, but against the **synthesized gate-level netlist** instead of the original RTL code.

**How it works:**
- The gate-level netlist (output of Yosys/synthesis) is simulated using the same stimulus as the RTL testbench.
- Often, a **timing-annotated** netlist (using an SDF file) is used so that real gate delays are also modeled, not just functional behavior.

**Why we need it:**
- **Verifies Synthesis Correctness:** Confirms the synthesis tool didn't introduce any functional bugs while converting RTL into gates.
- **Catches Synthesis-Simulation Mismatches:** Some RTL constructs simulate one way at the RTL level but synthesize into hardware that behaves differently — GLS is how these bugs are caught before tape-out.
- **Timing Verification:** With delay annotation, GLS can reveal timing violations (e.g., race conditions, hold-time issues) that a purely functional RTL simulation would never show, since RTL simulation typically assumes zero-delay logic.

<img width="900" height="400" alt="GLS Flow" src="" />

---

#### 2. Synthesis-Simulation Mismatch
**What it is:** A situation where the RTL simulation and the gate-level (post-synthesis) simulation produce **different results** for the same testbench, even though both were derived from the "same" design.

**Why it happens:**
- **Incomplete Sensitivity Lists:** Missing signals in an `always @(...)` sensitivity list can cause RTL simulators to behave differently than synthesized hardware (which is always "fully sensitive" to all its actual inputs).
- **Blocking vs. Non-Blocking Misuse:** Using the wrong assignment type in sequential or combinational blocks can cause the simulator to model a different order of operations than what actual hardware physically does.
- **Non-Synthesizable Constructs:** Constructs like `initial` blocks, delays (`#10`), or certain loop constructs simulate fine in RTL but are either ignored or interpreted differently by the synthesis tool, since real hardware has no concept of "simulation time."
- **Ambiguous/Incomplete Coding:** For example, incomplete `if-else` or `case` statements can infer unintended latches during synthesis — a mismatch the RTL simulation won't reveal, but the gate-level one will.

**Why it matters:** If left uncaught, this class of bug means your chip's *actual physical behavior* differs from what you verified in simulation — a serious, expensive problem to discover after fabrication.

---

#### 3. Blocking and Non-Blocking Statements in Verilog
Verilog provides two types of procedural assignment, and choosing the correct one is critical for both correct simulation and correct synthesis.

- **Blocking Assignment (`=`):**
  - Executes **sequentially**, in the exact order written — the next statement in the block waits ("blocks") until the current one completes.
  - Intended for modeling **combinational logic**.
  - **Example:**
```verilog
    always @(*) begin
        y = a & b;
        z = y | c;   // uses the just-updated value of y
    end
```

- **Non-Blocking Assignment (`<=`):**
  - All right-hand sides are evaluated first, and **all assignments happen simultaneously** at the end of the time step — mimicking how flip-flops physically update on a clock edge.
  - Intended for modeling **sequential (clocked) logic**.
  - **Example:**
```verilog
    always @(posedge clk) begin
        q1 <= d;
        q2 <= q1;   // uses q1's value BEFORE this clock edge, not after
    end
```
    This models a correct 2-stage shift register, because `q2` gets the *old* value of `q1`, exactly like real flip-flops updating in parallel.

---

#### 4. Caveats with Blocking Statements
Using blocking assignments (`=`) in the wrong context is one of the most common causes of synthesis-simulation mismatch.

- **Using Blocking in Sequential Logic:**
```verilog
  always @(posedge clk) begin
      q1 = d;
      q2 = q1;   // BUG: uses the just-updated q1, not the old value
  end
```
  Because blocking assignments execute immediately and sequentially, `q2` incorrectly gets the **new** value of `q1` in the same clock edge — collapsing what should be a 2-stage shift register into effectively a 1-stage delay in simulation. Real hardware (flip-flops) can never behave this way, causing RTL simulation to diverge from the actual synthesized gate-level behavior.

- **Order-Dependent Bugs:** Since blocking statements execute top-to-bottom, simply reordering two blocking statements inside an `always` block can silently change simulation results — a fragile and error-prone pattern for sequential logic.

- **General Rule of Thumb:**
  | Logic Type | Assignment to Use |
  | :--- | :--- |
  | Combinational (`always @(*)`) | Blocking (`=`) |
  | Sequential (`always @(posedge clk)`) | Non-Blocking (`<=`) |

  Mixing these up is one of the top sources of synthesis-simulation mismatches in real RTL designs.
