# RISC-V\_Tapeout\_Programm

RISC-V tapeout program Organized by IITGN and VSD



\## \*\*Week 0:\*\*

\### Getting started with Digital VLSI SOC Design and Planning:

A chip is designed with a goal of running any specific application. The application that we want to perform in our chip is first coded in a programming language like C.

Steps in Chip modelling:

1\. \*\*GCC:\*\* In the first step we want to ensure that the application is correct by construction with the GCC compiler.

In this step we compile the C code in GCC and measure the output/response of the code/appplication (O0).

2\. \*\*Specification modelling:\*\* In this step we model the specification of the chip in a C environment and test its output with the GCC compiled code. The response/output of this steps(O1) must match with the GCC compiled code using a C language testbench.

3\. \*\*RTL Architecture:\*\* After specs of chip are finalized a soft copy of hardware language is written which is an abstract version of hardware language like Verilog. This is an intermediate level between HDL and C model. The architecture is tested using same testbench in C language and its output(O2) is matched with C model of previous step.

4\. \*\*SoC design flow:\*\* Now the design is divided into Processor and Peripherals/IPs.

&nbsp;	i. For Processor gate level netlist is created and full PD flow is done for it.

&nbsp;	ii. Peripherals/IPs are like blocks that can be used multiple times. We have Macros that are digital IPs that can be synthesized and Analog IPs which interact with analog signals of outside world and are designed using mosfet transistors for that we need functional RTL as synthesis is not required.

Then all these components of the design are integrated together with GPIOs for designing the hardware. 

The output of the SoC is then measured(O3) and compared with RTL architecture for testing.

