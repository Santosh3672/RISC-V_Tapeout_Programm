# RISCV Tapeout Week5

This repository documents Week 5 of the RISC-V Tapeout Program. It covers an introduction to OpenROAD, its installation process, and the use of OpenROAD to perform floorplanning and placement on a GCD design.

---

<details>
<summary>Introduction to Openroad</summary>

## Introduction to Openroad


OpenROAD is an innovative open-source project that aims to revolutionize VLSI digital design by making it more accessible and automated. Its primary goal is to provide a complete RTL-to-GDSII flow that can run autonomously within 24 hours without requiring manual intervention.

### Key Features

1. **Automated Design Flow**
   - Complete RTL to GDSII automation
   - No manual intervention required
   - 24-hour turnaround target

2. **Open Source Benefits**
   - Permissively licensed EDA suite
   - Supports modern design methodologies
   - Enables broader access to IC design capabilities

3. **Advanced Technologies**
   - Integrated machine learning optimization via AutoTuner
   - Cloud computing support through COPILOT
   - Distributed computing capabilities

4. **Community-Driven Development**
   - Active global community
   - Proven track record in successful tapeouts
   - Continuous improvements and updates

### Target Users
- Startups developing custom silicon
- Academic researchers
- Independent engineers
- Companies seeking open-source alternatives

This tool democratizes IC design by removing barriers like:
- High costs of proprietary tools
- Need for specialized expertise
- Limited access to advanced design capabilities



</details>


<details>
<summary>Installation of Openroad</summary>

## Installation of Openroad

This section details the step-by-step process to install OpenROAD and its dependencies.

### Prerequisites
- Linux-based system
- Git
- C++ compiler
- Required packages (will be installed via setup script)

### Installation Steps

1. **Clone the Repository**
   ```bash
   git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts
   cd OpenROAD-flow-scripts```


2. **Run setup script**
    ```bash
    sudo ./setup.sh```

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p1.png)

3. **Build OpenRoad**
    ```bash
    ./build_openroad.sh --local```

  - Builds OpenROAD with local settings
  - Creates build_openroad.log for debugging
  - Check log for any missing dependencies (e.g., cudd package)


![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p2.png)


4. **Verify Installation**
    ```bash
    source ./env.sh
    yosys -help  
    openroad -help``

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p3.png)

Yosys installation

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p4.png)
Openroad installation

Troubleshooting Tips
If missing packages are reported:

Review build_openroad.log
Install missing dependencies
Rebuild OpenROAD
Repeat until all dependencies are satisfied
Common Issues:

Missing system packages
Compiler version mismatches
Environment variable conflicts
Note: Installation may require multiple iterations to resolve all dependencies. Using online resources like Perplexity AI can help quickly resolve package-related issues.

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
