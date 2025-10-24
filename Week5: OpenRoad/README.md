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

### Troubleshooting Guide

#### When Missing Packages Are Reported:

1. **Log Review**
   - Check `build_openroad.log` for specific error messages
   - Look for dependency requirements and version conflicts

2. **Installing Dependencies**
   - Install missing packages using system package manager
   - Follow error messages to identify required packages
   - Verify package versions match requirements

3. **Rebuild Process**
   ```bash
   ./build_openroad.sh --clean  # Clean previous build
   ./build_openroad.sh --local  # Rebuild with local settings```

4. **Iterative Resolution**
    - Address each missing dependency
    - Rebuild after installing packages
    - Continue until build completes successfully


</details>

<details>
<summary>Directory Structure of OpenROAD</summary>

## Directory Structure of OpenROAD

### OpenROAD-flow-scripts/

OpenROAD-flow-scripts/  
├── Tools/ # Tools used in the OpenROAD flow  
│ ├── Autotuner # Machine learning optimization module  
│ ├── OpenROAD # Core OpenROAD tool  
│ └── yosys # Logic synthesis tool  
│  
├── Flow/ # RTL to GDSII flow structure  
├── docs/ # OpenROAD documentation  
├── Docker/ # Docker image configurations  
├── Jenkins/ # CI/CD pipeline setup  
├── etc/ # Dependency installer scripts  
└── setup_env.sh # Environment setup script  

### Flow Directory Structure

Flow/  
├── design/ # Design files for different nodes  
├── platform/ # Technology node specific data  
│ ├── libs/ # Library files  
│ ├── lef/ # Layout files  
│ ├── gds/ # Physical layout data  
│ └── drc/ # Design rule check files  
│  
├── scripts/ # RTL to GDS flow scripts  
├── test/ # Test configurations  
├── tutorial/ # Tutorial materials  
├── util/ # Utility scripts  
└── Makefile # Build automation file  



</details>

<details>
<summary>Floorplan and Placement with Openroad</summary>

## Floorplan and Placement with Openroad

### Initial Setup
The design configuration is specified in `flow/Makefile`:
- Design variable: `DESIGN_CONFIG ?= ./designs/nangate45/gcd/config.mk`
- Uses GCD design on nangate45 (45nm open-source PDK)

### Configuration Details (config.mk)
1. **Design Inputs**
   - Design name
   - Verilog files (RTL)
   - Constraints file (.sdc)
2. **Platform Configuration**
   - Platform name: nangate45
3. **Synthesis Parameters**
   - ABC_AREA settings
4. **Floorplan/Placement Inputs**
   - Core utilization
   - PDN TCL file


### Running Logic Synthesis for GSD design on nangate45:
Use following command to run synthesis in the flow directory:  
`make synth`  

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p5.png)

**Generated Directories:**
- Log/: Contains synthesis logs  
- Reports/: Synthesis reports  
- Results/: Output files for next steps  


![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p6.png)

Synth stattistics for gcd design  

### Executing Floorplan on synth netlist:  

Use command `make floorplan`  

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p7.png)
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p8.png)

In log file following subtasks logs are generated:

1. **2_1_floorplan**
   - Initializes design
   - Reads libs and LEF files
   - Checks setup
   - Repairs tie_lo and tie_hi fanout

2. **2_2_floorplan_macro**
   - Handles macro placement
   - Places large macro blocks

3. **2_3_floorplan_tapcell** 
   - Adds tapcells
   - Manages substrate/well connections

4. **2_4_floorplan_pdn**
   - Adds PDN grid
   - Creates power distribution network

These are the subtasks of floorplan.

In the results directory, OpenROAD Database (ODB) files are created for each intermediate step with the same name as the log files. Additionally generated:
- SDC files
- floorplan.tcl
- Final ODB (2_floorplan.odb)

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p9.png)

To view floorplan in OpenROAD GUI:
```bash
cd results/nangate45/gcd/base/
openroad -gui -db 2_floorplan.odb
```

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p10.png)

### Executing Placement on Floorplan DB:

Use the command `make place` to execute placement on the floorplan DB.

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p11.png)
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p12.png)

### Placement Subtasks and Outputs

1. **3_1_place_gp_skip_io (Global Placement - Skip IO)**
   - Performs initial global placement when IO pins are unplaced
   - Optimizes cell positions before IO pin assignment
   - Generates initial placement database
   ![Global Placement Skip IO](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p13.png)

2. **3_2_place_iop (IO Placement)**
   - Handles IO pin placement if not already completed
   - Places pins around chip periphery
   - Creates database with placed IO pins
   ![IO Placement](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p14.png)

3. **3_3_place_gp (Global Placement with IO)**
   - Executes complete global placement including placed IO pins
   - Optimizes overall cell locations
   - Produces globally placed design database
   ![Global Placement](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p15.png)

4. **3_4_place_resized (Cell Resizing)**
   - Optimizes cell sizes
   - Adds buffers to critical nets
   - Creates resized and buffered design database
   ![Cell Resizing](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p16.png)

5. **3_5_place_dp (Detailed Placement)**
   - Performs final detailed placement
   - Legalizes all cell positions
   - Ensures design rule compliance
   - Generates final placed design database
   ![Detailed Placement](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week5%3A%20OpenRoad/Image%20W5/W5p17.png)

To view the final placement results:
```bash
cd results/nangate45/gcd/base/
openroad -gui -db 3_place.odb
```

## Conclusion:
In this repository we were able to install openroad-flor_scripts in our system and perform Floorplan and Placement of GCD design at nanogate45 platform.


</details>
