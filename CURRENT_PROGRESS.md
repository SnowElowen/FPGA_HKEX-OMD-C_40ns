# SnowSakura-FPGA — Current Progress

## 2026-11-26: Golden GTH IP Frozen

**SnowSakura has reached its Golden GTH IP milestone. Scrambled RAW32 is moving through the SFP+ optical laboratory path on ZU15EG, and an HKEX OMD-C laboratory packet has been recovered and decoded. The next objective is a 43 ns-class A/B market-data architecture.**

The July GTH Top milestone established the hardware foundation. The November Golden GTH freeze establishes the reusable dual-buffer-bypass IP baseline for HKEX and future CME, LSE and NASDAQ laboratory projects.

## Active Hardware Baseline

| Item | Configuration |
|---|---|
| Device / tools | `xczu15eg-ffvb1156-2-i`, Vivado 2022.2, GT Wizard 1.7 |
| Serial interface | 10.3125 Gb/s, RAW32 TX and RX |
| Optical direction | X0Y6/SFP2 TX → OM4 → X0Y7/SFP1 RX |
| GT channels / master | X0Y6 and X0Y7; X0Y7 TX/RX master |
| GT buffers | **TX and RX Buffer Bypass**, RX bypass MULTI |
| Reference / PLL | 125 MHz MGTREFCLK → QPLL0; 200 MHz freerun |
| Clock sources | TXPROGDIVCLK / RXOUTCLKPMA |
| Nominal RAW32 user clock | 322.265625 MHz, ≈ 3.103030 ns |
| Fabric timing over-constraint | 322.56 MHz |
| Laboratory source | Scrambled RAW32 HKEX OMD-C frames |

The Golden GTH recreation Tcl and XCI define the frozen IP configuration. The top, XDC and scrambler are retained with the matching laboratory project.

## 43 ns A/B Attack Plan

| Allocation | Budget |
|---|---:|
| GTH PMA/PCS and associated physical path | ≈ 18 ns |
| Existing fabric baseline | 6 cycles ≈ 18.62 ns |
| Planned additional A/B arbitration | 2 cycles ≈ 6.21 ns |
| **Integrated wire-to-wire design target** | **≈ 42.82 ns — 43 ns class** |

The six-cycle fabric baseline follows the project's upstream-launch-to-TX-release accounting. The two-cycle allocation is the next A/B integration target; the ≈18 ns GTH allowance is a design budget. Independent hardware timestamping will measure the integrated wire-to-wire result and its latency distribution.

## Completed Milestones

- [x] GTH Top milestone and ZU15EG board bring-up.
- [x] Golden GTH IP freeze with TX/RX Buffer Bypass.
- [x] External native PRBS31 short-window board validation in the matching dual-bypass diagnostic build: `ila_checker=0x95`, `good_count=0xFFFFF`, approximately 2^20 RX cycles / 3.25 ms, with no current checker error or error/loss-of-lock sticky.
- [x] Scrambled RAW32 optical TX, RX capture and short-window laboratory packet decode.
- [x] RX alignment and framing demonstrated on the laboratory stream.
- [x] HKEX OMD-C Add Order message-header event demonstrated in captured-stream simulation.
- [x] Post-route timing and Post-Implementation SDF Timing Simulation for the documented implementations.
- [x] Earlier RX Buffer ON laboratory build demonstrated OMD-C parsing, order-state processing, price-level aggregation, top-of-book and snapshot export.

## Latest Evidence

| Evidence | Result |
|---|---|
| [Physical implementation and RX timing](img/Snipaste_2026-11-26_04-58-17.png) | RX clock-group setup **+1.223 ns**, hold **+0.012 ns**, pulse-width **+0.564 ns**. Displayed GTH RX → first capture-register path: setup **+1.316 ns**, delay **1.708 ns**, 0 reported logic levels. |
| [RX-to-parser simulation](img/Snipaste_2026-11-26_05-04-33.png) | Displayed cursor interval **9.320 ns**; `locked=1`, `marker_error=0`, message size `0x0020` and type `0x001E` with parser-valid. |
| [RX optical ILA](img/Snipaste_2026-11-26_05-45-35.png) | Captures changing RAW32 receive data, normalized output activity and probe flags. |
| [TX scrambled RAW32 ILA](img/Snipaste_2026-11-26_05-45-46.png) | Captures changing scrambled RAW32 words at the TX probe. |

The timing screenshot belongs to the shown fixed-phase implementation. The simulation interval, transceiver allocation and full wire-to-wire latency refer to different measurement boundaries.

## Next Targets

- [ ] **Line A/B arbitration:** integrate message-level sequence handling, ordering, duplicate suppression, conflict handling and gap/recovery within the two-cycle allocation.
- [ ] **10GbE NIC replay:** drive market-data stimulus into the FPGA through a separate validation path.
- [ ] **KU040 hardware timestamping:** build an independent nanosecond timestamp and measurement system.
- [ ] **Latency distribution:** measure integrated end-to-end P50, P99 and P99.9.
- [ ] **Long-duration BER:** extend physical-link measurement beyond the completed short-window PRBS31 validation.
- [ ] Extend the hardware foundation to **CME, LSE and NASDAQ** laboratory projects.

## Earlier Hardware Evidence

The July RX Buffer ON build remains part of SnowSakura's development record. It demonstrated programmed TX, optical reception, bounded alignment, OMD-C parser events, downstream order-state/aggregation/TOB/snapshot processing, an Eye Scan, routed timing and implemented timing simulation. Its reported WNS `+0.239 ns`, WHS `+0.017 ns`, and **12.420 ns across four intervals between five registered boundaries** belong to that implementation.

The [evidence chain](EVIDENCE_CHAIN.md) links the July and November captures and identifies their build and measurement boundaries. The [README](README.md) contains the project architecture and full development history.
