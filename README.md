# vsdStdCellList_sky130
This repository contains the standard cells designed for the characterization for the following tool ----- and the cells designed here are used as input to the characterization flow.



- [Custom Standard Cell Design using Skywater 130nm PDK](#custom-standard-cell-design-using-skywater-130nm-pdk)


  - [Verification of generated Liberty File with OpenLane](#verification-of-generated-liberty-file-with-openlane)
    - [OpenLane Requirements](#openlane-requirements)
    - [Custom Cells and Skywater 130nm Cells:](#custom-cells-and-skywater-130nm-cells)
    - [Synthesis](#synthesis)
    - [Floor-planning](#floor-planning)
    - [Placement](#placement)
    - [CTS](#cts)
    - [Routing:](#routing)
  - [Future Works:](#future-works)

Standard Cells Designed list

1.Nand2
2.Nand3
3.Nand4
4.o21ai
5.inv
6.inv_8x
7.buf2
8.conb



Standard cell design and characterization in openlane
Objective
The goal of the project is to design a single height standard cell and plug this custom cell into a more complex design and perform it's PnR in the openlane flow. The standard cell chosen is a basic CMOS inverter and the design into which it's plugged into is a pre-built picorv32a core.

About PicoRV32
PicoRV32 is a CPU core that implements the RISC-V RV32IMC Instruction Set. It can be configured as RV32I, RV32IC, RV32IM, or RV32IMC core; where the suffixes stand for:

M - Multiply extension
I - Base Integer Instructions
C - Compressed Instructions
PicoRV32 is free and open hardware licensed under the ISC license. All features and data-sheet related to picoRV32 core can be obtained here.

Standard cell layout design in Magic
The proposed inverter for the design is a single height standard cell, so the dimensions needs to be a multiple of the single height place site; which for sky130 node has a nomenclature of unithd with dimensions(in microns): 0.46 x 2.72 (width x height) for sky130_fd_sc_hd PDK variant. The magic tool is invoked with sky130 tech file as magic -T sky130A.tech & (the magic tech file (sky130A.tech) has also been included in this repo under /libs as reference).

Thus, the first step in magic layout tool is to create a bounding box with a width of 1.38 (3 x width(unithd)) and height of 2.72. This can be done by using command property FIXED_BBOX {0 0 138 272} in magic tkcon window.

alt text

This is followed by defining the ground and power segments (in metal 1), the respective contacts and finally the layout of the logic part. Same procedure can be followed for any standard cell layout.
alt text

Note:
The layout (also included as the part of this repo, viz., sky130_inv.mag) can be viewed in magic layout window as magic -T sky130A.tech sky130_inv.mag &

Create port definition
Once the layout is ready, the next step is extracting LEF file for the cell. However, certain properties and definitions need to be set to the pins of the cell which aid the placer and router tool. For LEF files, a cell that contains ports is written as a macro cell, and the ports are the declared PINs of the macro. Our objective is to extract LEF from a given layout (here of a simple CMOS inverter) in standard format. Defining port and setting correct class and use attributes to each port is the first step. The easiest way to define a port is through Magic Layout window and following are the steps:

In Magic Layout window, first source the .mag file for the design (here inverter). Then Edit >> Text which opens up a dialogue box.
alt text

For each layer (to be turned into port), make a box on that particular layer and input a label name along with a sticky label of the layer name with which the port needs to be associated. Ensure the Port enable checkbox is checked and default checkbox is unchecked as shown in the figure:
alt text

In the above two figures, port A (input port) and port Y (output port) are taken from locali (local interconnect) layer. Also, the number in the textarea near enable checkbox defines the order in which the ports will be written in LEF file (0 being the first).

For power and ground layers, the definition could be same or different than the signal layer. Here, ground and power connectivity are taken from metal1 (Notice the sticky label)
VPWR	VGND
alt text	alt text
Set port class and port use attributes for a layout
Post port definition, the next step is setting port class and port use attributes. The "class" and "use" properties of the port have no internal meaning to magic but are used by the LEF and DEF format read and write routines, and match the LEF/DEF CLASS and USE properties for macro cell pins. Valid classes are: default, input, output, tristate, bidirectional, inout, feedthrough, and feedthru. Valid uses are: default, analog, signal, digital, power, ground, and clock. These attributes are set in tkcon window (after selecting each port on layout window. A keyboard shortcut would be repeatedly pressing s till that port gets highlighed) as:

alt text

Additional:
You can delete or remove any port by first selecting the port (key s) and then executing below two commands in order (in tkcon window):

port remove
label erase
Defining LEF properties and extracting LEF file
Certain properties needs to be set before writing the LEF. As mentioned before, these values are fetched by placer and router to determine, for instance, site where a cell needs to be placed. Macro cell properties common to the LEF/DEF definition but that have no corresponding database interpretation in magic are retained using the cell property method in magic. There are specific property names associated with the LEF format. Once the properties are set, lef write command writes the LEF file with the same nomenclature as that of the layout (.mag) file.

alt text

Plugging custom LEF to openlane flow
If a new custom cell needs to be plugged into openlane flow, include the lefs (the one extracted in Step-5) as below:

In the design's config.tcl file add the below line to point to the lef location which is required during spice extraction.

    set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
Include the below command to include the additional lef into the flow:

    set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
  
    add_lefs -src $lefs
Run the interactive flow as described here.

Note: A sample inverter magic file (sky130_inv.mag) has been included as a reference resource.

Observations
The custom inverter successfully included in the picorv32a design. Below is the final routed picorv32a design with the custom cell zoomed in and highlighted.

alt text

Challenges
The biggest challenge was with the legalization of the cell. Initial iteration showed illegal positioning of the cell away from the standard cell rails.
alt text

Upon closer inspection, the issue seemed to be with the dimensions of the drawn power and ground rails and also with the positioning of local1 -> metal1 contacts which was then corrected for further iterations.

− All metal tracks of the same layer (metal1, metal2, etc) for the same purpose (signal or
power) must have the same width. If, for example, a metal1 track for signal connection is
0.5μm wide, then all other signal connections in metal1 in the library must also be 0.5μm
wide. If a metal1 power pin is 2μm wide, all cells in the library must use 2μm wide metal1
power connections.
− All power/ground pins should have the same width and should run in the same directions – all
horizontal or all vertical.
− Power/ground pins should be in the form of rail at the top/bottom ends of the cell.
− Attempts must be made to lay signal tracks of the same layer in the same direction.
− For any two adjacent signal track in the same metal layer running in the same direction,
center-to-center pitch (defined below) must be either the same or an integer multiply of a
minimal pitch value (called routing pitch).


## Verification of generated Liberty File with OpenLane
### OpenLane Requirements
* Install OpenLane as mentioned in repo [OpenLANE Built Script](https://github.com/nickson-jose/openlane_build_script)
* [OpenLane Workshop repo for understanding openLane flow](https://github.com/mayurpohane17/OpenLANE-Sky130-Workshop)
### Custom Cells and Skywater 130nm Cells:

  * **Skywater Library cells**: sky130_fd_sc_hd__buf_2, and sky130_fd_sc_hd__conb_1.
  * **Custom cells**: sky130_vsdbuf_1x, sky130_vsdbuf_2x, sky130_vsdclkbuf_4x, sky130_vsdinv_1x, sky130_vsdinv_8x, sky130_vsdnand2_1x, sky130_vsdnand3_1x, sky130_vsdnand4_1x, sky130_vsdo21ai_1x and sky130_vsddfxtp_1.     
* **Liberty File**: [sky_mod1.lib](sta_results/sky_mod1.lib)    

### Synthesis
* Designed used for verification: picorv32a   
* Edit config.tcl to -      
  <img src="images/configtcl.png" alt="config" width="450"/>
* Synthesis Result:   
  <img src="images/synthesis.png" alt="config" width="300"/>

### Floor-planning
* Command: `run_floorplan`   
* Layout can be viewed in magic with predefined taps, io pins and decoupling Caps: `magic -T ~/sky130A.tech lef read ~/merged.lef def read picorv32a.floorplan.def`

### Placement
 * Command: `run_placement`
 * Layout:    
    <img src="images/placement.png" alt="placement" width="450"/>

### CTS
* Command: `run_cts`
* Layout:   
  <img src="images/cts.png" alt="placement" width="350"/>

### Routing:
* Command: `run_routing`
* Encountered high congestion error, need to add more type of cells in the library to reduce density and re-check port position in the custom cells for routing tracks.
  
## Future Works: 
* Improve the layout of custom cells for the routing(375 iter-> 81 Overflows)

# Acknowledgement
- Kunal Ghosh, Co-founder, VSD Corp. Pvt. Ltd
- Openlane team, Efabless corporation
- Tim Edwards, Senior Vice President of Analog and Design at efabless corporation
- Nickson Jose, VLSI Engineer

- Prithivi Raj K, National Institute of Technology Tiruchirapalli

# Contact Information
- Praharsha Mahurkar, BE Electronics and Telecommunication, Maharashtra Institute of Technology, Pune, 	praharshapm@gmail.com
- Kunal Ghosh, Co-founder, VSD Corp. Pvt. Ltd. kunalpghosh@gmail.com
