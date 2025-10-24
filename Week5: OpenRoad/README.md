# RISCV Tapeout Week5

This repository documents Week 5 of the RISC-V Tapeout Program. It covers an introduction to OpenROAD, its installation process, and the use of OpenROAD to perform floorplanning and placement on a GCD design.

---

<details>
<summary>Introduction to Openroad</summary>

## Introduction to Openroad

OpenROAD is an open-source project focused on simplifying and democratizing VLSI digital design. It provides an autonomous, 24-hour RTL-to-GDSII flow that requires no manual intervention. The project plays a vital role in the open-source hardware ecosystem by enabling startups, academic researchers, and independent engineers to access advanced IC design capabilities without relying on costly proprietary tools or specialized expertise.

Key Features:
• Fully automated digital design flow from RTL to GDSII  
• Open-source, permissively licensed EDA suite supporting modern design methodologies  
• Integrated machine learning tools for optimization (e.g., AutoTuner)  
• Cloud and distributed computing support for faster design turnaround (COPILOT)  
• Strong community-driven development with widespread adoption in global tapeouts


</details>


<details>
<summary>Installation of Openroad</summary>

## Installation of Openroad
Steps to install Openroad:

1. Clone the Openroad repository:
```git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts
cd OpenROAD-flow-scripts```
2. Run the setup script:
``` sudo ./setup.sh
```

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p1.png)

3. Build the OpenRoad with local setting:
```
./build_openroad.sh --local
```
A log file is dumped `build_openroad.log`
Check it for any error and fix any missing packages like cudd.

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p2.png)


4. Verify Installation
``source ./env.sh
yosys -help  
openroad -help``
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p3.png)

Yosys installation
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p4.png)
Openroad installation

During installation of Openroad I faced challenges in missing packages. To resolve them I had to fix those packages and reinstall openroad, after 3 such iterations all issues were resolved. To resolve the issues faster I used perplexity AI and applied the solutions. 

</details>

<details>
<summary>Directory Structure</summary>

## Directory Structure

OpenRoad directory structure:
OpenROAD-flow-scripts:
	-Tools: Contains all the tools for Openroad flow: Autotuner, Openroad, yosys, etc
	- Flow: File structure to run RTL to GDS flow for designs
	- docs: Documentation for Openroad and the flow
	- Docker: Include openroad version in docker image
	- Jenkins: To manage continuous integration pipelines for rapid, automated build and test verification
	- etc: Has dependency installer script
	- setup_env.sh: Source file to setup Openroad


Flow directory structure:
    - design: contains design information for different technology nodes
    - platform: contains data for different technodes like, libs,lef,gds,drc rules
    - scripts: scripts for RTL to GDS flow
    - test: 
    - tutorial
    - util
    - Makefile: Make build automation file for Openroad

</details>

<details>
<summary>Floorplan and Placement with Openroad</summary>

## Floorplan and Placement with Openroad

nside flow/Makefile file we can see that the design variable is set to `DESIGN_CONFIG ?= ./designs/nangate45/gcd/config.mk`
It is a GCD design on nangate45 technode an opensource PDK on 45nm.

In the config.mk file we can see following information present:
1. Design input: Design name, Verilog files(RTL), and constraints file(.sdc).
2. Platform name:
3. Inputs for synthesis: ABC_AREA
4. Inputs for Floorplan and placement: Core utilization, PDN tcl file.


### Running Logic Synthesis for GSD design on nangate45:
Use following command to run synthesis in the flow directory:
`make synth`

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p5.png)

It will create following directories:
1. Log
2. Reports
3. Results: for subsequent steps

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p6.png)

Synth stat for gcd 

### Executing Floorplan on synth netlist:
Use command `make floorplan`

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p7.png)
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p8.png)

In log file following subtasks logs are generated:
1. 2_1_floorplan: Initializes design, reads libs,lef, checks setup and reapaits tie_lo and tie_hi fanout
2. 2_2_floorplan_macro:  Macro placement
3. 2_3 floorplan_tapcell: Adds tapcell
4. 2_4 floorplan_pdn: Adds PDN grid.
These are the subtasks of floorplan.


In the results directory Openroad Database (odb) are created for each intermediate steps with same name as that of log files. Along with that sdc and floorplan.tcl file and final odb 2_floorplan.odb is also generated. 

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p9.png)

To view floorplan view with openroad use command 
```cd results/nangate45/gcd/base/
openroad -gui -db  2_floorplan.odb```

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p10.png)

### Executing Placement on Floorplan DB:

Use the command `make place` to execute placement on the floorplan DB.

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p11.png)
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p12.png)

Placement has following subtasks under it:
1. 3_1_place_gp_skip_io: If Iopins are unplaced it will do global placement prior to IO placement.
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p13.png)

2. 3_2_place_iop: Perform IO placement if it is not done
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p14.png)

3. 3_3_place_gp: Global placement with Iopins placed.
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p15.png)

4. 3_4_place_resized: Performs resizing of cells and buffering of nets
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p16.png)

5. 3_5_place_dp: Detail placement stage
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p17.png)

Conclusion:
In this repository we were able to install openroad-flor_scripts in our system and perform Floorplan and Placement of GCD design at nanogate45 platform.


</details>
