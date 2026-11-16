# picorv32a-openlane-physical-design

This repository documents the complete RTL-to-GDSII physical design flow for the **PicoRV32A** RISC-V 32-bit core using the open-source OpenLANE automated ASIC toolchain and the SkyWater 130nm PDK (`sky130A`).

Rather than relying on default tutorial settings, this repository highlights aggressive **Power, Performance, and Area (PPA) optimizations**, rigorous timing closure, and complete DRC/Antenna sign-off.

---

## 📋 Table of Contents
1. [Project Overview](#project-overview)
2. [Baseline vs. Optimized Configuration](#baseline-vs-optimized-configuration)
3. [Design Flow & Execution Steps](#design-flow--execution-steps)
4. [Slew/Capacitance Violation Analysis & Closure](#slewcapacitance-violation-analysis--closure)
5. [Repository Structure](#repository-structure)
6. [Layout Snapshots](#layout-snapshots)

---

## 1. Project Overview
* **Target Core:** `picorv32a` (RISC-V RV32I)
* **PDK & Library:** `sky130A` (`sky130_fd_sc_hd` High-Density Standard Cell Library)
* **Target Clock Period:** `5.000 ns` (200 MHz)
* **Toolchain:** OpenLANE (Yosys, OpenROAD, Magic, TritonRoute, Klayout)

---

## 2. Baseline vs. Optimized Configuration
To demonstrate active engineering contribution and optimization beyond default runs, custom overrides were introduced in `config_optimized.tcl`:

| Parameter | Baseline (Default) | Optimized (Engineering Contribution) | Impact & Rationale |
| :--- | :--- | :--- | :--- |
| **`SYNTH_STRATEGY`** | `AREA 0` | `DELAY 1` | Prioritizes timing over area to successfully meet the tight 5.0 ns clock constraint. |
| **`FP_CORE_UTIL`** | `50%` | `65% – 70%` | Shrinks core footprint and silicon die area while relying on detailed routing congestion resolution. |
| **`PL_TIME_DRIVEN`** | `0` | `1` | Forces the RePLace engine to prioritize critical timing paths during global placement. |
| **`PL_ROUTABILITY_DRIVEN`** | `0` | `1` | Incorporates congestion maps during placement to prevent dense routing blockages. |
| **`CTS_TARGET_SKEW`** | `200 ps` | `50 – 100 ps` | Synthesizes a tightly balanced clock tree via TritonCTS to improve hold margins. |
| **`GLB_RT_MAX_DIODE_INS_ITERS`** | `1` | `5` | Grants FastRoute extra iteration passes to mitigate antenna violations and guarantee DRC cleanliness. |

---

## 3. Design Flow & Execution Steps
The complete RTL-to-GDSII flow was driven via the OpenLANE interactive shell environment:
1. **Synthesis (`run_synthesis`):** Logic translation and technology mapping via Yosys and ABC.
2. **Floorplanning (`run_floorplan`):** Core dimensioning, pin placement, tap/decap insertion, and Power Distribution Network (PDN) generation.
3. **Placement (`run_placement`):** Global and detailed placement with timing- and routability-driven options enabled.
4. **Clock Tree Synthesis (`run_cts`):** H-tree clock distribution balancing using TritonCTS.
5. **Post-CTS Resizer Timing (`run_resizer_timing`):** Automated buffering and fixing of minor hold/setup violations.
6. **Routing (`run_routing`):** Global routing (FastRoute) and detailed routing (TritonRoute) achieving zero DRC violations.
7. **Sign-off:** SPEF extraction, Static Timing Analysis (OpenSTA), and physical verification (Magic/Klayout DRC/LVS).

---

## 4. Slew/Capacitance Violation Analysis & Closure
During initial post-placement static timing analysis, intensive net loads led to transient signal degradation:
* **The Symptom:** Max slew reports flagged several high-fanout multiplexer select and control paths (e.g., net loads on pins like `_41353_/S0`) exceeding the `1.50 ns` limit with slews reaching up to `8.52 ns` (negative slack of `-7.02 ns`). Capacitance limits on high-fanout drivers (such as `_41051_/X`) also showed max capacitance violations (`-0.40 slack`).
* **The Resolution:** 
  * Enabled automated timing-driven resizing and buffer insertion via `run_resizer_timing`.
  * Upgraded weak driver cells to higher drive-strength standard cells (scaling up buffers and mapping to robust drivers like `sky130_fd_sc_hd__buf_2` and stronger drive variants).
  * Re-ran STA sign-off confirming complete timing closure with **WNS = 0.00 ns** and **TNS = 0.00 ns**.

---

## 5. Repository Structure
```text
picorv32a-openlane-physical-design/
├── reports/                  # Extracted OpenSTA timing and Magic DRC logs
├── snapshots/                # Visual layout validation (Magic / Klayout)
└── README.md

## ACKNOWLEDGEMENT

- **Physical Design & Lab Expert, VLSI System Design (VSD):** [Nickson Jose](https://github.com/nickson-jose)
- **Co-founder of VLSI System Design (VSD):** [Kunal Ghosh](https://github.com/kunalg123/)
