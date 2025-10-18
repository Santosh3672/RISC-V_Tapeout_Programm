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
