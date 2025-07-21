## IO Placer Revision

The IO Mode is set to **random equidistant** as a default, meaning all of the cells are spaced uniformly. 

To change, run this command: **set ::env(FP_IO_MODE) 2**, then run the floorplan again.

This command will set the mode of the IO pins to 2 which allows them to overlap and such. In OpenLANE, mode 2 causes the pins to be designated based on a Hungarian Matching Algorithm (combinatorial optimization methods). 

## SPICE Deck Creation for CMOS Inverter

**SPICE Deck**: A text file that contains a netlist (list of circuit components and how they're connected), the TAPS points for the outputs, and the inputs that needed to be provided. 
* Values: length and width of both types (PMOS and NMOS) of transistors. Ideally, the PMOS should be 2-3 times wider than the NMOS to balance their drive strengths and achieve symmetrical switching behavior.

## SPICE Simulation Lab for CMOS Inverter

* **.op** - starts the simulation with a voltage sweep
* **Model file** holes process-specific data for 0.25 micrometer NMOS/PMOS
![spice_deck](https://github.com/user-attachments/assets/8106c80b-f20f-44f1-a494-18db2f355ff9)

### SPICE Simulation:
* Open NGSPICE
* Source the circuit file with **source <filename>**
* Use **run** to run the simulation
* Use **setplot** to choose the simulation type
* Use **display** to view nodes and plot out on a graph.
![spice_display](https://github.com/user-attachments/assets/08ebed3e-28f2-4d2c-b176-54c52e68fa4b)

## Switching Threshold Vm

**Switching Threshold**:
* Vin = Vout --> point where both NMOS and PMOS are at saturation.
* When these are turned on, it causes short-circuit current --> direct Vdd to GND leakage
  
**Propagation Delay**: time it takes for the output to switch after the input switches.

## Static and Dynamic Simulation of CMOS Inverter

**DC Transfer Analysis**
* **.dc Vin 0 2.5 0.05** - looks for the input gate voltage for values from 0-2.5V at increments of 0.05V
* DC Transfer Analysis done to find Vm
* Transient analysis done to find propagation delay of when a pulse is applied to the CMOS.
![spice_dynamic_sim](https://github.com/user-attachments/assets/66e3d379-47d6-4b40-8837-4c0a514bcfda)

**Steps to do the transient analysis:**
1. **Source** file again
2. **run** again
3. **setplot** again
4. **tran2** after the question mark
5. **display** to see options
6. **plot out vs time in** for replot
![transient_analysis_spice](https://github.com/user-attachments/assets/7b0d104a-bb24-4a14-8242-c5d06ebabdae)

## Lab Steps to GitClone VSDSTD Cell Design
1. On the openlane directory (/openlane), do **git clone https://github.com/nickson-jose/vsdstdcelldesign**
2. Make a copy of sky130A.tech file to the vsdstd cell design directory using the **cp** command.
3. Do **magic -T sky130A.tech sky130_inv.mag &** in order to get a visual.
![inverter_visual](https://github.com/user-attachments/assets/9371a38c-05d7-41c4-828e-a46691b085c7)
## CMOS Fabrication Process
1. **Select a substrate** - where the design will be fabricated on. P doped silicon substrate is most commonly used but the substrate type can vary based on the intended application and design requirements.
2. **Create an Active Region** - grow 40nm of silicon dioxide on the substrate as an insulator. Then put 80nm of silicon nitride onto the insulator and then 1 micrometer of photoresist on top. 
![active_region1](https://github.com/user-attachments/assets/b93f1955-f197-471d-b951-5349c9ca0182)
Then put a mask-1 layer on top of the photoresist in locations where you don't want to get hit by the UV light. 
![active_region2](https://github.com/user-attachments/assets/2ba67a2f-3ffe-4f40-8646-472991052afc)
Then the UV light is applied to remove the layers on the unmasked areas. Then remove mask-1, etch off the open silicon nitride, and rmeove the photoresist. By putting the chip in an oxdiation furnace, the oxide will grow in the other areas. Finally, remove the silicone nitride layer using phosporic acid to have the remains of p-substrate and silicon dioxide.
3. **Formation of N-well and P-well** - place another layer of photoresist on the chip then use another mask to only cover half of the substrate. Then put boron on top of everything so that only the uncovered part of the substrate has a P-well. 
![cmos_p-well](https://github.com/user-attachments/assets/f3ec425f-2382-42d8-acd1-95bc483074b9)
Then do the same exact process, but cover the P-well and isolate the other side, implant it with Phosphorus ions to create an N-well on the other side. 
![cmos_n-well](https://github.com/user-attachments/assets/5cd56f28-8552-41fd-852e-717841e67993)
Finally, the entire thing is placed inside a furnace, diffusing both wells. 
![cmos_furnace](https://github.com/user-attachments/assets/8b334aab-bbb6-4921-98f8-46ea88787b8a)
4. **Formation of Gate Terminal** - the gate terminal is where the threshold voltage is controlled.
![threshold_voltage_math](https://github.com/user-attachments/assets/e61c89d3-f6ac-4dbb-a66d-b33f7076a85b)
* Leave photoresist layers on the areas to be protected, then put mask-4 on top.
* Apply UV light.
* Then dope ions into each half of the substrate, but use much lower energy ions for each respective well. In order to fix the oxide that is damaged by the implantation of ions, the extra silicon dioxide is removed using hydrofluoric acid, then regrown using silicon dioxide on the p-substrate to control the next occuring oxide thickness.
* Then grow a polysilicon layer.
* Mask-6 is added and etch using photolithography.
* Mask-6 is etched off to form the gate terminal.
![cmos_formation_of_gate](https://github.com/user-attachments/assets/d9ca6a94-7c1d-4c5d-b543-85255b92d737)
5. **Lightly Doped Drain (LDD) Formation**:
* Mask 7 is used for NMOS while mask 8 is used for PMOS.
* Heavily doped impurities of N+ for NMOS and P+ for PMOS are added for the source and drain. The lightly doped impurity is only needed to help space the source and drain.
* Silicon dioxide protects the lightly doped regions. 
![cmos_LDD](https://github.com/user-attachments/assets/3f6dd1af-2fbf-4064-ae9d-44a6aaf5299a)
6. **Source and Drain Formation**:
* Thin screen of silicon oxide avoids channeling (when implanatations dig too deep into substrates)
* Mask-9 for N+ implantation and Mask-10 for P+ implantation
![cmos_source_drain](https://github.com/user-attachments/assets/6eea173e-57b4-4288-a954-0957811431d8)
* High temperature annealing by putting it into a high-temperature furnace. 
![cmos_source_drain_furnace](https://github.com/user-attachments/assets/fa67db50-f767-4761-9f46-f63d5a410481)
7. **Local Interconnect Formation**:
* Thin silicon oxide is removed to open up the source, drain, and gate.
* Use titanium for low resistance.
* Titatium diselenide is used as the local interconnects and heat it in an N2 ambient.
* Mask-11 is formed and titanium nitride is etched off using RCA Cleaning.
![cmos_rca_cleaning](https://github.com/user-attachments/assets/3c1b7810-d7ad-4d2f-b164-e5efbf7c0934)
* Removing the rest of photoresist leaves you with titanium nitride local interconnects.
8. **Higher Level Metal Formation**:
* Leave silicon dioxide doped with boron or phosphorus on the wafer.
* Surface is polished with chemical mechanical polishing (CMP).
* Photolithography creates contact holes.
![cmos_contact_holes](https://github.com/user-attachments/assets/59bee8a6-2011-44c3-8086-e599be18415f)
* Mask-12 = first contact holes
* Mask-13 = first Aluminum contact layer
* Mask-14 = second contact holes
* Mask-15 = second Alumnium contact layer
* Mask-16 = making contact to topmost layer
![cmos_final_masks](https://github.com/user-attachments/assets/efcf6db8-e37c-4b9e-b80b-dcd3b646b437)
## Lab Introduction to Sky130 Basic Layers Layout and LEF Using Inverter
* Use **what** in tkcon to get specific details on things.
* Use **s** to see what components a component is connected to.
## Lab Steps to Create STD Cell Layout and Extract SPICE Netlist
**SPICE extraction of custom inverter layout**:
* **extract all** - will extract to the location of the .inv file
* **ext2 spice cthres 0 rthres 0** - extracts parasitic info
* **ext2spice** - makes the SPICE file
![tkcon1](https://github.com/user-attachments/assets/80daaddf-64fd-48e3-a537-0003a5df82ba)
Go into the terminal directory and run **vim sky130_inv.spice**
![spice_director](https://github.com/user-attachments/assets/84a4a3e5-1427-45aa-97cd-4b31b1aec7e1)
![spice_netlist](https://github.com/user-attachments/assets/5a9d893b-5546-45e7-92b3-99c1cd5cb807)
## Sky130 Tech File Labs
### Create Final SPICE Deck
Modify the **sky130_inv.ext** file as shown below.
![sky130invext_modified](https://github.com/user-attachments/assets/d385ac22-96c8-4dbb-adaf-b58bb98dd408)

Start up NGSPICE and load the SPICE file for simulation:
![ngspice_load](https://github.com/user-attachments/assets/eb8033b5-954e-4c12-b2b0-b8eae073940b)
































