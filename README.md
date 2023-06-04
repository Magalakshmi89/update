
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

![Screenshot 2023-05-31 164033](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/d9e6d284-c451-4f91-be64-a44a5052480e)

- Opensource RTL Designs: github, librecores, opencores
- Opensource EDA tools: QFlow, OpenROAD, OpenLANE
- Opensource PDK data: Google Skywater130 PDK

The ASIC flow objective is to convert RTL design to GDSII format.

#### Simplified RTL2GDS Flow

![Screenshot 2023-05-31 165557](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/7a8d6f60-712e-4068-9923-100fb08aab23)

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
![Screenshot 2023-05-31 135059](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/8c82a609-5a69-4aa7-abab-a5c8605d4783)


![Screenshot 2023-05-31 232255](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/93f2a8b2-86c0-4ed1-8158-d1eaca9efe54)

#### Review of files & Synthesis step
* A "runs" folder is generated within the picorv32a folder.
* The merged file is created during the merging operation in the pircorv32a design preparation (it merges lef and techlef files)
* Next, we run the synthesis of picorv32a design in the openlane interactive terminal:

`run_synthesis`


![Screenshot 2023-05-31 234758](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/75ec70a2-a8c9-4185-bd76-25bce7690f6e)

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

![Screenshot 2023-06-01 140815](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/4807dbfb-f12e-4fe1-8b4f-82bd0c2f84b4)

* The synthesis statistics report is as follows:

![Screenshot 2023-06-01 140458](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/5d1a8ff6-abf3-4636-8cd3-adbc30ad5771)


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

![Screenshot 2023-06-01 144634](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/410d3471-7ec5-4d35-abae-93f5cb06c230)

#### Decoupling capacitors

![capacitor](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/0a412ae6-6028-409c-847e-951a53eebffb)


Pre-placed cells must then be surrounded with decoupling capacitors (decaps). Its is capacitor in which is used to decouple critical cells from main power supply in order to protect from disturbance in the power distribution lines.Decaps are huge capacitors charged to power supply voltage and placed close the logic circuit. Their role is to decouple the circuit from power supply by supplying the necessary amount of current to the circuit. They pervent crosstalk and enable local communication.

#### Power Planning

Each block on the chip, however, cannot have its own decap unlike the pre-placed macros. Therefore, a good power planning ensures that each block has its own VDD and VSS pads connected to the horizontal and vertical power and GND lines which form a power mesh.

![power](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/949f330e-f2f8-45f1-be37-f50945c5445c)


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

![Screenshot 2023-06-02 112011](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/fe6eb0b7-be51-480b-95ef-fd96b57c5893)


 ![Screenshot 2023-06-02 112052](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/b7642452-e897-4efc-9d35-b162420a7248)
 
 
 ![Screenshot 2023-06-02 112146](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/f3edc3b5-ec5f-4c41-be37-fdb110c026ea)
 

![Screenshot 2023-06-02 112324](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/c16098ab-d397-416c-ae87-0c057c31c403)

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

![Screenshot 2023-06-02 125604](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/5c018148-6026-4beb-99da-f3dede7d5f36)

Zoomed-in views of the standard cell placement:

![Screenshot 2023-06-02 125645](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/03101aa0-2efd-479a-be63-3e2c51771c63)


***Note: Power distribution network generation is usually a part of the floorplan step. However, in the openLANE flow, floorplan does not generate PDN. The steps are - floorplan, placement CTS and then PDN***


### Standard Cell Design Flow

![Screenshot 2023-06-02 133703](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/48f81cd7-702b-4115-90a3-dbc32f5e3efe)

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


![Screenshot 2023-06-02 190500](https://github.com/Magalakshmi89/Sky-130-PD-Workshop--VSD-Tapeout-Program/assets/135096629/9f8f13d6-f26c-47d5-b208-3f872cbdd0a0)

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




