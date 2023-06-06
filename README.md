
# Physical Design Using OpenLANE/Sky130

This project is done in the course ["Physical Design using OpenLANE/Sky130"](https://www.vlsisystemdesign.com/advanced-physical-design-using-openlane-sky130/) by VLSI System Design Corporation. In this project a complete RTL to GDSII flow for PicoRV32a SoC is executed with Openlane using Skywater130nm PDK. 


## Table of Contents
1. Overview 
2. Inception of Opensource EDA
   - How to Talk to computers?
   - SOC Design & OpenLANE
     - Components of opensource digital ASIC design
     - Simplified RTL2GDS Flow
     - OpenLANE ASIC Flow
   - Opensource EDA tools
     - OpenLANE design stages
     - OpenLANE Files
     - Invoking OpenLANE
     - Design Preparation Step
     - Review of files & Synthesis step
3. Floorplanning & Placement and library cells
   - Floorplanning considerations
     - Utilization Factor & Aspect Ratio
     - Pre-placed cells
     - Decoupling capacitors
     - Power Planning
     - Pin Placement
     - Floorplan run on OpenLANE & view in Magic
   - Placement
     - Placement Optimization
     - Placement run on OpenLANE & view in Magic
   - Standard Cell Design Flow
   - Standard Cell Characterization Flow
   - Timing Parameter Definitions
4. Design library cell
   - SPICE Deck creation & Simulation
   - CMOS inverter Switching Threshold Vm
   - 16 Mask CMOS Fabrication
   - Inverter Standard cell Layout & SPICE extraction
   - Inverter Standard cell characterization
   - Magic Features & DRC rules
5. Timing Analysis & CTS
   - Create port definition
   - Standard Cell LEF generation
   - Integrating custom cell in OpenLANE
   - Post-synthesis timing analysis
   - Clock Tree Synthesis
6. Final steps in RTL2GDS
   - Power Distribution Network generation
   - Routing
   - GDSII
   - LEF
7. Design folder   
8. Differences from older OpenLANE versions
9. Summary
   - VLSI NON INTERACTIVE OPENLANE FLOW


### Inception of Opensource EDA

### How to talk to computers?

RISC-V Instruction Set Architecture (ISA) is a language used to talk to computers. If a user wishes to run a certain application software on a computer, its corresponding C/C++/Java program must be converted into instructions by the compliler. The ouput of the compiler is hardware dependent. These instructions go as inputs to the assembler which outputs binary language that the hardware logic in the chip layout can make sense of. According to the bits received, the digital logic consisting of gates performs the function required by the user of the application software.From the user to RISC-V to implenmentation and finally to Layout.

![compiler](https://github.com/Magalakshmi89/update/assets/135096629/7166f555-849a-4d3a-bd39-63e209069ff9)


### SoC Design & OpenLANE

#### Components of opensource digital ASIC design
The Application Specific Integrated Circuit (ASIC) requires three enablers or elements - Resistor Transistor Logic Intellectual Property (RTL IPs), Electronic Design Automation (EDA) Tools and Process Design Kit (PDK) data.

![Screenshot 2023-05-31 164033](https://github.com/Magalakshmi89/update/assets/135096629/f216ff83-72b0-4011-ab9a-a307ce146eb4)


- Opensource RTL Designs: github, librecores, opencores
- Opensource EDA tools: QFlow, OpenROAD, OpenLANE
- Opensource PDK data: Google Skywater130 PDK

The ASIC flow objective is to convert RTL design to GDSII format.

#### Simplified RTL2GDS Flow

![Screenshot 2023-05-31 165557](https://github.com/Magalakshmi89/update/assets/135096629/89af795f-ed9a-4ef3-95a8-ecb39668acb2)

- Synthesis: RTL Converted to gate level netlist using standard cell libraries (SCL).
- Floor & Power Planning:Planning defines the size of the chip/block,preplacing Hard Macros and thereby detemining the routing areas between   them.Power Planning provide power ti evary Macros,standard cells.It connects Power to chip by considering isssues like EM and IR Drop.
- Placement: Placing cells on floorplan rows aligned with sites
  - Global Placement: for optimal position of cells(not necessary legal)
  - Detailed Placement: for legal positions
  - Clock Tree Synthesis: To deliver the clock to all Sequential Elements (eg.FlipFlops)
- Routing: Implement the interconnect using the available metal.
  - Global : Generate routing guides.
  - Detailed : Uses routing Guides to iimplement actual wiring.
- Signoff: Physical (DRC, LVS) and Timing verifications (STA)

#### OpenLANE ASIC Flow

![3  opensource ASIC digital designs-1](https://user-images.githubusercontent.com/83152452/185787620-8c999b89-2580-477d-aa20-1156d3e996c8.png)
 
![Screenshot 2023-05-31 171816](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/086f43fb-0f82-43ee-aed9-1e29c08cc38a)


From conception to product, the ASIC design flow is an iterative process that is not static for every design. The details of the flow may change depending on ECO’s, IP requirements, DFT insertion, and SDC constraints, however the base concepts still remain. The flow can be broken down into steps:

Logic Design and Verification
 - Design starts with a specification
 - Text description or system specification language
   Example: C, SystemC, SystemVerilog
 - RTL Description
 - Automated conversion from system specification to RTL possible
   Example: Cadence C-to-Silicon Compiler
 - Most often designer manually converts to Verilog or VHDL
 - Verification
 - Generate test-benches and run simulations to verify functionality
 - Assertion based verification
 - Automated test-bench generatio

RTL Synthesis and Verification
- RTL Synthesis
- Automated generation of generic gate description from RTL description
- Logic optimization for speed and area
- State machine decomposition, datapath optimization, power optimization
- Modern tools integrate global place-and-route capabilities
- Library Mapping
- Translates a generic gate level description to a netlist using a target library
- Functional or Formal Verification
- HDL ambiguities can cause the synthesis tool to produce incorrect netlist
- Rerun functional verification on the gate level netlist
- Formal verification
   - Model checking: prove that certain assertions are true
   - Equivalence checking: compare two design descriptions

Static Timing Analysis
- Checks temporal requirements of the design
- Uses intrinsic gate delay information and estimated routing loads to exhaustively
evaluate all timing paths
- Requires timing information for any macro-blocks e.g. memories
- Will evaluate set-up and hold-time violations
- Special cases need to be flagged using timing constraints (more later)
- Reports “slack time”

Test Insertion and Power Analysis
- Insert various DFT features to perform device testing using Automated Test Equipment
(ATE) and system level tests
- Scan enabled flip-flops and scan chains
  Automatic Test Pattern Generation (ATPG) tools generate test vectors to
perform logic and parametric testing
- Built-in Self Test
  Logic: Based on LFSR (random-patterns) and MISR (signature) (LBIST)
  Memory: Implements various memory testing algorithms (MBIST)
- Boundary-Scan/JTAG
   Enables board/system level testing
- More on DFT and test insertion later
- Power Analysis
- Power analysis tools predict power consumption of the circuit
- Either test vectors or probabilistic activity factors used for estimation

Floorplanning 
- To plan the silicon area and create a robust power distribution network (PDN) to power each of the individual      components of the synthesized netlist. 
- Macro placement and blockages must be defined before placement.
- In power planning we create the ring which is connected to the pads which brings power around the edges of the chip.

Placement 
- Place the standard cells on the floorplane rows, aligned with sites defined in the technology lef file.
- Placement is done in two steps: Global and Detailed. 

CTS 
- Clock tree synteshsis is used to create the clock distribution network that is used to deliver the clock to all sequential elements.

Routing 
- Implements the interconnect system between standard cells using the remaining available metal layers after CTS and PDN generation. 
- The routing is performed on routing grids to ensure minimal DRC errors.



### Opensource EDA tools

OpenLANE utilises a variety of opensource tools in the execution of the ASIC flow:
Task | Tool/s
------------ | -------------
RTL Synthesis & Technology Mapping | [yosys](https://github.com/YosysHQ/yosys), abc
Floorplan & PDN | init_fp, ioPlacer, pdn and tapcell
Placement | RePLace, Resizer, OpenPhySyn & OpenDP
Static Timing Analysis | [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA)
Clock Tree Synthesis | [TritonCTS](https://github.com/The-OpenROAD-Project/OpenLane)
Routing | FastRoute and [TritonRoute](https://github.com/The-OpenROAD-Project/TritonRoute) 
SPEF Extraction | [SPEF-Extractor](https://github.com/HanyMoussa/SPEF_EXTRACTOR)
DRC Checks, GDSII Streaming out | [Magic](https://github.com/RTimothyEdwards/magic), [Klayout](https://github.com/KLayout/klayout)
LVS check | [Netgen](https://github.com/RTimothyEdwards/netgen)
Circuit validity checker | [CVC](https://github.com/d-m-bailey/cvc)


#### OpenLANE Files

The openLANE file structure looks something like this:

- skywater-pdk: contains PDK files provided by foundry
- open_pdks: contains scripts to setup pdks for opensource tools 
- sky130A: contains sky130 pdk files

#### Invoking OpenLANE and Design Preparation 

Openlane can be invoked using docker command followed by opening an interactive session.
flow.tcl is a script that specifies details for openLANE flow.

```
docker
./flow.tcl -interactive
package require openlane 0.9
```

```
prep -design picorv32a
```

![Screenshot 2023-05-31 135059](https://github.com/Magalakshmi89/update/assets/135096629/78b63cea-0da9-4676-8fab-1f5ba60288d9)

![Screenshot 2023-05-31 232255](https://github.com/Magalakshmi89/update/assets/135096629/b77d7853-85b9-400f-9019-31200973e452)

#### Review of files & Synthesis step
* A "runs" folder is generated within the picorv32a folder.
* The merged file is created during the merging operation in the pircorv32a design preparation (it merges lef and techlef files)
* Next, we run the synthesis of picorv32a design in the openlane interactive terminal:

`run_synthesis`

![Screenshot 2023-05-31 234758](https://github.com/Magalakshmi89/update/assets/135096629/979fff60-24da-41c3-a98c-0c97d8fef679)

* The yosys and ABC tools are utilised to convert RTL to gate level netlist.
* Calcuation of Flop Ratio:
```
Flop ratio = Number of D Flip flops 
             ______________________
             Total Number of cells
```

```
dfxtp_4 = 1613,
Number of cells = 14876,
Flop ratio = 1613/14876 = 0.1084 = 10.84%
```
* We may check the success of the synthesis step by checking the synthesis folder for the synthesized netlist file (.v file)
* The synthesis statistics report can be accessed within the reports directory. It is usually the last yosys file since files are listed chronologically by date of modification.
* The synthesis timings report are as follows:

![Screenshot 2023-06-01 140815](https://github.com/Magalakshmi89/update/assets/135096629/e0957811-c12f-41a7-a4ee-9fec4fdfc66f)

* The synthesis statistics report is as follows:

![Screenshot 2023-06-01 140458](https://github.com/Magalakshmi89/update/assets/135096629/298d65fd-17f2-4804-8780-88fb9752cb9d)

## Floorplanning & Placement and library cells

### Floorplanning considerations

#### Utilization Factor & Aspect Ratio  

Two parameters are of importance when it comes to floorplanning namely, Utilisation Factor and Aspect Ratio. They are defined as follows:

```
Utilisation Factor =  Area occupied by netlist
                     __________________________
                        Total area of core
```

```
Aspect Ratio =  Height
               ________
                Width
```
                                  
A Utilisation Factor of 1 signifies 100% utilisation leaving no space for extra cells such as buffer. However, practically, the Utilisation Factor is 0.5-0.6. Likewise, an Aspect ratio of 1 implies that the chip is square shaped. Any value other than 1 implies rectanglular chip.

#### Pre-placed cells

Once the Utilisation Factor and Aspect Ratio has been decided, the locations of pre-placed cells need to be defined. Pre-placed cells are IPs comprising large combinational logic which once placed maintain a fixed position. Since they are placed before placement and routing, the are known as pre-placed cells.

![Screenshot 2023-06-01 144634](https://github.com/Magalakshmi89/update/assets/135096629/b9361550-2425-4286-955b-e1c725d43e8c)

#### Decoupling capacitors

![capacitor](https://github.com/Magalakshmi89/update/assets/135096629/3b2b5157-93ba-4142-8e10-fd2664d5ded0)

Pre-placed cells must then be surrounded with decoupling capacitors (decaps). Its is capacitor in which is used to decouple critical cells from main power supply in order to protect from disturbance in the power distribution lines.Decaps are huge capacitors charged to power supply voltage and placed close the logic circuit. Their role is to decouple the circuit from power supply by supplying the necessary amount of current to the circuit. They pervent crosstalk and enable local communication.

#### Power Planning

Each block on the chip, however, cannot have its own decap unlike the pre-placed macros. Therefore, a good power planning ensures that each block has its own VDD and VSS pads connected to the horizontal and vertical power and GND lines which form a power mesh.

![power](https://github.com/Magalakshmi89/update/assets/135096629/0f43cf42-1aef-443e-82d8-0d51e63d27d7)


#### Pin Placement
 The connectivityy information between the gates is coded using VHDL/Verilog Language and is called Netlist. The place between the core and die is utilised for placing pins.Then, logical placement blocking of pre-placed macros is performed so as to differentiate that area from that of the pin area.


 To run the picorv32a floorplan in openLANE:
 ```
 run_floorplan
 
 ```
 ![openlane - placement - run_floorplan](https://user-images.githubusercontent.com/83152452/185788662-a73dca29-98a9-4b78-9297-61f8f9bc7a47.png)
 

Post the floorplan run, a .def file will have been created within the ```results/floorplan``` directory. 
We may review floorplan files by checking the ```floorplan.tcl```. 
The system defaults will have been overriden by switches set in ```conifg.tcl``` and further overriden by switches set in ```sky130A_sky130_fd_sc_hd_config.tcl```.
 
To view the floorplan, Magic is invoked after moving to the ```results/floorplan``` directory:

```
magic -T /home/devipriya/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.min.lef def read picorv32a.def &
```
![Screenshot 2023-06-02 112011](https://github.com/Magalakshmi89/update/assets/135096629/a108632a-6bd7-486f-94e6-b098d486cf9a)

 ![Screenshot 2023-06-02 112052](https://github.com/Magalakshmi89/update/assets/135096629/b1bdd222-b260-44b7-bae2-07b46b5c8715)

  ![Screenshot 2023-06-02 112146](https://github.com/Magalakshmi89/update/assets/135096629/8a51d94d-c44b-445a-bc2e-3e4060336179)

![Screenshot 2023-06-02 112324](https://github.com/Magalakshmi89/update/assets/135096629/2ad7cf00-bb4e-42d6-96e6-e3303d5da138)

* One can zoom into Magic layout by selecting an area with left and right mouse click followed by pressing "z" key. 
* Various components can be identified by using the ```what``` command in tkcon window after making a selection on the component.
* Zooming in also provides a view of decaps present in picorv32a chip.
* The standard cell can be found at the bottom left corner.


### Placement 

#### Placement Optimization

The next step in the OpenLANE ASIC flow is placement. The synthesized netlist is to be placed on the floorplan. Placement is perfomed in 2 stages:

1. Global Placement: It finds optimal position for all cells which may not be legal and cells may overlap. Optimization is done through reduction of half parameter wire length.
2. Detailed Placement: It alters the position of cells post global placement so as to legalise them.

Legalisation of cells is important from timing point of view. 

#### Placement run on OpenLANE & view in Magic

Command:

```
run_placement
```

![openlane - placement - run_placement-1](https://user-images.githubusercontent.com/83152452/185789104-a2284118-7e56-439e-abac-bb1ba669e72b.png)

The objective of placement is the convergence of overflow value. If overflow value progressively reduces during the placement run it implies that the design will converge and placement will be successful. Post placement, the design can be viewed on magic within ```results/placement``` directory:

```
magic -T /home/devipriya/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.max.lef def read picorv32a.def &

```

![Screenshot 2023-06-02 125604](https://github.com/Magalakshmi89/update/assets/135096629/5e7baece-3133-4a51-a791-473e1e58d510)

Zoomed-in views of the standard cell placement:

![Screenshot 2023-06-02 125645](https://github.com/Magalakshmi89/update/assets/135096629/8414fb6a-58ec-4b8f-9c73-a1b8e31bb61f)


***Note: Power distribution network generation is usually a part of the floorplan step. However, in the openLANE flow, floorplan does not generate PDN. The steps are - floorplan, placement CTS and then PDN***


### Standard Cell Design Flow

![Screenshot 2023-06-02 133703](https://github.com/Magalakshmi89/update/assets/135096629/726c0773-10d3-4b92-918f-bd5ff8ba792d)

Standard cell design flow involves the following:

1. Inputs: PDKs, DRC & LVS rules, SPICE models, libraries, user-defined specifications. 
2. Design steps: Circuit design, Layout design (Art of layout Euler's path and stick diagram), Extraction of parasitics, Characterization (timing, noise, power).
3. Outputs: CDL (circuit description language), LEF, GDSII, extracted SPICE netlist (.cir), timing, noise and power .lib files

### Standard Cell Characterization Flow

A typical standard cell characterization flow includes the following steps:

1. Read in the models and tech files
2. Read extracted spice netlist
3. Recognise behaviour of the cell
4. Read the subcircuits
5. Attach power sources
6. Apply stimulus to characterization setup
7. Provide necessary output capacitance loads
8. Provide necessary simulation commands

The opensource software called GUNA can be used for characterization. Steps 1-8 are fed into the GUNA software which generates timing, noise and power models.

### Timing Parameter Definitions

![Screenshot 2023-06-02 190500](https://github.com/Magalakshmi89/update/assets/135096629/c520b997-e606-4ea2-bbba-96dbca226b59)

Timing defintion | Value
------------ | -------------
slew_low_rise_thr  | 20% value
slew_high_rise_thr |  80% value
slew_low_fall_thr | 20% value
slew_high_fall_thr | 80% value
in_rise_thr | 50% value
in_fall_thr | 50% value
out_rise_thr | 50% value
out_fall_thr | 50% value

```
rise delay =  time(out_fall_thr) - time(in_rise_thr)

Fall transition time: time(slew_high_fall_thr) - time(slew_low_fall_thr)

Rise transition time: time(slew_high_rise_thr) - time(slew_low_rise_thr)
```

A poor choice of threshold points leads to neative delay value. Therefore a correct choice of thresholds is very important.

## Design library cell

OpenLANE allows users to make changes to environment variables on the fly. For instance, if we wish to change the pin placement from equidistant to some other style of placement we may do the following in the openLANE flow:

```
set ::env(FP_IO_MODE) 2
```

### SPICE Deck creation & Simulation

A SPICE deck includes information about the following:

1. Model description
2. Netlist description
3. Component connectivity
4. Component values
5. Capacitance load
6. Nodes
7. Simulation type and parameters
8. Libraries included

### CMOS inverter Switching Threshold Vm

Thw sitching threshold of a CMOS inverter is the point on the transfer characteristic where Vin equals Vout (=Vm). At this point both PMOS and NMOS are in ON state which gives rise to a leakage current
### Inverter Standard cell Layout & SPICE extraction

The Magic layout of a CMOS inverter will be used so as to intergate the inverter with the picorv32a design. To do this, inverter magic file is sourced from [vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign) by cloning it within the ```openlane_working_dir/openlane``` directory as follows:

```
git clone https://github.com/nickson-jose/vsdstdcelldesign
```

This creates a vsdstdcelldesign named folder in the openlane directory.

To invoke magic to view the ```sky130_inv.mag``` file, the sky130A.tech file must be included in the command along with its path. To ease up the complexity of this command, the tech file can be copied from the magic folder to the vsdstdcelldesign folder.

The sky130_inv.mag file can then be invoked in Magic very easily

```

![Screenshot 2023-06-03 163558](https://github.com/Magalakshmi89/update/assets/135096629/65562ac3-3bad-45a8-8dc5-152bbf4b565a)

```
![Screenshot 2023-06-03 164536](https://github.com/Magalakshmi89/update/assets/135096629/8cb7438e-8fde-4014-b1e1-45700c773e58)

![Screenshot 2023-06-03 164611](https://github.com/Magalakshmi89/update/assets/135096629/633e5300-4237-430d-94c3-3f19c8d1f60a)

In Sky130 the first layer is called the local interconnect layer or Locali.

To verify whether the layout is that of CMOS inverter, verification of P-diffusiona nd N-diffusion regions with Polysilicon can be observed.

Other verification steps are to check drain and source connections. The drains of both PMOS and NMOS must be connected to output port (here, Y) and the sources of both must be connected to power supply VDD (here, VPWR).

**LEF or library exchange format:**
A format that tells us about cell boundaries, VDD and GND lines. It contains no info about the logic of circuit and is also used to protect the IP.

**SPICE extraction:**
Within the Magic environment, following commands are used in tkcon to achieve .mag to .spice extraction:

```
extract all
ext2spice cthresh 0 rethresh 0
ext2spice
```

![Screenshot 2023-06-03 170501](https://github.com/Magalakshmi89/update/assets/135096629/4bdee41e-afa1-44b4-8e6c-255aa4ef40e9)


This generates the ```sky130_in.spice``` file. This SPICE deck is edited to include ```pshort.lib``` and ```nshort.lib``` which are the PMOS and NMOS libraries respectively. In addition, the minimum grid size of inverter is measured from the magic layout and incorporated into the deck as: ```.option scale=0.01u```. The model names in the MOSFET definitions are changed to ```pshort.model.0``` and ```nshort.model.0``` respectively for PMOS and NMOS. 

Finally voltage sources and simulation commands are defined as follows:

```
VDD VPWR 0 3.3V
VSS VGND 0 0
Va A VGND PUSLE(0V 3.3V 0 0.1ns 0.1 ns 2ns 4ns)
.tran 1n 20n
.control
run 
.endc
.end
```

The final sky130_inv.spice file is modified to:

![3  spice file](https://user-images.githubusercontent.com/83152452/185789670-f5d25504-f809-404f-ab6f-a60214661134.png)

For simulation, ngspice is invoked in the terminal:

```
ngspice sky130_inv.spice
```

The output "y" is to be plotted with "time" and swept over the input "a":

```
plot y vs time a
```


The waveform obtained is as shown:
![Screenshot 2023-06-05 221705](https://github.com/Magalakshmi89/update/assets/135096629/228b6965-6346-45d4-94bd-b92e7de9619b)


![Screenshot 2023-06-06 011059](https://github.com/Magalakshmi89/update/assets/135096629/4b9c93c8-b0ef-4e6e-aeb6-a0ac19a0aa4d)

The spikes in the output at switching points is due to low capacitance loads. This can be taken care of by editing the spice deck to increase the load capacitance value.

### Inverter Standard cell characterization

Four timing parameters are used to characterize the inverter standard cell:

1. Rise transition: Time taken for the output to rise from 20% of max value to 80% of max value
2. Fall transition- Time taken for the output to fall from 80% of max value to 20% of max value
3. Cell rise delay = time(50% output rise) - time(50% input fall)
4. Cell fall delay  = time(50% output fall) - time(50% input rise)

The above timing parameters can be computed by noting down various values from the ngspice waveform.

```Rise transition = (2.25 - 2.1923) = 57.7ps```

```Fall transition = (4.08806 - 4.04758) = 40.48ps```

```Cell rise delay = (2.20968 - 2.14919) = 60.49ps```

```Cell fall delay = (4.0724 - 4.05) = 22.4ps```



## Timing Analysis & CTS

A requirement for ports as specified in ```tracks.info``` is that they should be at intersection of horizontal and vertical tracks. The CMOS Inverter ports A and Y are on li1 layer. It needs to be ensured that they're on the intersection of horizontal and vertical tracks. We access the tracks.info file for the pitch and direction information:

![image](https://user-images.githubusercontent.com/55539862/185734357-8cd10f22-9a23-4cef-af83-f029b344b473.png)

To ensure that ports lie on the intersection point, the grid spacing in Magic (tkcon) must be changed to the li1 X and li1 Y values. Convergence of grid and tracks can be achieved using the following command:

```
grid 0.46um 0.34um 0.23um 0.17um
```

![1  grid activation](https://user-images.githubusercontent.com/83152452/185789868-4a759399-9ee7-4b94-8b11-93283ed4ebfa.png)

![2  grid cells with tkcon window command nd magic layout changes](https://user-images.githubusercontent.com/83152452/185789878-18b441ec-af32-4f3f-b123-2908b864c6fb.png)


### Create port definition

Once the layout is ready, the next step is extracting LEF file for the cell. However, certain properties and definitions need to be set to the pins of the cell which aid the placer and router tool. For LEF files, a cell that contains ports is written as a macro cell, and the ports are the declared PINs of the macro. Our objective is to extract LEF from a given layout (here of a simple CMOS inverter) in standard format. Defining port and setting correct class and use attributes to each port is the first step. 

The easiest way to define a port is through Magic Layout window and following are the steps:

- In Magic Layout window, first source the .mag file for the design (here inverter). Then **Edit >> Text** which opens up a dialogue box.

In the above two figures, port A (input port) and port Y (output port) are taken from locali (local interconnect) layer. Also, the number in the textarea near enable checkbox defines the order in which the ports will be written in LEF file (0 being the first).


### Standard Cell LEF generation

Before the CMOS Inverter standard cell LEF is extracted, the purpose of ports must be defined:

Select port A in magic:
```
port class input
port use signal
```
Select Y area
```
port class output
port class signal
```
Select VPWR area
```
port class inout
port use power
```

Select VGND area
```
port class inout
port use ground
```

LEF extraction can be carried out in tkcon as follows:

```
lef write
```
![3  lef file extraction](https://user-images.githubusercontent.com/83152452/185790311-f5a68ba1-9e1d-47b1-ae47-d7a6f1a723d2.png)

This generates ```sky130_vsdinv.lef``` file.

### Opening MAGIC tool:
![Image](https://github.com/srsapireddy/Images/blob/main/175.png?raw=true) </br>
* Open met3.mag file using  MAGIC:
![Screenshot 2023-06-06 023142](https://github.com/Magalakshmi89/update/assets/135096629/6ea0dca8-2458-42da-868b-d65582360a0a)

* Reference: https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#m3
![Screenshot 2023-06-06 103940](https://github.com/Magalakshmi89/update/assets/135096629/103ac62a-93ee-49f8-a8fa-bd12279af065)

![Image](https://github.com/srsapireddy/Images/blob/main/178.png?raw=true) </br>

![Screenshot 2023-06-06 104916](https://github.com/Magalakshmi89/update/assets/135096629/63a340f0-1dea-453e-a70d-4e48a2942446)
* Open file poly.mag </br>

![Screenshot 2023-06-06 113913](https://github.com/Magalakshmi89/update/assets/135096629/5730f2fa-1b4b-4617-914b-124b3fb75759)

* Considering poly.9
* Poly resistor spacing to poly or spacing (no overlap) to diff/tap 0.480 µm

* Reference: https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#poly
![Screenshot 2023-06-06 122733](https://github.com/Magalakshmi89/update/assets/135096629/3cffd651-0f20-41ae-97ae-6a974f0c68b5)


## Lab steps to convert grid info to track information
* Open inverter mag file </br>
![Image](https://github.com/srsapireddy/Images/blob/main/189.png?raw=true) </br>
* LEF file have the information about the input, power, and ground ports.
* We need to extract LEF file from the .mag file.
1.The input port and output port must lie on the vertical and horizontal tracks.
2.Width of the standard cell must be in odd multiples of the track pitch.
3.Height should be in the order of vertical pitch.
* Press g to get the grid view.
![Screenshot 2023-06-06 122733](https://github.com/Magalakshmi89/update/assets/135096629/2150a1b2-6595-4ef5-8f6b-ea4eb7472e77)

![Screenshot 2023-06-06 122835](https://github.com/Magalakshmi89/update/assets/135096629/ca0e62fa-8c5b-40d4-b0c6-bb31c26a7cc7)

### Convert magic layout to std cell LEF
* The width of the standard cell should be in the odd multiples of the X-pitch.
* Port information is required only when we need to extract the LEF file. When we extract the LEF file, these ports are defined as pins of the macro.
* Converting labels to ports:
* How does the tool know A is input and Y is an output port?
* We use port class and port use attributes.
* Once these parameters are set, we can extract the LEF file from a cell/ macro.
* Before extracting the LEF, we need to give the cell a custom name
* Creating a LEF file </br>

### Plugging this LEF file into PICORV32A – move the LEF file into src folder
 We need to have a library which has our cell definition for synthesis.
Here the tool should map vsdinverter cell to the synthesis flow. We need to modify our config.tcl file next.
                     Then `run_synthesis`
We can see that 1554 instances of our custom cell is added to the picorv32a design.


### Integrating custom cell in OpenLANE

In order to include the new standard cell in the synthesis, copy the sky130_vsdinv.lef file to the ```designs/picorv32a/src``` directory  
Since abc maps the standard cell to a library abc there must be a library that defines the CMOS inverter. The ```sky130_fd_sc_hd_typical.lib``` file from ```vsdstdcelldesign/libs``` directory needs to be copied to the ```designs/picorv32a/src``` directory (Note: the slow and fast library files may also be copied).

Next, ```config.tcl``` must be modified:
```
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```


In order to integrate the standard cell in the OpenLANE flow, invoke openLANE as usual and carry out following steps:

```
prep -design picorv32a -tag RUN_2022.08.17_16.22.21 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
```
![Screenshot 2023-06-06 144703](https://github.com/Magalakshmi89/update/assets/135096629/f04c7e9b-d0ff-40b2-8f79-921c2975b317)

Next floorplan is run, followed by placement:

```
run_floorplan
run_placement
```

To check the layout invoke magic from the ```results/placement``` directory:

```
magic -T /home/vsduser/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.max.lef def read picorv32a.def &
```

Since the custom standard cell has been plugged into the openLANE flow, it would be visible in the layout.

![sky130_vsdinv](https://user-images.githubusercontent.com/83152452/185794679-cd0de4df-30a6-46bd-b6dd-c73a727d85f4.jpeg)


### Post-synthesis timing analysis

Timing analysis is carried out outside the openLANE flow using OpenSTA tool. For this, a new file ```pre_sta.conf``` is created. This file would be reqiured to carry out the STA analysis. Invoke OpenSTA outside the openLANE flow as follows:

```
sta pre_sta.conf
```

Since clock tree synthesis has not been performed yet, the analysis is with respect to ideal clocks and only setup time slack is taken into consideration. The slack value is the difference between data required time and data arrival time. The worst slack value must be greater than or equal to zero. If a negative slack is obtained, following steps may be followed:
1. Change synthesis strategy, synthesis buffering and synthesis sizing values 
2. Review maximum fanout of cells and replace cells with high fanout

The lef file generated:

![5  generated lef file](https://user-images.githubusercontent.com/83152452/185791680-36453e1a-5b7b-44e4-866f-4d0be2e63b8e.png)

```
## Fixing Slack
SYNTH_BUFFERING: Find any high fanout nets. We want high fanout nets to be buffers.
SYNTH_SIZING: Upsizing or downsizing a buffer based on delay strategy.
SYNTH_DRIVING_CELL: Cell that drives the input port. </br>
![Image](https://github.com/srsapireddy/Images/blob/main/211.png?raw=true)  </br>
Settings to reduce tns and wns.
Run synthesis, floorplan and placement steps.  </br>
 
Check if vsd inverter is added after floorplan stage.
Check layout after placement stage
![Screenshot 2023-06-06 183230](https://github.com/Magalakshmi89/update/assets/135096629/bef2dea4-6b1b-4e2e-b7bb-4752ba43bb8e)
![Screenshot 2023-06-06 183408](https://github.com/Magalakshmi89/update/assets/135096629/515437be-a5ae-424a-b195-72e6f4239fd9)
![Screenshot 2023-06-06 183653](https://github.com/Magalakshmi89/update/assets/135096629/3df18fb6-213a-41a7-8227-cd367f4a6ba0)

We can see that vsd inverter is inserted after placement stage.
Abatement between the cells takes place to share power and ground between the cells.

### Static Timing Analysis (With Ideal Clocks)
* In ideal clock network clock tree is not yet built.
* Identifying combinational path:
* Pre-layout STA will not yet include effects of clock buffers and net-delay due to RC parasitics (wire delay will be derived from PDK library wire model).

Checking Timing analysis with ideal clocks using openSTA Lab: </br>
![Image](https://github.com/srsapireddy/Images/blob/main/300.png?raw=true) </br>
![Image](https://github.com/srsapireddy/Images/blob/main/301.png?raw=true) </br>
![Image](https://github.com/srsapireddy/Images/blob/main/302.png?raw=true) </br>
We can see that the setup slack is MET.
pre_sta.conf file: </br>
![Image](https://github.com/srsapireddy/Images/blob/main/303.png?raw=true) </br>
my_base.sdc file </br>
![Image](https://github.com/srsapireddy/Images/blob/main/305.png?raw=true) </br>
PATH:   </br>
![Image](https://github.com/srsapireddy/Images/blob/main/304.png?raw=true) </br>

## Optimize synthesis to reduce setup violations
* Hold analysis have significance after CTS.
* The delay of any cell is a function of input slew (input transition) and output load. More the values, more the delay.
* Optimizing the fanout value:
* Set parameter for fanout:
![Image](https://github.com/srsapireddy/Images/blob/main/223.png?raw=true) </br>
* Then run synthesis, floorplan, and placement to check slack.
* Commands:
`report_net -connections _02682_` </br>
`replace_cell _41882_ sky130_fd_sc_hd__buf_4` </br>
`report_checks -fields {cap slew nets} -digits 4` </br>
`report_checks -from _18671_ -to _18739_ -fields {cap slew nets} -digits 4` </br>
`report_wns` </br>
`report_tns` </br>
`report_worst_slack -max` </br>
`write_verilog designs/picorv32a/runs/date/results/synthesis/picorv32.v` </br>
## Analyze timing with a real clock using OpenSTA
* Run `openroad` to do timing analysis. OpenSTA is integrated inside the openroad.
* Here we need to read lef and def files to create db.
![Image](https://github.com/srsapireddy/Images/blob/main/235.png?raw=true) </br>
![Image](https://github.com/srsapireddy/Images/blob/main/236.png?raw=true) </br>
 
* Create a db </br>
![Image](https://github.com/srsapireddy/Images/blob/main/237.png?raw=true) </br>
* Check if db is created  </br>
![Image](https://github.com/srsapireddy/Images/blob/main/238.png?raw=true) </br>
* Read db , Verilog, and library files
![Image](https://github.com/srsapireddy/Images/blob/main/239.png?raw=true) </br>
* read_liberty $::env(LIB_SYNTH_COMPLETE) // we are not using LIB_MAX & LIB_MIN because we ran our cts for one corner that is the typical corner. </br>
![Image](https://github.com/srsapireddy/Images/blob/main/240.png?raw=true) </br>
* Read sdc file </br>
![Image](https://github.com/srsapireddy/Images/blob/main/241.png?raw=true) </br>
## SDC FILE UPDATED
![Image](https://github.com/srsapireddy/Images/blob/main/242.png?raw=true) </br>
* PATH: picorv32a/src/my_base.sdc
* To calculate actual cell delay in clock path:
![Image](https://github.com/srsapireddy/Images/blob/main/243.png?raw=true) </br>
* Check Slack: </br>
* Hold Slack: </br>
![Image](https://github.com/srsapireddy/Images/blob/main/244.png?raw=true) </br>
* Setup Slack: </br>
![Image](https://github.com/srsapireddy/Images/blob/main/245.png?raw=true) </br>
* We have built the clock tree for typical but analyzing for min_max corners.
* Exit from the openroad
* Here we need to include the typical library for typical analysis.
![Image](https://github.com/srsapireddy/Images/blob/main/246.png?raw=true) </br>
* Reading typical lib (screenshot - USE LIB_SYNTH OR LIB_TYPICAL variable) </br>
![Image](https://github.com/srsapireddy/Images/blob/main/247.png?raw=true) </br>
* Slack for typical corner: </br>
![Image](https://github.com/srsapireddy/Images/blob/main/248.png?raw=true) </br>
![Image](https://github.com/srsapireddy/Images/blob/main/249.png?raw=true) </br>
* Both are met.


## Clock Tree Synthesis

The purpose of building a clock tree is enable the clock input to reach every element and to ensure a zero clock skew. H-tree is a common methodology followed in CTS.
Before attempting a CTS run in TritonCTS tool, if the slack was attempted to be reduced in previous run, the netlist may have gotten modified by cell replacement techniques. Therefore, the verilog file needs to be modified using the ```write_verilog``` command. Then, the synthesis, floorplan and placement is run again. To run CTS use the below command:

```
run_cts
```

The CTS run adds clock buffers in therefore buffer delays come into picture and our analysis from here on deals with real clocks. Setup and hold time slacks may now be analysed in the post-CTS STA anlysis in OpenROAD within the openLANE flow:

```
openroad
write_db pico_cts.db
read_db pico_cts.db
read_verilog /openLANE_flow/designs/picorv32a/runs/03-07_11-25/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock (all_clocks)
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```

Slack at the end of STA for typical corner:

![6  cts report-1](https://user-images.githubusercontent.com/83152452/185791682-94f98a0a-56ce-4409-9a66-796675ac5d39.png)


## Final steps in RTL2GDS

### Power Distribution Network generation

Unlike the general ASIC flow, Power Distribution Network generation is not a part of floorplan run in OpenLANE. PDN must be generated after CTS and post-CTS STA analyses:

```
gen_pdn
```

We can confirm the success of PDN by checking the current def environment variable: ``` echo $::env(CURRENT_DEF) ```

![pdn def report-1](https://user-images.githubusercontent.com/83152452/185791987-6a53d110-667f-4243-a27f-056ed20e0154.png)


- `gen_pdn` - Generates the Power Distribution network
- The power distribution network has to take the `design_cts.def` as the input def file.
- This will create the grid and the straps for the Vdd and the ground. These are placed around the standard cells.
- The standard cells are designed such that it's height is multiples of the space between the Vdd and the ground rails. Here, the pitch is `2.72`. Only if the above conditions are adhered it is possible to power the standard cells.
- The power to the chip, enters through the `power pads`. There is each for Vdd and Gnd
- From the pads, the power enters the `rings`, through the `via`
- The `straps` are connected to the ring. Vdd straps are connected to the Vdd ring and the Gnd Straps are connected to the Gnd ring. There are horizontal and the vertical straps
- Now the power has to be supplied from the straps to the standard cells. The straps are connected to the `rails` of the standard cells
- If macros are present then the straps attach to the `rings` of the macros via the `macro pads` and the pdn for the macro is pre-done.
- There are definitions for the straps and the railss. In this design straps are at metal layer 4 and 5 and the standard cell rails are at the metal layer 1. Vias connect accross the layers as required.


### Routing 

OpenLANE uses the TritonRoute tool for routing. There are 2 stages of routing:

1. Global routing: Routing region is divided into rectangle grids which are represented as course 3D routes (Fastroute tool).

![global routing](https://user-images.githubusercontent.com/83152452/185791992-5d0dadb2-4dc1-493a-bd24-b531298d6442.png)

3. Detailed routing: Finer grids and routing guides used to implement physical wiring (TritonRoute tool).

![detailed routing](https://user-images.githubusercontent.com/83152452/185791995-c8204a98-b61b-48a3-a1ec-642da35d7589.png)

Features of TritonRoute:

1. Honouring pre-processed route guides
2. Assumes that each net satisfies inter guide connectivity
3. Uses MILP based panel routing scheme
4. Intra-layer parallel and inter-layer sequential routing framework

Running routing step in TritonRoute as part of openLANE flow:

```
run_routing
```

- `run_routing` - To start the routing
- The options for routing can be set in the `config.tcl` file. 
- The optimisations in routing can also be done by specifying the routing strategy to use different version of `TritonRoute Engine`. There is a trade0ff between the optimised route and the runtime for routing.
- For the default setting picorv32a takes approximately 30 minutesaccording to the current version of TritonRoute.
- This routing stage must have the `CURRENT_DEF` set to `pdn.def`
- The two stages of routing are performed by the following engines:
    - Global Route : Fast Route
    - Detailed Route : Triton Route
- Fast Route generates the routing guides, whereas Triton Route uses the Global Route and then completes the routing with some strategies and optimisations for finding the best possible path connect the pins.

## GDSII

GDS Stands for Graphic Design Standard. This is the file that is sent to the foundry and is called as "tape-out". 

*Fact- Earlier, the GDS files were written on magnetic tapes and sent out to the foundry and hence the name "tape-out"*

In openLane use the command `magic`

The GDSII file is generated in the `results/signoff/magic` directory.

No DRC errors are found.

The layout pictures are shown below:

![3  picorv32a mag layout - without black metal](https://user-images.githubusercontent.com/83152452/185795420-9af80389-7205-4a18-ac1b-0f95bd0f9d5c.png)

![3  picorv32a mag layout](https://user-images.githubusercontent.com/83152452/185795424-bb2cf6ee-87ef-4e60-ac54-5e793d8cf3f4.png)

Zoom in view:

![3  picorv32a mag zoom in view layout](https://user-images.githubusercontent.com/83152452/185795427-f2a5b5c1-00ef-4c39-aaf5-81e04eb9cb50.png)


### LEF 

```picorv32a.lef.mag``` file generated is shown below:

![4  picorv32a lef mag layout](https://user-images.githubusercontent.com/83152452/185795429-6fbb2ed0-e787-46d0-a4f7-198ae3bd8fd2.png)

## Design folder


```

picorv32a
├── config.tcl
├── runs
│   ├── RUN_2022.08.17_16.22.21
│   │   ├── config.tcl
│   │   ├── logs
│   │   │   ├── cts
│   │   │   ├── cvc
│   │   │   ├── floorplan
│   │   │   ├── klayout
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   ├── reports
│   │   │   ├── cts
│   │   │   ├── cvc
│   │   │   ├── floorplan
│   │   │   ├── klayout
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   ├── results
│   │   │   ├── cts
│   │   │   ├── cvc
│   │   │   ├── floorplan
│   │   │   ├── klayout
|   |   |   ├── signoff
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   └── tmp
│   │       ├── cts
│   │       ├── cvc
│   │       ├── floorplan
│   │       ├── klayout
│   │       ├── magic
│   │       ├── placement
│   │       ├── routing
│   │       └── synthesis
├── src
|   ├── picorv32a.v

```

## Differences from older OpenLANE versions

- The reports in the latest version is not generated in the terminal, we have to verify them from the reports/results folder.
- SPEF extraction need not be externally performed in the new version. It has been integrated into the OpenLANE flow.


## Summary

### VLSI NON INTERACTIVE OPENLANE FLOW

``` 

cd OpenLane/ 
make mount 
{ If Error occurs use the below commands in OpenLane directory:
sudo chown $USER /var/run/docker.sock 
PYTHON_BIN=python3 make mount
}

./flow.tcl -design picorv32a 

```

STEPS RUNNING:

```

[STEP 1] : Running Synthesis.
[STEP 2] : Running Single-Corner Static Timing Analysis.
[STEP 3] : Running Initial Floorplanning, Setting Core Dimensions.
[STEP 4] : Running IO Placement.
[STEP 5] : Running Power planning with power {VPWR} and ground {VGND}.
[STEP 6] : Generating PDN.
[STEP 7] : Performing Random Global Placement.
[STEP 8] : Running Placement Resizer Design Optimizations.
[STEP 9] : Writing Verilog.
[STEP 10] : Running Detailed Placement.
[STEP 11] : Running Placement Resizer Timing Optimizations.
[STEP 12] : Writing Verilog, Routing.
[STEP 13] : Running Global Routing Resizer Timing Optimizations.
[STEP 14] : Writing Verilog.
[STEP 15] : Running Detailed Placement.
[STEP 16] : Running Global Routing, Starting FastRoute Antenna Repair Iterations.
[STEP 17] : Running Fill Insertion.
[STEP 18] : Writing Verilog.
[STEP 19] : Running Detailed Routing, No DRC violations after detailed routing.
[STEP 20] : Writing Verilog, Running parasitics-based static timing analysis.
[STEP 21] : Running SPEF Extraction at the min process corner.
[STEP 22] : Running Multi-Corner Static Timing Analysis at the min process corner.
[STEP 23] : Running SPEF Extraction at the max process corner.
[STEP 24] : Running Multi-Corner Static Timing Analysis at the max process corner.
[STEP 25] : Running SPEF Extraction at the nom process corner...
[STEP 26] : Running Single-Corner Static Timing Analysis at the nom process corner...
[STEP 27] : Running Multi-Corner Static Timing Analysis at the nom process corner...
[STEP 28] : Running Magic to generate various views, Streaming out GDS-II with Magic, Generating MAGLEF views...
[STEP 29] : Streaming out GDS-II with Klayout...
[STEP 30] : Running XOR on the layouts using Klayout...
[STEP 31] : Running Magic Spice Export from LEF...
[STEP 32] : Writing Powered Verilog.
[STEP 33] : Writing Verilog.
[STEP 34] : Running LEF LVS.
[STEP 35] : Running Magic DRC, Converting Magic DRC Violations to Magic Readable Format, Converting Magic DRC Violations to Klayout Database, Converting DRC Violations to RDB Format, No DRC violations after GDS streaming out, Running Antenna Checks.
[STEP 36] : Running OpenROAD Antenna Rule Checker.
[STEP 37] : Running CVC, Saving final set of views, 
Saving runtime environment, 
Generating final set of reports, Created manufacturability report at 'designs/riscv/runs/RUN_2022.06.07_10.39.52/reports/manufacturability.rpt', 
Created metrics report at 'designs/riscv/runs/RUN_2022.06.07_10.39.52/reports/metrics.csv', 
There are no max slew violations in the design at the typical corner, There are no hold violations in the design at the typical corner, There are no setup violations in the design at the typical corner.

[SUCCESS]: Flow complete.

```


### VLSI INTERACTIVE OPENLANE FLOW

```

cd OpenLane/ 
make mount 
{ If Error occurs use the below commands in OpenLane directory:
sudo chown $USER /var/run/docker.sock 
PYTHON_BIN=python3 make mount
}

./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
run_floorplan
run_placement
run_cts
run_routing
run_magic
run_magic_spice_export
run_magic_drc
run_netgen
run_magic_antenna_check


