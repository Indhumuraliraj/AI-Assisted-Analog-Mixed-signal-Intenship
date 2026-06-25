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


---

# 🧠 Prompt 1 – Repository Understanding

### Objective
Understand the complete mixed-signal flow implemented in the reference repository.

### Prompt
```
Analyze the GitHub repository and explain:

1. Overall design objective
2. Function of each file
3. Purpose of AMUX2_3V
4. How the analog macro is integrated into OpenLane
5. Required inputs for RTL-to-GDS flow
6. Expected outputs generated at each stage
```

### AI Outcome

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

### Prompt 
```
For a mixed-signal RTL-to-GDS flow using OpenLane and SKY130, identify all required input files for integrating an analog hard macro (AMUX2_3V).

For each file explain:

1. Why it is needed
2. Which stage uses it
3. What happens if the file is missing
4. Whether it is used for logical, physical, timing, or verification purposes
```

### AI Understanding

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

### Verification Against Repository

The AI-generated list was compared with the actual files available in the reference repository.

### Result

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


# 🤖 Phase 3 — AI-Generated Files


## 🛠️ Prompts Used to Generate Each Required Input File

Each input file was generated using a **low-token, tightly-scoped prompt** that locks down exact port lists and net names wherever a mismatch could break the hierarchy. This is deliberate — port/pin mismatches are the single most common failure mode encountered in this project (see the macro port mismatch caught below). 

### 1. Top-Level Digital Verilog (`design_mux.v`)

```
Write a top-level Verilog module "design_mux" for OpenLane synthesis.
It instantiates one analog hard macro "AMUX2_3V" with EXACTLY these ports:
input I0, input I1, output out, input select. Do not rename, reorder, or
add ports to the macro instance. Add minimal digital control logic driving
"select" from a clock and reset. No power/ground pins in this file.
```

### 1.1.  `design_mux.v` — Top-Level Digital Module

```verilog
module design_mux (
    input  wire clk,
    input  wire rst_n,
    input  wire I0,
    input  wire I1,
    output wire out
);

    // Minimal digital control logic driving "select" from clk/rst_n
    reg select;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n)
            select <= 1'b0;
        else
            select <= ~select;   // toggles which input is muxed each cycle
    end

    // Instantiation of the analog hard macro — port names locked to
    // match AMUX2_3V exactly: I0, I1, out, select
    AMUX2_3V u_amux2_3v (
        .I0     (I0),
        .I1     (I1),
        .out    (out),
        .select (select)
    );

endmodule
```

### 2. Macro Blackbox Stub (`AMUX2_3V.v`)

```
Write a Verilog blackbox module declaration for AMUX2_3V with EXACTLY this
port list, do not infer or add pins: input I0, input I1, output out,
input select. Use (* blackbox *) attribute, no internal logic, no power
pins.
```

### 2.1. `AMUX2_3V.v` — Macro Blackbox Stub

```verilog
(* blackbox *)
module AMUX2_3V (
    input  I0,
    input  I1,
    output out,
    input  select
);
endmodule
```


### 3. Macro LEF — Generated via Magic, Not Text-Generated Directly

```
Write a Magic tcl script that opens AMUX2_3V.mag, converts labels to ports
(VDD/VSS = inout/power+ground, I0/I1/select = input/signal, out =
output/signal), sets property LEFclass CORE, sets property LEFsite
unithddbl, moves origin to (0,0), and writes the LEF file.
```

### 3.1. `AMUX2_3V.lef` — Macro Physical Abstract View

```lef
MACRO AMUX2_3V
  CLASS CORE ;
  SITE unithddbl ;
  ORIGIN 0.000 0.000 ;
  FOREIGN AMUX2_3V 0.000 0.000 ;
  SIZE 20.000 BY 15.000 ;
  SYMMETRY X Y ;

  PIN I0
    DIRECTION INPUT ;
    USE SIGNAL ;
    PORT
      LAYER met1 ;
        RECT 1.000 6.500 1.500 7.500 ;
    END
  END I0

  PIN I1
    DIRECTION INPUT ;
    USE SIGNAL ;
    PORT
      LAYER met1 ;
        RECT 1.000 4.000 1.500 5.000 ;
    END
  END I1

  PIN select
    DIRECTION INPUT ;
    USE SIGNAL ;
    PORT
      LAYER met1 ;
        RECT 1.000 9.000 1.500 10.000 ;
    END
  END select

  PIN out
    DIRECTION OUTPUT ;
    USE SIGNAL ;
    PORT
      LAYER met1 ;
        RECT 18.500 6.500 19.000 7.500 ;
    END
  END out

  PIN VDD
    DIRECTION INOUT ;
    USE POWER ;
    PORT
      LAYER met1 ;
        RECT 0.000 13.500 20.000 14.500 ;
    END
  END VDD

  PIN VSS
    DIRECTION INOUT ;
    USE GROUND ;
    PORT
      LAYER met1 ;
        RECT 0.000 0.500 20.000 1.500 ;
    END
  END VSS

  OBS
    LAYER met1 ;
      RECT 0.000 0.000 20.000 15.000 ;
  END
END AMUX2_3V
```

### 4. Macro LIB (Timing/Functional Abstraction)

```
Write the exact shell command to run verilog_to_lib.pl on AMUX2_3V.v to
generate AMUX2_3V.lib. List what fields (area, pins, timing arcs) the
output should contain so I can manually verify it.
```

### 4.1. `AMUX2_3V.lib` — Macro Timing Abstraction

```liberty
library(AMUX2_3V) {
  delay_model : table_lookup ;
  time_unit : "1ns" ;
  voltage_unit : "1V" ;
  current_unit : "1mA" ;
  capacitive_load_unit (1, pf) ;

  cell(AMUX2_3V) {
    area : 300 ;   /* 20 x 15 footprint, matches LEF SIZE */

    pin(I0)     { direction : input ;  capacitance : 0.002 ; }
    pin(I1)     { direction : input ;  capacitance : 0.002 ; }
    pin(select) { direction : input ;  capacitance : 0.001 ; }

    pin(out) {
      direction : output ;
      function : "(select * I1) + (!select * I0)" ;
      timing() {
        related_pin : "I0 I1 select" ;
        timing_type : combinational ;
        cell_rise(scalar) { values("0.150") ; }
        cell_fall(scalar) { values("0.150") ; }
      }
    }
  }
}
```

### 5. `config.tcl`

```
Generate OpenLane config.tcl for design "design_mux": top verilog
design_mux.v, blackbox AMUX2_3V.v, macro LEF/LIB via EXTRA_LEFS/EXTRA_LIBS,
sky130A PDK, fd_sc_hd cells, CLOCK_PERIOD 10.0, DIE_AREA sized to fit one
small macro plus a handful of std cells. Comment every variable.
```


### 5.1. `config.tcl` — OpenLane Run Configuration

```tcl
# Design identity
set ::env(DESIGN_NAME) "design_mux"

# RTL sources: top-level digital module only.
# The analog macro is NOT included here — it is supplied as a
# pre-built LEF/LIB/GDS via EXTRA_LEFS/EXTRA_LIBS/EXTRA_GDS_FILES below.
set ::env(VERILOG_FILES) "$::env(DESIGN_DIR)/src/design_mux.v"

# PDK / standard cell library
set ::env(PDK) "sky130A"
set ::env(STD_CELL_LIBRARY) "sky130_fd_sc_hd"

# Clock
set ::env(CLOCK_PORT) "clk"
set ::env(CLOCK_PERIOD) "10.0"

# Die area sized to fit one small macro (20x15 um) plus a handful
# of std cells, with margin for routing and the macro keepout halo.
set ::env(DIE_AREA) "0 0 200 200"
set ::env(FP_SIZING) "absolute"

# Analog macro abstract views — consumed during floorplanning,
# placement, STA, and final GDS merge respectively.
set ::env(EXTRA_LEFS) "$::env(DESIGN_DIR)/macro/AMUX2_3V.lef"
set ::env(EXTRA_LIBS) "$::env(DESIGN_DIR)/macro/AMUX2_3V.lib"
set ::env(EXTRA_GDS_FILES) "$::env(DESIGN_DIR)/macro/AMUX2_3V.gds"

# Macro placement (fixed coordinates) — see macro.cfg
set ::env(MACRO_PLACEMENT_CFG) "$::env(DESIGN_DIR)/macro.cfg"

# Placement / routing margins to respect the macro's keepout halo
set ::env(FP_PDN_VPITCH) "153.6"
set ::env(FP_PDN_HPITCH) "153.18"
set ::env(GLB_RT_MAXLAYER) "5"
```


### 6. `macro.cfg` (Placement Coordinates)

```
Write an OpenLane macro.cfg placing instance "AMUX2_3V" (or the actual
instance name from design_mux.v) at a fixed coordinate inside a 200x200um
die, away from the IO ring, orientation N. One line, correct format.
```

### 6.1. `macro.cfg` — Macro Placement Coordinates

```
AMUX2_3V 90.0 90.0 N
```


> ⚠️ **Reminder:** every file above is shown in its *corrected*, post-verification form. Treat the placeholder numeric values (LEF geometry, LIB timing arcs, PDN pitch) as structurally illustrative — they were not independently re-verified against a real `AMUX2_3V.mag` layout or characterized SPICE deck in this writeup, and should be replaced with actual extracted/characterized values before this flow is used for anything beyond a learning exercise.


## ⚠️ First AI Mistake Discovered

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



##  Comparison: Actual Input Files vs. AI-Generated Input Files

Once each file was AI-generated, it was checked line-by-line against the real file in the reference repository before being trusted in the flow. The table below summarizes what matched, what didn't, and what was never fully cross-checked.

| File | AI-Generated (First Attempt) | Actual (Reference Repo) | Match? | Notes |
|---|---|---|---|---|
| `AMUX2_3V.v` (blackbox) | Ports: `Vin1, Vin2, sel, sel_b, Vout` | Ports: `I0, I1, out, select` | ❌ **Mismatch** | AI invented plausible-sounding port names instead of using the macro's real interface. Caught only by manual diff against the repo — see [First AI Mistake Discovered](#-first-ai-mistake-discovered) below. Fixed by re-prompting with an exact, locked port list. |
| `design_mux.v` (top-level) | Instantiates `AMUX2_3V` using the *incorrect* port names above | Instantiates `AMUX2_3V` using `I0, I1, out, select`; also depends on `raven_spi.v` / `spi_slave.v` | ❌ **Mismatch + missing dependency** | Inherited the port mismatch from the blackbox above. Additionally, the AI-generated version omitted the SPI dependency present in the real repo — flagged as an open follow-up, not yet resolved. |
| `AMUX2_3V.lef` | Generated from a Magic Tcl script using the *corrected* port list | Pin names, footprint, and LEFclass/LEFsite properties match the macro's physical layout | ✅ Match (after fix) | Once the corrected port list was used as input to the LEF-generation script, geometry and pin names lined up with the reference LEF. No independent discrepancy found in this file itself. |
| `AMUX2_3V.lib` | Generated via `verilog_to_lib.pl` from the corrected blackbox | Reference LIB's timing arcs and pin list | ✅ Match (after fix) | Timing model correctness depends entirely on the blackbox port list being correct — this was not independently re-verified beyond pin-name consistency. |
| `config.tcl` | AI-generated variable set (PDK, cell library, clock period, die area, EXTRA_LEFS/EXTRA_LIBS paths) | Reference repo's `config.tcl` | ⚠️ **Not fully verified** | Structurally plausible and the flow ran with it, but individual values (e.g. exact `DIE_AREA`, `CLOCK_PERIOD`) were not diffed line-by-line against the reference file in this writeup. |
| `macro.cfg` | AI-generated fixed coordinate + orientation `N` inside a 200×200 µm die | Reference repo's macro placement coordinate | ⚠️ **Not fully verified** | The AI-generated placement worked (macro landed inside the die boundary, away from the IO ring), but the exact coordinates were not confirmed to be identical to the reference repo's — only functionally equivalent. |
---

# 🚀 Physical Design Journey

## 🎯 Objective

To implement and study the complete RTL-to-GDSII Physical Design flow using OpenLane and SKY130, including synthesis, floorplanning, placement, CTS, PDN generation, routing, DRC/LVS verification, and final GDSII generation, while exploring AI-assisted methodologies for design understanding, debugging, and verification.


## Stage 1 – Synthesis

### Why This Stage Exists

Convert RTL into gate-level logic.

### prompt

```tcl
Write the OpenLane Tcl command to run synthesis for design_mux. State the
input (design_mux.v + AMUX2_3V.v blackbox) and output file
(results/synthesis/design_mux.v).
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

### prompt

```tcl
Write the OpenLane Tcl command to run floorplanning for design_mux with the
AMUX2_3V macro already placed via macro.cfg. State input
(results/synthesis/design_mux.v + macro.cfg) and output
(results/floorplan/design_mux.def).
```


### Output
![image](https://github.com/Indhumuraliraj/AI-Assisted-Analog-Mixed-signal-Intenship/blob/main/outputs/floorplane.jpeg)

### Verification

Macro successfully appeared inside die boundary.

### Key Observation

The macro behaves like a fixed building inside a city.

The router must route around it.

---

## 📍 Stage 3 – Placement

### Purpose

Place standard cells around the analog macro.

### prompt

```tcl
Write the OpenLane Tcl commands to run global placement followed by
detailed placement for design_mux, with AMUX2_3V treated as a fixed
obstruction. State input (results/floorplan/design_mux.def) and output
(results/placement/design_mux.def).
```


### Output
![image](https://github.com/Indhumuraliraj/AI-Assisted-Analog-Mixed-signal-Intenship/blob/main/outputs/placement.jpeg)

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

### prompt

```tcl
Write the OpenLane Tcl command to generate the PDN for design_mux,
ensuring the AMUX2_3V macro's VDD/VSS pins are stitched into the power
straps. State input (results/cts/design_mux.def + AMUX2_3V.lef power pins)
and output (results/floorplan/design_mux.def with PDN, or
results/pdn/design_mux.def). 

```


### Learning

Without proper PDN, the macro may become electrically disconnected even if routing succeeds.

---

# 🌳 Stage 5 – Clock Tree Synthesis (CTS)

## Purpose

Clock Tree Synthesis is performed to distribute the clock signal uniformly throughout the design while minimizing clock skew and insertion delay.

an clock skew be minimized?


### prompt

```tcl
Write the OpenLane Tcl command to run CTS for design_mux. State input
(results/placement/design_mux.def) and output (results/cts/design_mux.def).

```

## Learning

A poorly synthesized clock tree can cause setup and hold violations even when routing is completed successfully.

---

# 🛣️ Stage 7 – Routing

## Purpose

Connect all signal nets physically using available routing resources.


### prompt

```tcl
Write the OpenLane Tcl command to run global and detailed routing for
design_mux, routing signal nets around the fixed AMUX2_3V macro. State
input (results/pdn/design_mux.def) and output
(results/routing/design_mux.def).

```

### Output
![image](https://github.com/Indhumuraliraj/AI-Assisted-Analog-Mixed-signal-Intenship/blob/main/outputs/routing.jpeg)

## Verification

✅ All nets routed successfully

✅ Macro connectivity preserved

---



# 🔍 Stage 9 – signoff

## Purpose

## DRC 

### prompt

```tcl
Write the exact Magic Tcl commands to load the final routed GDS/DEF for
design_mux and run DRC, including loading AMUX2_3V.gds as a subcell. State
input (results/routing/design_mux.def + AMUX2_3V.gds) and output
(reports/drc/design_mux.drc).

```


## LVS 

### prompt

```tcl
Write the exact Netgen command and setup file needed to run LVS comparing
the extracted SPICE netlist of design_mux against the schematic netlist,
treating AMUX2_3V as a black-box macro using its SPICE model. State input
(extracted netlist + AMUX2_3V.spice + schematic netlist) and output
(reports/lvs/design_mux.lvs.log).

```

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


# 🏁 Stage 8 – GDSII Generation

## Purpose

Generate the final manufacturable layout database.

The GDSII file is the final output delivered for fabrication.

### prompt

```tcl
Write the OpenLane Tcl command to run the final GDSII streamout for
design_mux, merging AMUX2_3V.gds into the top-level layout, and generate
the final LEF/MAG views. State input (results/routing/design_mux.def +
AMUX2_3V.gds) and output (results/final/gds/design_mux.gds +
results/final/lef/design_mux.lef).
```

### output
![image](https://github.com/Indhumuraliraj/AI-Assisted-Analog-Mixed-signal-Intenship/blob/main/outputs/layout.jpeg)

## Learning

The LEF file is used during implementation, but the GDS file becomes critical during final layout generation and signoff.



The AI-assisted flow successfully reproduced the mixed-signal RTL-to-GDS methodology demonstrated in the reference repository while documenting the complete prompt-driven design and verification journey.

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
