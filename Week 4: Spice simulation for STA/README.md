# RISCV Tapeout Week3 🚀

This repository documents Week 4 of the RISCV tapeout program, covering CMOS circuit design and Spice simulation.

---

<details>
<summary>Day 1: Basics NMOS Id vs Vds</summary>

## Day 1: Basics — NMOS Id vs Vds

NMOS and PMOS transistors form the foundation of logic gates:

- NMOS devices are N-channel MOSFETs with a P-type substrate, SiO2 isolation, n+ source/drain, gate oxide, and a gate made from polysilicon or metal. Terminals: Gate (G), Source (S), Drain (D), Body (B).
- PMOS devices use an N-type substrate and p+ diffusion regions.

A MOSFET operates in three main modes:

1. Cutoff mode
   - Condition: `VGS < Vt` (threshold voltage).
   - Source, drain, and body junctions are off; no inversion channel forms → transistor behaves like an open circuit.
   - No drain current flows.
   - Bulk (body) bias effect: a nonzero `VSB` changes `Vt` (positive `VSB` increases `Vt`).

2. Resistive / Linear / Triode region
   - Condition: `VGS > Vt` and `VDS < VGS − Vt`.
   - Drain current (exact): `ID = Kn * [(VGS − Vt) * VDS − 2 * VDS^2]`
   - For small `VDS`: `ID ≈ Kn * (VGS − Vt) * VDS` (acts like a voltage-controlled resistor).
   - `Kn` depends on process, `W`, and `L`.

3. Saturation region
   - Condition: `VGS > Vt` and `VDS ≥ VGS − Vt`.
   - Channel pinches off near the drain; current approximately independent of `VDS`.
   - Ideal saturation current: `ID ≈ (Kn / 2) * (VGS − Vt)^2`.
   - With channel-length modulation: `ID = (Kn / 2) * (VGS − Vt)^2 * (1 + λ * VDS)` where `λ` models the dependence on `VDS`.

Introduction to SPICE
- SPICE uses device models (from the foundry/PDK) and circuit netlists to compute waveforms and operating points based on the equations above and many fitted parameters (e.g., `VTH0`, `U0`, etc.).

How to perform SPICE simulation
1. Verify the setup: ensure model parameters (Kn, Vto, etc.) match the target technology node.
2. Create a SPICE netlist for the circuit.

Basic steps when writing a SPICE netlist:
- Define nodes.
- Define device elements connecting nodes, e.g.:
  - `M1 vdd n1 0 0 nmos W=1.8u L=1.2u`
    - `M1` = device name, followed by terminals, model name, and parameters.
- Include the PDK/model `.lib` file that contains device model definitions.

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d1p2.png)

Labs
- For labs we used the repository: https://github.com/kunalg123/sky130CircuitDesignWorkshop/tree/main
- Directory of interest:
  - `sky130CircuitDesignWorkshop/design/sky130_fd_pr/` — contains `cells/` and `models/`.

Objective: Plot Id vs Vds for different Vgs values.

Spice netlist used:

```spice
* Model / Netlist Description
.param temp=27

* Including sky130 library files (adjust path if needed)
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* Circuit netlist
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

* Simulation commands
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control
run
display
setplot dc1
.endc

.end
```

Components of the SPICE netlist
1. Parameter definition: `temp = 27`
2. Include `.lib` from the models directory (the `sky130_fd_pr__nfet_01v8` model is referenced).
3. Circuit elements: NFET, resistor, and voltage sources.
4. Simulation: DC sweep for `Vdd` (acts as `Vds`) and `Vin` (acts as `Vgs`).

Steps to perform SPICE simulation using ngspice:
1. Run:
   ```
   ngspice ./day1_nfet_idvds_L2_W5.spice
   ```
   ![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d1p3.png)

2. Plot the drain branch current vs Vds:
   - Example: `plot -vdd#branch`
   - Note: branch current sign may be negative due to SPICE sign conventions — use the sign or absolute value as needed.
   ![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d1p4.png)

We have Id on the Y axis and Vds on the X axis; different traces correspond to different `Vgs`.

W/L ratio = 5/2 = 2.5

</details>


<details>
<summary>Day 2: Simulation at lower technode and effect of Velocity saturation</summary>

## Day 2: Simulation at lower technode and effect of Velocity saturation

This section compares long- and short-channel MOSFET behavior (L = 2 µm vs L = 150 nm) to illustrate velocity saturation and its effect on Id–Vgs and Id–Vds characteristics.

### Short‑channel example (L = 150 nm)

Spice netlist used:
```spice
*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description



XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.375 l=0.15

R1 n1 in 55

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands

.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control

run
display
setplot dc1
.endc

.end
```

Spice waveform:

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d2p1.png)

W/L ratio = 0.375/0.15

Notes:
- L reduced to 150 nm to study velocity saturation.
- W changed from default 390 µm to 375 µm so W/L ≈ 2.5 (kept comparable to long-channel case).

Observation:
- For the short-channel device, saturation drain current shows an approximately linear dependence on Vgs (due to velocity saturation), unlike the quadratic dependence predicted for long-channel devices.


---

### Id vs Vgs — long vs short channel

Long-channel (example, L = 2 µm) netlist:
```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description

XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=2 l=5

R1 n1 in 55

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands

.op
.dc Vin 0 1.8 0.1 Vdd 0 2.5 2.5

.control

run
display
setplot dc1
.endc

.end
```

Simulation waveform (long channel):
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d2p2.png)

Observation:
- Id varies roughly quadratically with Vgs for the long-channel device (as per square-law behavior).

Short-channel Id vs Vgs netlist (repeat for comparison):
```spice
*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description



XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.375 l=0.15

R1 n1 in 55

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands

.op
.dc Vin 0 1.8 0.1 Vdd 0 2.5 2.5

.control

run
display
setplot dc1
.endc

.end
```

Simulation waveform (short channel):
![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d2p3.png)

Observation:
- Id increases approximately linearly with Vgs for the short-channel device because carriers reach velocity saturation; the square‑law no longer holds.

Definitions / notes:
- Vdsat: technology-dependent saturation voltage where carriers begin to saturate in velocity.
- Threshold voltage, mobility, and other parameters are taken from the sky130 model file included via the `.lib` directive.
- Keeping comparable W/L between cases isolates the effect of channel length scaling.

</details>


<details>
<summary>Day 3: Cmos Voltage Transfer characteristic (VTC)</summary>

## Day 3: Cmos Voltage Transfer characteristic (VTC)

Voltage transfer characteristics of aCMOS is plot of Vin vs Vout, it is done to analyse the input to output behavior of CMOS logic circuit.
It is calculated by evaluating Load curve of Pmos and Nmos and overlapping them. 

Spice deck to evaluate VTC:

 ``*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description


XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15


Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands

.op

.dc Vin 0 1.8 0.01

.control
run
setplot dc1
display
.endc

.end``

W/L ratio of pmos is 5.6 is greated than W/L of nmos of 1.8 because mobility of nmos is higher than that of pmos.

To plot VTC use command `plot out vs in`on ngspice.

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p1.png)

To find switching threshold we need to find at which value Vin and Vout are equal. For that we zoomed VTC curve by dragging with right click.

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p2.png)
 
Transition model:
It is a plot of output response of inverter when input is switching dynamically:
spice deck:

``*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description


XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15


Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)

*simulation commands

.tran 1n 10n

.control
run
.endc

.end``


PULSE function is used to simulate: Rising and falling of Vin between 0 and 1.8V at certain time interval.

To plot the curve use `plot out vs time in`

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p3.png)

To find rising transition we find time difference when Vin and Vout are at Vdd/2 or 0.9V for rising edge of Vout. Similarly falling delay is calculated at falling edge of Vout.

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p4.png)

From image above rise delay = (2.48-2.15) ns = 0.33ns = 330ps

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p5.png)

From image above fall delay is 4.33-4.05 ns = 0.28ns = 280ps.
Simulation of Cmos with equal WL ratio of Pmos and Nmos:
Wn = 0.36, Ln = 0.15, Wp = 0.42, Lp = 0.15
Wp of 0.36 was not available so we choose the closest one:
VTC plot:

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p6.png)

Switching threshold extracted = 0.817V

Transition plot:

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p7.png)


Rise transition :

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p8.png)

Extracted rise transition delay = 2.76 – 2.15 ns = 610ps.


Fall transition :

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p9.png)


Extracted fall transition delay = 4.32 – 4.05 ns = 270ps.

Similarly we tried this experiment for various values of Wp keeping Wn = 0.36, Ln = Lp = 0.15um constant.
With this ratio of Wp/Lp : Wn/Lp is varied and results are tabulated below:

Values set in spice circuit	Values extracted from spice simulation
Wp (um)	Wp/Lp	Wn/Ln	Wp/Lp : Wn/Lp	Switching threshold Vm(V)	Rise transition delay (ps)	Fall transition delay (ps)
0.42	2.8	2.4	1.166	0.817	610	270
0.84	5.6	2.4	2.333	0.876	330	280
1.26	8.4	2.4	3.5	0.881	234	286
1.68	11.2	2.4	4.66	0.9038	179.8	288.6
2.1	14	2.4	5.833	0.917	147.8	291.9

Observation:
* When Pmos and Nmos have equal WL ratio The switching threshold is lower than half of Vdd.
* Rise transition is very high and fall transition delay is less when Wp/Lp is similar to that of Nmos.
* This is due to high mobility of Nmos.
* When Pmos WL ratio is increased the Rise transition is reduced significantly at first then starts to saturates. At the same time Fall trastition increases slowly.
* At when the Pmos WL ratio is in the range of  3.5-4.66 time that of Mnos the switching threshold value is 0.9 (Vdd/2)
* When Pmos WL ratio is in range of 2.333-3.5 the rise and fall transition are equal.

Key takeaway for STA:
For Clock tree synthesis where we want equal transition of rise and fall we can use the above table and use Wp in the range of 0.84 to 1.26.
For critical paths where slack is negative we can increase Wp to reduce the rise transition time and meet timing.
For case where timing path is not critical we can use Wp value 0.42 to have low area of Pmos.


</details>
