---
tags: [memory, hardware]
---

# Memory Technologies

> [!summary] Summary
> The layers of the [[Memory Hierarchy]] are only possible because different physical storage technologies offer very different speed/density/cost/volatility trade-offs. This note surveys the technologies actually used at each level.

## 1. SRAM (Static RAM) — registers, caches

- Stores each bit in a **cross-coupled flip-flop-like cell** (typically 6 transistors), see [[Sequential Logic Circuits]] for the underlying latch structure.
- **Fast** (single-digit ns access), but low density (6 transistors/bit) and expensive — used only where speed matters most: registers and on-chip caches (L1/L2/L3).
- No need for refresh — holds its value as long as powered ("static").

## 2. DRAM (Dynamic RAM) — main memory

- Stores each bit as **charge on a capacitor**, accessed through a single transistor (1T1C cell) — much denser than SRAM (1 transistor vs 6).
- **Must be periodically refreshed** (typically every ~64 ms) because capacitor charge leaks — this is the "dynamic" in DRAM, and refresh overhead is one reason DRAM is slower than SRAM.
- Organized into ranks, banks, rows, columns; a **row buffer** holds the currently "open" row — accessing another column in the same row (row hit) is much faster than switching rows (row miss, needing a precharge + activate cycle).
- Modern types: **DDR4/DDR5 SDRAM** (synchronous, double data rate — transfers data on both clock edges), **LPDDR** (low-power variants for mobile), **HBM (High Bandwidth Memory)** (stacked DRAM dies with a very wide bus, used in GPUs — see [[GPU Architecture]]).

## 3. Non-volatile / secondary storage

| Technology | Mechanism | Notes |
|---|---|---|
| NAND Flash (SSD) | Floating-gate transistors storing charge | No moving parts, fast random access; writes require block erase first; limited write endurance (wear leveling needed) |
| HDD | Magnetic domains on spinning platters, read/written by a moving head | Cheapest per GB at large scale, but slow (ms-scale seek times) due to mechanical movement |
| Optical (CD/DVD/Blu-ray) | Reflectivity changes read by laser | Mostly obsolete for primary storage, used for archival/distribution |

## 4. Emerging / non-traditional memory

- **MRAM (Magnetoresistive RAM)**: stores bits via magnetic orientation — non-volatile, fast, unlimited endurance; used in niche/embedded applications.
- **ReRAM / PCM (Phase-Change Memory, e.g., Intel Optane)**: bit stored as a material's resistance state (crystalline vs amorphous for PCM) — positioned to fill the latency/persistence gap between DRAM and SSD ("storage-class memory").
- **FeRAM (Ferroelectric RAM)**: uses ferroelectric material polarization — low power, non-volatile, used in specialized/embedded contexts.

## 5. Read-Only and firmware memory

- **ROM (mask ROM)**: contents fixed at fabrication — essentially obsolete for general use.
- **PROM/EPROM/EEPROM**: programmable after fabrication; EEPROM/Flash-based variants are electrically erasable, used for BIOS/UEFI firmware, microcontroller program storage.

## 6. Key comparison

| Technology | Volatile? | Relative speed | Relative density/cost | Typical role |
|---|---|---|---|---|
| SRAM | Yes | Fastest | Lowest density, priciest/bit | Registers, caches |
| DRAM | Yes | Fast | Medium density | Main memory |
| NAND Flash | No | Medium | High density, cheap/bit | SSDs |
| HDD | No | Slow | Highest density, cheapest/bit | Bulk/archival storage |

## See also
- [[Memory Hierarchy]]
- [[Cache Memory]]
- [[Sequential Logic Circuits]] — the flip-flop cell SRAM is built from

#memory #hardware
