## Introductory Components

**Package** - The external casing that protects the chip and enables electrical connection to a circuit board.

**Die** - The actual piece of silicon containing the chip’s integrated circuits.

**Pads** - Metallic terminals on the die that allow electrical signals to enter/leave the chip.

**Core** - The main, central processing area of the chip. 

**Macro** - A pre-designed, reusable block of logic. A common example of macros are memory macros (SRAM, DRAM, etc.)

**Foundry IPs** - Proprietary circuit blocks tied to specific manufacturing processes, only producible at certain foundries.

## Introduction to RISC-V
* Hello
 * Hello

## RTL To GDSII Flow

**RTL (Register Transfer Level)** - describes the logic behavior/data flow and writes the design in a hardware description language (HDL).

**GDSII (Graphic Design System II)** - final physical layout data used by a foundry for mask creation.

1. **Logic Synthesis** - converts the RTL into a gate-level netlist (description of a circuit's components and connections) based on a standard cell library. Static timing analysis (STA) is often used to check if the synthesized netlist meets the predetermined timing constraints.
  
2. **Floorplanning** - Defining the physical layout of major functional blocks (such as macros, standard cell areas, and memories). This phase directly view chip performance, area, power, and routability by primarily focusing on minimizing wire lengths, reducing any sort of signal/timing delays, optimizing power distribution by minimizing congestion (Power Distribution Network (PDN)), and maximizing the utilization of the given chip area. 

3. **Placement** - exact physical locations of components are determined within the floorplan area (previous phase). The placement is meant to minimize wirelength (minimizing delay and congestion in the process), still meet timing/area constraints, and reduce power by shortening interconnects. This begins with **global placement** which is a rough, approximated placement plan, and is followed by **detailed placement** which adjusts cell locations to avoid overlaps and align with their rows.

4. **Clock Tree Synthesis (CTS)** - building and optimizing the clock distribution network. **Main Objectives**: minimize clock skew (difference in arrival times of the clock at different parts of the chip) and minimize clock latency (time it takes for the clock signal to get from the source to components). Buffers and inverters are inserted to distribute the clock signal and various different types of structures can be used depending on the design goals (H-tree, X-tree, mesh, etc.).

5. **Routing** - interconnections between the components are done here. This is done through the usage of multiple metal layers in which each of these layers form physical wiring paths across the chip. This is defined within a Process Design Kit (PDK). 


## OpenLANE Commands + Navigation
