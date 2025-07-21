## IO Placer Revision

The IO Mode is set to **random equidistant** as a default, meaning all of the cells are spaced uniformly. 

To change, run this command: **set ::env(FP_IO_MODE) 2**, then run the floorplan again.

This command will set the mode of the IO pins to 2 which allows them to overlap and such. In OpenLANE, mode 2 causes the pins to be designated based on a Hungarian Matching Algorithm (combinatorial optimization methods). 

## SPICE Deck Creation for CMOS Inverter

**SPICE Deck**: A text file that contains a netlist (list of circuit components and how they're connected), the TAPS points for the outputs, and the inputs that needed to be provided. 

