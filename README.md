# openlane-soc-design

RTL-to-GDSII physical design flow using OpenLANE & Sky130 PDK | VSD SoC Design and Planning Workshop.

## Digital VLSI SoC Design and Planning — RTL to GDSII

> A 2-week hands-on workshop on complete RTL-to-GDSII flow for digital VLSI SoC design, organised by **VSD (VLSI System Design)** in collaboration with NASSCOM. This repository documents my learning, lab outputs, and key takeaways from each day.

### Day 1 — Inception of Open-Source EDA, OpenLANE & Sky130 PDK

**Understanding the Chip Package**

When we look at any embedded board and point to what we call the "chip," we're actually looking at the **package** — a protective casing around the actual silicon die. The real chip sits in the centre of this package and communicates with the outside world via **wire bonding** — tiny wires that connect the chip's pads to the package pins.

**Inside the Chip: Core, Pads, and Die**

Zooming into the chip itself, all signals between the chip and the external world pass through **pads** placed around the periphery. The region enclosed by the pads is called the **core** — this is where all the actual digital logic lives.
# 📖 Day 1: Exploring the Basics of SoC Design and Open-Source EDA

As I dive into the world of physical design, I wanted to document my foundational understanding of how chips are actually built and the open-source tools that make this possible. Here are my key takeaways from getting started with the RTL-to-GDSII flow.

---

## 🧩 My Understanding of the Chip Package

When I look at an embedded board and point to the black square I usually call the "chip," I'm actually just looking at the **package**—the protective plastic or ceramic casing. The *real* silicon chip sits right in the centre of this package. It communicates with the outside world through **wire bonding**, which are incredibly tiny wires connecting the chip's internal pads to the pins I see on the outside of the package.

### Zooming Inside: Core, Pads, and the Die

If I zoom into the chip itself, I can see how the layout is structured:
*   **Pads:** Placed around the periphery, these act as the gateways. All signals passing between the chip and the external world go through here.
*   **Core:** The region enclosed by the pads. This is the heart of the chip where all the actual digital logic lives. 
*   **Die:** Together, the core and the pads form the die, which is the fundamental manufacturing unit of the chip.

### Key Terminology I'm Tracking
*   **Foundry:** The physical fabrication plant where the chips are actually manufactured.
*   **Foundry IPs:** Specialized IP blocks (like PLLs or SRAMs) that require deep, process-specific knowledge to implement properly.
*   **Macros:** Reusable, purely digital logic blocks.

---

## 🌉 From Software to Silicon: The ISA Bridge

I find the transformation from a high-level program to physical hardware really fascinating. When a C program runs on a chip, it goes through a multi-layer transformation:

1.  My C code is first compiled into **RISC-V assembly** (or whichever Instruction Set Architecture I'm targeting).
2.  The assembler then converts those instructions into **binary machine code** (0s and 1s).
3.  For the hardware to understand this, there needs to be an **RTL implementation** of that specific ISA.
4.  Finally, that RTL is synthesized and pushed through the full **PnR (Place and Route)** flow to become a physical hardware layout.

Ultimately, the system software stack (OS → Compiler → Assembler) acts as the crucial bridge translating what I write as a programmer into what the hardware physically executes.

---

## 🔓 Why Open-Source EDA is a Game Changer

For a fully open-source ASIC design flow to exist, I learned that we need three critical components:
1.  **RTL Designs:** Open-source hardware descriptions (like those found on opencores.org).
2.  **EDA Tools:** The software for synthesis, placement, routing, and verification.
3.  **PDK Data:** The Process Design Kit, which contains the manufacturing rules and standard cell libraries specific to a foundry's process node.

Historically, PDKs were closely guarded secrets distributed only under strict NDAs, making actual chip design practically inaccessible to hobbyists or independent students like me. That all changed in **June 2020**, when Google and SkyWater Technology released the **Sky130 PDK**. As the world's first open-source PDK, it was a massive milestone that blew the doors open for the VLSI community.

---

## ⚙️ OpenLANE and the Automated RTL to GDSII Flow

To bring my designs to life, I'm using **OpenLANE**. It's an automated, open-source flow that wraps around several EDA tools to take an RTL netlist all the way to a final GDSII layout file (the file you'd send to a foundry). 

Here is a breakdown of the specific tools OpenLANE uses under the hood for each stage of the flow:

| Flow Stage | Open-Source Tool(s) Used |
| :--- | :--- |
| **Synthesis** | Yosys, ABC |
| **Floorplan & PDN** | OpenROAD |
| **Placement** | OpenROAD |
| **CTS (Clock Tree Synthesis)** | TritonCTS |
| **Routing** | FastRoute, TritonRoute |
| **SPEF Extraction** | OpenRCX |
| **GDS Streaming** | Magic, KLayout |
| **Timing Analysis** | OpenSTA |
| **DRC & LVS (Physical Verification)** | Magic, Netgen |

## 🛠️ Lab Session: Interactive OpenLane Flow & Synthesis

To get hands-on with the flow, I needed to launch OpenLane in interactive mode. This allows for step-by-step execution and inspection of the design at each individual stage.

### 1. Invoking the OpenLane Environment
First, I navigated to the working directory, mounted the Docker container, and launched the tool.

```bash
# Navigate to the OpenLane directory
cd ~/Desktop/OpenLane

# Mount the Docker container
make mount

# Launch the OpenLane flow in interactive mode
./flow.tcl -interactive

### 2. Package Inclusion and Design Preparation
Once inside the OpenLane interactive shell (`%` prompt), I loaded the required OpenLane packages and set up my specific design (`picorv32a`). 

```tcl
# Require the OpenLane package
package require openlane 0.9

# Prepare the picorv32a design
prep -design picorv32a
```
*Note: The `prep` command creates a new run directory tagged with the current date and time, and merges the technology LEF and cell LEF files into a single `merged.lef` file.*

**Terminal State Before Synthesis:**
![Before Synthesis](images/before%20synthesis.png)

### 3. Logic Synthesis
With the design prepared, I ran logic synthesis using Yosys. This step translates the RTL (Verilog) into a logic gate-level netlist mapped to the Sky130 standard cell library.

```tcl
run_synthesis
```
**Synthesis Execution & Completion:**
![After Synthesis](images/after%20synthesis.png)

### 4. Calculating the Flop Ratio
After synthesis completed, I analyzed the generated netlist statistics in the terminal to determine the Flop Ratio. This ratio represents the proportion of D-Flip Flops (DFFs) relative to the total number of cells in the synthesized design.

The formulas used for this calculation are:

$$ \text{Flop Ratio} = \frac{\text{Number of D Flip-Flops}}{\text{Total Number of Cells}} $$

$$ \text{Percentage of DFFs} = \text{Flop Ratio} \times 100 $$

**Finding the Total Number of Cells:**
![Flop Ratio Total Cells](images/flop%20ratio1.png)
**Finding the Number of D-Flip Flops (`sky130_fd_sc_hd__dfxtp_2`):**
![Flop Ratio DFFs](images/flop%20ratio2.png)

Based on my specific terminal output, I identified the following values:

*   **Total Number of Cells:** 15,762
*   **Number of D-Flip Flops (`sky130_fd_sc_hd__dfxtp_2`):** 1,613

Plugging my values into the formula:

$$ \text{Flop Ratio} = \frac{1613}{15762} = 0.1023347 $$

This means my design has a **Flop Ratio of 10.23%**.


---

## 📖 Day 2 — Floorplanning and Introduction to Library Cells

### Chip Floorplanning — Core Area and Utilisation
Floorplanning is about deciding where everything goes on the chip. Two key parameters drive this:

*   **Utilisation Factor** = `(Area occupied by Netlist) / (Total Core Area)`
    *   A utilisation of 0.5-0.6 is typical — you want room for buffers, routing, etc.
*   **Aspect Ratio** = `Height / Width of the core`
    *   A ratio of 1 means a square; anything else is a rectangle.

### Pre-Placed Cells and Decoupling Capacitors
**Pre-placed cells** (like memories, PLLs, and complex IP blocks) are fixed in position before automated placement runs. Their location is determined manually based on connectivity and power intent.

**Decoupling capacitors** are placed around pre-placed cells to act as local charge reservoirs — they compensate for voltage drops caused by switching activity and ensure these blocks see clean power.

### Power Planning — Mesh vs Ring
A good power grid uses both **power rings** around the core and a **power mesh** across the chip. Multiple VDD and VSS rails are distributed in both metal layers so that every standard cell has a nearby power tap, minimising IR drop and electromigration risk.

### Pin Placement and Logical Cell Blockage
Input and output pins are placed along the chip boundary. The relative placement of pins is guided by connectivity — a pin that drives logic deep in the core should be closer to that logic. The area between the core and the die boundary (I/O ring area) is blocked from automated cell placement to reserve it for pin buffers and ESD cells.

### 🛠️ Lab — Floorplan and Placement
### 1. Floorplanning and I/O Pin Placement
During the floorplan stage, the default I/O pin placement was modified to cluster the pins together. By default, OpenLane sets the `FP_IO_MODE` to `1` (equidistant spacing). We overrode this global environment variable to `2` to stack the pins based on a specific algorithm.

**Commands (inside OpenLane interactive shell):**

```tcl
# Modify the I/O pin placement mode
set ::env(FP_IO_MODE) 2

# Run the floorplan stage
run_floorplan
```

**Viewing the Floorplan in Magic:**
To verify the clustered pins visually, the layout is opened in Magic. *(Note: The LEF file must be read before the DEF file to prevent standard cell read errors).*

```bash
# Open a standard terminal and navigate to the OpenLane root
cd ~/Desktop/OpenLane/designs/picorv32a/runs/RUN_2026.08.10_10.56.09/results/floorplan

# Launch Magic from the results/floorplan directory
magic -T /home/vscode/.ciel/sky130A/libs.tech/magic/sky130A.tech &

# Inside the Magic tkcon console:
lef read ../../tmp/merged.nom.lef
def read picorv32a.def
```
![Magic tkcon Window](images/tkcon%20window.png)
**Final Layout after Floorplanning (Showing clustered I/O pins):**
![Layout after Floorplanning](images/layout%20after%20floorplanning.png)

**Clustered Components:**
![Clustered Components](images/clustered%20.png)

**Floorplan Verification:**
![Verification Floorplanning](images/verification%20floorplanning.png)

### 2. Placement
Following a successful floorplan, global and detailed placement are executed to assign physical coordinates to the standard cells (moving them out of the origin and into standard cell rows).

**Command (inside OpenLane interactive shell):**

```tcl
run_placement
```

**Viewing Placement in Magic:**

```bash
# Launch Magic from the results/placement directory
magic -T /home/vscode/.ciel/sky130A/libs.tech/magic/sky130A.tech &

# Inside the Magic tkcon console:
lef read ../../tmp/merged.nom.lef
def read picorv32a.def
```
---

## 📖 Day 3 — Design and Characterisation of Library Cells using Magic & ngspice

### CMOS Inverter — SPICE Deck
To characterise a standard cell, we write a SPICE netlist describing the PMOS and NMOS transistors along with their W/L ratios, supply voltage, input stimulus, and load capacitance.

**Key parameters we extract from simulation:**
*   **Rise time** — 20% to 80% of output rising edge
*   **Fall time** — 80% to 20% of output falling edge
*   **Propagation delay** — 50% input to 50% output

### Detailed 16-Mask CMOS Fabrication Process
The physical fabrication of a CMOS integrated circuit relies on 16 photolithography masks to define active regions, wells, gates, implants, contacts, and multi-layer interconnections:

1. **Mask 1 — Active Region (Substrate / LOCOS):** Defines the active silicon regions where NMOS and PMOS transistors will be built, separating them using Local Oxidation of Silicon (LOCOS) or STI (Shallow Trench Isolation).
2. **Mask 2 — N-Well:** Defines the N-well regions on the P-substrate to house the PMOS transistors.
3. **Mask 3 — P-Well:** Defines the P-well regions (in a twin-tub process) to optimize NMOS threshold voltages.
4. **Mask 4 — Polysilicon Gate Patterning:** Patterns the deposited polysilicon layer to form the gate electrodes for both NMOS and PMOS transistors.
5. **Mask 5 — N- Select (Lightly Doped Drain - N- LDD):** Implants a low-concentration n-type dopant near the NMOS gate edges to mitigate hot-carrier effects.
6. **Mask 6 — P- Select (Lightly Doped Drain - P- LDD):** Implants a low-concentration p-type dopant near the PMOS gate edges.
7. **Mask 7 — N+ Source/Drain Implantation:** Defines and heavily implants the $N^+$ regions for the NMOS source/drain terminals and N-well body contacts.
8. **Mask 8 — P+ Source/Drain Implantation:** Defines and heavily implants the $P^+$ regions for the PMOS source/drain terminals and P-substrate body contacts.
9. **Mask 9 — Contact Vias (Contacts to Gate/Active):** Opens contact windows through the Inter-Level Dielectric (ILD) down to the polysilicon gates and $N^+/P^+$ active regions.
10. **Mask 10 — Metal 1 Routing:** Patterns the first metal layer (Metal 1 / Local Interconnect) to connect internal standard cell transistors.
11. **Mask 11 — Via 1:** Defines vertical connection cuts (Via 1) through the dielectric between Metal 1 and Metal 2 layers.
12. **Mask 12 — Metal 2 Routing:** Patterns the second metal layer for inter-cell routing and distribution.
13. **Mask 13 — Via 2:** Defines connection cuts (Via 2) between Metal 2 and higher-level metal layers.
14. **Mask 14 — Metal 3 / Top Metal:** Patterns the upper metal layer, commonly used for global power ($V_{DD}$) and ground ($V_{SS}$) bus routing.
15. **Mask 15 — Terminal / Pad Definition:** Opens access cuts through the top passivation layer to allow bonding wire connections to the I/O pads.
16. **Mask 16 — Passivation / Protective Layer:** Patterns the final glass passivation layer ($Si_3N_4$ / $SiO_2$) to protect the entire die from moisture, mechanical damage, and contamination.

### 🛠️ Lab — Cloning and Characterising a Custom Inverter Cell

**Cloning the Standard Cell Repository**
```bash
git clone [https://github.com/nickson-jose/vsdstdcelldesign.git](https://github.com/nickson-jose/vsdstdcelldesign.git)
```
