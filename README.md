## Module 1: Core Concepts & Tools
This module lays the groundwork for the RTL design flow — covering how a design is simulated and verified before it's built, and how it moves from behavioral Verilog to a synthesizable gate-level netlist using open-source tools like Icarus Verilog and Yosys.

- What are design, testbench, and simulator?
- What is Icarus Verilog?
- The "Good Mux" Lab
- RTL vs. Gate-Level Netlist
- What is Synthesis?
- What is Yosys?
- What can we do with Yosys?

## Module 2: Timing Libraries and Synthesis Strategies
This module dives into how synthesis tools use standard cell libraries to make real timing, area, and power trade-offs, and compares hierarchical vs. flat synthesis strategies along with the role of flip-flops in sequential design.

- What does a `.lib` contain?
- Application of different "flavours" of standard cells
- What is `timing.lib`?
- What is hierarchical synthesis?
- What is flat synthesis?
- Hierarchical vs. Flat Synthesis
- Which one is preferred when?
- Why do we need Flip-Flops (Flops)?
- Asynchronous vs. Synchronous Set/Reset
- Hardware-efficient math: Multiplying by 2 and 8
