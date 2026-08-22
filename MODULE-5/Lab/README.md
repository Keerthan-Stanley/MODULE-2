### Module 5 (Lab): Conditional Constructs & Loops in Verilog

CONTENTS
1. Lab: If-Case Constructs
2. Lab: Incomplete If / Case (Latch Inference)
3. Lab: For Loop and For-Generate

This section walks through synthesizing the constructs from the class section using Yosys, to directly observe priority logic, latch inference, and structural replication.

---

#### 1. Lab: If-Case Constructs

**Goal:** Synthesize both an `if-else` version and a `case` version of the same 4:1 mux logic, and compare the resulting gate structures.

**Design (`mux_if.v`):**
```verilog
module mux_if(input [1:0] sel, input a, b, c, d, output reg y);
    always @(*) begin
        if (sel == 2'b00) y = a;
        else if (sel == 2'b01) y = b;
        else if (sel == 2'b10) y = c;
        else y = d;
    end
endmodule
```

**Design (`mux_case.v`):**
```verilog
module mux_case(input [1:0] sel, input a, b, c, d, output reg y);
    always @(*) begin
        case (sel)
            2'b00: y = a;
            2'b01: y = b;
            2'b10: y = c;
            default: y = d;
        endcase
    end
endmodule
```

**Steps:**
```bash
yosys
read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog mux_if.v
synth -top mux_if
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
stat
show
```
Repeat for `mux_case.v`.

**What to check:** Compare the `stat` output (cell count/area) of both — the `case` version often synthesizes to a flatter, more balanced mux structure, while `if-else` may show a priority-encoded chain.

---

#### 2. Lab: Incomplete If / Case (Latch Inference)

**Goal:** Deliberately write incomplete conditional logic and confirm Yosys infers a latch.

**Buggy design (`incomplete_if.v`):**
```verilog
module incomplete_if(input sel, input a, output reg y);
    always @(*) begin
        if (sel)
            y = a;
        // no else!
    end
endmodule
```

**Steps:**
```bash
yosys
read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomplete_if.v
synth -top incomplete_if
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
stat
show
```
**What to check:** The `stat` output will show a `$_DLATCH_` (or similarly named latch cell) in the cell count — confirming a latch was inferred, even though this was intended as pure combinational logic. Also run Yosys's synthesis with `synth` and watch for a warning like `Latch inferred for signal y` in the log.

**Fix and re-run:**
```verilog
module incomplete_if_fixed(input sel, input a, output reg y);
    always @(*) begin
        if (sel)
            y = a;
        else
            y = 1'b0;
    end
endmodule
```
Re-synthesize and confirm the latch disappears from `stat`.

---

#### 3. Lab: For Loop and For-Generate

**Goal:** Compare a behavioral `for` loop (bit-summing) against a structural `generate for` (hardware replication), and inspect the netlists.

**Design A — `for` loop (`bit_sum.v`):**
```verilog
module bit_sum(input [7:0] data, output reg [3:0] sum);
    integer i;
    always @(*) begin
        sum = 0;
        for (i = 0; i < 8; i = i + 1)
            sum = sum + data[i];
    end
endmodule
```

**Design B — `generate for` (`adder_array.v`):**
```verilog
module full_adder(input a, b, cin, output sum, cout);
    assign sum = a ^ b ^ cin;
    assign cout = (a & b) | (b & cin) | (a & cin);
endmodule

module adder_array(input [3:0] a, b, input cin, output [3:0] sum, output cout);
    wire [4:0] carry;
    assign carry[0] = cin;
    genvar j;
    generate
        for (j = 0; j < 4; j = j + 1) begin : fa_inst
            full_adder fa (.a(a[j]), .b(b[j]), .cin(carry[j]), .sum(sum[j]), .cout(carry[j+1]));
        end
    endgenerate
    assign cout = carry[4];
endmodule
```

**Steps:**
```bash
yosys
read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog adder_array.v
synth -top adder_array
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
**What to check:** In the `show` schematic, you should see **4 distinct, physically repeated full-adder instances** (`fa_inst[0]` through `fa_inst[3]`), each with identical internal structure — direct visual proof of structural hardware replication via `generate`, as opposed to the single flattened adder-tree logic that the `for` loop in `bit_sum.v` would produce.
