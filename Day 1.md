## Introductory Components

**Package** - The external casing that protects the chip and enables electrical connection to a circuit board.

**Die** - The actual piece of silicon containing the chip’s integrated circuits.

**Pads** - Metallic terminals on the die that allow electrical signals to enter/leave the chip.

**Core** - The main, central processing area of the chip. 

**Macro** - A pre-designed, reusable block of logic. A common example of macros are memory macros (SRAM, DRAM, etc.)

**Foundry IPs** - Proprietary circuit blocks tied to specific manufacturing processes, only producible at certain foundries.

## Introduction to RISC-V
* Open-source Instruction Set Architecture (ISA) - defines how software communicates with hardware. 
* An ISA defines the rules/instructions that a compiler must follow in order to translate high-level programming (like C) into assembly/machine code. 

## Software to Hardware Flow
1. **Application Software** - The program/app that a user wants to write or run. These programs are human-readable so that its easy to describe complex logic and algorithms (using high-level code).
2. **System Software** - bridges the high-level program to the physical hardware. Includes the **Operating System** (manages tasks like memory allocation and device communication so that the program can run smoothly), the **Compiler** (translates the high-level code to assembly/machine language using the ISA), and the **Assembler** (converts assembly code into actual binary machine code).
3. **Hardware** - The CPU and supporting hardware load the binary and begin execution.

![softwareapp_hardware-flow](https://github.com/user-attachments/assets/44a56fec-9d5f-4267-9448-ee961500cb93)

## System on Chip Design with OpenLANE

**Application Specific IC (ASIC) Digital Design**: 
* HDLs and RTL IPs
* **EDA Tools** - automate key steps like synthesis, floorplanning, placement, etc.
* **Process Design Kit (PDK)** - set of files that describe the fabrication process with design rules, standard cells, I/O libraries, etc. It is an essential interface between chip deisgners and the fab.
* **Fabrication Separation** - fabless companies focus only on design, pure-play foundries handle manufacturing. 

### RTL To GDSII Flow

**RTL (Register Transfer Level)** - describes the logic behavior/data flow and writes the design in a hardware description language (HDL).

**GDSII (Graphic Design System II)** - final physical layout data used by a foundry for mask creation.

1. **Logic Synthesis** - converts the RTL into a gate-level netlist (description of a circuit's components and connections) based on a standard cell library. Static timing analysis (STA) is often used to check if the synthesized netlist meets the predetermined timing constraints.
  
2. **Floorplanning** - Defining the physical layout of major functional blocks (such as macros, standard cell areas, and memories). This phase directly view chip performance, area, power, and routability by primarily focusing on minimizing wire lengths, reducing any sort of signal/timing delays, optimizing power distribution by minimizing congestion (Power Distribution Network (PDN)), and maximizing the utilization of the given chip area. 

3. **Placement** - exact physical locations of components are determined within the floorplan area (previous phase). The placement is meant to minimize wirelength (minimizing delay and congestion in the process), still meet timing/area constraints, and reduce power by shortening interconnects. This begins with **global placement** which is a rough, approximated placement plan, and is followed by **detailed placement** which adjusts cell locations to avoid overlaps and align with their rows.

4. **Clock Tree Synthesis (CTS)** - building and optimizing the clock distribution network. **Main Objectives**: minimize clock skew (difference in arrival times of the clock at different parts of the chip) and minimize clock latency (time it takes for the clock signal to get from the source to components). Buffers and inverters are inserted to distribute the clock signal and various different types of structures can be used depending on the design goals (H-tree, X-tree, mesh, etc.).

5. **Routing** - interconnections between the components are done here. This is done through the usage of multiple metal layers in which each of these layers form physical wiring paths across the chip. This is defined within a Process Design Kit (PDK). 


## OpenLANE Basic Overview

OpenLANE is an open-source toolchain that takes RTL and makes it into a chip layout file ready for manufacturing without any manual steps needed. 
* StriVe - SoC family
* Tuned for the SkyWater 130nm PDK
* Run Modes:
  * Autonomous - fully automated design flow
  * Interactive - manual control for custom tuning

### **Stages of the OpenLANE Flow**

1. **Synthesis Exploration** - tries different logic synthesis strategies and generates reports on timing/area to find the best one
2. **Design Exploration** - evaluates performance across multiple configurations and runs regression tests to catch issues.
3. **Design for Test (DFT)** - adds test-specific logic to help check chip functionality after manufacturing. Scans chains for test access, ATPG to generate test patterns, and fault simulation to preduct possible chip failures.
4. **Physical Implementation (PnR)** - automated placement/routing of components and wires, done using OpenROAD. 
 
![openlane_flow_stages](https://github.com/user-attachments/assets/6d34b96d-cd13-4b5b-87c4-83a0592d4ae7)

**Logic Equivalence Check (LEC)** - makes sure that the netlist still performs the same function after being optimized for things like timing or clock tree insertion.

**Antenna Diodes** - protects the chip during fabrication. Long wires can build up charge that damages gate layers and so the diodes drain that charge. 

## Beginning to use OpenLANE

Begin by opening the virtual machine and then open terminal. 

1. All the files are stored in **Desktop/work/tools**
2. Navigate to **openlane_working_dir**
3. Go into the **pdks**
4. Enter sky130A and open up libs.ref. If you do ls -ltr when in the sky130A directory, you will notice there are two files. Libs.tech contains all of the tool specific files while libs.ref contains all of the foundry related processes.
5. Do **ls -ltr** while in the libs.ref directory. We will be working with sky130_fd_sc_hd.

Here are all of the steps done:
![openlane_ss1](https://github.com/user-attachments/assets/4010ec20-de47-4882-9bb4-bd25fb45f765)

### Design Preparation (Note: all of the instructions will be done from a Mac perspective)

1. Run **./flow.tcl -interactive** in the OpenLANE directory
2. Run **package require openlane 0.9** to load the necessary package. This step must be dont everytime OpenLANE is reopened.
3. Run **prep -design designs/ci/picorv32a** to make a place where Openlane can store the results. 
![openlane_ss4](https://github.com/user-attachments/assets/fbb3ac47-3579-4a2e-8bf9-645757409f24)
4. Then do **run_synthesis** to do synthesis. 
![openlane_ss9](https://github.com/user-attachments/assets/afe3a38f-dbde-4955-
5. Then go check the synthesis reports to calculate the flop ratio.
![openlane_ss11](https://github.com/user-attachments/assets/2e31f980-4a39-4d30-9404-c1e579289b2b)
90f8-2a1a2e141439)
6. Look at the results and calculate the flop ratio.
![openlane_ss12](https://github.com/user-attachments/assets/a419d79c-cb05-4acc-9232-8644b3a89365)

Flop ratio: 1596/9819 = 0.16254201038
DFF % = 0.16254201038 * 100 = 16.254201038 %
