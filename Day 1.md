### RTL To GDSII Flow

**RTL (Register Transfer Level)** - describes the logic behavior/data flow and writes the design in a hardware description language (HDL).

**GDSII (Graphic Design System II)** - final physical layout data used by a foundry for mask creation.

1. **Logic Synthesis** - converts the RTL into a gate-level netlist (description of a circuit's components and connections) based on a standard cell library. Static timing analysis (STA) is often used to check if the synthesized netlist meets the predetermined timing constraints.
  
2. **Floorplanning** - 
