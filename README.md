# 512-Bit Custom SRAM Array IC Design

**Authors**: [Kevelyn Lin](https://github.com/WanYing1224), [Yuxiang Luo](https://github.com/lllyxxxx)

A custom CMOS 512-bit Static Random-Access Memory (SRAM) memory macro designed, laid out, and verified in **Cadence Virtuoso Studio** (gpdk045 technology) for **EE577A: VLSI System Design** at the University of Southern California (USC).

This repository contains the complete transistor-level schematics, custom DRC/LVS-clean layouts, parasitically extracted views (Quantus PEX), and ADE Maestro simulation environments for transient waveform verification.

---

## 📌 Architecture Overview

The memory block is structured as a **32-word × 16-bit** (512 bits total) SRAM array.

### Memory Map & Address Decoding
* **5-Bit Input Address Bus (`A[4:0]`)**: Latch-buffered on the rising edge of the system clock.
* **Row Decoder (`A[4:2]`)**: Upper 3 bits drive a 3-to-8 Row Decoder to select and activate 1 of 8 Wordlines (`WL[7:0]`).
* **Column Decoder (`A[1:0]`)**: Lower 2 bits drive a 2-to-4 pre-decoder and a 16-bit 4-to-1 Column Selector MUX to multiplex 64 bitline pairs down to 16 data channels.

### Hierarchical Block Diagram
```text
                     +---------------------------------------+
                     |     5-Bit Address Register (DFFs)     |
                     +-------------------+-------------------+
                                         |
                       +-----------------+-----------------+
                       | A[4:2]                            | A[1:0]
                       v                                   v
             +-------------------+               +-------------------+
             |  3-to-8 Row Dec.  |               |  2-to-4 Col Dec.  |
             +---------+---------+               +---------+---------+
                       |                                   |
                       v (Wordlines WL[7:0])               v (CSB / CS signals)
  +--------------------+-----------------------------------+--------------------+
  |                                                                             |
  |   +---------------------------------------------------------------------+   |
  |   |                   Precharge & Equalization Network                  |   |
  |   +----------------------------------+----------------------------------+   |
  |                                      |                                      |
  |                                      v                                      |
  |   +---------------------------------------------------------------------+   |
  |   |               8 x 16 SRAM Cell Bank Array (512 Bits)                |   |
  |   +----------------------------------+----------------------------------+   |
  |                                      |                                      |
  |                                      v                                      |
  |   +---------------------------------------------------------------------+   |
  |   |             Bitlines (BL / BL_B) & Column Selector MUX              |   |
  |   +-------------------+-----------------------------+-------------------+   |
  |                       |                             |                       |
  |                       v                             v                       |
  |           +-----------------------+     +-----------------------+           |
  |           |  16-Bit Write Driver  |     | 16-Bit Sense Amplifier|           |
  |           +-----------------------+     +-----------+-----------+           |
  |                                                     |                       |
  |                                                     v                       |
  |                                         +-----------------------+           |
  |                                         | 16-Bit Output Latch / |           |
  |                                         |     DFF Register      |           |
  |                                         +-----------------------+           |
  +-----------------------------------------------------------------------------+
```

---

## ⏱ Control Timing & Constraints

To avoid bitline contention and maintain read/write noise margins, control signals must follow strict pulse width and nesting constraints:

| Control Signal | Minimum Pulse Width | Design Constraint / Reason |
| :--- | :---: | :--- |
| `precharge_en` | **~0.7 ns** (OFF time) | Must turn **OFF** before Wordlines turn **ON** to prevent PMOS precharge to NMOS pull-down contention. |
| `decoder_en` | **~0.8 ns** (ON time) | Must stay active long enough for the Write Driver to discharge bitline capacitance to 0V. |
| `write_en` | **~0.6 ns** (ON time) | Must be nested completely within `decoder_en` to prevent writing to unintended rows. |
| `read_en` | **~0.3 ns** (ON time) | Opens briefly to pass the bitline voltage differential into the Sense Amp, then isolates it. |
| `sense_en` | **~0.4 ns** (ON time) | Fires strictly **after** `read_en` isolates the amplifier to quickly resolve valid outputs. |

---

## 📊 Extracted Macro Performance Summary

Post-layout parasitic extraction (Cadence Quantus PEX) metrics for the full memory macro:

| Metric | Value | Notes |
| :--- | :---: | :--- |
| **SRAM Bank Area** | **$15.68 \times 20.32\ \mu\text{m}^2$** | Core $8 \times 16$ SRAM Cell Bank |
| **Read Access Time** | **1.228 ns** | Measured from clock to valid data output |
| **Write Cycle Time** | **0.739 ns** | Time needed to latch and overwrite internal 6T storage nodes |
| **Max Operating Frequency** | **500 MHz** | Verified under extracted transient simulation |
| **Average Read Energy** | **46.74 fJ/op** | Normalized per operational cycle |
| **Average Write Energy** | **52.58 fJ/op** | Normalized per operational cycle |
| **Leakage Power** | **36.83 $\mu\text{W}$** | Static dissipation |

---

## 🛠 Layout & Physical Verification

All standard cells, peripheral drivers, decoders, and top-level layouts were designed in Cadence Virtuoso Layout Suite XL and passed complete physical verification:

* **DRC (Design Rule Checking)**: Clean using Pegasus DRC.
* **LVS (Layout Versus Schematic)**: 100% Matching using Pegasus LVS across all hierarchy blocks.
* **PEX (Parasitic Extraction)**: Extracted via Cadence Quantus PEX for post-layout back-annotation.

### Verified Layout Sub-Blocks:
1. **6T SRAM Cell** ($2.545\ \mu\text{m} \times 0.460\ \mu\text{m}$)
2. **$8 \times 16$ SRAM Bank** ($15.68\ \mu\text{m} \times 20.32\ \mu\text{m}$)
3. **16-bit Precharge & Equalization Network**
4. **2-to-4 Pre-Decoder & 3-to-8 Row Decoder**
5. **5-bit Address Input Register**
6. **16-bit 4-to-1 Column Selector Multiplexer**
7. **16-bit Write Driver Array**
8. **16-bit Differential Sense Amplifier Array**
9. **16-bit Output Data Latch / Register**
10. **Top-Level Integrated 512-bit SRAM Macro**

---

## 📁 Repository Structure

```text
512Bit_SRAM_ARRAY_DESIGN/
├── 512bit_SRAM_Array/                 # Integrated 512-bit SRAM Memory Macro (Schematic & Layout)
├── 128bit_SRAM_Array/                 # 128-bit Sub-array Macro
├── 5bit_Address_Register/             # Synchronous 5-bit Address Input Register
├── 3to8_Row_Decoder/                  # Wordline Decoder Block
├── 2to4_decoder/                      # Column MUX Decoder
├── 16bit_Column_Selector_MUX/         # Pass-gate Column Selector Array
├── 16bit_Sense_Amplifier/             # Differential Sense Amplifier Array
├── 16bit_Write_Driver/                # Bitline Write Driver Array
├── 16bit_Output_Register/             # Output Data Register with Reset
├── 16bit_Latch/                       # Read Data Isolation Latch
├── SRAM/                              # Base Custom 6T SRAM Bitcell
├── SRAM_BANK/                         # 8x16 Bitcell Core Storage Bank
├── precharge/ & precharge_extend/     # Bitline Precharge & Equalization Network
├── AND2X1/, AND3X1/, DFFQX1/, DFFRX1/ # Cell Primitives & Flip-Flops
└── vector_files/                      # Digital Testbench Stimulus Vectors (.vec)
```

---

## 🧪 How to Run Verification Simulations

1. Add the repo directory to your `cds.lib` file in Cadence Virtuoso:
   ```text
   DEFINE EE577a_S26 ./512Bit_SRAM_ARRAY_DESIGN
   ```
2. Open Virtuoso Studio and navigate to `EE577a_S26` $\rightarrow$ `512bit_SRAM_Array` $\rightarrow$ `maestro`.
3. Load the pre-configured ADE Maestro state from `active.state`.
4. Ensure the testbench points to the appropriate digital vector file in `vector_files/` (e.g., `sram_array_9924884571.vec` or `sram_512bit_final1.vec`).
5. Execute the **Transient Analysis** simulation to observe the write/read access waveforms.
