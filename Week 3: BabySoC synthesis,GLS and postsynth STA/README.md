# RISCV Tapeout Week3 🚀

This repository documents Week 3 of RISCV tapeout programm, covering synthesis of BabySoC, GLS, STA fundamentals and Post synthesis STA of BabySoC.

---

<details>
<summary> Synthesis and GLS of BabySoC</summary>

## Synthesis and GLS of BabySoC
As per our previous weeks repo gate level simulation is performed to verify the functionality of a design after the synthesis process. Here actual logic gates are considered for simulation.

### Steps to perform Synthesis of BabySoC

1. Launch Yosys from VSDBabySoC/src area:
`` cd VSDBabySoC/src/
	yosys ``

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p1.png)

2. Read the verilog files of vsdbabysoc and all synthesised modules instantiated in it (rvmyth and clk_gate)

``read_verilog ./module/vsdbabysoc.v
read_verilog -I./include ../output/compiled_tlv/rvmyth.v
read_verilog -I./include ./module/clk_gate.v``

Image W3d1p2

3. Read the library files of standard cells and analog blocks (PLL and DAC)

``read_liberty -lib ./lib/avsdpll.lib
read_liberty -lib ./lib/avsddac.lib
read_liberty -lib ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib``

Image W3d1p3

4. Synthesize the top module vsdbabysoc
``synth -top vsdbabysoc``

Image W3d1p4

5. Map the flipflops with standard cells:
``dfflibmap -liberty ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib``

Image W3d1p5

6. Optimize design and do technology mapping:
``opt
abc -liberty ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D} ``

7. Perform final cleanup like design flattening, setting undefined values to a constant value, cleaning unused inst and wires and renaming.

``flatten
setundef -zero
clean -purge
rename -enumerate
``
Image W3d1p6

8. Print statistics of synthesis:
`` stat ``

Image W3d1p7

9. Write the syn netlist for GLS:
`` write_verilog -noattr ../output/synth/vsdbabysoc.synth.v ``

Image W3d1p8

### Steps to perform GLS of BabySoC

1. Simulate the netlist using following command:

``iverilog -o ./output/post_synth_sim/post_synth_sim.out \
-DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 \
./src/module/testbench.v -I ./src/include/ -I src/module/ -I src/gls_model/ -I ./output/synth/``

Explaination: -o used to define the output generated file name
POST_SYNTH_SIM and FUNCTIONAL varialbes are enabled and UNIT_DELAY varialbe is set to 1.
testbench location is defined and include directories are mentioned including the synth netlist path, std cell verilog files and other related files.


2. Execute the output file of simulation and view the waveform in gtkwave
`` ./output/post_synth_sim/post_synth_sim.out
gtkwave post_synth_sim.vcd``

Image W3d1p9

GLS waveform:
Complete waveform
 Image W3d1p10

Zoomed waveform showing reset:

Image W3d1p11

Zoomed waveform showing transition of Out

Image W3d1p12

Observation:
From GLS the RV_TO_DAC[9:0] waveform is a pattern of increasing and decreasing waveform as it is sum of first n integers and then in reverse order.

Before reset is applied the output of rvmyth is undefined after reset is applied it starts giving valid diigtal output.
The output of rvmyth post reset starts with 17 then 0,1,3,.. sum of first n integers as per functional simulation.
       Clk has 8 cycles in 1 cycle of REF confirming that the PLL is 8x multipliers as per functional simulation. 

Value of out is analog which is in digital format and transitions from 0 to 1 when output of RVMYTH crosses the halfway mark of 512.

Waveform of VCO_IN is same as REF with some delay as per functional simulations. 
Enb_VCO is set to 1 which is required for normal operation of PLL.
VREFH is set to 3.3v and VREFL to 0V for input to DAC reference voltage.


Conclusion: The response of all signals in GLS are observed and are similar with the result of functional simulation, hence.
		GLS == Functional output


</details>

<details>

<summary>STA fundamentals</summary>

## STA fundamentals

    • Timing Path Analysis: Combinational paths are checked between start points (flop clock pin/input port) and endpoints (flop data pin/output port).
    • Arrival Time: Time taken by a signal to travel from start to endpoint.
    • Required Time: Time required for a signal to meet setup and hold constraints at endpoint.
    • Slack: Difference between required and arrival time; negative slack indicates setup/hold constraint violation.
Types of Setup/Hold Analysis:
    • Reg2reg: Register to register paths
    • In2reg: Input port to register
    • Reg2out: Register to output ports
    • In2out: Input to output ports
    • Clock gating: Clock gating flop to clock pin of flop
    • Recovery/Removal: Register to reset/set pin of flop
    • Data-to-data check: Checks for signals propagating reset value
    • Latch timing: Latches provide time borrowing/giving in pipelines
Analysis of Other Timing Properties:
    • Slew (transition analysis): Data max/min—max for power, min for timing
    • Clock max/min: Tighter margin due to frequent switching
    • Load: Fanout and capacitance max/min
    • Clock latency/skew: Difference in clock arrival times at flops
    • Pulse width: Ensure no excessive degradation


Graph-Based Timing Analysis:
    • Delays converted to Directed Acyclic Graph (DAG) nodes
    • Actual Arrival Time (AAT): Computed from delays in DAG, max value for setup, min for hold at nodes with multiple fan-in
    • Required Arrival Time (RAT): Expected signal transition time, computed by backtracing constraints from endpoint
    • Slack: RAT - AAT, calculated for all DAG nodes to locate violations
Analysis Techniques:
    • GBA (Graph-Based Analysis): Considers worst-case delays
    • PBA (Path-Based Analysis): Pin/node convention; logic gate delays mapped to gate pins

Transistor-Level Analysis:
    • Flops made of back-to-back positive/negative latches
    • Setup time: Minimum pre-edge stability interval for data; includes inverter and transmission gate delays
    • Clk-to-Q delay: Time from clock to Q output via transmission gate/inverter
    • Hold time: Often zero if value is pre-stored

    Image W3d2p1

Jitter and Noise Margin:
    • Eye diagram: Overlapping clock waveforms showing voltage droop/bounce
    • Jitter extraction: Noise region and reliable data window identified for STA calculations

OCV (On-Chip Variation):
    • Etching differences: Affect gate width/length, drain current, and delay
    • Oxide thickness: Impacts MOSFET capacitance/resistance and delay
    • Delay histogram: Shows increase or decrease (derate) from nominal delay
Clock Push/Pull:
    • Push: Positive delay addition in clock path
    • Pull: Negative delay addition in clock path
    • Setup analysis: Capture clock pulled
    • Hold analysis: Launch clock pulled, capture clock pushed
Pessimism Removal: Common clock path derates removed as pessimism is not warranted in shared paths

</details>

<details>

<summary>STA of BabySoC</summary>

## STA of BabySoC

Steps to install of OpenSTA:
1. Install required packages:
``sudo apt-get install cmake clang gcc tcl swig bison flex libeigen3-dev libz-dev tcl-dev``

2. Clone the OpenSTA repository:

``git clone https://github.com/parallaxsw/OpenSTA``

3. Build Open STA
``mkdir OpenSTA/build && cd OpenSTA/build
cmake ..
make
``
4. Invoke Opensta using:
``sta``

Image W3d3p1

Post Synthesis STA of BabySoC:

Input scripts for OpenSTA:

Execute from ./BabySoC/VSDBabySoC/src area:

``read_liberty -min ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -max ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -min ./lib/avsdpll.lib
read_liberty -max ./lib/avsdpll.lib
read_liberty -min ./lib/avsddac.lib
read_liberty -max ./lib/avsddac.lib
read_verilog ../output/synth/vsdbabysoc.synth.v
link_design vsdbabysoc
read_sdc ./sdc/vsdbabysoc_synthesis.sdc

report_checks -fields {nets cap slew input_pins fanout} -digits {4} -path_delay max -sort_by_slack > setup_report.txt
report_checks -fields {nets cap slew input_pins fanout} -digits {4} -path_delay min -sort_by_slack >  hold_report.txt``


Parts of script:
1. Reading library: Libraries of standard cells and analog macros PLL and DAC are read. `-max` and `-min` option are used for setup and hold analysis respectively.

2. Reading the synthesized netlist

3. Linking the netlist with libraries: If linking is succesful it will output `1` in terminal.

4. Reading timing constraints:
5. Reporting timing and saving in a txt file.

Setup reports:
Image W3d3p2

Observation: 
It is a reg2reg setup timing path from reg `_10450_` to `_10015_`, the reg2reg datapath contains 1 clkinv (8121) and 1 o211ai (8599) cells. Having a total delay of 9.7556 ns which is data arrival time. Required data time is 11ns (clock period) – 0.1386ns (library setup time) = 10.8614ns.

As arrival time is less than required time it is passing with a slack of 10.8614 – 9.7556 = 1.060ns.

This setup path is with least slack so it is a critical path for setup.

Hold report 
Image W3d3p3

It is a reg2reg hold timing path from reg `_9493_` to `_10335_` there is no combinational logic between the two regs so the data arrival time is the delay of the startpoint which is 0.2749ns.
The data required time is library hold time which is -0.0346ns. As Arrival time is after the the required time the path is passing for hold with a slack of 0.2749 - (-0.0346) = 0.3096ns.

This hold path is with least slack so it is a critical path for hold.



To view graphical report we installed Pathview a powerful tool to visualize timing report from opensta and primetime.
Details on installation and usage can be found here: (https://github.com/kanndil/PathView/tree/1e3ac1b9517269c97a6a94d829ca40cedc8273f3)

Following is the timing graph of the setup and hold paths discussed previously.

Image W3d3p4
Image W3d3p5


</details>