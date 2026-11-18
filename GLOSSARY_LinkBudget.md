# RF-Knowledge[GLOSSARY.md](https://github.com/user-attachments/files/35035621/GLOSSARY.md)
# Link Budget Glossary — LTE & 5G NR

A term-by-term reference that grows with each project stage. Each entry has: **what it is**, **formula**,
**typical values**, **why it matters**, and **the interview angle**.

Stage coverage: ✅ Stage 1 (core engine) · ⏳ Stage 2 (propagation) · ⏳ Stage 3 (O2I, site count, UI)

---

## Contents

1. [Units: dB, dBm, dBi, dBd](#1-units-db-dbm-dbi-dbd)
2. [Link budget](#2-link-budget)
3. [Downlink (DL) and Uplink (UL)](#3-downlink-dl-and-uplink-ul)
4. [Transmit power](#4-transmit-power)
5. [Antenna gain](#5-antenna-gain)
6. [Cable / feeder loss](#6-cable--feeder-loss)
7. [EIRP](#7-eirp)
8. [Body loss](#8-body-loss)
9. [Resource block (PRB), subcarrier spacing, numerology](#9-resource-block-prb-subcarrier-spacing-numerology)
10. [Channel vs occupied bandwidth](#10-channel-vs-occupied-bandwidth)
11. [Thermal noise and −174 dBm/Hz](#11-thermal-noise-and-174-dbmhz)
12. [Noise figure (NF)](#12-noise-figure-nf)
13. [Noise floor](#13-noise-floor)
14. [SINR, SNR, C/I](#14-sinr-snr-ci)
15. [MCS and BLER](#15-mcs-and-bler)
16. [Attenuated Shannon model](#16-attenuated-shannon-model)
17. [Receiver sensitivity](#17-receiver-sensitivity)
18. [Uplink PRB allocation trade-off](#18-uplink-prb-allocation-trade-off)
19. [Rx diversity / combining gain](#19-rx-diversity--combining-gain)
20. [TMA / MHA](#20-tma--mha)
21. [Massive MIMO array gain and the double-count trap](#21-massive-mimo-array-gain-and-the-double-count-trap)
22. [Path loss](#22-path-loss)
23. [MAPL — Maximum Allowable Path Loss](#23-mapl--maximum-allowable-path-loss)
24. [Limiting link and UL/DL imbalance](#24-limiting-link-and-uldl-imbalance)
25. [Shadow fading (log-normal) and σ](#25-shadow-fading-log-normal-and-σ)
26. [Shadow fade margin: cell-edge probability](#26-shadow-fade-margin-cell-edge-probability)
27. [Cell-area coverage probability (Jakes)](#27-cell-area-coverage-probability-jakes)
28. [Fast fading (Rayleigh / Rician)](#28-fast-fading-rayleigh--rician)
29. [Interference margin (noise rise / IoT)](#29-interference-margin-noise-rise--iot)
30. [Implementation margin](#30-implementation-margin)
31. [Control-channel overhead](#31-control-channel-overhead)
32. [Penetration loss / O2I](#32-penetration-loss--o2i)
33. [Propagation model](#33-propagation-model)
34. [Path-loss exponent n](#34-path-loss-exponent-n)
35. [Cell radius, ISD and site count](#35-cell-radius-isd-and-site-count)
36. [Dimensioning vs planning vs optimisation](#36-dimensioning-vs-planning-vs-optimisation)

---

## 1. Units: dB, dBm, dBi, dBd
**What:** decibels express ratios on a log scale. `dB = 10·log10(P2/P1)`.
- **dBm**: absolute power vs 1 mW. 0 dBm = 1 mW; 30 dBm = 1 W; 46 dBm ≈ 40 W.
- **dBi**: antenna gain vs an isotropic radiator. **dBd**: vs a half-wave dipole. `dBi = dBd + 2.15`.

**Rules:** dBm + dB = dBm · dBm − dBm = dB · never add two dBm values directly (convert to mW first).
+3 dB ≈ ×2, +10 dB = ×10.

**Interview angle:** "Two 43 dBm carriers combined?" → 46 dBm, not 86.

## 2. Link budget
**What:** accounting of every gain and loss between transmitter and receiver, solved for the largest
path loss the link can tolerate (MAPL). It is the first step of **dimensioning**: MAPL → cell radius → site count.

**Why it matters:** it is how operators estimate how many sites a rollout needs before detailed planning.

## 3. Downlink (DL) and Uplink (UL)
**DL:** base station (eNB/gNB) → UE. **UL:** UE → base station. Each has its own budget; the shorter range wins.

## 4. Transmit power
**What:** power at the transmitter antenna port, per carrier.
**Typical:** LTE macro 43–46 dBm (20–40 W); NR C-band 46–50 dBm; UE power class 3 = 23 dBm (200 mW),
class 2 / HPUE = 26 dBm, FWA CPE up to 26–28 dBm.
**Interview angle:** the ~25 dB gap between base station and UE power is the main reason the UL limits coverage.

## 5. Antenna gain
**What:** how much an antenna concentrates power in its main beam vs isotropic. It does not create power; it
redirects it. **Typical:** macro panel 15–18 dBi; massive-MIMO array 22–25 dBi; handset ≈ 0 dBi (often −3 to 0);
FWA CPE 8–15 dBi.

## 6. Cable / feeder loss
**What:** loss in feeders, jumpers, connectors, diplexers between radio and antenna. Applies on both DL (transmit)
and UL (receive). **Typical:** AAU/integrated radio 0–1 dB; RRU at top of mast 0.5–1 dB; radio at the bottom with
long feeder 2–5 dB.

## 7. EIRP
**What:** Effective Isotropic Radiated Power — the power an isotropic antenna would need to match the main beam.
**Formula:** `EIRP = P_tx + G_tx − L_cable` (UE side: `− L_body`).
**Typical:** LTE macro ~60–63 dBm; NR C-band massive MIMO ~70–75 dBm; handset ~20 dBm.
**Interview angle:** regulators limit EIRP, not just power (FCC, IFT in Mexico).

## 8. Body loss
**What:** absorption by the user's head and hand. **Typical:** 2–4 dB handheld voice/data; 0 dB for CPE or IoT
on a wall. Applied once per direction, always on the UE side.

## 9. Resource block (PRB), subcarrier spacing, numerology
**PRB:** 12 subcarriers in frequency (one slot in time). **SCS:** LTE fixed 15 kHz → PRB = 180 kHz.
NR numerology μ: SCS = 15·2^μ kHz (15/30/60/120 kHz). At 30 kHz a PRB = 360 kHz.
**Why it matters:** bandwidth sets noise; PRB count sets capacity and the UL noise trick.

## 10. Channel vs occupied bandwidth
**Channel bandwidth:** the licensed block (e.g. 10 MHz). **Occupied (transmission) bandwidth:** PRBs actually
used: `N_PRB × 12 × SCS`. LTE 10 MHz → 50 PRB → 9 MHz; NR 100 MHz @ 30 kHz → 273 PRB → 98.28 MHz.
Noise is computed on occupied bandwidth (sources: TS 36.101, TS 38.101-1 Table 5.3.2-1).

## 11. Thermal noise and −174 dBm/Hz
**What:** random electron motion noise in any conductor. **Formula:** `N = k·T·B`;
k = 1.380649×10⁻²³ J/K, T = 290 K. `k·T = 4.0×10⁻²¹ W/Hz = −204 dBW/Hz = −174 dBm/Hz`.
`N_thermal[dBm] = −174 + 10·log10(B_Hz)`.
**Values:** 180 kHz → −121.4 dBm; 9 MHz → −104.4 dBm; 98.28 MHz → −94.1 dBm.
**Interview angle:** derive −174 on a whiteboard.

## 12. Noise figure (NF)
**What:** how much noise the receiver chain adds beyond thermal, in dB. Lower is better.
**Typical:** base station 2–3.5 dB (FR1); handset 7–9 dB (FR1), 10–13 dB (FR2).
**Related:** Friis cascade formula — the first stage (LNA, or TMA) dominates the chain NF, which is why a TMA helps.

## 13. Noise floor
**Formula:** `Noise floor = −174 + 10·log10(B) + NF`. The level the wanted signal must beat.

## 14. SINR, SNR, C/I
- **SNR:** signal / noise. **C/I:** carrier / interference. **SINR:** signal / (interference + noise) — the one
  that drives modulation choice in a loaded network.
**Interview angle:** at the cell edge in a loaded network, interference often dominates noise, which is why an
interference margin is needed.

## 15. MCS and BLER
**MCS:** modulation and coding scheme (QPSK…256QAM, code rate). **BLER:** block error rate; link adaptation
targets ~10% initial BLER. Each MCS needs a minimum SINR; vendors publish SINR-vs-MCS curves. Atoll calls these
"bearers" with "selection thresholds".

## 16. Attenuated Shannon model
**What:** first-order link-level abstraction from 3GPP TR 36.942 Annex A.2:
`Thr = α · B · log2(1 + SINR)`, bounded by SINR_min and a max spectral efficiency.
**Parameters:** DL α = 0.6, max 4.4 b/s/Hz; UL α = 0.4, max 2.0 b/s/Hz; SINR_min = −10 dB.
**Use in this project:** derive the required SINR from a target cell-edge throughput.
**Limit:** not a substitute for vendor BLER curves; fine for dimensioning.

## 17. Receiver sensitivity
**What:** weakest signal the receiver decodes at the required quality.
**Formula:** `Sens = −174 + 10·log10(B) + NF + SINR_req`. More negative = better.

## 18. Uplink PRB allocation trade-off
**What:** fewer UL PRBs concentrate the fixed 23 dBm in less bandwidth → less noise → better sensitivity
by `10·log10(N_full / N_alloc)`. **Catch:** for the *same* throughput, fewer PRBs need a higher SINR, which can
cancel the gain. Range improves only if the edge throughput target is lowered. (See Stage 1 notebook, §12.)

## 19. Rx diversity / combining gain
**What:** the base station receives on several antennas and combines them (MRC/IRC), improving effective SINR.
**Typical:** 2RX ≈ 3 dB, 4RX ≈ 5–6 dB, 8RX ≈ 8–9 dB. UL only (handset Rx diversity is usually inside UE sensitivity).

## 20. TMA / MHA
**What:** Tower (Mast) Head Amplifier: an LNA at the antenna that cancels feeder loss on the uplink and lowers
system NF. **Typical gain on the budget:** 1–3 dB, mainly with long feeders. Not needed with AAUs.

## 21. Massive MIMO array gain and the double-count trap
**What:** a 32T32R/64T64R array forms beams; its quoted gain (22–25 dBi) may already include coherent combining
of all elements. Adding a separate 12 dB "Rx combining gain" can overstate UL MAPL by ~10 dB.
**Interview angle:** "How does your vendor define the array gain?" shows real design maturity.

## 22. Path loss
**What:** attenuation of the signal between the two antennas due to distance, frequency and environment.
**Free space:** `FSPL[dB] = 32.44 + 20·log10(f_MHz) + 20·log10(d_km)`. Real environments lose much more (Stage 2).

## 23. MAPL — Maximum Allowable Path Loss
**What:** largest path loss the link survives with all margins respected.
**Formula:** `MAPL = EIRP − Sensitivity + G_rx − L_rx − Margins`.
**Use:** MAPL + a propagation model → cell radius (Stage 2).
**Typical:** LTE macro outdoor 140–165 dB depending on service and margins.

## 24. Limiting link and UL/DL imbalance
**What:** the direction with the smaller MAPL sets the cell radius. **Imbalance** = MAPL_DL − MAPL_UL.
Usually UL-limited for handhelds; DL-limited can happen with high-gain CPE or narrow UL grants.

## 25. Shadow fading (log-normal) and σ
**What:** slow variation of received power around the model's median because of obstacles, at the same distance.
In dB it is Gaussian with standard deviation σ. **Typical:** 6–8 dB suburban/urban, up to 10–12 dB dense urban/indoor.
**Atoll:** set per clutter class as "model standard deviation".

## 26. Shadow fade margin: cell-edge probability
**Formula:** `M = σ · Q⁻¹(1 − P_edge) = σ · Φ⁻¹(P_edge)`.
**Values (σ = 8 dB):** 75% → 5.4 dB; 90% → 10.3 dB; 95% → 13.2 dB.

## 27. Cell-area coverage probability (Jakes)
**What:** fraction of the cell *area* covered, higher than the edge probability for the same margin because
inner locations have spare signal. Jakes' closed form depends on σ and the path-loss exponent n.
**Values (σ = 8 dB, n = 3.76):** 95% area ≈ 8.5 dB margin ≈ 86% at the edge.
**Interview angle:** clarify whether a customer's "95% coverage" means area or edge — it changes the margin by ~5 dB.

## 28. Fast fading (Rayleigh / Rician)
**What:** rapid variation over fractions of a wavelength from multipath. Mostly absorbed by link-level SINR
curves, HARQ and diversity; sometimes a fast-fade margin is added for slow-moving users.

## 29. Interference margin (noise rise / IoT)
**What:** extra margin for neighbour-cell interference raising the effective noise floor as load grows.
`IoT = (I + N) / N`. **Typical:** DL 3–6 dB, UL 1–3 dB (higher in heavily loaded urban).
Tied to the cell load assumed in dimensioning.

## 30. Implementation margin
**What:** gap between ideal models and real hardware/software (channel estimation, phase noise). **Typical:** 1–3 dB.

## 31. Control-channel overhead
**What:** DL power and resources consumed by reference signals and control channels (PDCCH, PBCH, CRS in LTE),
not available to the data channel. **Typical:** ~1 dB in first-order budgets.

## 32. Penetration loss / O2I
**What:** loss for an outdoor site serving an indoor user. 3GPP TR 38.901 §7.4.3 gives low-loss and
high-loss (IRR glass, concrete) models plus indoor depth loss (0.5 dB/m). **Typical:** 10–15 dB low-loss,
20–27 dB high-loss at 3.5 GHz. *(Detailed in Stage 3.)*

## 33. Propagation model ⏳
**What:** formula (empirical, semi-empirical or deterministic) predicting median path loss vs distance, frequency,
heights and environment. **Examples:** Okumura-Hata (150–1500 MHz), COST-231 Hata (1500–2000 MHz),
3GPP TR 38.901 UMa/UMi/RMa (0.5–100 GHz), Atoll SPM (calibrated with drive tests), ray tracing.
*(Detailed in Stage 2.)*

## 34. Path-loss exponent n ⏳
**What:** how fast path loss grows with distance: PL ∝ 10·n·log10(d). Free space n = 2; urban 3–4; indoor 4+.

## 35. Cell radius, ISD and site count ⏳
**What:** radius R where model path loss = MAPL. Hexagonal 3-sector site area ≈ 1.95·R²; omni ≈ 2.6·R².
Sites = area to cover / site area. *(Stage 3.)*

## 36. Dimensioning vs planning vs optimisation
- **Dimensioning:** link budget + capacity → how many sites (spreadsheet level, this project).
- **Planning:** real site locations, terrain, clutter, antennas, predictions (Atoll, iBwave).
- **Optimisation:** tune the live network from KPIs and drive tests (Carolina's current daily work).

**Interview angle:** this project bridges the three — a dimensioning engine, explained the way a planning tool
works, built by an optimisation engineer.
