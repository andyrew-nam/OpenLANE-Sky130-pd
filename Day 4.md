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





