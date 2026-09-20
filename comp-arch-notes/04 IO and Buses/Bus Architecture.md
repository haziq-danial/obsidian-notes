---
tags: [io, bus, interconnect]
---

# Bus Architecture

> [!summary] Summary
> A bus is a shared communication pathway connecting the CPU, memory, and I/O devices. This note covers bus types, arbitration, and the evolution from shared parallel buses to modern point-to-point serial interconnects.

![[bus-architecture.svg]]

## 1. The three logical bus types

| Bus | Carries | Direction |
|---|---|---|
| Address bus | The memory/IO address being accessed | Unidirectional (CPU → memory/device) |
| Data bus | The actual data being transferred | Bidirectional |
| Control bus | Timing/coordination signals (read/write, clock, interrupts, bus grant) | Mixed |

The **width** of the address bus determines the maximum addressable memory ($2^{\text{width}}$ bytes, e.g., a 32-bit address bus addresses 4 GB). The width of the data bus affects how many bytes can be transferred per cycle.

## 2. Bus arbitration

When multiple devices (CPU, DMA controller, multiple cores) can request the bus, an **arbiter** decides who gets it next.

- **Daisy chain**: a priority signal passes physically from device to device; simple, but low-priority devices can starve.
- **Centralized (bus grant) arbitration**: a dedicated arbiter grants bus ownership based on a priority scheme (fixed priority, round-robin) to prevent starvation.
- **Distributed arbitration**: devices collectively negotiate access without a central arbiter (e.g., collision-detection schemes like classic Ethernet's CSMA/CD).

## 3. Synchronous vs asynchronous buses

- **Synchronous bus**: all devices share a common clock; simple, fast, but every device must run at the bus's clock speed, and bus length is limited by clock-skew tolerances.
- **Asynchronous bus**: uses a handshake protocol (request/acknowledge signals) instead of a shared clock — accommodates devices of very different speeds, easier to extend, but has more per-transaction overhead.

## 4. From shared parallel buses to point-to-point serial links

Classic buses (ISA, PCI, the old "front-side bus") were **shared and parallel**: all devices on one set of wires, one transaction at a time, width fixed by the number of physical wires.

Modern high-speed interconnects instead use **point-to-point serial links** with packet-based protocols:

| Interconnect | Used for | Notes |
|---|---|---|
| PCIe (PCI Express) | Peripherals (GPUs, SSDs, NICs) | Point-to-point serial lanes, switched topology, scalable by adding lanes |
| QPI / UPI (Intel), Infinity Fabric (AMD) | CPU-to-CPU, CPU-to-memory-controller | High-bandwidth coherent interconnect for multi-socket/multi-die systems |
| NVLink (NVIDIA) | GPU-to-GPU, GPU-to-CPU | Very high bandwidth for [[GPU Architecture|GPU]]/accelerator interconnects |
| SATA / NVMe | Storage | NVMe rides directly over PCIe lanes for much lower latency than legacy SATA/AHCI |

> [!note] Why the shift to serial
> At high frequencies, keeping many parallel wires electrically synchronized (avoiding skew/crosstalk) becomes harder than just running a few differential-pair serial lanes very fast and packetizing the data. This mirrors a broader trend: today's "buses" are increasingly switched point-to-point networks rather than literally shared wires.

## 5. Memory-mapped I/O vs port-mapped I/O

- **Memory-mapped I/O (MMIO)**: device registers appear at specific addresses within the normal memory address space; the CPU uses ordinary load/store instructions to access them.
- **Port-mapped I/O (PMIO)**: a separate address space with dedicated `IN`/`OUT` instructions (used historically on x86).

MMIO is simpler for compilers/software (no special instructions needed) and dominates modern system design; see [[IO Systems and Interrupts]] for how the CPU actually notices when a device needs attention.

## See also
- [[IO Systems and Interrupts]]
- [[Von Neumann vs Harvard Architecture]]
- [[Multicore and Multiprocessing]] — interconnects between cores/sockets

#io #bus
