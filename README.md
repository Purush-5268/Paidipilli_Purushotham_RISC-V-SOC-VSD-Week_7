🚀 *Week 7 VLSD Physical Design – RTL → GDS → SPEF Full Flow*

---

### 📘 **1.  Overview**

Modern VLSI design evolves from transistor innovations to full-chip physical implementation.
From *Planar → FinFET → GAA → CFET* and scaling limits like leakage, parasitics, contact resistance, and interconnect bottlenecks, we now depend heavily on **PDN**, **CTS**, **routing**, and **accurate parasitic extraction** to ensure real-chip performance.

BabySoC physical design demonstrates:

* How RTL becomes a physical layout
* How macros + std cells behave in placement & routing
* How PDN enables power integrity
* How CTS balances the clock tree
* How Raw routing → DRC-clean routing is achieved
* How SPEF extraction captures R & C parasitics
* How STA uses SPEF to give accurate timing

This week you perform a **real ASIC flow** using OpenROAD.

---

## 🛠 **2. Installing & Setting Up ORFS**

#### ✔ Clone OpenROAD-flow-scripts

```
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts
cd OpenROAD-flow-scripts
sudo ./setup.sh
```

#### ✔ Build OpenROAD

```
./build_openroad.sh --local
```

#### ✔ Source environment

```
source ../env.sh
```

#### ✔ Verify

```
yosys -help
openroad -help
```

---

## 📂 **3. Preparing VSDBabySoC Design**

Create directory:

```bash
mkdir ~/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/
```
**Copy all the folders to openroad folder** 
Copy into it:

* `src/` → vsdbabysoc.v, rvmyth.v, clk_gate.v
* `lef/` → avsddac.lef, avsdpll.lef
* `gds/` → avsddac.gds, avsdpll.gds
* `lib/` → avsddac.lib, avsdpll.lib
* `vsdbabysoc_synthesis.sdc`
* `macro.cfg`
* `pin_order.cfg`

### 1\. 🚀 Go to the `flow` Directory

All commands should be run from here.

```bash
cd ~/OpenROAD-flow-scripts/flow
```

### 2\. 🧹 Clean Up (Optional)

```bash
rm -rf designs/src/designs
```

### 3\. 📂 Create Sub-folders

This will create the `gds`, `include`, `lef`, and `lib` folders inside your `designs/sky130hd/vsdbabysoc/` directory.

```bash
mkdir -p designs/sky130hd/vsdbabysoc/{gds,include,lef,lib}
```

### 4\. 📋 Copy All Project Files

**Remember to change `~/Downloads/vsdbabysoc_files/`** to your actual source path.

```bash
# Set your source path (CHANGE THIS!)
export MY_SOURCE_FILES=~/Downloads/vsdbabysoc_files

# --- Copy Verilog files ---
cp $MY_SOURCE_FILES/{vsdbabysoc.v,rvmyth.v,clk_gate.v} designs/src/vsdbabysoc/

# --- Copy GDS files ---
cp $MY_SOURCE_FILES/{avsddac.gds,avsdpll.gds} designs/sky130hd/vsdbabysoc/gds/

# --- Copy Include files (using a wildcard) ---
cp $MY_SOURCE_FILES/sandpiper*.vh designs/sky130hd/vsdbabysoc/include/

# --- Copy LEF files ---
cp $MY_SOURCE_FILES/{avsddac.lef,avsdpll.lef} designs/sky130hd/vsdbabysoc/lef/

# --- Copy LIB files ---
cp $MY_SOURCE_FILES/{avsddac.lib,avsdpll.lib} designs/sky130hd/vsdbabysoc/lib/

# --- Copy SDC and CFG files ---
cp $MY_SOURCE_FILES/{vsdbabysoc_synthesis.sdc,macro.cfg,pin_order.cfg} designs/sky130hd/vsdbabysoc/
```

### 5\. ✍️ Create the `config.mk` File

This command will create and write the entire `config.mk` file for you. **Just copy and paste this whole block into your terminal and press Enter.**

```bash
cd ~/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc$
gedit config.mk
```
#### Copy This and Makee Changes According to your Paths
```bash
### BASIC DESIGN SETUP ###
export DESIGN_NICKNAME = vsdbabysoc
export DESIGN_NAME     = vsdbabysoc
export PLATFORM        = sky130hd
export RTLMP_ENABLE = 0
### VERILOG FILES ###
export VERILOG_FILES = \
    $(DESIGN_HOME)/src/vsdbabysoc/vsdbabysoc.v \
    $(DESIGN_HOME)/src/vsdbabysoc/rvmyth.v \
    $(DESIGN_HOME)/src/vsdbabysoc/clk_gate.v

export VERILOG_INCLUDE_DIRS = $(DESIGN_DIR)/include

### SDC ###
export SDC_FILE = $(DESIGN_DIR)/vsdbabysoc_synthesis.sdc

### EXTRA FILES (LEF/GDS/LIB) ###
export ADDITIONAL_LEFS = \
    $(DESIGN_DIR)/lef/avsddac.lef \
    $(DESIGN_DIR)/lef/avsdpll.lef

export ADDITIONAL_GDS = \
    $(DESIGN_DIR)/gds/avsddac.gds \
    $(DESIGN_DIR)/gds/avsdpll.gds

export ADDITIONAL_LIBS = \
    $(DESIGN_DIR)/lib/avsddac.lib \
    $(DESIGN_DIR)/lib/avsdpll.lib

### CLOCK ###
export CLOCK_PORT = CLK
export CLOCK_NET  = CLK
export CLOCK_PERIOD = 50.0

### FLOORPLAN ###
export DIE_AREA  = 0 0 3000 3000
export CORE_AREA = 20 20 2980 2980

### MACRO PLACEMENT ###
export MACRO_PLACEMENT_CFG = $(DESIGN_DIR)/macro.cfg

### DENSITY ###
export PL_TARGET_DENSITY = 0.10
export PL_MACRO_HALO_X = 300
export PL_MACRO_HALO_Y = 300

### TAP/DECAP/PDN ###
export FP_TAPCELL_DIST = 40
export RUN_TAP_DECAP_INSERTION = 0
export PDN_TCL = $(PLATFORM_DIR)/pdn.tcl

### CTS ###
export SKIP_GATE_CLONING = 1
export CTS_BUF_DISTANCE  = 600
export GRT_ADJUSTMENT = 0.7
export GRT_ADJUSTMENT_FACTOR = 0.7

export GRT_ANTENNA_MARGIN = 2
export GRT_REPAIR_MODE = 1

### MISC ###
export REMOVE_ABC_BUFFERS = 1
export MAGIC_ZEROIZE_ORIGIN = 0
export MAGIC_EXT_USE_GDS = 1
```
> 📸 <img width="1920" height="1080" alt="1 config mk" src="https://github.com/user-attachments/assets/0bee8ae9-afd0-49f4-96fa-c763fe174435" />

### 6\. ✅ Verify Your Setup

Run this command to check if all your files are in the right place:

```bash
ls -l designs/src/vsdbabysoc/ && ls -lR designs/sky130hd/vsdbabysoc/
```

You should see your Verilog files listed first, followed by the `config.mk`, `macro.cfg`, and all the sub-folders (`gds`, `lef`, `lib`) with their files.

**If everything looks correct, you are ready to run the flow.**

---

### 🚀 **4. Running the Automated RTL→GDS Flow**

Move to flow directory:

```
cd flow
```

---

#### **5️⃣ Synthesis**

```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk synth
```

📸 **Synthesis Output Images**

> 📸<img width="1920" height="1080" alt="3 command to run make design " src="https://github.com/user-attachments/assets/efbc2513-7c2c-4871-a0f2-3e589557c80b" />

> 📸 Files Created
> <img width="1200" height="500" alt="4 files created" src="https://github.com/user-attachments/assets/040579a6-4e2f-4148-941c-36aa248a0a2f" />

> 📸 yosys netlist
> <img width="1920" height="1080" alt="4 netlist file" src="https://github.com/user-attachments/assets/3757fe8c-72a4-4c15-bc56-8c710876a2f8" />
> 📸 Synth_Stats
> <img width="1920" height="1080" alt="4 synth check txt" src="https://github.com/user-attachments/assets/d376cc63-63e5-4832-b38a-de29e52a129a" />
><img width="1920" height="1080" alt="4 synth stat txt" src="https://github.com/user-attachments/assets/c75996ba-2b6c-4f3d-a72b-04d8417b31cf" />

---

#### **6️⃣ Floorplan**

```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan
```

📸 **Floorplan Images**

> 📸 floorplan run
> <img width="1920" height="1080" alt="5 floorplan run" src="https://github.com/user-attachments/assets/f307de3b-dec5-4ea3-9afd-8f6d8faa3eab" />

> 📸 floorplan files
> <img width="1920" height="1080" alt="6 floor plan files" src="https://github.com/user-attachments/assets/fc1fdbc2-33f8-4eda-ac9d-ce0019569436" />

**TO View Floorplan**
```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_floorplan
```

> 📸 floorplan output
> <img width="1920" height="1080" alt="6 flooorplan output" src="https://github.com/user-attachments/assets/121368ab-1922-44cf-956a-9a430b334bd4" />

#### **7️⃣ Placement**

```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place
```

📸 **Placement Images**

> 📸 placement done
> <img width="1920" height="1080" alt="7 placement done" src="https://github.com/user-attachments/assets/97e5d99a-7baf-4daa-89a7-114859c7f09d" />

> 📸 placement files
> <img width="1910" height="465" alt="7 placement files" src="https://github.com/user-attachments/assets/fb5484a7-55ba-44b7-869a-0679e0f79a19" />
**To View the OUTPUT**
```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place
```
> 📸 placement screenshot
> <img width="1920" height="1080" alt="7 placement screenshot" src="https://github.com/user-attachments/assets/88ef8c4c-1369-491c-90c4-f5ea96341838" />

**To View the below make sure the checkmark on heatmaps on the left panel**

> 📸 placement heatmap
> <img width="1920" height="1080" alt="7 placement heatmaps" src="https://github.com/user-attachments/assets/cdc4ded6-68dd-43a1-973c-098b372be0d3" />


> 📸 placement heatmap zoom
> <img width="1920" height="1080" alt="7 placement heatmaps  zoom" src="https://github.com/user-attachments/assets/6892a111-598a-45a0-8e88-a3c507f9f283" />

---

#### **8️⃣ CTS (Clock Tree Synthesis)**

```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts
```

📸 **CTS Images**

> 📸 run cts
> <img width="1920" height="1080" alt="8 run cts" src="https://github.com/user-attachments/assets/1a231cde-5b57-4c89-9757-177ea5dcddf7" />

> 📸 cts done
> <img width="1920" height="1080" alt="8 cts done" src="https://github.com/user-attachments/assets/4222493e-4b44-4d0d-8ed8-c3bc40b828f5" />

> 📸 cts files created
> <img width="1920" height="520" alt="8 cts files created " src="https://github.com/user-attachments/assets/8fae151b-7039-49dd-a589-c2eae70d5ac1" />

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_cts
```

> 📸 cts output
> <img width="1920" height="1080" alt="8 cts output" src="https://github.com/user-attachments/assets/d0ddf270-a0a3-42eb-a6c2-d1e9ed53806d" />
---


### **9️⃣ Tapcells, Blockages, PDN Adjustments (Before Routing — MUST CHECK)**

Before running **Routing**, these checks *must* be done to avoid congestion, overflow, and PDN-related errors.

---

#### ✅ **A. Clean Old Placement & CTS Data (VERY IMPORTANT)**

If you changed **PL_TARGET_DENSITY**, macro placement, pin constraints, tapcell spacing —
OpenROAD will reuse old `.odb` files unless you clean them.

#### **🧹 Clean old placement**

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk clean_place
```

#### **🧹 Clean old CTS**

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk clean_cts
```

#### **▶️ Re-run Placement**

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place
```

#### **▶️ Re-run CTS**

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts
```

### ✅ **B. Check Blockages (After Floorplan / After Placement)**

#### **🔍 Global logs check**

```bash
grep -R "Blockages" -n logs/sky130hd/vsdbabysoc
```

#### **🔍 Blockages in floorplan & placement**

```bash
grep -n "Blockages" logs/sky130hd/vsdbabysoc/base/*.log || true
```

### ✅ **C. Fix Tapcell Spacing (If tapcells too dense or too sparse)**

Edit:

```bash
nano flow/platforms/sky130hd/tapcell.tcl
```

Replace the line:

```
tapcell \
  -distance 60 \
  -tapcell_master "$::env(TAP_CELL_NAME)"
```

✔ This reduces density
✔ Helps global router
✔ Reduces blockages

#### **📸 tapcell images**

> <img width="1920" height="1080" alt="9 tapecell" src="https://github.com/user-attachments/assets/43af6870-d37e-46a5-b32c-5d4f2e155552" />

### ✅ **D. Check Tapcell Count**

```bash
grep -R "Inserted" logs/sky130hd/vsdbabysoc | grep tap
```
> ![Uploading 9 tapecell count to 3097.png…]()
---

### ✅ **E. Congestion Pre-Check BEFORE Routing**

#### **🔍 Routing resources**

```bash
grep -n "Routing resources analysis" logs/sky130hd/vsdbabysoc/base/5_1_grt.log -n -A12 || true
```

#### **🔍 Congestion report (very important)**

```bash
head -n 60 reports/sky130hd/vsdbabysoc/base/congestion.rpt || true
```

If congestion > 5% → routing WILL FAIL.

---

### ✅ **F. Verify Macro Placement**

```bash
grep -n "place_macro" results/sky130hd/vsdbabysoc/base/2_2_floorplan_macro.tcl
```

Your output:

```bash
1:place_macro -macro_name {dac} -location {60.665 540.745} -orientation R0
2:place_macro -macro_name {pll} -location {60.68 1182.79} -orientation R180
```

✔ Macro placement is correct.
✔ No overlap.
✔ Good spacing.

#### 📸 **Images to include in README**

> <img width="1920" height="1080" alt="9 pdn tcl comment out" src="https://github.com/user-attachments/assets/e81c8c01-69e0-42e3-bf60-220de783425f" />


### **🔟 Routing**

Now safely run routing:

```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk route
```

> <img width="1920" height="1080" alt="Screenshot from 2025-11-16 19-56-45" src="https://github.com/user-attachments/assets/2e554178-d2c8-4edd-ac06-cc8837ad553d" />

### **🟦 11. Post-Route SPEF Extraction**

```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk spef
```

---

<details>
<summary>Theory</summary>

### 50 Years of Microprocessor Trend Data
> <img width="698" height="336" alt="image" src="https://github.com/user-attachments/assets/6831ea34-d442-46f2-9fa7-10b36c4d7185" />
Transistors increased exponentially (Moore’s Law). Single-thread performance rose until ~2005 then slowed due to power and thermal limits. Clock frequency plateaued; power consumption increased sharply. Multi-core evolution replaced frequency scaling. Around 2007, mobile computing demanded high efficiency. Post-2010, datacenter-scale computing dominated, requiring energy-efficient architectures.

### CMOS Evolution and Next-Gen Candidates
> <img width="1190" height="594" alt="image" src="https://github.com/user-attachments/assets/97038383-d47b-49fa-95c7-326a73074f9a" />
  
CMOS scaling now depends on material innovation (MoS₂, Ge), patterning advances (EUV, High-NA EUV), new gate stacks (HKMG, NC-FET, TFET), better interconnects (Ru, semi-metals), and device structures (FinFET, GAAFET, VFET, 3DS-FET). DTCO and STCO optimize technology + design + system simultaneously.

### Transistor Evolution (Planar → FinFET → GAA)
><img width="1126" height="654" alt="image" src="https://github.com/user-attachments/assets/5ff79748-af0a-4ad8-ab94-a3197dfdb409" />
 
Planar MOSFETs had poor channel control. FinFETs introduced 3-D fins for lower leakage. GAA wraps the channel entirely for best electrostatic control and is used for advanced nodes (<3 nm). Each step improves performance, power efficiency, and scalability.

### CMOS Technology Inflection Points
> <img width="1202" height="678" alt="image" src="https://github.com/user-attachments/assets/a07dc7ea-cbc8-457f-997f-24e807699ad2" />

Key innovations by nodes:  
180 nm – voltage scaling  
130 nm – Cu interconnects  
90 nm – strained silicon  
65 nm – eSiGe  
45 nm – HKMG  
22 nm – FinFET  
7/5 nm – EUV + advanced patterning  
Scaling shifted from simple shrinking to material + structure innovation.

### Standard Cell Area Scaling (DDB, SDB, COAG, BS-PDN)
> <img width="1024" height="576" alt="image" src="https://github.com/user-attachments/assets/2d374f73-9f59-490c-85b2-e13c749b0edc" />
 
DDB and SDB improve diffusion isolation. COFG/COAG place contacts over gates to shrink cell area. BS-PDN moves power rails to the backside, reducing IR drop and enabling smaller standard cells.

### Variability Evolution (Planar → FinFET → NW)
> <img width="1162" height="590" alt="image" src="https://github.com/user-attachments/assets/b4bb43d0-bbdb-4817-bffc-151cc01a2628" />

Planar devices had high variability (~130 mV). FinFETs reduced it (~14 mV). Nanowire/GAA reduced further (~7 mV). Improved gate control and reduced dopant fluctuation drive this improvement.

### Parasitic Resistance (Planar, FinFET, GAA, CFET)
> <img width="1192" height="678" alt="image" src="https://github.com/user-attachments/assets/7e433444-3965-417b-95a7-9eecbc6a2ac6" />

Planar: Rc < Rch  
FinFET: Rc ≈ Rch  
GAA/CFET: Rc ~ 3× Rch  
Smaller contacts → larger resistance → challenges for scaling.

### Parasitic Resistance Improvement Techniques
> <img width="1188" height="672" alt="image" src="https://github.com/user-attachments/assets/ddb19d2f-9109-4092-b776-e1c0dfe4cb60" />

Breakdown of Rc sources shows most contribution from interface + MOL + contact. Improving doping, lowering barrier height, and optimizing BEOL/MOL dramatically reduce Rc.

### Parasitic Capacitance (Ceff) Scaling
><img width="1008" height="620" alt="image" src="https://github.com/user-attachments/assets/116bc58f-8ae9-40ed-8b24-09b3d15f1561" />
  
At 22 nm, Cfr dominates; at 14/10 nm, Cpc-ca increases; at 7 nm, Cg increases. Low-k spacers (SiBCN) and air-gap spacers reduce Ceff and improve delay.

### Need for New Channel Materials (2D Materials)
> <img width="1188" height="682" alt="image" src="https://github.com/user-attachments/assets/705f050d-5474-4623-a8cf-daa5f564ff86" />

MoS₂ offers high effective mass, ideal thickness (~0.65 nm), good bandgap, and low dielectric constant → reduces direct source-drain tunneling and leakage at sub-5 nm.

### Sub-5nm Transistor Scaling Challenges
Direct S-D tunneling grows rapidly; materials need high effective mass and uniform atomic layers. Lower CD relative to Cox needed to maintain electrostatics. 2D materials solve these issues.

### 1 nm Gate-Length MoS₂ Transistor
Uses CNT gate + MoS₂ channel + high-k ZrO₂. Demonstrates feasibility of atomic-scale FETs with excellent control and low leakage.

### All-2D MOSFET
Transistor built using graphene electrodes, MoS₂ channel, and h-BN dielectric. Shows high mobility, high on/off ratio, and strong electrostatic control, ideal for future nodes.

### Non-Planar Device Challenges
Hard to form high-quality single-crystal semiconductor on 3-D surfaces, increasing variability and complexity.

### Monolithic 3D CMOS
Stacked NMOS + PMOS layers reduce area, improve performance, and shorten interconnect lengths. Vertical logic integration increases density.

### Interconnect Evolution (Cu → Ru → Barrier-less)
Cu dual-damascene (7 nm), single-damascene (5 nm), barrier optimization (3 nm), Ru subtractive processes (<18 nm), and future barrier-less tall lines (<15 nm) reduce resistance and improve reliability.

### Selective Barriers in Cu BEOL
Selective TaN barriers improve Cu reliability, reduce resistivity, and optimize gap-fill for advanced scaling.

### Back-Side Power Delivery Network (BS-PDN)
BS-PDN reduces IR drop drastically, improves performance, shrinks standard cell height, and shortens power routes by moving power rails underneath the device layer.


</details>
