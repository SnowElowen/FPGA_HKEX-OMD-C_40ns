# SnowSakura-FPGA — Hardware and Timing Evidence

## 2026-11-26 — Golden GTH IP Frozen, Optical RAW32 Running

SnowSakura's Golden GTH IP is frozen. The ZU15EG laboratory system has transmitted scrambled RAW32 over SFP+ and recovered an HKEX OMD-C laboratory packet on the receive path. The project now moves toward **43 ns A/B architecture**, with sequence-aware arbitration and independent hardware latency measurement as the next targets.

This index brings the implementation, simulation and hardware captures together, while preserving the earlier RX Buffer ON results as part of the engineering history. See the [project README](README.md) for the architecture and development log.

## Golden GTH Build Identity

| Item | Configuration |
|---|---|
| Repository | `SnowElowen/FPGA_HKEX-OMD-C_40ns` |
| Device | `xczu15eg-ffvb1156-2-i` |
| Vivado / IP | Vivado 2022.2 / GT Wizard 1.7 |
| Serial rate and width | 10.3125 Gb/s, RAW32 TX and RX |
| Optical direction | X0Y6/SFP2 TX → OM4 → X0Y7/SFP1 RX |
| GT lanes / master | X0Y6 and X0Y7; TX/RX master X0Y7 |
| Buffers | TX and RX Buffer Bypass; RX bypass mode MULTI |
| Reference and PLL | 125 MHz MGTREFCLK, QPLL0; 200 MHz freerun |
| User clocks | TXPROGDIVCLK / RXOUTCLKPMA |
| Nominal RAW32 clock | 322.265625 MHz, period ≈ 3.103030 ns |
| Fabric over-constraint | 322.56 MHz |
| Source | Scrambled RAW32 HKEX OMD-C laboratory stream |

The Golden GTH XCI and recreation Tcl establish the reusable IP configuration. The screenshots below identify their own implementation or capture; the Golden IP freeze does not make their individual build histories interchangeable.

## November 2026 — Evidence on the Board and in Vivado

| Evidence | Observation | Layer |
|---|---|---|
| [ZU15EG implementation and RX timing](img/Snipaste_2026-11-26_04-58-17.png) | RX clock-group setup **+1.223 ns**, hold **+0.012 ns**, pulse-width **+0.564 ns**; shown GTH RX → `raw0_q_reg` path setup **+1.316 ns**, data-path delay **1.708 ns**, 0 reported logic levels | Implemented timing, fixed-phase build; displayed requirement 3.1 ns |
| [RX-to-parser waveform](img/Snipaste_2026-11-26_05-04-33.png) | Cursor interval **9.320 ns**; `locked=1`, `marker_error=0`; parser-valid event shows message size `0x0020` and type `0x001E` (Add Order) | Captured-stream simulation |
| [RX optical ILA](img/Snipaste_2026-11-26_05-45-35.png) | RX RAW32 data and normalized output change; RX probe flags show activity. A short optical laboratory capture was decoded into the OMD-C packet | On-board RX capture / laboratory decode |
| [TX scrambled RAW32 ILA](img/Snipaste_2026-11-26_05-45-46.png) | Changing scrambled RAW32 words are visible at the TX probe | On-board TX capture |
| External native PRBS31 diagnostic, 2026-11-10 | Matching dual-bypass diagnostic build reported `ila_checker=0x95`, `good_count=0xFFFFF`, and no current checker error or error/loss-of-lock sticky; approximately 2^20 RX cycles / 3.25 ms | Short-window board validation |
| Single-path post-implementation SDF timing simulation | Previously completed in February 2026; project report records the registered fast path and selected routed timing metrics | Implemented timing simulation |

The 9.320 ns cursors identify an interval in the displayed RX/parser simulation. The 18 ns transceiver allowance below and the complete wire-to-wire latency are separate quantities.

## 43 ns A/B Latency Budget

| Component | Allocation |
|---|---:|
| GTH PMA/PCS and associated physical-path allowance | ≈ 18 ns |
| Existing fabric baseline, upstream launch → TX release | 6 cycles ≈ 18.62 ns |
| Planned additional A/B arbitration | 2 cycles ≈ 6.21 ns |
| **Integrated design target** | **≈ 42.82 ns — 43 ns class** |

Cycle values use the nominal 322.265625 MHz clock. The project’s existing six-cycle accounting and the additional two-cycle A/B allowance are the design budget; the transceiver allocation is an estimate. The integrated A/B endpoints and full wire-to-wire measurement will be established in the next implementation and hardware measurement stage.

## Development Status

- [x] ZU15EG bring-up and GTH Top milestone.
- [x] Golden GTH IP freeze with TX/RX Buffer Bypass.
- [x] Short-window external native PRBS31 board validation in the matching diagnostic build.
- [x] Scrambled RAW32 optical laboratory transmission and short-window receive capture / decode.
- [x] RX alignment and framing demonstrated on the laboratory stream.
- [x] OMD-C Add Order message-header event demonstrated in captured-stream simulation.
- [x] Post-route timing and post-implementation SDF timing simulation for the documented implementations.
- [ ] Integrated two-cycle Line A/B arbitration, deduplication, sequence and recovery.
- [ ] 10GbE NIC market-data replay.
- [ ] KU040 independent hardware nanosecond timestamping.
- [ ] End-to-end latency distribution: P50, P99 and P99.9.
- [ ] Long-duration BER measurement.

---

## July 2026 — Earlier RX Buffer ON Evidence

| Boundary | Public artifact | Exact observation | Evidence level |
|---|---|---|---|
| Golden top TX launch | [TX frame launch](img/2026-07-25_ila_tx_frame_launch.png) | frame start, busy transition, first programmed word | real hardware ILA |
| Golden top complete TX frame | [Complete frame](img/2026-07-25_ila_tx_complete_frame.png) | sequential word indices and programmed words before idle | real hardware ILA |
| Optical receiver margin | [IBERT Eye Scan](img/2026-07-23_ibert_eye_scan_open_ui_77_78.png) | open UI 77.78%, open area 6720, configured `1e-10` dwell BER | real hardware RX sampler |
| Raw32 receive/alignment | [RX capture](img/2026-07-23_rx_capture_parser_fields.png) | `align_locked = 1`, registered 12-word capture | real hardware RXUSRCLK ILA |
| Packet-to-parser contract | [Valid-chain capture](img/2026-07-23_packet_valid_parsed_valid.png) | capture index 0…11, packet-valid pulse, parsed-valid pulse, no parsed error | real hardware ILA |
| Repeated packet handling | [Continuous parser](img/2026-07-23_continuous_multi_packet_parser.png) | repeated Add/Modify output across two-message, Heartbeat, and one-message forms | real hardware ILA |
| Parser-to-state | [Book closure](img/2026-07-23_exchange_simulator_book_closure.png) | registered order-state activity with `position_error = 0` | real hardware ILA |
| Registered book export | [Snapshot closure](img/2026-07-23_exchange_simulator_snapshot_closure.png) | version 1, offer level `0x12`, price `0x00007A12`, quantity `0x3E8` | real hardware ILA |
| Five registered boundaries | [Post-Implementation Timing Simulation](img/2026-07-25_post_impl_rx3_parser1_tx1.png) | 12.420 ns across four intervals of RX3 + Parser1 + TX1 | implemented netlist simulation |
| Full implemented timing | [Timing summary](img/2026-07-25_post_impl_timing_summary.jpeg) | WNS `+0.239 ns`, WHS `+0.017 ns`, zero failing endpoints | post-route STA |
| RX registered boundary | [0.900 ns path audit](img/2026-07-25_rx_registered_path_0900ns.png) | 0 LUT levels, 0.546 ns total, `+0.379 ns` slack | post-route STA |

## HKEX OMD-C Laboratory Packet Arithmetic

| Packet | Byte arithmetic | Encoded value |
|---|---:|---:|
| Add Order | 16-byte Packet Header + 32-byte Msg30 | `PktSize = 16'h0030`, `MsgCount = 1` |
| Heartbeat | 16-byte Packet Header, no message | `PktSize = 16'h0010`, `MsgCount = 0` |
| Add + Modify | 16-byte Packet Header + 32-byte Msg30 + 28-byte Msg31 | `PktSize = 16'h004C`, `MsgCount = 2` |

The Heartbeat has no MsgType. Any retained MsgType bus value is invalid when `MsgCount = 0` and is ignored.


## Next Engineering Targets

The immediate fast-path challenge is message-level Line A/B arbitration: preserve ordering, suppress duplicates, detect gaps and handle conflicts within the two-cycle design allocation. Precomputed eligibility, localized control and routed timing analysis drive the implementation.

NIC replay will exercise market-data ingestion as a separate validation path. The KU040 timestamp system will provide independent nanosecond-scale hardware measurements of the integrated path and its P50, P99 and P99.9 distribution. CME, LSE and NASDAQ laboratory projects will reuse the Golden GTH foundation.

The July observations above belong to their own RX Buffer ON builds; the November captures and dual-bypass diagnostic have their stated build and test scope. Each new result is added with its matching mode and evidence.

