# RISCV Tapeout Week3 🚀

This repository documents Week 3 of the RISCV tapeout program, covering synthesis of BabySoC, gate-level simulation (GLS), static timing analysis (STA) fundamentals, and post-synthesis STA of BabySoC.

---

<details>
<summary>Synthesis and GLS of BabySoC</summary>

## Synthesis and GLS of BabySoC

Gate-level simulation is performed to verify the functionality of a design after synthesis, using actual logic gates.

### Steps to Perform Synthesis of BabySoC

1. **Launch Yosys from the VSDBabySoC/src directory:**
    ```sh
    cd VSDBabySoC/src/
    yosys
    ```
    ![Yosys launch](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p1.png)

2. **Read the Verilog files for vsdbabysoc and all instantiated modules:**
    ```sh
    read_verilog ./module/vsdbabysoc.v
    read_verilog -I./include ../output/compiled_tlv/rvmyth.v
    read_verilog -I./include ./module/clk_gate.v
    ```
    ![Read Verilog files](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p2.png)

3. **Read the library files for standard cells and analog blocks (PLL and DAC):**
    ```sh
    read_liberty -lib ./lib/avsdpll.lib
    read_liberty -lib ./lib/avsddac.lib
    read_liberty -lib ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    ```
    ![Read library files](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p3.png)

4. **Synthesize the top module:**
    ```sh
    synth -top vsdbabysoc
    ```
    ![Synthesize top module](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p4.png)

5. **Map flip-flops to standard cells:**
    ```sh
    dfflibmap -liberty ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    ```
    ![Map flip-flops](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p5.png)

6. **Optimize design and perform technology mapping:**
    ```sh
    opt
    abc -liberty ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
    ```

7. **Final cleanup: flatten design, set undefined values, clean unused instances/wires, and rename:**
    ```sh
    flatten
    setundef -zero
    clean -purge
    rename -enumerate
    ```
    ![Final cleanup](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p6.png)

8. **Print synthesis statistics:**
    ```sh
    stat
    ```
    ![Synthesis statistics](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p7.png)

9. **Write the synthesized netlist for GLS:**
    ```sh
    write_verilog -noattr ../output/synth/vsdbabysoc.synth.v
    ```
    ![Write netlist](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p8.png)

---

### Steps to Perform GLS of BabySoC

1. **Simulate the netlist:**
    ```sh
    iverilog -o ./output/post_synth_sim/post_synth_sim.out \
    -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 \
    ./src/module/testbench.v -I ./src/include/ -I src/module/ -I src/gls_model/ -I ./output/synth/
    ```
    - `-o` specifies the output file name.
    - `POST_SYNTH_SIM`, `FUNCTIONAL`, and `UNIT_DELAY` macros are enabled.
    - Testbench and include directories are specified.

2. **Execute the simulation and view the waveform:**
    ```sh
    ./output/post_synth_sim/post_synth_sim.out
    gtkwave post_synth_sim.vcd
    ```
    ![GLS waveform](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p9.png)

- **Complete GLS waveform:**
    ![Complete waveform](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p10.png)

- **Zoomed waveform (reset):**
    ![Zoomed reset](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p11.png)

- **Zoomed waveform (OUT transition):**
    ![Zoomed OUT](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d1p12.png)

---

### Observations

- The `RV_TO_DAC[9:0]` waveform shows a pattern of increasing and decreasing values, representing the sum of first n integers and then in reverse order.
- Before reset, the output of rvmyth is undefined; after reset, valid digital output is produced.
- Post-reset, rvmyth outputs 17, then 0, 1, 3, ... (sum of first n integers).
- The clock (`CLK`) has 8 cycles for every 1 cycle of `REF`, confirming the PLL is an 8x multiplier.
- The value of `OUT` (analog) transitions from 0 to 1 when the output of RVMYTH crosses the halfway mark of 512.
- `VCO_IN` waveform matches `REF` with some delay.
- `Enb_VCO` is set to 1 for normal PLL operation.
- `VREFH` is set to 3.3V and `VREFL` to 0V for DAC reference voltage.

**Conclusion:**  
The response of all signals in GLS matches the results of functional simulation, confirming:
```
GLS == Functional output
```

</details>

---

<details>
<summary>STA Fundamentals</summary>

## STA Fundamentals

**Timing Path Analysis:**
- Combinational paths are checked between start points (flop clock pin/input port) and endpoints (flop data pin/output port).
- **Arrival Time:** Time taken by a signal to travel from start to endpoint.
- **Required Time:** Time required for a signal to meet setup and hold constraints at endpoint.
- **Slack:** Difference between required and arrival time; negative slack indicates setup/hold constraint violation.

**Types of Setup/Hold Analysis:**
- Reg2reg: Register to register paths
- In2reg: Input port to register
- Reg2out: Register to output ports
- In2out: Input to output ports
- Clock gating: Clock gating flop to clock pin of flop
- Recovery/Removal: Register to reset/set pin of flop
- Data-to-data check: Checks for signals propagating reset value
- Latch timing: Latches provide time borrowing/giving in pipelines

**Other Timing Properties:**
- Slew (transition analysis): Data max/min—max for power, min for timing
- Clock max/min: Tighter margin due to frequent switching
- Load: Fanout and capacitance max/min
- Clock latency/skew: Difference in clock arrival times at flops
- Pulse width: Ensure no excessive degradation

**Graph-Based Timing Analysis:**
- Delays converted to Directed Acyclic Graph (DAG) nodes
- Actual Arrival Time (AAT): Computed from delays in DAG, max value for setup, min for hold at nodes with multiple fan-in
- Required Arrival Time (RAT): Expected signal transition time, computed by backtracing constraints from endpoint
- Slack: RAT - AAT, calculated for all DAG nodes to locate violations

**Analysis Techniques:**
- GBA (Graph-Based Analysis): Considers worst-case delays
- PBA (Path-Based Analysis): Pin/node convention; logic gate delays mapped to gate pins

**Transistor-Level Analysis:**
- Flops made of back-to-back positive/negative latches
- Setup time: Minimum pre-edge stability interval for data; includes inverter and transmission gate delays
- Clk-to-Q delay: Time from clock to Q output via transmission gate/inverter
- Hold time: Often zero if value is pre-stored

![Transistor-level timing](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d2p1.png)

**Jitter and Noise Margin:**
- Eye diagram: Overlapping clock waveforms showing voltage droop/bounce
- Jitter extraction: Noise region and reliable data window identified for STA calculations

![Eye diagram](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d2p2.png)

**OCV (On-Chip Variation):**
- Etching differences: Affect gate width/length, drain current, and delay
- Oxide thickness: Impacts MOSFET capacitance/resistance and delay
- Delay histogram: Shows increase or decrease (derate) from nominal delay

**Clock Push/Pull:**
- Push: Positive delay addition in clock path
- Pull: Negative delay addition in clock path
- Setup analysis: Capture clock pulled
- Hold analysis: Launch clock pulled, capture clock pushed

**Pessimism Removal:**  
Common clock path derates removed as pessimism is not warranted in shared paths.

</details>

---

<details>
<summary>STA of BabySoC</summary>

## STA of BabySoC

### Installing OpenSTA

1. **Install required packages:**
    ```sh
    sudo apt-get install cmake clang gcc tcl swig bison flex libeigen3-dev libz-dev tcl-dev
    ```

2. **Clone the OpenSTA repository:**
    ```sh
    git clone https://github.com/parallaxsw/OpenSTA
    ```

3. **Build OpenSTA:**
    ```sh
    mkdir OpenSTA/build && cd OpenSTA/build
    cmake ..
    make
    ```

4. **Invoke OpenSTA:**
    ```sh
    sta
    ```
    ![OpenSTA launch](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d3p1.png)

---

### Post-Synthesis STA of BabySoC

**Input scripts for OpenSTA:**  
Execute from `./BabySoC/VSDBabySoC/src` directory:

```sh
read_liberty -min ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -max ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -min ./lib/avsdpll.lib
read_liberty -max ./lib/avsdpll.lib
read_liberty -min ./lib/avsddac.lib
read_liberty -max ./lib/avsddac.lib
read_verilog ../output/synth/vsdbabysoc.synth.v
link_design vsdbabysoc
read_sdc ./sdc/vsdbabysoc_synthesis.sdc

report_checks -fields {nets cap slew input_pins fanout} -digits {4} -path_delay max -sort_by_slack > setup_report.txt
report_checks -fields {nets cap slew input_pins fanout} -digits {4} -path_delay min -sort_by_slack > hold_report.txt
```

**Script Steps:**
1. Read libraries for standard cells and analog macros (PLL and DAC) using `-max` and `-min` for setup and hold analysis.
2. Read the synthesized netlist.
3. Link the netlist with libraries (successful linking outputs `1`).
4. Read timing constraints.
5. Report timing and save to text files.

---

### Setup and Hold Reports

- **Setup Report:**
    ![Setup report](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d3p2.png)

    - Example: Reg2reg setup timing path from reg `_10450_` to `_10015_`, passing with a slack of 1.060 ns (critical path for setup).

- **Hold Report:**
    ![Hold report](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d3p3.png)

    - Example: Reg2reg hold timing path from reg `_9493_` to `_10335_`, passing with a slack of 0.3096 ns (critical path for hold).

---

### Visualizing Timing Reports

- **Pathview:**  
  Pathview is a tool to visualize timing reports from OpenSTA and Primetime.  
  [Installation and usage details](https://github.com/kanndil/PathView/tree/1e3ac1b9517269c97a6a94d829ca40cedc8273f3)

- **Timing Graphs:**
    ![Setup path graph](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d3p4.png)
    ![Hold path graph](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%203%3A%20BabySoC%20synthesis%2CGLS%20and%20postsynth%20STA/Image%20W3/W3d3p5.png)

</details>