### Module 5 (Class): Conditional Constructs & Loops in Verilog

CONTENTS
1. If-Case Constructs
2. Incomplete If / Case
3. For Loop and For-Generate

This section covers how conditional logic and iteration are expressed in Verilog, and the coding pitfalls that cause synthesis tools to infer unintended hardware (like latches) if these constructs are written carelessly.

---

#### 1. If-Case Constructs
**What they are:** `if-else` and `case` are the two primary ways to describe conditional behavior in Verilog. Both are used inside procedural blocks (`always`), but they synthesize differently and are suited to different design intents.

- **`if-else`:**
  - Naturally implies **priority logic** — the first matching condition takes precedence over the rest.
  - Synthesizes into a chain of priority-encoded multiplexers.
  - **Example:**
```verilog
    always @(*) begin
        if (sel == 2'b00)
            y = a;
        else if (sel == 2'b01)
            y = b;
        else
            y = c;
    end
```

- **`case`:**
  - Naturally implies **parallel selection** — all branches are evaluated as independent, equally-weighted options (no implicit priority).
  - Synthesizes into a more balanced multiplexer structure, often more efficient in area/timing for pure select-based logic.
  - **Example:**
```verilog
    always @(*) begin
        case (sel)
            2'b00: y = a;
            2'b01: y = b;
            2'b10: y = c;
            default: y = d;
        endcase
    end
```

**When to use which:** Use `if-else` when conditions genuinely have priority (e.g., interrupt handling, exception logic). Use `case` when selecting between mutually exclusive, equally-weighted options (e.g., mux logic, opcode decoding) — it's usually clearer and can synthesize to more optimal hardware.

---

#### 2. Incomplete If / Case
**What it is:** A coding mistake where not every possible condition or case value is explicitly assigned an output — causing the synthesis tool to infer a **latch** to "hold" the output's previous value when no branch matches.

**Why this happens:** In combinational logic, every output must have a defined value for *every* possible input combination. If your code doesn't specify what happens in some scenario, the only way real hardware can behave consistently with that "gap" is by remembering ("holding") the last value — which is exactly what a latch does. This is almost always **unintended** and a bug, since combinational logic is supposed to have no memory.

**Incomplete `if` — Example:**
```verilog
always @(*) begin
    if (sel)
        y = a;
    // BUG: no 'else' — what should y be when sel = 0?
end
```
Since `y` has no assignment when `sel` is `0`, the synthesis tool infers a latch that holds `y`'s previous value in that case.

**Incomplete `case` — Example:**
```verilog
always @(*) begin
    case (sel)
        2'b00: y = a;
        2'b01: y = b;
        // BUG: no case for 2'b10, 2'b11, and no 'default'
    endcase
end
```
Same problem — `sel = 2'b10` or `2'b11` leaves `y` unassigned, inferring a latch.

**The fix:** Always include a final `else` for `if-else` chains, and always include a `default` case for `case` statements, even if it just reassigns a safe fallback value.

---

#### 3. For Loop and For-Generate
Verilog has two constructs that look similar (`for`) but serve completely different purposes, and mixing them up is a common source of confusion for beginners.

- **`for` loop (inside procedural blocks):**
  - Used inside an `always` or `initial` block.
  - Does **not** create multiple hardware instances — it's unrolled at synthesis/simulation time to describe repetitive *behavior*, typically for operating on parts of a signal (e.g., a vector) using the same logic.
  - **Example — summing bits of a vector:**
```verilog
    integer i;
    always @(*) begin
        sum = 0;
        for (i = 0; i < 8; i = i + 1)
            sum = sum + data[i];
    end
```
    This describes one adder tree operating on 8 bits — not 8 separate hardware blocks.

- **`generate for` (structural/instance replication):**
  - Used **outside** procedural blocks, at the module level.
  - Used to **instantiate multiple copies of hardware** (modules, gates, or logic) at compile time — a true structural replication.
  - Requires a `genvar` instead of a regular `integer`.
  - **Example — instantiating 4 full adders:**
```verilog
    genvar j;
    generate
        for (j = 0; j < 4; j = j + 1) begin : adder_inst
            full_adder fa (.a(a[j]), .b(b[j]), .cin(carry[j]), .sum(sum[j]), .cout(carry[j+1]));
        end
    endgenerate
```
    This creates **4 actual, physically distinct full-adder instances** wired together — real repeated hardware, not a behavioral loop.

**Key distinction:**
| Construct | Where used | What it produces |
| :--- | :--- | :--- |
| `for` loop | Inside `always`/`initial` | Repetitive *behavior* on existing signals (no new hardware instances) |
| `generate for` | Module-level, outside procedural blocks | Multiple *physical instances* of hardware (structural replication) |
