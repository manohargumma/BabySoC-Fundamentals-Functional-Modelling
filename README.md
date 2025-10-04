# Week 2 – BabySoC Fundamentals & Functional Modelling

This write-up focuses on four key areas that form the foundation of SoC design and explain how **BabySoC** helps us learn these concepts step by step.

## 📑 Table of Contents
# Table of Contents

- [What is a System-on-Chip (SoC)?](#1-what-is-a-system-on-chip-soc)
- [Components of a Typical SoC](#2-components-of-a-typical-soc)
- [Why BabySoC is a Simplified Model for Learning SoC Concepts](#3-why-babysoc-is-a-simplified-model-for-learning-soc-concepts)
- [Role of Functional Modelling Before RTL and Physical Design](#4-role-of-functional-modelling-before-rtl-and-physical-design)

- [VSDBabySoC Modeling](#vsdbabysoc-modeling)
   [RVMYTH Modeling](#rvmyth-modeling)  
- [PLL Modeling](#pll-modeling)  
- [DAC Modeling](#dac-modeling)  
- [Step by step modeling walkthrough](#step-by-step-modeling-walkthrough)  
- [OpenLANE installation](#openlane-installation)
- [Post-synthesis simulation](#post-synthesis-simulation)
  
- [Static timing analysis using OpenSTA](#static-timing-analysis-using-opensta)
 

## 1. What is a System-on-Chip (SoC)?  

A **System-on-Chip (SoC)** is an integrated circuit that consolidates all the essential building blocks of a computing system into a **single piece of silicon**.  
Unlike traditional designs where CPU, memory, and I/O devices exist on separate chips connected via a motherboard, an SoC brings them together into one unit, reducing power consumption, increasing performance, and shrinking the physical footprint.  

SoCs are everywhere around us:  
- 📱 Smartphones (Apple A-series, Snapdragon, MediaTek)  
- ⌚ Smartwatches and wearable devices  
- 🚗 Automotive electronics (ADAS controllers, infotainment chips)  
- 🌐 IoT devices (ESP32, Raspberry Pi Pico)  

The essence of an SoC lies in **integration** — a complete digital ecosystem compressed into a single chip.  

---

## 2. Components of a Typical SoC  

At its core, every SoC consists of a few **fundamental building blocks**:  

- 🖥️ **Central Processing Unit (CPU):**  
  The heart of the SoC. Handles instructions, executes operations, and controls data flow. In many modern SoCs, CPUs are based on RISC architectures (like ARM or RISC-V).  

- 💾 **Memory:**  
  On-chip RAM, ROM, and cache memory for fast access. Some SoCs also include external memory controllers for DRAM or Flash.  

- 🔌 **Peripherals:**  
  Interfaces that allow the SoC to talk to the outside world, such as UART, SPI, I²C, GPIO, timers, and ADC/DAC modules.  

- 🔗 **Interconnects:**  
  The "nervous system" of the SoC. This includes buses, crossbars, or NoCs (Network-on-Chip) that connect CPU, memory, and peripherals together.  

Each of these blocks is integrated and optimized to ensure the SoC operates as a **self-sufficient computing platform**.  

---

## 3. Why BabySoC is a Simplified Model for Learning SoC Concepts  

Industry-grade SoCs are **incredibly complex** — billions of transistors, multiple cores, GPUs, accelerators, high-speed memory controllers, and advanced security features. Designing and understanding them directly is overwhelming for beginners.  

🍼 **BabySoC** is an educational SoC designed to simplify this learning curve.  
It includes only a few carefully chosen blocks:  

- **RVMYTH RISC-V core** – a compact open-source CPU core  
- **PLL (Phase-Locked Loop)** – generates stable and accurate clocks  
- **10-bit DAC (Digital-to-Analog Converter)** – demonstrates how digital signals can be converted into analog voltages  

By limiting the complexity, BabySoC allows learners to:  
- Focus on the **core principles** of integration  
- Understand how different blocks interact inside a chip  
- Gain confidence before advancing to full-scale RTL or physical design  

In short, BabySoC acts as a **stepping stone** between theoretical SoC concepts and practical chip design.  

---

## 4. Role of Functional Modelling Before RTL and Physical Design  

Chip design follows a structured flow: **Concept → Functional Model → RTL → Synthesis → Physical Design → Fabrication.**  
Among these steps, **functional modelling** plays a critical role in verifying the system’s behavior *before* time and resources are spent on RTL or silicon fabrication.  

Functional modelling allows us to:  
- 🧪 **Simulate behavior** of the SoC blocks  
- 🔍 Detect errors in design logic at an early stage  
- 📊 Visualize waveforms (using tools like GTKWave)  
- 🛠️ Test integration between CPU, memory, and peripherals  

Tools such as **Icarus Verilog** (for compiling and simulating designs) and **GTKWave** (for analyzing signal activity) are used here.  

👉 Without functional modelling, moving directly into RTL or physical design would be risky. Errors caught late in the process can result in costly chip re-spins.  


#  VSDBabySoC-modeling

## 📂 Project Structure  


VSDBabySoC/<br>
├── src/<br>
│ ├── include/ # Header files (*.vh) with macros/parametersM<br>
│ │ └── sandpiper.vh<br>
│ ├── module/ # Verilog design files<br>
│ │ ├── vsdbabysoc.v # Top-level SoC module<br>
│ │ ├── rvmyth.v # RISC-V CPU core<br>
│ │ ├── avsdpll.v # PLL module<br>
│ │ ├── avsddac.v # DAC module<br>
│ │ └── testbench.v # Testbench for simulation<br>
├── output/ # Compiled outputs & simulation files<br>
└── compiled_tlv/ # Holds intermediate compiled file<br>



---

## RVMYTH Modeling

**Goal:** provide a compact RISC-V–like CPU (rvmyth) that is simple to read and easy to simulate.

**Key features**
- Minimal instruction subset (e.g., add, load, store, branch)
- Simple register file (x0..x31 behavior)
- Program memory interface (ROM or testbench-provided instruction stream)
- Cleanly separated datapath and control logic
- Parameterized width and reset behavior (via `include/*.vh`)

---

## PLL Modeling

**Goal:** model clock generation behavior for simulation without requiring transistor-level PLL or an analog simulator.

**Modeling approach**
- Behavioral Verilog that generates an output clock (`clk_out`) derived from an input (`clk_in`) with configurable multiplication/division factors.
- Includes simple lock detection logic (`locked` signal) and a programmable lock-up delay to mimic real-world PLL lock time.

---

## DAC Modeling

**Goal:** provide a simple digital-to-analog converter behavioral model usable in purely digital testbenches.

**Modeling approach**
- Verilog module that accepts a parallel digital input (e.g., 8/10/12-bit) and outputs a digital-encoded "analog" stream.
- Optional PWM-mode testbench to produce waveforms suitable for GTKWave visualization.

---

## Step by step modeling walkthrough

1.**First we need to install some important packages:**
```bash
  $sudo apt-get update
  $sudo apt-get install -y python3 python3-pip git iverilog gtkwave docker.io make build-essential
  $sudo apt-get remove -y containerd.io
  $ sudo apt-get update<br>
   $  sudo apt-get install -y docker.io
  $  sudo systemctl enable docker
  $  sudo systemctl start docker
  $  docker --version
  $  pip install sandpiper-saas
  $  python3 -m venv ~/.venvs/babysoc
  $  source ~/.venvs/babysoc/bin/activate
  $  pip install --upgrade pip
  $  pip install sandpiper-saas
  $  sandpiper-saas --version
  $  sandpiper-saas -h
```
<br><a href="https://ibb.co/20KGvhLN"><img src="https://i.ibb.co/dsK8PM3g/Screenshot-from-2025-10-03-18-49-27.png" alt="Screenshot-from-2025-10-03-18-49-27" border="0"></a><br>
2. **Now we can clone this repository in an arbitrary directory (we'll choose home directory here)**
```bash
$ cd VLSI 
$ git clone https://github.com/manili/VSDBabySoC.git
$cd VSDBabtSoC
```
This repository includes **`rvmyth.tlv`**, a TL-Verilog source file describing the RVMYTH RISC-V core.
It contains the pipeline stages, instruction decoding, ALU, branch logic, memory interface, and register file connections.<br><br>
3.  **Convert TL-Verilog to SystemVerilog**
 ```bash
$sandpiper-saas -i src/module/rvmyth.tlv -o rvmyth.v     --bestsv --noline -p verilog --outdir output/compiled_tlv     --default_includes
```
<a href="https://ibb.co/qYBgCSRc"><img src="https://i.ibb.co/Mkn1fvRt/Screenshot-from-2025-10-04-19-56-41.png" alt="Screenshot-from-2025-10-04-19-56-41" border="0"></a><br>
-check location<br>
 output/compiled_tlv/rvmyth.v<br>
 output/compiled_tlv/rvmyth_gen.v<br>
<a href="https://ibb.co/WNDH0kpn"><img src="https://i.ibb.co/208FSvYq/Screenshot-from-2025-10-04-20-02-14.png" alt="Screenshot-from-2025-10-04-20-02-14" border="0"></a><br>
4. **Run simulation with Icarus Verilog**<br>
```bash
$ iverilog -g2012 -o output/pre_synth_sim/pre_synth_sim.out -DPRE_SYNTH_SIM     -I src/include -I output/compiled_tlv -I src/module     -y src/include     src/module/testbench.v     src/module/vsdbabysoc.v     output/compiled_tlv/rvmyth_gen.v     src/module/avsdpll.v src/module/avsddac.v
```

5.**Execute simulation**<br>
```bash
$ make synt
$ make post_synth_sim
```
6.**View waveform in GTKWave**<br>
```bash
$gtkwave output/post_synth_sim/post_synth_sim.vcd
```
### Notes

* `BOGUS_USE` warnings can be ignored; they are placeholders for unused logic.
* Always run **Sandpiper** first to translate `.tlv` into `.v` before compiling.
## openlane-installation

The OpenLANE and SKY130 installation can be done by following the steps in this repository:
👉 [openlane_build_script](https://github.com/nickson-jose/openlane_build_script)

More information on OpenLANE:

* [The-OpenROAD-Project/OpenLane](https://github.com/The-OpenROAD-Project/OpenLane)
* [efabless/openlane](https://github.com/efabless/openlane)

### Installation Summary

Clone OpenLANE repository
```bash
#Clone OpenLANE repository
$ git clone https://github.com/The-OpenROAD-Project/OpenLane.git
$ cd OpenLane/

# Build OpenLANE
$ make openlane

# Build SKY130 PDK
$ make pdk

# Run quick test
$ make test
```

**Note:**

* Currently using **OpenLANE commit** `8580c248a995b575f7734813b80bb6c4aa82d4f2`.
* Docker image version: **2021.09.09_03.00.48**.

---

## post-synthesis-simulation

The first step in the ASIC design flow is **synthesis** of the RTL code, followed by a simulation of the synthesized netlist (Gate-Level Simulation).
This ensures the synthesized design matches the RTL functionality.

### Synthesizing with Yosys

* In OpenLANE:

  * RTL synthesis is performed by **Yosys**.
  * Technology mapping is done by **ABC**.
  * Timing reports are generated by **OpenSTA**.

#### Run Synthesis

```bash
$ cd ~/VSDBabySoC
$ make synth
```

* Result: `output/synth/vsdbabysoc.synth.v`

---

### Post-Synthesis Simulation (GLS)

There is a known issue for GLS simulation in SKY130.
We fixed it by modifying the following file:

**File:**
`$YOUR_PDK_PATH/sky130A/libs.ref/sky130_fd_sc_hd/verilog/sky130_fd_sc_hd.v`

**Fix:**
Change

```verilog
`endif SKY130_FD_SC_HD__LPFLOW_BLEEDER_FUNCTIONAL_V
```

to

```verilog
`endif // SKY130_FD_SC_HD__LPFLOW_BLEEDER_FUNCTIONAL_V
```

#### Run GLS

```bash
$ cd ~/VSDBabySoC
$ make post_synth_sim
```

* Result: `output/post_synth_sim/post_synth_sim.vcd`

#### View Waveform

```bash
$ gtkwave output/post_synth_sim/post_synth_sim.vcd
```

---
### Result
<a href="https://ibb.co/j9chrkZp"><img src="https://i.ibb.co/RGx2DT45/Screenshot-from-2025-10-03-19-58-53.png" alt="Screenshot-from-2025-10-03-19-58-53" border="0"></a><br>
**Post-Synthesis Simulation**<br>

After completing the synthesis of my Verilog design, I performed post-synthesis simulation to verify the functional correctness at the gate-level. The simulation was done using Icarus Verilog and the results were viewed in GTKWave.<br>

In the waveform, signals such as reset, REF, VCO_IN, VREFH, VREFL, EN_CP, EN_VCO, and OUT were monitored. The behavior of the design shows that after the reset is de-asserted, the reference and control signals properly drive the output.<br>

Post-synthesis simulation is important because it checks whether the synthesized netlist (generated after logic synthesis) still preserves the intended functionality of the RTL code. It also helps to identify any mismatches introduced during synthesis.<br>
## Yosys final Report
```bash
=== vsdbabysoc ===

 Number of wires:               5559
 Number of wire bits:           5559
 Number of public wires:        1323
 Number of public wire bits:    1323
 Number of memories:               0
 Number of memory bits:            0
 Number of processes:              0
 Number of cells:               5552
   avsddac                         1
   avsdpll1v8                      1
   sky130_fd_sc_hd__a211o_2        1
   sky130_fd_sc_hd__a21o_2         4
   sky130_fd_sc_hd__a21oi_2       19
   sky130_fd_sc_hd__a221o_2       56
   sky130_fd_sc_hd__a22o_2        32
   sky130_fd_sc_hd__a2bb2o_2      14
   sky130_fd_sc_hd__a2bb2oi_2     12
   sky130_fd_sc_hd__a311o_2        1
   sky130_fd_sc_hd__a31o_2         7
   sky130_fd_sc_hd__a31oi_2        3
   sky130_fd_sc_hd__a32o_2         8
   sky130_fd_sc_hd__a41o_2         1
   sky130_fd_sc_hd__and2_2        38
   sky130_fd_sc_hd__and3_2         5
   sky130_fd_sc_hd__and4b_2        1
   sky130_fd_sc_hd__buf_1        885
   sky130_fd_sc_hd__conb_1         6
   sky130_fd_sc_hd__dfxtp_2     1144
   sky130_fd_sc_hd__inv_2       1026
   sky130_fd_sc_hd__mux2_1       513
   sky130_fd_sc_hd__nand2_2        3
   sky130_fd_sc_hd__nand4_2       32
   sky130_fd_sc_hd__nor2_2        61
   sky130_fd_sc_hd__nor2b_2        1
   sky130_fd_sc_hd__nor4_2         2
   sky130_fd_sc_hd__o2111a_2       1
   sky130_fd_sc_hd__o2111ai_2     65
   sky130_fd_sc_hd__o211a_2        4
   sky130_fd_sc_hd__o21a_2         6
   sky130_fd_sc_hd__o21ai_2        9
   sky130_fd_sc_hd__o221a_2      955
   sky130_fd_sc_hd__o221ai_2       2
   sky130_fd_sc_hd__o22a_2       427
   sky130_fd_sc_hd__o2bb2a_2      23
   sky130_fd_sc_hd__o2bb2ai_2      2
   sky130_fd_sc_hd__o311a_2        2
   sky130_fd_sc_hd__o31a_2        10
   sky130_fd_sc_hd__o32a_2        15
   sky130_fd_sc_hd__or2_2         48
   sky130_fd_sc_hd__or2b_2        32
   sky130_fd_sc_hd__or3_2         35
   sky130_fd_sc_hd__or4_2         36
   sky130_fd_sc_hd__or4b_2         3

 Area for cell type \avsddac is unknown!
 Area for cell type \avsdpll1v8 is unknown!

 Chip area for module '\vsdbabysoc': 58173.298800
```
# static-timing-analysis-using-opensta
