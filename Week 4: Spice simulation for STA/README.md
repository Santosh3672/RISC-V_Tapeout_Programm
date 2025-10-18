# RISCV Tapeout Week3 🚀

This repository documents Week 4 of the RISCV tapeout program, covering CMOS circuit design and Spice simulation.

---

<details>
<summary>Day 1: Basics NMOS Id vs Vds</summary>

## Day 1: Basics NMOS Id vs Vds

NMOS and PMOS transistors form the foundation of logic gates:
    • NMOS devices are N-channel MOSFETs with a P-type substrate, SiO2 isolation, n+ source/drain, gate oxide, and a gate made from polysilicon or metal. The four terminals are Gate (G), Source (S), Drain (D), and Body (B).
    • PMOS devices use an N-type substrate and p+ diffusion regions.

A mosfet operates on 3 modes:

1. Cutoff mode:
        ◦ Gate-to-source voltage VGS<Vt (threshold voltage).
        ◦ Source, drain, and body grounded; p-n junctions are off.
        ◦ No inversion channel forms; transistor behaves like an open circuit.
        ◦ No current flows from source to drain.
    • Bulk Biasing effect:
        ◦ Positive substrate-to-source voltage VSB reverse biases the source-substrate junction.
        ◦ Increases depletion region width near source, reducing channel charge.
        ◦ Requires higher VGS to form inversion channel; increases Vt.
        ◦ Raises threshold voltage, pushing transistor toward cutoff mode.

Image W4d1p1




2. Resistive Mode of Operation (also called Linear or Triode Region):
    • Condition: VGS>Vt and VDS<VGS−Vt
    • Drain current equation:
      ID=Kn[(VGS−Vt)VDS−2VDS2] 
    • For small VDS, VDS2 term is negligible, simplifying to:
      ID≈Kn(VGS−Vt)VDS 
    • Here, Kn is the process transconductance parameter depending on channel width (W), length (L), and foundry-specific process parameters.
    • In this mode, the transistor behaves like a voltage-controlled resistor, with linear dependence of drain current on VDS.

3. Saturation region: 
Conditions for saturation mode:
    • VGS>Vt (gate-to-source voltage greater than threshold)
    • VDS≥VGS−Vt (drain-to-source voltage exceeds overdrive voltage)
    • As VDS in linear mode increases and reaches VDS=VGS−Vt, the voltage difference between gate and drain becomes zero.
    • The channel starts to disappear at the drain side, causing the pinch-off phenomenon.
    • In this mode, the voltage across the channel stays constant at VGS−Vt regardless of further increase in VDS.
    • Drain current saturates and is given by:
Id = Kn/2* (Vgs – Vt)2 
and no longer depends on VDS.

To account for channel length modulation, the drain current equation modifies to:
Id = Kn/2* (Vgs – Vt)2 *(1 + λ*Vds)

where λ models the slight increase of current with VDS in saturation.


Introduction to SPICE:
SPICE is an engine that has the formulas described above and has parameters from foundry. 
Based on that it evaluates a circuit to derive correct waveform of the cell. 

How to perform SPICE simulation:
1. Verifying the setup: Ensure model parametrs (Kn, Vto,etc) are coming from correct technology node.
2. SPICE netlist: Create a correct SPICE netlist for your circuit
Steps in writing a SPICE netlist:

* Define nodes (where there are no obstruction of device elements)
* Define device elements between each node in following way
eg `M1 vdd n1 0 0 nmos W=1.8u L=1.2u`
M1 = Mosfet, nodes where terminal of M1 are connected to, name of device nmos, parameter of nmos

Image W4d1p2

* Get models of devices: The model file contains MODEL definition of all device nmos for our example and store its parameter like VTH0, U0, etc.
This needs to be included in the spice netlist



Labs:
For labs we used github repo from: https://github.com/kunalg123/sky130CircuitDesignWorkshop/tree/main

directory structure:
sky130CircuitDesignWorkshop/design/sky130_fd_pr/
Has cells and models directories that contains device parameters for analysis.

Objective: To Plot Id vs Vds for different Vgs values.

Spice netlist used:
``*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description



XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2

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

.end``

Components of spice netlist:

1. Parameters definition: Temp = 27
2. Including .libs from models directory, here the `sky130_fd_pr__nfet_01v8` is defined
3. Circuit definition containing nfet, resistor and voltage sources.
4. Simulation condition defined for Vdd(Vds) and Vin (Vgs)


Steps to perform SPICE simulation using ngspice:
1. Open ngspice with spice circuit:
	``ngspice ./day1_nfet_idvds_L2_W5.spice``
Image W4d1p3

2. Plot graph using
	``plot -vdd#branch``
negative sign is used because the convention in spice simulation is negative. 
Image W4d1p4

We have Id on Y axis and Vds on X axis, different plots are for different Vgs.

W/L ratio = 5/2 = 2.5

</details>