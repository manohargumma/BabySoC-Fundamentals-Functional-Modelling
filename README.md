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
  - [How to synthesize the design](#how-to-synthesize-the-design)
  - [Synthesis using Yosys](#synthesis-using-yosys)
  - [Post synthesis simulation (GLS)](#post-synthesis-simulation-gls)
  - [Yosys final report](#yosys-final-report)
- [Static timing analysis using OpenSTA](#static-timing-analysis-using-opensta)
  - [Static timing analysis on the design](#static-timing-analysis-on-the-design)

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

1. **First we need to install some important packages:**
   ```bash
   git clone https://github.com/manili/VSDBabySoC.git
   cd VSDBabySoC

