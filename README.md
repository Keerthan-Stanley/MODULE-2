##  Core Concepts & Tools

CONTENTS
1. What are design, testbench, and simulator?
2. What is Icarus Verilog?
3. The "Good Mux" Lab
4. What is Synthesis?
5. RTL vs. Gate-Level Netlist
6. What is Yosys?
7. What can we do with Yosys?

This section breaks down the foundational concepts, files, and tools used throughout this chip design program.

### 1. Design, Testbench, and Simulator
To verify that a digital circuit works before manufacturing it, we rely on a simulation flow consisting of three main parts:
*   **Design (DUT - Design Under Test):** This is the actual Verilog RTL code that describes the logic and functionality of the hardware circuit we want to build.
*   **Testbench:** A separate Verilog file used to test the design. It is not meant to be synthesized into hardware. Instead, it generates input signals (stimulus), feeds them into the design, and monitors the outputs to ensure the design behaves exactly as expected.
*   **Simulator:** A software tool that takes the Design and Testbench, processes the inputs over time, and models how the hardware will behave. It outputs a waveform file (usually `.vcd`) so we can visually check signal transitions.
   
*   <img width="1111" height="640" alt="Screenshot 2026-08-08 233200" src="https://github.com/user-attachments/assets/d41ac6e1-273d-45ca-b743-e184a5cf7b61" />


### 2. Icarus Verilog (iverilog)
`iverilog` is a free, open-source Verilog simulation tool. In our workflow, it acts as the compiler. It takes the Verilog design file and the testbench file, compiles them together, and generates an executable binary. When this binary is run, it produces the Value Change Dump (`.vcd`) file, which we then visualize using **GTKWave**.

<img width="1250" height="552" alt="Screenshot 2026-08-08 233218" src="https://github.com/user-attachments/assets/2f415f00-2e5f-43f4-810e-f563984f5a1f" />


### 3. The "Good Mux" Lab
The `good_mux.v` (a standard 2:1 Multiplexer) serves as our primary introductory lab exercise. It is used to demonstrate the complete end-to-end flow: 
1. Writing the RTL behavior.
2. Writing a testbench to toggle the select lines and inputs.
3. Simulating it with `iverilog` to verify the waveform.
4. Passing it to the synthesis tool to see how a behavioral multiplexer is converted into standard logic gates.

<img width="552" height="173" alt="Screenshot 2026-08-08 194558" src="https://github.com/user-attachments/assets/0cd0bc1f-797a-46c5-aa8d-db8fe9cdcfd5" />


### 4. RTL vs. Gate-Level Netlist
Throughout the design process, the code transforms from behavioral abstraction to physical implementation. 

| Feature | RTL (Register Transfer Level) | Gate-Level Netlist |
| :--- | :--- | :--- |
| **What is it?** | Human-readable Verilog code describing *what* the circuit should do (behavior & data flow). | A structural text file describing *how* the circuit is built using actual standard cells (AND, OR, Flops). |
| **Technology** | Technology-independent. The same RTL can be used for any manufacturing process. | Technology-dependent. It is mapped to a specific library (e.g., SkyWater 130nm PDK). |
| **Creation** | Written by the RTL Designer. | Generated automatically by the Synthesis tool. |
| **Readability** | High (uses `if-else`, `always` blocks, `assign`). | Low (looks like a massive list of interconnected logic gate instances). |

### 5. What is synthesis?
In digital design, synthesis (specifically logic synthesis) is the automated process of translating abstract, human-readable Register Transfer Level (RTL) code into a structural, physical representation called a gate-level netlist.

Think of it like a software compiler: just as a compiler translates C++ code into machine code (1s and 0s) that a processor can execute, a synthesis tool translates Verilog code into a network of actual logic gates that can be physically manufactured on a silicon chip.

<img width="1390" height="882" alt="Screenshot 2026-08-08 210735" src="https://github.com/user-attachments/assets/3d1947ab-d789-4ffc-be7a-f5d52b70c306" />


### 6. What is Yosys?
**Yosys** is a robust, open-source framework for Verilog RTL synthesis. If `iverilog` is used to *test* the logic, Yosys is used to *build* the logic. It acts as the bridge between abstract behavioral code and physical hardware gates.

<img width="1270" height="670" alt="Screenshot 2026-08-08 233308" src="https://github.com/user-attachments/assets/2d3a09bc-63a2-4c42-a475-53c1f4b5263a" />


### 7. What can we do with Yosys?
Yosys performs several critical tasks in the VLSI frontend flow:
*   **RTL Parsing:** It reads and interprets the Verilog RTL code.
*   **Logic Optimization:** It simplifies Boolean expressions, removes redundant logic, and optimizes the design to save area and power.
*   **Technology Mapping:** It takes the optimized generic logic and maps it directly to the physical standard cells provided in a Process Design Kit (like the `sky130_fd_sc_hd` library).
*   **Netlist Generation:** It outputs the final Gate-Level Netlist (`.v` file) which is then passed to the physical design (layout) team.
*   **Visualizing Logic:** It can generate graphical schematics of the synthesized logic using the `show` command, helping designers visually inspect how their code was mapped to hardware.
