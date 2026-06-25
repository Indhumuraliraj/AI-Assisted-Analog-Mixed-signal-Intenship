# 🤖 AI-Assisted Mixed-Signal RTL-to-GDS Flow for AMUX2_3V using SKY130/OpenLane

## 🎯 Goal

The goal of this project is to reproduce the mixed-signal RTL-to-GDS flow shown in the `vsdmixedsignalflow` repository using AI-assisted prompts instead of manually copying files.

Unlike a traditional implementation, this project demonstrates how AI tools can assist throughout the complete physical design flow:

✅ Understanding the repository

✅ Identifying required files

✅ Generating OpenLane configurations

✅ Creating blackbox models

✅ Debugging LEF/DEF issues

✅ Running OpenLane stages

✅ Verifying final GDS generation

---

# 🧠 Understanding the Problem

- The reference repository contains an analog macro called **AMUX2_3V**.

- The challenge is that OpenLane is a digital physical design tool.

- OpenLane cannot directly understand transistor-level analog layouts.

Therefore, the analog macro must be converted into abstract representations that digital tools can understand.

## Required Macro Views

| File             | Why It Is Needed        |
| ---------------- | ----------------------- |
| LEF              | Physical abstract view  |
| LIB              | Timing information      |
| GDS              | Physical implementation |
| Verilog Blackbox | Logical representation  |
| SPICE            | LVS verification        |

## Key Learning

Before this project, I thought GDS alone was sufficient.

After studying the repository using AI prompts, I learned that OpenLane primarily uses LEF and LIB during implementation, while GDS is merged only at the final stage.

---

# 🧠 Prompt 1 – Repository Understanding

## Objective
Understand the complete mixed-signal flow implemented in the reference repository.

## Prompt
```
Analyze the GitHub repository and explain:

1. Overall design objective
2. Function of each file
3. Purpose of AMUX2_3V
4. How the analog macro is integrated into OpenLane
5. Required inputs for RTL-to-GDS flow
6. Expected outputs generated at each stage
```

## AI Outcome

Generated explanation of:

AMUX2_3V
design_mux
LEF
LIB
GDS
Verilog Blackbox
config.tcl
macro.cfg
Verification

---

# 📂 Phase 2 – Input File Study

Before running OpenLane, I wanted to understand why each file exists, where it is used in the flow, and what would happen if the file was missing.
Instead of directly using the files from the reference repository, I first used AI to analyze the mixed-signal design requirements.

## Prompt 
```
For a mixed-signal RTL-to-GDS flow using OpenLane and SKY130, identify all required input files for integrating an analog hard macro (AMUX2_3V).

For each file explain:

1. Why it is needed
2. Which stage uses it
3. What happens if the file is missing
4. Whether it is used for logical, physical, timing, or verification purposes
```

## AI Understanding

The AI identified the following files as mandatory for successful mixed-signal integration:

| File | Purpose |
| ------------ | ------------------ |
| design_mux.v | Top-level RTL | 
| AMUX2_3V.v | Analog blackbox |
| AMUX2_3V.lef | Physical footprint |
| AMUX2_3V.lib | Timing model |
| AMUX2_3V.gds | Final layout merge |
| config.tcl | OpenLane setup |
| macro.cfg | Macro placement |

## Verification Against Repository

The AI-generated list was compared with the actual files available in the reference repository.

## Result

 All major files required for synthesis, floorplanning, placement, routing, timing analysis, and final GDS generation were correctly identified.

###  Key Learning

I learned that LEF does not contain transistor information.

It only describes:

* Cell dimensions
* Pin locations
* Routing blockages
* Placement information

This allows OpenLane to treat the analog block as a fixed hard macro.

---

# 🤖 Phase 3 – AI Generated Files

The following files were generated or validated using AI prompts.

### Generated Files
design_mux.v
AMUX2_3V.v 
AMUX2_3V.lef
AMUX2_3V.lib 
AMUX2_3V.gds 
config.tcl 
macro.cfg


---



# ⚠️ First AI Mistake Discovered

AI initially generated:

```verilog
module AMUX2_3V(
Vin1,
Vin2,
sel,
sel_b,
Vout
);
```

However, the actual repository used:

```verilog
module AMUX2_3V(
I0,
I1,
out,
select
);
```

## Problem

Port names did not match.

## Why This Is Dangerous

If port names differ:

* Synthesis succeeds
* Placement succeeds
* Routing may succeed

But LVS and functionality fail.

## Fix

Compared AI-generated blackbox with actual repository files.
Updated the port list.

### Lesson Learned

Never trust AI-generated interfaces without verification.

---

# 🚀 Physical Design Journey

## 🎯 Objective

To implement and study the complete RTL-to-GDSII Physical Design flow using OpenLane and SKY130, including synthesis, floorplanning, placement, CTS, PDN generation, routing, DRC/LVS verification, and final GDSII generation, while exploring AI-assisted methodologies for design understanding, debugging, and verification.

🤖 AI Prompt Used
```
Act as a VLSI physical design engineer using OpenLane on SKY130. I have a 
mixed-signal design "design_mux" with a digital top module and one analog 
hard macro "AMUX2_3V" (LEF/LIB/Verilog blackbox already prepared, paths in 
config.tcl). Walk me through the full RTL-to-GDS flow as a single ordered 
list of OpenLane interactive Tcl commands, in this exact order, with one 
line of comment above each explaining what file it consumes and what file 
it produces:

1. prep + add macro LEFs
2. synthesis
3. floorplanning
4. IO placement
5. global + detailed placement
6. tap/decap insertion
7. clock tree synthesis (CTS)
8. power delivery network (PDN) generation
9. routing
10. parasitics extraction (SPEF)
11. Magic DRC
12. LVS (Netgen)
13. final GDSII + LEF/MAG generation

For each stage, also tell me the exact result file name to check 
(e.g. results/<stage>/design_mux.def) and the one most common failure mode 
for a mixed-signal design with a fixed macro at that stage.
```
---

## Stage 1 – Synthesis

### Why This Stage Exists

Convert RTL into gate-level logic.

### Command

```tcl
run_synthesis
```

### Verification

The analog macro remained untouched.

### Why This Matters

If OpenLane tries to synthesize the analog macro, the entire mixed-signal flow breaks.

---

## 🏗️ Stage 2 – Floorplanning

### Purpose

Allocate physical space for:

* Standard cells
* Routing resources
* Analog macro

### Command

```tcl
init_floorplan
```


## Output
![image]()

### Verification

Macro successfully appeared inside die boundary.

### Key Observation

The macro behaves like a fixed building inside a city.

The router must route around it.

---

## 📍 Stage 3 – Placement

### Purpose

Place standard cells around the analog macro.

### Command

```tcl
run_placement
```


## Output
![image]()

### Verification

Cells legalized successfully.

### Learning

Placement quality directly impacts routing congestion later.

---

## ⚡ Stage 4 – Power Delivery Network

### Purpose

Connect all cells and macros to:

* VDD
* GND

### Command

```tcl
gen_pdn
```

### Learning

Without proper PDN, the macro may become electrically disconnected even if routing succeeds.

---

# 🌳 Stage 5 – Clock Tree Synthesis (CTS)

## Purpose

Clock Tree Synthesis is performed to distribute the clock signal uniformly throughout the design while minimizing clock skew and insertion delay.

an clock skew be minimized?


## Command

```tcl
run_cts
```

## Learning

A poorly synthesized clock tree can cause setup and hold violations even when routing is completed successfully.

---

# 🛣️ Stage 7 – Routing

## Purpose

Connect all signal nets physically using available routing resources.


## Command

```tcl
run_routing
```

## Output
![image]()

## Verification

✅ All nets routed successfully

✅ Macro connectivity preserved

---

# 🏁 Stage 8 – GDSII Generation

## Purpose

Generate the final manufacturable layout database.

The GDSII file is the final output delivered for fabrication.

## Learning

The LEF file is used during implementation, but the GDS file becomes critical during final layout generation and signoff.

---

# 🔍 Stage 9 – Final Layout Verification

## Purpose

Verify that the generated layout is physically and logically correct.


# ❌ Errors Encountered During Implementation


## Error 1

```text
DEF Read: encountered 3688 errors total
```

### Initial Reaction

At first, I assumed the flow had completely failed.

### AI Debug Prompt

```text
Analyze DEF import errors in Magic and identify possible LEF mismatches.
```

### AI Suggestions

* Check LEF pins
* Check routing layers
* Check macro dimensions
* Check obstruction definitions

### Result

Several LEF inconsistencies were identified.

---

## Error 2

```text
DEF Read: encountered 596 errors total
```

### Observation

Error count reduced from:

3688 ➜ 596

### Why This Was Important

This proved that the fixes were working.

Even though the problem was not fully solved, progress was measurable.

---

## Error 3

```text
DRC = 1267
```

### AI Suggested

* Increase halo
* Increase keepout margin
* Re-run placement
* Verify LEF geometry

### Result

DRC violations reduced significantly.

### Layout Inspection

Performed using:

* Magic
* KLayout

## Results

✅ No critical DRC violations

✅ Layout successfully generated

✅ Macro integrated correctly

✅ Final layout comparable to reference repository


The AI-assisted flow successfully reproduced the mixed-signal RTL-to-GDS methodology demonstrated in the reference repository while documenting the complete prompt-driven design and verification journey.

---

# 🎉 Final Success

## Final DRC Result

```text
DRC = 0
```
![image]()

### What This Means

✅ No design rule violations

✅ Layout accepted by Magic

✅ Closest match to reference repository

---

# 📊 Comparison with Reference Repository

| Feature        | Reference Repo | AI Flow |
| -------------- | -------------- | ------- |
| Synthesis      | ✅              | ✅       |
| Floorplan      | ✅              | ✅       |
| Placement      | ✅              | ✅       |
| CTS            | ✅              | ✅       |
| Routing        | ✅              | ✅       |
| DRC Clean      | ✅              | ✅       |
| GDS Generation | ✅              | ✅       |

---

# 🤔 Was AI Actually Useful?

## What AI Did Well

✅ Repository understanding

✅ File identification

✅ OpenLane guidance

✅ LEF debugging suggestions

✅ Configuration generation

## What AI Did Poorly

❌ Incorrect port names

❌ Assumed macro interfaces

❌ Could not automatically identify root cause of every DEF error

## Final Conclusion

AI significantly accelerated learning and debugging, but engineering verification remained essential. The final success was achieved through a combination of AI assistance, OpenLane execution, Magic verification, and manual analysis.

---

#  Key Lessons Learned

1. Never trust AI-generated interfaces without verification.
2. LEF correctness is critical for mixed-signal integration.
3. A layout view can appear correct while still containing DEF import errors.
4. DRC reports are more reliable than screenshots.
5. Mixed-signal physical design requires both digital and analog abstractions.
6. AI is an accelerator, not a replacement for verification.

---

#  Conclusion

This project successfully reproduced the mixed-signal RTL-to-GDS methodology of the reference repository using AI-assisted prompting.

The complete journey—from repository understanding, file generation, OpenLane execution, debugging, error rectification, and final DRC-clean layout generation—demonstrates how AI can be effectively integrated into modern VLSI physical design workflows while maintaining engineering validation at every stage.
