# AI-Assisted Analog Mixed-Signal Internship
AI-Assisted Mixed-Signal RTL-to-GDS Flow for AMUX2_3V using SKY130/OpenLane

Author

Indhu V

Reference Repository

vsdmixedsignalflow Reference Repository

---

Project Objective

The objective of this work is to reproduce the mixed-signal RTL-to-GDS flow demonstrated in the reference repository using AI-assisted prompting instead of manually copying files.

The project demonstrates how AI tools such as ChatGPT and Codex can be used to:

- Understand the mixed-signal design flow
- Identify required input files
- Generate configuration files
- Create OpenLane setup files
- Generate LEF/LIB/Verilog abstractions
- Execute the RTL-to-GDS flow
- Debug flow failures
- Verify final outputs

The complete workflow was documented from repository understanding through final GDS generation.

---

Reference Design Understanding

The reference design contains:

Analog Macro

AMUX2_3V

A transistor-level analog 2:1 multiplexer implemented in SKY130.

Digital Top Module

design_mux

Digital wrapper controlling the analog multiplexer.

Mixed-Signal Challenge

OpenLane cannot directly understand transistor-level analog layouts.

Therefore, the analog macro must be represented through:

File| Purpose
LEF| Physical abstract view
LIB| Timing abstraction
GDS| Physical implementation
Verilog Blackbox| Logical abstraction
SPICE| LVS verification

---

AI-Assisted Workflow

Phase 1 – Repository Understanding

AI Prompt

«Analyze the repository and explain every file required to integrate AMUX2_3V into OpenLane.»

AI Output

Generated understanding of:

- design_mux.v
- AMUX2_3V.v
- AMUX2_3V.lef
- AMUX2_3V.lib
- AMUX2_3V.gds
- config.tcl
- macro.cfg

Verification

Compared AI explanation against the reference repository.

Result

✅ Correct identification of all required inputs.

---

Phase 2 – Input File Identification

Required Files

Input File| Used In
design_mux.v| Synthesis
AMUX2_3V.v| Blackbox
AMUX2_3V.lef| Floorplan
AMUX2_3V.lib| STA
AMUX2_3V.gds| Final Merge
config.tcl| OpenLane
macro.cfg| Macro Placement

---

Phase 3 – AI Generated Files

Generated Using Codex

Verilog Blackbox

"AMUX2_3V.v"

OpenLane Configuration

"config.tcl"

Macro Placement File

"macro.cfg"

LEF Validation Script

"fix_amux_lef.tcl"

---

AI Generated vs Actual Files

Observation

AI Generated Blackbox

module AMUX2_3V(
    Vin1,
    Vin2,
    sel,
    sel_b,
    Vout
);

Actual Reference Blackbox

module AMUX2_3V(
    I0,
    I1,
    out,
    select
);

Error

Port mismatch.

Fix

Compared against reference files and updated the blackbox port list.

Lesson Learned

AI assumptions must always be verified against the actual design hierarchy.

---

Physical Design Flow

Stage 1 – Synthesis

Inputs

- design_mux.v
- AMUX2_3V.v
- config.tcl

Command

run_synthesis

Outputs

design_mux.v
design_mux.sdf

Verification

AMUX2_3V remained as a single blackbox instance.

---

Stage 2 – Floorplanning

Inputs

- Synthesized Netlist
- LEF
- Macro Placement File

Command

init_floorplan

Outputs

design_mux.def
design_mux.odb

Verification

Macro placed successfully inside the die area.

---

Stage 3 – Placement

Command

run_placement

Outputs

design_mux.def
design_mux.pnl.v
design_mux.nl.v

Verification

Standard cells legalized around the macro.

---

Stage 4 – Clock Tree Synthesis (CTS)

Command

run_cts

Outputs

design_mux.sdc
design_mux.def

Verification

Clock tree successfully generated.

---

Stage 5 – Power Delivery Network (PDN)

Command

gen_pdn

Verification

Power grid connected correctly to macro power pins.

---

Stage 6 – Routing

Command

run_routing

Outputs

design_mux.def

Verification

All signal nets routed successfully.

---

Stage 7 – Design Rule Check (DRC)

Command

run_magic_drc

---

Errors Faced During Implementation

Error 1

Message

DEF Read: encountered 3688 errors total

Root Cause

LEF and DEF mismatch.

AI Debug Prompt

«Analyze Magic DEF import errors caused by LEF pin mismatch and suggest fixes.»

Fix Attempt

Verified:

- Layer names
- Pin names
- Obstruction definitions

---

Error 2

Message

DEF Read: encountered 596 errors total

Observation

Partial improvement after LEF correction.

---

Error 3

Message

DRC = 1267

Root Cause

Incorrect macro abstraction and routing issues.

AI Suggested Fixes

- Increase halo
- Increase keepout margin
- Verify LEF geometry
- Re-run placement

---

Final Result

DRC Status

DRC = 0

Status

✅ Clean Layout Achieved

---

Signoff Outputs Generated

design_mux.gds
design_mux.mag
design_mux.lef
design_mux.lib
design_mux.spice
design_mux.sdf

---

Output Files Generated at Each Stage

Stage| Output
Synthesis| Netlist
Floorplan| DEF
Placement| DEF + ODB
CTS| DEF + SDC
Routing| Routed DEF
Signoff| GDS / LEF / LIB
Final| Complete Design Database

---

Comparison with Reference Repository

Item| Reference Repo| AI Flow
Synthesis| ✅| ✅
Floorplan| ✅| ✅
Placement| ✅| ✅
CTS| ✅| ✅
Routing| ✅| ✅
DRC| ✅| ✅
GDS Generation| ✅| ✅

Key Difference

The reference repository manually provides all required files.

This project demonstrates that AI-generated prompts can be used to:

- Understand the flow
- Generate files
- Debug errors
- Execute OpenLane stages
- Verify outputs

while achieving the same mixed-signal RTL-to-GDS methodology.

---

Conclusion

This work successfully demonstrates an AI-assisted mixed-signal RTL-to-GDS flow using SKY130 and OpenLane.

AI tools were used throughout repository analysis, file generation, configuration creation, debugging, and verification. Real OpenLane execution produced floorplan, placement, routing, signoff, and final GDS outputs.

Multiple DEF and DRC issues were encountered and investigated using AI-guided debugging before achieving a clean DRC result.

The project proves that AI can significantly accelerate mixed-signal physical design learning and development while still requiring engineering verification at every stage.

---

References

1. Praharsha Mahurkar, vsdmixedsignalflow
2. OpenLane Documentation
3. SKY130 Open-Source PDK
4. Magic VLSI Layout Tool
5. Netgen LVS Tool
