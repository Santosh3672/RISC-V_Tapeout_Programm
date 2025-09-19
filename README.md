# RISCV Tapeout program

<details>

&nbsp;	<summary> Week 0:  Digital VLSI SoC Planning and Tool Installation </summary>

## Week 0: Digital VLSI SoC Planning and Tool Installation

### Getting started with Digital VLSI SOC Design and Planning:

A chip is designed with a goal of running any specific application. The application that we want to perform in our chip is first coded in a programming language like C.

Steps in Chip modelling:

1. **GCC:** In the first step we want to ensure that the application is correct by construction with the GCC compiler.

In this step we compile the C code in GCC and measure the output/response of the code/application (O0).

2. **Specification modelling:** In this step we model the specification of the chip in a C environment and test its output with the GCC compiled code. The response/output of this steps(O1) must match with the GCC compiled code using a C language testbench.

3. **RTL Architecture:** After specs of chip are finalized a soft copy of hardware language is written which is an abstract version of hardware language like Verilog. This is an intermediate level between HDL and C model. The architecture is tested using same testbench in C language and its output(O2) is matched with C model of previous step.

4. **SoC design flow:** Now the design is divided into Processor and Peripherals/IPs.

 	i. For Processor gate level netlist is created and full PD flow is done for it.

 	ii. Peripherals/IPs are like blocks that can be used multiple times. We have Macros that are digital IPs that can be synthesized and Analog IPs which interact with analog signals of outside world and are designed using mosfet transistors for that we need functional RTL as synthesis is not required.

Then all these components of the design are integrated together with GPIOs for designing the hardware.

The output of the SoC is then measured(O3) and compared with RTL architecture for testing.



IMAGE HERE



5. **Physical Design Flow**: The integrated SoC is now converted to logic gates and the gates are planned on a physical die. It includes Floowplanning, Placement, CTS, Routing.

After Physical design we generate GDSII file (graphical data stream information interchange) it contains information about the layers that are required for fabrication.

On the GDSII we check DRC to check if it can be manufacture and LVS to check it functions as per the SoC . After these checks are passed we send the GDSII to fabrication which is called as **tapeout**,

After fabrication we get the chips back from foundry which is called **tapein**, the chip is then used for package and the output of the chip(O4) is measured at board level which is again compared with SoC output.



IMAGE HERE

IMAGE HERE



### Tool Installation:

**Yosys**

Command used



`$ sudo apt-get update

$ git clone https://github.com/YosysHQ/yosys.git

$ cd yosys

$ sudo apt install make (If make is not installed please install it)

$ sudo apt-get install build-essential clang bison flex \\

libreadline-dev gawk tcl-dev libffi-dev git \\

graphviz xdot pkg-config python3 libboost-system-dev \\

libboost-python-dev libboost-filesystem-dev zlib1g-dev

$ make config-gcc

$ make

$ sudo make install`



IMAGE HERE



**Iverilog**

Command Used



`$ sudo apt-get update

$ sudo apt-get install iverilog`



IMAGE HERE



**gtkwave**

Command Used



`$ sudo apt-get update

$ sudo apt-get install iverilog`



IMAGE HERE

</details>

