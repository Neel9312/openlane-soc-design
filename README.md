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
