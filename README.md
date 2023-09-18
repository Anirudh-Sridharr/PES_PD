# PES_PD
## ASIC Physical design course repository

Ok so what is a chip from the inside?

A packaging has ports to connect outside, a padding connects these ports to the logic units on the inside, riscv for example.

A chip has industry IPs[Intellectual property] namely the PLL, SRAM, ADC and DAC!

![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/29536d2e-f6c4-4035-81e2-ac175803143f)



### Openlane: 
So we're here, the new opensource tool that started out as a tapeout experiment  The main goal is to be able to produce a full 
GDSII with no violations in LVS, DRC and Timing.

It is tuned for sky130 pdk but is also compatible withXFAB180 and GF130G

In openlane you van begin with a verilog code and end with a final layout, a straight out full flow

_Openlane ASIC flow:_

Openlane is based off other opensource tools like yosys, Qflow, ABC, openroad, Magic, FAULT

Now the flow:
![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/b7e5bf9d-a269-4033-b72c-c4eece8e8174)

Synth exploration:  offers a plot for how Delay and area is affected by the used synthesis strategy
 
Design exploration: Offers a report on runtime routing strategy etc and regression testing 

DFT[Design for test]: Automatic test pattern generation, test pattern compaction, Fault coverage, Fault simulation, Scan insertion

A fabricated metal wire fragment can act as an antenna and collect charges and hence screw a lot of things up for which we either use a top layer
(Bridging or use antenna diodes)

Openlane uses fake diodes for simulation purposes and replaces during LVS 

OpenSTA and Magic come in the last


### *LAB-1*

So we see into the folders using ```cd``` and ``` ls -ltr```, we find the library definition files inside, the sky130nm technology along with other tools, 
``` pdks/sky130A/libs.ref```  has all the necessary library definition modules eg: sky130_fd_sc_hd etc
 ``` pdks/sky130A/libs.tech```  has all the tool oriented stuff like ngspice, magic, lib, etc.

All these modules used for simulating whatever we want to rig up in verilog, our code is seen in terms of these files, this is the language of our simulation


![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/b35a7a82-75d1-4f99-adca-73cdbf3d1091)
![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/778583a3-2ac4-466f-a754-01a8cc110e97)

 under  ```~/openlane/designs/picorv32``` we have 

![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/855c1a12-8fe8-46a1-8018-338cf483ee94)

The clock periods in the last 2 called files are 20 and 12 respectively. 

Priority is given highest to the sky130A module and then the config.tcl .

Then we prepare the picorv32a directory for the design flow using 
'''prep -design picorv32a'''
picorv32a here is just the directory name, not a command in itself it could be anything 

Now a "runs" directory gets created under picorv32a

![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/e738c516-461c-49d2-a94c-93dd23a7e4d9)

All these are to store data from simulations and all that have been run, results and reports will have the necessary in terms of output

There may be many config.tcl files, they aren't to be confused as their addresses and purposes are different, they store essential parameters for the software to refer and run

Openlane allows for modification of these config files for projects, you can change certain parameters to suit the design and have it work anyways, with the change applied 


Now back on the Openlane cmd instantiated using docker, we can use ```run_synthesis``` to use yosys and automatically run abc and all those syntheses 

https://github.com/efabless/openlane has everything about the openlane tool, user guides and installation. This repo can be cloned to use 


Now that our synthesis is done, we have a few parameters that we can determine using the results given out after the synthesis, like flop ratio ,

It is the [(Number of flops)/(Number of cells)]

![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/defdb92a-b577-4ea8-9b4b-29a082413743)

Now as discussed earlier, we do have reports and results directories we can get stuff out of, let's check that out

We go to the picorv32a/runs/(Dateandtime)/results/synthesis and do ```gedit picorv32a.synthesis.v``` to see the netlist generated from the synthesis

![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/62e2d77a-f909-4783-a67c-45b44347291f)

##  _*Day 2:*_

Basic chip floor planning conditions:
The first step of physical design is to define the height and width of core and die. 
So we first know the netlist then try to define a size on the core for the gates, flops and all that, assume 1x1 sq unit for standard cells and
flops then we place them occupying as less area as possible and call that the minimum area that would be occupied by the netlist on the core.
Now, a netlist may or may not occupy all the area in the core of the silicon wafer. Thus we have a term known as the utilization factor 

U = [(Area of netlist)/(Area of core)]

Usually, we have a utilization factor of about 0.5-0.6

We can also define aspect ratio: Height/width of the core. Basically about how narrow or wide the cell is 

_Modularity:_
We can pick a block of a circuit like a mux made out of gates in old computers and keep it aside, have it's ports extended
instead of replicating the circuitry and wasting resources. 

Now the placement of these cells (Modules) on the chip is called floor planning 

_*Power planning*_

_Decoupling Capacitors:_
From the supply to the transistor circuit, there's a voltage drop that the interconnect offers. This makes it possible for signals to get messed
up in the defined noise margin of the CMOS circuitry involved. To avoid this, we connect a capacitor in parallel with the CMOS network, 
that can charge up to Vdd and has minimal wiring to reach the CMOS network. This way, Vdd is always maintained at the plate of the capacitor 
as it charges from the supply rail and ensures close to maximum supply for the network as the network gets it's supply from the capacitor

During placement and floor planning, it looks like this:
![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/a3cda194-d0b9-449d-97cd-f622c2a9d94d)

Blocks are modules and DECAP is decoupling capacitor

This ensures proper voltage supply on a local scale but on the global it doesn't.

_Ground bounce:_
So a bus can be considered as a series of capacitors with logic 1 meaning fully charged and 0 meaning discharged, 
during inversion the discharged charge and the charged discharge. This causes slight instability in the output 
capacitance which may exhibit a pulse of something like high potential at the grounded terminal. This is known as ground bounce
The other way round where the positive terminal of the capacitor shows a pulse of a low voltage is called voltage droop.

These and other global issues can be avoided with the help of having multiple supplies. 
i.e. a supply mesh from above with occasional nodes to draw from instead of  just one lossy supply system

_*Pin placement:*_
We have a circuit block to be implemented, In the form of a netlist we tie the same input nodes etc, together as done during synthesis,
then we draw these lines to the wall of the die from the cells and connect them to the outside using pins
![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/ae37a20f-9fbd-4712-ac04-978cac829ab6)

Clock pins due to the nature of continuous input need low resistance paths 
We then insulate the rest of the cell from the pins individually 


###  _*Floorplanning using openlane*_

Floorplanning is next to synthesis
When we see the README.md file in
```~Desktop/work/tools/openlane_working_dir/openlane/configuration```
We have detailed descriptions of variables used during the processes with their elaborations and default values. These values can be adjusted to our need.


![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/668cb42e-d341-455e-9559-b867984df4ac)

The default floorplan parameters can be found in floorplan.tcl 


![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/dfe526c5-1588-4ad8-90b9-9b2f83d22938)

*So our priorities: floorplan.tcl < config.tcl < sty130_fd_sc_hd.tcl*

![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/a726c986-82bd-45fc-87fe-2b7459916d8f)

This is after running the floorplanning. On observing the 7-pdn.def and log files we see details of the floorplan including metal width,
clock and all that in logs and the die size and all that in picorv32a.floorplan.def


![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/579140e6-d397-4bc7-982c-72f2756751d2)

The floorplan can be viewed in text on the picorv32a.floorplan.def flie, this can also be rigged up in magic using 

```
magic -T ~Desktop/work/tools/Openlane_working_dir/pdks/sky130A/libs.tech/sky130A.tech lef  ~Desktop/work/tools/Openlane_working_dir/openlane/designs/picorv32A/runs/datetime/tmp/merged.lef def read picorv32a.floorplan.def 
```

![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/dadcb3c3-cfbd-477e-abd6-dca2044c2d7b)

This is the layout 

We zoom in to see the pins, decoupling capacitors etc.

![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/a85e58f5-3c5c-4e6c-a821-20cca101d486)

The + sign is a pin

###  _*Library binding and placement*_

An subcircuit that is made and it's netlist is bound physically, 
we can put it's parameters in the library file. Now the library can store variable parameters better worded as multiple parameters as well.
Differently scaled versions of the same cells/netlist. Now we take these defined blocks from the library and place it in the floorplan

![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/27b70ed5-e863-45a1-a199-7b3c36b0d52f)

The modules are placed as close as possible to their i/o nodes and under unavoidable situations of far distance, we place repeaters along the long path (Buf) to maintain signal integrity as the signal loses out travelling long paths


###  _placement in openlane_
We use the same command as used for viewing floorplan but rig up the file from placement and not floorplan

![image](https://github.com/Anirudh-Sridharr/PES_PD/assets/140473803/12da6884-24c3-4626-8ca2-9724fdfa5e29)

The globally placed chip. 



### _Cell design flow:_


**Inputs for cell design flow**

Cell design flow refers to the process of creating and optimizing individual digital logic cells that are part of a standard cell library. These libraries contain a set of pre-designed, characterized, and reusable logic gates, flip-flops, and other basic building blocks used in the design of integrated circuits.
These libraries include  PDK, DRC and LVS rules, SPICE models, libraries, user-defined specifications.
User derfined specifications like Pin location, drawn gate lenght are added to the libarary by the library developer.

**Circuit Design**

 Circuit design:Implment function using nmos and pmos and then derive the network graph. Derive the Euler's path and stick diagram from the graph.
 
**Layout design**
 Convert stick diagram according to the DRC rules
Extraction of parasitics,extracted spice list

**Characterization**
timing ,noise power.libs functions
Read in the models and tech files and generate extracted spice Netlist. Read the subcircuits and attach power sources. Apply stimulus to characterization setup, provide neccesary output capacitance loads and provide neccesary simulation commands.



## General timing characterization parameters

**Timing threshold definitions**

![image](https://github.com/Anirudh-Ravi123/pes_pd/assets/142154804/10949764-a9f2-4860-9f8c-90d092ea32ef)



**Propagation Delay**
The time difference between when the transitional input reaches 50% of its final value and when the output reaches 50% of its final value. 

```
Propagation delay=time(out_fall_thr)-time(in_rise_thr)
```

**Transition Time**
The time it takes the signal to move between states is the transition time , where the time is measured between 10% and 90% or 20% to 80% of the signal levels.

```
Rise transition time = time(slew_high_rise_thr) - time (slew_low_rise_thr)
```


```
Fall transition time = time(slew_high_fall_thr) - time (slew_low_fall_thr)
```








