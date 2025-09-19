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

4. **SoC design flow:**  The design is divided into two components.  
    i. **Processor**
    - A gate-level netlist is created.
    - Full Physical Design (PD) flow is performed, including floorplanning, placement, routing, CTS, STA, and signoff.

    ii. **Peripherals / IPs**
    - These are modular blocks that can be reused across designs.
    - There are two types of IPs:
        - **Digital IPs (Macros):**
            - Can be synthesized.
            - Used for standard digital logic functions.
        - **Analog IPs:**
            - Interact with external analog signals.
            - Designed using MOSFET transistors.
            - Require functional RTL only; synthesis is not needed.

    Then all these components are integrated together with GPIOs for designing the hardware. The output of the SoC is then measured(O3) and compared with RTL architecture for testing.



IMAGE HERE



5. **Physical Design Flow**: The integrated SoC is now converted to logic gates and the gates are planned on a physical die.  

    i. **Physical Design**
    - Floorplanning
    - Placement
    - CTS
    - Routing  

    ii. **GDSII generation** : A Graphical data stream information interchange (GDSII) contains information about layers required for fabrication of chip.  

    iii. Verification checks
    - **DRC (Design Rule Check)** ensures manufacturability.
    - **LVS (Layout vs. Schematic)** ensures functional correctness.

    iv. Tapeout and Tapein  
    - **Tapeout**: GDSII is sent to the foundry for fabrication.
    - **Tapein**: Fabricated chips are received from the foundry.  

    v. Board-Level Validation
    - The chip is packaged and tested at board level.
    - Output (**O4**) is compared with expected SoC output.



IMAGE HERE

IMAGE HERE



### Tool Installation:

**Yosys**

Command used

```
$ sudo apt-get update

$ git clone https://github.com/YosysHQ/yosys.git

$ cd yosys

$ sudo apt install make (If make is not installed please install it)

$ sudo apt-get install build-essential clang bison flex \

libreadline-dev gawk tcl-dev libffi-dev git \

graphviz xdot pkg-config python3 libboost-system-dev \

libboost-python-dev libboost-filesystem-dev zlib1g-dev

$ make config-gcc

$ make

$ sudo make install
```


IMAGE HERE



**Iverilog**

Command Used



```
$ sudo apt-get update

$ sudo apt-get install iverilog
```


IMAGE HERE



**gtkwave**

Command Used


```
$ sudo apt-get update

$ sudo apt-get install iverilog
```


IMAGE HERE

</details>

