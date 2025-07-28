## Lab Steps to Convert Grid Info to Track Info
* OpenLANE is a PnR tool (no logic parts of .mag info is used)
* .lef files need to be extracted
* Tracks.info file contains specifications for the metal routes

![tracks-info](https://github.com/user-attachments/assets/dea0c964-a11c-4ba9-ba23-76d376d74674)

The format for each line is **(name) (axis) (offset) (track pitch)**

First guideline for standard cell layout:
* Input and output ports must be on vertical and horizontal track intersections

Second guideline for standard cell layout:
* Width must be on the odd multiples of the x (horizontal) pitch, while the length must be on the odd multiples of the y (vertical) pitch.

**Press G** to enable grid and type **grid 0.46um 0.34um 0.23um 0.17um** in the tkcon window. 

![grid-tkcon](https://github.com/user-attachments/assets/d9f0c05b-268c-4f35-83c2-54925977e78f)

The first guideline is satsified as the pins now lie on the intersections. The second guideline is also satisfied as the width and height is both odd. 

## Lab Steps to Convert Magic Layout to STD Cell LEF

Lef files contain the physical layout and standard cell library of components used in the design. The information is very detailed and usually spans across all information about the specific physical characteristics of cells in the library. 

Refer to here for specific info on setting the port definitions: https://github.com/nickson-jose/vsdstdcelldesign#create-port-definition

Use **lef write** in the tkcon terminal to make a new .lef file with a file name of your choice (in my case, its called "sky130_vsdinv.lef")

![vsdinv-lef](https://github.com/user-attachments/assets/768112cf-54f9-4b89-8607-f0dd144c0e79)

Here is what the lef file should look like. 

![pins_check](https://github.com/user-attachments/assets/bf0dcf71-376a-4414-9996-2eef4db84a5b)

The number that you set for the ports (during the defining stage) will each be shown in the lef file individually. 

## Introduction of Timing Libs and Steps to Include New Cell in Synthesis

Copy the lef file created in the previous step into the picorv32a/src directory. 

Go to the **libs** directory in vsdstdcelldesign. 

Open up the **sky130_fd_sc_hd__fast.lib** file. This libtery file contains all of the timing and power characteristics for the cells. The "slow", "typical", and "fast" relates to different voltages and temperatures. 

Now copy all of the liberty files from the vsdstdcelldesign **libs** directory into the src directory like such: 

![copy-libfiles](https://github.com/user-attachments/assets/e1a8abeb-5620-4339-b4c7-bf243cbbd9f1)

Make sure your config file looks like this: 

![config-tcl](https://github.com/user-attachments/assets/b75757a8-d4a8-4698-9a79-420f17881a18)

Then go to the OpenLANE terminal and write **prep -design designs/ci/picorv32a -tag (run name) -overwrite**

Next, run **set lefs [glob $::env(DESIGN_DIR)/src/*.lef]** and then **add_lefs -src $lefs**.

![pre-run-synthesis](https://github.com/user-attachments/assets/f8eb5f8b-a037-4bef-a4d8-d72eadd77f04)

Finally, **run_synthesis** and make sure that sky130_vsdinv is written in the design. 



## Introduction to Delay Tables

The enable pin heavily affects the clk in logic gates. In an AND gate, if the enable pin is 1, the clk will be propagated to the rest of the circuit (this is the only case). In an OR gate, if the enable pin is 0, the clk will be propagated to the rest of the circuit (this is the only case). This is advantageous for clock gating (a common power-saving technique) because if you gate the clock, then you can prevent unnecessary clocking of modules when they are not currently being used. 

![buffer-cts](https://github.com/user-attachments/assets/ca79ff10-3797-45c8-a59f-8a7469e56b41)

In the past, we've made generalized assumptions about clock trees present in the image above. Ideally, buffers on different levels should have differing values for capacitance and load because as long as the buffers on the same level have the same characteristics, the delay will remain constant. In reality, the capacitance and load varies and the input transition varies. 

## Delay Table Usage Part 1

![power-aware-cts](https://github.com/user-attachments/assets/27b4f7e6-e6a5-4586-8f30-f4231540f932)


* Delay tables help visualize this by being a two-dimensional table with one axis representing input transition and the other axis representing load output. 
* The intersections of these axises are the calculated delay. Every time of gate will have their own delay table (AND gate, OR gate, XOR gate, etc.).
* The PMOS and NMOS sizes will be corresponding to the label (causing a varying resistance and varying RC constant).
* Each and every size of a buffer type will have its own exclusive delay table.
* It is possible to build an equation from a given set of data in a delay table and use it to estimate the rest of the values in the table.

## Delay Table Usage Part 2

![power-aware-cts](https://github.com/user-attachments/assets/27b4f7e6-e6a5-4586-8f30-f4231540f932)

* In the image above, since the buffers in level 2 have the same input transition times, capacitative loads, and sizes, they have equal delays.
* The skew value is calculated by looking at the difference of delay of buffers at the same level.
* Negative skew can be extremely bad when designs are scaled up to large sizes with large timing conflicts (as even a small negative skew can stack up) --> must be caught in early clock tree creation

## Lab Steps to Configure Synthesis Settings to Fix Slack and Include VSDINV

**ADD SCREENSHOTS LATER**

1. Look at the synthesis results and check the strategy by doing **echo $::env(SYNTH_STRATEGY)** in the Openlane terminal. By setting the synth strategy to 1 (through **set ::env(SYNTH_STRATEGY) 1**, the area should increase but the timing should also improve. SYNTH_STRATEGY value can range from 0 to 3 (inclusive) and as the value gets higher, timing is further optimized but area increases the most as well.
2. Check SYNTH_BUFFERING and make sure it is 1 (0 means disabled, 1 means enabled). If it is 0, use the set command to set it to 1. When SYNTH_BUFFERING is 1, that means that the buffer will be used to reduce delay in wires.
3. Make sure SYNTH_SIZING is 1 and set it to 1 if necessary. When SYNTH_SIZING is 1, that means that all cell sizes will be modified to further optimize timing.
4. Make sure echoing SYNTH_DRIVING_CELL gives the same cell that has higher drive strength.
5. **run_synthesis** in the Openlane terminal once again and slack should be reduced.

The chip area should naturally increase as the SYNTH_STRATEGY was changed from 0 to 1. Slack should reduce though (check at the near bottom of the produced netlist). 

Now **run_floorplan** in the Openlane terminal. If it comes up with some errors, run these instead:

**init_floorplan**
**place_io**
**global_placement_or**
**detailed_placement**
**tap_decap_or**
**detailed_placement**

Then your results should be found here:

**ADD SCREENSHOT OF PWD**

Then in a normal terminal (not Openlane environment), run **magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def** which should open up this:

**ADD SCREENSHOT HERE**

If we zoom in, we can see our custom cell:

**ADD ZOOMED IN SS HERE**

The overlap is to ensure the power and ground rails are shared between cells.

We can also run **expand** in the tkcon window to more accurately see how the power/ground overlap the power/ground pins of touching cells. 

**ADD SCREENSHOT OF EXPANDED MAGIC**

## Setup Timing Analysis and Introduction to Flip-Flop Setup Time

Initially, take an ideal clock where the clock tree is not yet built. We'll do timing analysis on this ideal clock to understand basic parameters and later on, the same will be done on a real clock. 

We'll first work with a single clock with these specifications: **clock frequency (F) = 1GHz, clock period (T) = 1/F = 1/GHz = 1ns**

![timing-analysis](https://github.com/user-attachments/assets/18ef0f31-ca7c-4907-8594-da297b2f4df5)

The setup has a launch flop and a capture flop connected by combinational logic with an ideal clock network connecting to both flops (no clock tree built yet). In the basic timing analysis, you'll begin by sending the rising edge at 0 to the launch flop and the rising edge at T to the capture flop. The timing analysis will be conducted from the time in between that. 

Let's assume that the combinational logic = **theta** delay. Then, the theta delay must be less than the time period (T, 1ns). 

The capture flop has many different MOSFETs and resistors inside of it. In a simplified nature, lets assume that it has two muxes inside of it like this: 

![timing-analysis-capture](https://github.com/user-attachments/assets/db3b2ce6-0d78-492a-8111-6ce012b0e509)

The mux delays in the capture flop will restrict your combinational delay requirement (theta) as some of the maximum alloted delay is taken up by the capture flop. This mux delay is called **setup time** as seen below:

![timing-analysis-setup-time](https://github.com/user-attachments/assets/e1447f47-a964-4c43-a788-6bb9e717476c)

## Introduction to Clock Jitter and Uncertainty

**Clock jitter**: deviation of clock signals from their ideal periodic positions.

In the image below, the yellow windows show the amount of jitter that is alloted for the clock signal before circuit failure occurs. 

![timing-analysis-jitter](https://github.com/user-attachments/assets/c83ca170-6ad6-411b-84f4-2bf08b0aca12)

Now, we will add that clock jitter as a variable **SU** uncertainty and consider it in our calculations. 

![timing-analysis-uncertainty](https://github.com/user-attachments/assets/37cae314-b7fd-45a7-83ae-4e62b3d4f038)

All of this is done with single clocks though so its quite simple. 

## Lab Steps to Configure OpenSTA for Post-Synth Timing Analysis

## Lab Steps to Optimize Synthesis to Reduce Setup Violations

## Lab Steps to do Basic Timing ECO

## Clock Tree Routing and Buffering using H-Tree Algorithm

Clock tree synthesis is a process of trying to distribute clock signals to all sequential blocks while minimizing skew and delay. Skew is the difference of arrival times of a clock signal in different parts of a circuit. Ideally, skew should be as close as possible to 0. Buffers and inverters are often used here. 

![cts-bad-tree](https://github.com/user-attachments/assets/4f168fe2-920e-4c6f-ba04-9ac660eb3a2e)

Above, we can see an example of a poorly mapped out clock tree. The skew is going to be significantly higher considering the wire lengths from FF1 and FF2 to clk1 are drastically different, leading to possible timing errors. 

In order to improve the tree, the clk1 will have a wire at the midpoint of the flipflops, from where connections will be made. Below, we can see how this would be configured. 

![cts-better-tree](https://github.com/user-attachments/assets/e66b6a84-c35f-49c4-9896-9f48d85c1042)

This is H-Tree routing. Essentially, calculating the distance from the clock to the destinations, finding the midpoint, then distributing it to the destinations, allowing the signal to reach all destinations at the same time (ideally). However, it is possible for signal distortion to occur (especially over long distances). This can be mitigated by adding buffers and inverters. Buffers just repeat an input signal and output it with high impedance. These clock buffers have equal rise and fall times for a constant signal propagation.

Note that this analysis was done with ideal clocks, not real clocks. 

## Crosstalk and Clock Net Shielding

**Clock net shielding**: take all of the clock nets and shield them. This protects the clock signal from the outside environment. This prevents crosstalk. Crosstalk is when activity from one channel causes disruption with activities in a nearby channel which leads to errors. The image below shoes yellow clock nets that are shielded.

![clock-net-shielding](https://github.com/user-attachments/assets/5c3b8a0c-dcbe-45a4-a85a-4b1472ebec9e)

When a regular wire is next to a shield, there is a coupling capacitor which can lead to glitch. When the aggressor has any sort of switching activity, the coupling capacitor can be strong enough to affect the net adjacent to it. Below, the image shows the aggressor in red, the capacitor in the middle, and the victim net on the bottom. The glitch can cause a dip in voltage (as seen in the image below) which can lead to several errors in data propagation. 

![glitch](https://github.com/user-attachments/assets/a93acfe5-3beb-4dd1-b868-fda92f0e68b9)

The glitch can be fixed by shielding the nets affected by the glitch, where the shield is either VDD or VSS. Since VDD and VSS does not switch, it will prevent the affected net from switching either. 

Delta delay can also happen from the coupling capacitor. It is possible for the coupling capacitor to briefly delay the signal while its trying to change from 0 to 1 or 1 to 0. This can lead to timing errors once again.

![crosstalk-delta-delay](https://github.com/user-attachments/assets/6dc90b31-8531-4092-8812-9f1edba17aec)

Often though, shielding all of the nets is implausible because it takes so much routing resources, and so most of the time, the clock nets are primarily shielded. 

## Lab Steps to Run CTS Using TritonCTS

## Lab Steps to Verify CTS Runs

## Setup Timing Analysis Using Real Clocks

Now, we'll be bringing clock trees to real clocks. With real clocks, the clock tree will now have buffers going into the launch flop and capture flop. We have to consider the clock delay now that occurs from the buffers in between the clock and flops. Seen below is the diagram. 

![timing-analysis-real-clocks](https://github.com/user-attachments/assets/cb5a8e5c-24c6-4d35-b33f-a0941e8b045f)

We can shift the edges by a value of delta1 and delta2. We also have to add back the delays of the setup time of the capture flop and the clock jitter windows. The righthand side of the equation seen below is called data required time and the left side is called data arrival time. If data arrival time is after data required time, then the resulting value is called **slack**. If slack is negative, that is not good and unexpected. Ideally, slack should always be positive or zero. 

So, the final timing analysis diagram is this:

![timing-analysis-fina](https://github.com/user-attachments/assets/6709c6c2-59ef-49ba-9150-bf7c932b9c38)

Now, lets take a look at hold timing analysis. Previously, we had only done setup timing analysis. Hold timing analysis is different as the edge at 0 will be sent to the launch flop to undergo combinational logic, and the edge at 0 will also be sent to the capture flop to check for hold condition.

The hold condition is that the combinational delay should be greater than the hold delay of the capture flop. 

![hold-timing-analysis1](https://github.com/user-attachments/assets/42c539b0-a6a6-40a8-846e-949672b70057)

The hold time of the capture flop is the amount of time it takes for mux2 to send an output. 

![hta-hold-time](https://github.com/user-attachments/assets/cd252ee3-5f39-49d6-9520-b08749598d99)

So, the capture flop is essentially telling the launch flop to not send any new data until the capture flop can send the existing data outside of itself. 

If again, you add clock buffers once again (like in a real circuit), you have to change the equation slightly to this:

![hta-buffers](https://github.com/user-attachments/assets/b1788d16-d1e7-48b5-b2f9-fada649661ba)

## Hold Timing Analysis Using Real Clocks

Technically, we can ignore uncertainty (jitter) as the delay for the launch flop and the capture flop will be the same. 

Notice that for hold analysis, the data arrival time should be greater than the data required time (opposite to setup timing analysis). 

![hta-slack](https://github.com/user-attachments/assets/686fb459-ed49-42bf-a874-4d7bf29941fe)

Now, lets make this diagram more realistic (visually). Skew is launch clock network delay minus the capture clock network delay. 

![hta-skew](https://github.com/user-attachments/assets/9a1e9554-3700-4eaf-a430-38eda1bf1b02)

Then, you also have the setup time equation: 

![hta-setup](https://github.com/user-attachments/assets/45cdf002-d27d-433c-98e3-5008fc2ba5a1)

And finally, here is the final hold time equation: 

![hta-hold-equation](https://github.com/user-attachments/assets/e0b743a8-6f83-4c3f-a7ee-58bc0e8d31ca)

There are a few other components in real circuits that can further complicate timing but this is a generalized, simple flow. 

## Lab Steps to Analyze Timing with Real Clocks Using OpenSTA

## Lab Steps to Execute OpenSTA with Right Timing Libraries and CTS Assignment

## Lab Steps to Observe Impact of Bigger CTS Buffers on Setup and Hold Timing










