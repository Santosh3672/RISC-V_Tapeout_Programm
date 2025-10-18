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


The VTC (Vin vs Vout) of a CMOS inverter shows switching behavior and the inverter switching threshold Vm. It is obtained by overlapping the NMOS pull‑down and PMOS load characteristics or by sweeping Vin and plotting out.

### DC VTC (DC sweep)

Spice deck to evaluate VTC (DC sweep):

 ````
 *Model Description
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

.end
````

Notes:
- The pMOS W/L is larger than nMOS W/L (Wp/Lp = 5.6 vs Wn/Ln = 1.8) to compensate for lower pMOS mobility.
- In ngspice use `plot out vs in` to view the VTC.

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p1.png)

- To find the switching threshold Vm, locate the point where Vin = Vout (zoom if necessary).

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p2.png)
 


### Dynamic response (transition)
To simulate the inverter switching dynamically, use a pulse input and transient simulation:



```spice
*Model Description
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

.end
```



- The PULSE source toggles Vin between 0 and 1.8 V with specified rise/fall times and periods.
- Plot `out` and `in` vs. `time` to measure delays.

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p3.png)

- Rising delay: time difference when both Vin and Vout cross Vdd/2 (0.9 V) on the rising edge.


![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p4.png)

From image above rise delay = (2.48-2.15) ns = 0.33ns = 330ps

- Falling delay: analogous measurement on the falling edge.

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d3p5.png)

From image above fall delay is 4.33-4.05 ns = 0.28ns = 280ps.

**Simulation of Cmos with equal WL ratio of Pmos and Nmos:**  
Wn = 0.36um, Ln = 0.15um, Wp = 0.42um, Lp = 0.15um

Wp of 0.36 was not available so we choose the closest one:  
**VTC plot:**

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


### Experiments: varying Wp (keeping Wn = 0.36, Ln = Lp = 0.15 µm)
We tested several Wp values to study VTC and transition delays. Wn = 0.36 µm, Ln = Lp = 0.15 µm.

| Wp (µm) | Wp/Lp | Wn/Ln | Wp/Lp : Wn/Ln | Switching threshold Vm (V) | Rise delay (ps) | Fall delay (ps) |
|---------:|------:|------:|---------------:|---------------------------:|----------------:|----------------:|
| 0.42     | 2.8   | 2.4   | 1.166          | 0.817                     | 610             | 270             |
| 0.84     | 5.6   | 2.4   | 2.333          | 0.876                     | 330             | 280             |
| 1.26     | 8.4   | 2.4   | 3.5            | 0.881                     | 234             | 286             |
| 1.68     | 11.2  | 2.4   | 4.66           | 0.9038                    | 179.8           | 288.6           |
| 2.10     | 14.0  | 2.4   | 5.833          | 0.917                     | 147.8           | 291.9           |


Observations:
- When pMOS and nMOS have equal W/L ratio, the switching threshold is below Vdd/2 (due to higher nMOS mobility).
- With pMOS W/L similar to nMOS W/L, rise delay is large and fall delay is small.
- Increasing pMOS W/L reduces rise delay significantly at first, then it starts to saturate; fall delay increases slowly.
- For Wp/Lp in the range ~3.5–4.66 × Wn/Ln the switching threshold approaches ≈ 0.9 V (Vdd/2).
- In the range ≈2.33–3.5 × Wn/Ln the rise and fall delays become approximately equal.

Key takeaways for STA:
- For clock-tree synthesis where balanced rise/fall is desired, use Wp in the range 0.84–1.26 µm (for the given process and Wn).
- For timing-critical paths with negative slack, increase Wp to speed up rising transitions.
- For non-critical paths, choose smaller Wp (e.g., 0.42 µm) to save area.

</details>


<details>
<summary>Day 4: Noise Margin</summary>

## Day 4: Noise Margin


Noise margins are extracted from the CMOS VTC by finding the two points where the slope of the Vout vs Vin curve is −1. These give the input thresholds that the next stage must recognize as logic 0 and logic 1.

Definitions
- Vol (Output Low): any output between 0 and Vol is considered logic 0.
- Voh (Output High): any output between Voh and Vdd is considered logic 1.
- Vil (Input Low): any input between 0 and Vil is considered logic 0.
- Vih (Input High): any input between Vih and Vdd is considered logic 1.



For correct cascading of stages we require:
Vdd > Voh > Vih > Vil > Vol > 0

Noise margin formulas:
- NMH = Voh − Vih
- NML = Vil − Vol

Any voltage between Voh and Vih: Considered as logic 1
Any voltage between Vil and Vol: Considered as logic 0


Noise margin Extraction procedure
1. Generate the inverter VTC (DC sweep of Vin) in ngspice and plot out vs in.
2. Zoom into the regions near the high and low transitions.
3. Find the two points where dVout/dVin = −1. Read out Voh, Vih (high side) and Vil, Vol (low side).
4. Compute NMH and NML using the formulas above.

Spice simulation deck used:

````spice
*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description


XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
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

.end
````


Plots are seen with `plot out`

To find region of -1 slope we zoomed the region of Voh and extracted slope by dragging with left click as shown below.

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d4p1.png)

Similarly the same was observed for Vol as shown in image below:

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d4p2.png)

From the image the extracted values are:  
- Voh = 1.7404 V
- Vol = 0.0812 V
- Vih = 1.0015 V
- Vil = 0.7470 V

Computed noise margins:
- NMH = Voh − Vih = 0.7389 V
- NML = Vil − Vol = 0.666 V

In this case Wn/Ln  = 2.4 and Wp/Lp = 6.66, Wp/Lp: Wn/Ln = 2.775
The experiment is repeated for different Wp to vary ratio of  Wp/Lp: Wn/Ln and results are tabulated below.

Table of measured values:

| Wp (µm) | Wp/Lp | Wn/Ln | Ratio (Wp/Lp : Wn/Ln) | Voh (V) | Vih (V) | Vil (V) | Vol (V) | NMH (V) | NML (V) |
|--------:|------:|------:|----------------------:|--------:|--------:|--------:|--------:|--------:|--------:|
| 0.42    | 2.8   | 2.4   | 1.166                 | 1.749   | 0.900   | 0.692   | 0.059   | 0.849   | 0.633   |
| 0.84    | 5.6   | 2.4   | 2.333                 | 1.7395  | 1.000   | 0.738   | 0.0832  | 0.7395  | 0.655   |
| 1.00    | 6.66  | 2.4   | 2.775                 | 1.7404  | 1.0015  | 0.747   | 0.0812  | 0.7389  | 0.666   |
| 1.26    | 8.4   | 2.4   | 3.5                   | 1.7396  | 0.9958  | 0.761   | 0.0672  | 0.7438  | 0.694   |
| 1.68    | 11.2  | 2.4   | 4.66                  | 1.7425  | 1.0308  | 0.7743  | 0.0776  | 0.7124  | 0.6967  |

Observations
- Increasing Wp strengthens the pMOS, raising Voh and improving NMH initially.
- After a point, noise margins change little: CMOS noise margin is robust to moderate Wp variation.
- The table quantifies how Wp affects Voh, Vil, and the margins for the given Wn and channel lengths.

</details>


<details>
<summary>Day 5: CMOS power supply and device variation robustness</summary>

## Day 5: CMOS power supply and device variation robustness

### Power supply variation:

This experiment sweeps Vdd to observe how the inverter VTC, switching threshold (Vm), and small‑signal gain change with supply voltage.

Spice deck used:

```spice
* Model / Netlist
.param temp=27

* Include sky130 models
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* Circuit
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0 0   sky130_fd_pr__nfet_01v8 w=0.36 l=0.15

Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in 0 1.8V

* Control: sweep Vdd from 1.8V down to 1.2V in steps of -0.2V
.control
let powersupply = 1.8
alter Vdd = {powersupply}
let voltagesupplyvariation = 0
dowhile voltagesupplyvariation < 6
  dc Vin 0 1.8 0.01
  let powersupply = powersupply - 0.2
  alter Vdd = {powersupply}
  let voltagesupplyvariation = voltagesupplyvariation + 1
end
plot dc1.out vs in dc2.out vs in dc3.out vs in dc4.out vs in dc5.out vs in dc6.out vs in \
     in xlabel "Input voltage (V)" ylabel "Output voltage (V)" title "Inverter DC characteristics vs supply"
.endc

.end
```

Notes:
- The control section runs multiple DC sweeps while decrementing Vdd by 0.2 V each iteration (1.8 V → 1.6 V → ... → 1.0 V).
- The plotted overlays show how the VTC and Vm shift with supply voltage. The 45° line (Vin = Vout) is included to extract Vm.


Spice waveform:

![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d5p1.png)

To find the gain we dragged the corresponding plot with left click.  
To extract the Vm the intersection of 45 degree tangent and plot is used.  

**Measured Vm and small‑signal gain for each supply:**

| Supply (V) | Switching threshold Vm (V) | Gain |
|-----------:|---------------------------:|-----:|
| 1.0        | 0.458                      | 11.4 |
| 1.2        | 0.530                      | 11.25|
| 1.4        | 0.700                      | 10.69|
| 1.6        | 0.790                      | 10.39|
| 1.8        | 0.8807                     | 9.64 |


**Observations:**
- Lower Vdd reduces Vm and increases the peak small‑signal gain (dVout/dVin) for the DC operating point shown.
- Lower Vdd also has low power consumption.
- Lower supply reduces dynamic drive strength: in transient operation the output may not fully reach Voh when Vdd is low, limiting switching margin and speed.


![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d5p2.png)

Example transient limitation:
- At Vdd = 1.0 V the output cannot fully charge to the nominal Voh during switching, illustrating a practical limit to supply scaling for fast, full‑swing outputs.

### Cmos Process / device variation:

To observe robustness to device sizing variations, Wp was increased significantly (example: Wp = 7 µm). The VTC shift is small:


![Image](https://github.com/Santosh3672/RISC-V_Tapeout_Programm/blob/main/Week%204%3A%20Spice%20simulation%20for%20STA/Image%20W4/W4d5p3.png)

**Observation:**
- Switching threshold is 0.99V, while for lower value of Wp it is in the range of 0.82 – 0.92V. 
- The variation in Vm is not much showing robustness of PMOS for process variation.

</details>
