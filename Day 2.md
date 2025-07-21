# Chip Floor Plans + Introduction to Library Cells

## Floorplanning Overview
1. Width of the core and die must be defined. The area of the netlist is essentially the space that all the standard cells take up and you can use it to find the **utilization factor** (Area of the netlist/area of the core). The utilization factor allows you to know how tightly packed the logic is and how easily the tools can route wires without congestion. The ideal ratio is about 60% to leave room for extra logic. **Aspect ratio** (height/width) is also calculated.
![aspect_ratio](https://github.com/user-attachments/assets/3ca69050-09c6-4daa-895f-be04f3b687a2)
2. **Preplaced Cells**: define the location of pre-placed cells. These are reusable blocks of logic that were previosuly created and need to be placed (can't be moved). Standard cells can be broken up and turned into black boxes that can be reused. 
3. **De-Coupling Capacitors**: blocks often switch lots of current and can cause many issues such as voltage drops and noise. Decoupling capacitors hold equivalent voltage to the main supply voltage and so all pre-placed cells get their power supply from the capacitors, leading to a stable outcome.
4. **Power Planning**: There are numerous power source pins with input and output pins being on different sides. When a lot of capacitors discharge at once, it may cause **ground bounce** which is a voltage fluctuation that could cause logic and timing errors, so many power sources are important. A power mesh is used which is a network of horizontal and vertical strips of power and ground. 
5. **Pin Placement**: I/O ports are located on the PADS. The placement of these ports are heavily reliant on the actual design of the cells on the cores. Clock ports are larger because they have to drive the cells continously and by making them larger, it reduces resistance.
![power_planning](https://github.com/user-attachments/assets/fc186057-a58b-462b-bf50-d7bc37a34f6a)
6. **Logical Cell Placement Blockage**: fills in the unused areas by the ports to ensure that the APR doesn't place any cells there.
![cell_placement_blockage](https://github.com/user-attachments/assets/43844175-918c-4a27-8c04-3e2818f0100b)

## OpenLANE Floorplanning

In OpenLANE, do **run_floorplan**.
![openlane_ss13](https://github.com/user-attachments/assets/cbe987f9-b402-4f65-8f41-91947c50acd9)

The results should be in the .def file of **/results/floorplan** of your most recent run.
![openlane_ss17](https://github.com/user-attachments/assets/f9d099b2-9f99-43eb-a442-e757284a6483)

This is what the results should look like. 
![openlane_ss18](https://github.com/user-attachments/assets/66755d4d-af3d-4552-9a03-dbc6041e9aaa)

## Magic Floorplanning

Go to the run file of the floorplan and use the following command in the screenshot to open magic inside this file. 
![openlane_ss19](https://github.com/user-attachments/assets/eca1d90d-7a90-4d22-992d-1cce4ed5b42c)

This is what should appear.
![openlane_ss20](https://github.com/user-attachments/assets/6ec119ac-5a8d-428f-82d3-ef5839af59c3)

**Basic Commands**:
* **s** to select the floorplan
* **v** to center view
* **z** to zoom in. Left click to create the bottom left corner boundary and rick click to create the top right corner boundary of the area you want to zoom in on. 

## Placement + Routing

**Binding the Netlist** - each logic component from the netlist is matched up to a physical cell from a standard cell library. 

**Cell Placement** - the cells are all arranged within the floorplan and all timing-critical elements are placed as close as possible to their connected pins to minimize any signal delay.

**Placement Optimization** - Buffers strengthen signals without changing any logic value by outputting the same value with a higher drive strength (meaning the signal can be pushed across longer wires to more loads without degrading). These buffers help maintain timing. 

Go back to the OpenLANE terminal and run the placement using **run_placement**. Then run the following magic command in the next screenshot.
![openlane_ss22](https://github.com/user-attachments/assets/a89b6a35-0862-43db-910a-b187e19995bb)

The following image will appear.
![openlane_ss23](https://github.com/user-attachments/assets/7d8e8e02-3d07-4fbb-b9dc-a540db8ec3a9)

## Cell Design Flow

**Inputs** - PDK (comes from foundry) includes design rules, SPICE models. Customizable parameters include: cell height/width, supply voltage, and pin locations.

**Design Steps**:
* Circuit design - build the actual logic circuit based on specs. Use SPICE sims to test functions.
* Layout design - arrange all of the transistors and wires using things like Euler paths and stick diagrams.
* Characterization - get performance metrics like timing, power, and noise. First, the foundry model is loaded, the SPICE netlist is loaded, recognize all of the circuit blocks, connect the power and ground, apply test inputs and waveforms, add output capacitance, and run the simulation commands.

**Use tools to generate output .lib files**.

## Key Timing Metrics

**Slew Thresholds**:
* Low = 20% of signal level
* High = 80%
* In/Out transition = 50%

**Propagation Delay**:
* Time for input change to cause output change
* Should always be positive --> if there  is a negative delay then there is an error in thresholds or general circuit design.

**Transition Time**:
* The time for a signal to move between threshold levels.

![slew_thresholds](https://github.com/user-attachments/assets/2fc52452-6d87-4749-9b65-267b7682c9f6)
