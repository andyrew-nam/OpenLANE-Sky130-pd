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

Use **lef write** in the tkcon terminal to make a new .lef file with a file name of your choice (in my case, its called "sky130_inv.lef")

![lef_write](https://github.com/user-attachments/assets/dd604a79-d3f8-4867-b692-6492c8dadd5a)

Here is what the lef file should look like. 

![pins_check](https://github.com/user-attachments/assets/bf0dcf71-376a-4414-9996-2eef4db84a5b)

## Introduction of Timing Libs and Steps to Include New Cell in Synthesis

Go to the **libs** directory in vsdstdcelldesign. 





