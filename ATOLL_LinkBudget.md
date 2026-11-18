[ATOLL_LINK_BUDGET.md](https://github.com/user-attachments/files/35035644/ATOLL_LINK_BUDGET.md)
# How Atoll Does the Link Budget — and Which Parameters You Set

> **Scope and honesty note.** This guide explains Atoll's calculation logic and maps every term of our
> link-budget engine to where it lives in Atoll. Field names follow Atoll 3.x (LTE/NR modules) as I know them;
> labels can differ slightly in Atoll 4.x. The official, authoritative sources are Forsk's **Atoll User Manual**
> and **Atoll Technical Reference Guide**, available to licensed users through Forsk's support portal.
> Treat this as a study map, then confirm details in the official manuals or a live Atoll session.

---

## 1. The key idea: Atoll does not compute one MAPL. It computes a link budget for every pixel.

Our notebook answers: *"what path loss can the link survive?"* (one number, MAPL), then a propagation model
turns it into one radius.

Atoll turns the problem around. For **every transmitter** and **every pixel (bin)** of the map it computes:

```
Received level (pixel) = Tx power
                       + Tx antenna gain  (from the 3D pattern at that azimuth/elevation, incl. tilts)
                       − Tx losses        (feeder, connectors, misc.)
                       − Path loss        (propagation model + terrain + clutter, stored in a path loss matrix)
                       − Shadowing margin (from clutter σ and the target cell-edge coverage probability)
                       − Indoor loss      (only if "indoor coverage" is ticked; from clutter class)
                       + Rx (terminal) gain − Rx losses (terminal + service body loss)
```

Then it compares that level (and the C/(I+N) built from all cells' levels) with **thresholds**:

* **Coverage by signal level / RSRP:** pixel is covered if level ≥ threshold. Your link-budget **sensitivity +
  margins** becomes this threshold.
* **Coverage by C/(I+N):** interference from real neighbours, weighted by their **traffic load**, replaces our
  fixed interference margin.
* **Bearer / throughput coverage:** the C/(I+N) at each pixel selects a **bearer** (MCS) using the reception
  equipment thresholds; throughput = bearer efficiency × resources. This replaces our attenuated-Shannon step.

**Interview one-liner:** *"A spreadsheet link budget gives you one MAPL and one radius under average conditions.
Atoll runs that same budget per pixel with the real terrain, clutter, antenna patterns, tilts and neighbour
interference, so the coverage area comes out irregular and realistic."*

---

## 2. Term-by-term mapping: our engine → Atoll

| Link budget term (our notebook) | Where you set it in Atoll | Notes / typical values |
|---|---|---|
| Tx power (site) | **Cells** table / cell properties: max power; LTE **RS EPRE**, NR SSB / reference signal power offsets | Atoll derives per-RE power (RSRP) from total power and bandwidth |
| Antenna gain | **Antennas** library (pattern files with gain, H/V patterns) + **Transmitter**: antenna model, height, azimuth, mechanical downtilt, electrical tilt | Gain is read from the 3D pattern in the pixel direction, not a single peak value |
| Massive MIMO / beamforming | NR **beam / smart-antenna** models on the transmitter (SSB and traffic beams) | Beam gain applied per pixel; avoids the double-count trap if configured once |
| Cable / feeder loss | **Transmitter → Transmission/Reception**: feeder type and length (Tx, Rx), connector and miscellaneous losses | Atoll sums them into total Tx / Rx losses |
| TMA / MHA, BS noise figure | **Equipment** (TMA, feeder, BTS) assigned on the transmitter | Atoll computes the system noise figure from the chain (Friis) |
| UE power, gain, NF, losses | **Terminals** table: max/min power, gain, losses, noise figure, number of antennas (MIMO), reception equipment | One terminal per device type (smartphone, CPE, IoT) |
| Body loss | **Services**: body loss | Different per service (voice vs data) |
| Required SINR | **Reception equipment**: bearer selection thresholds (C/(I+N) per bearer), quality (BLER) graphs, per mobility type | Vendor link-level curves; replaces attenuated Shannon |
| Occupied bandwidth, numerology | **Frequency bands** / global parameters: channel width, SCS (NR numerology), frame configuration, TDD pattern | Determines PRBs and noise bandwidth |
| Shadow fade margin | **Clutter classes**: model standard deviation (σ) and C/I standard deviation; **prediction properties**: cell edge coverage probability and "shadowing taken into account" | Atoll computes the margin per pixel from σ of that clutter class |
| Interference margin | **Cells**: DL traffic load (%), UL noise rise; or computed by **Monte Carlo simulations** from traffic maps | Explicit interference instead of a fixed dB margin |
| Penetration / O2I | **Clutter classes**: indoor losses (frequency dependent in newer versions); tick **indoor coverage** in the prediction | Our Stage 3 O2I model mirrors this |
| Propagation model | **Propagation models** folder: SPM (K1…K7, Kclutter, diffraction method, effective antenna height), Okumura-Hata, COST-Hata, 3GPP models, CrossWave / ray tracing | Assigned per transmitter; main and extended matrix with radius and resolution |
| Geographic data | **Geo** tab: DTM (terrain heights), clutter classes, clutter heights, vector layers (roads, buildings), population / traffic maps | Resolution drives accuracy and calculation time |

---

## 3. Atoll's propagation layer: SPM in one page

The **Standard Propagation Model (SPM)** is Atoll's default macro model: an empirical, Hata-like formula with
tunable coefficients, valid roughly 150–3500 MHz:

```
PL = K1 + K2·log(d) + K3·log(H_Txeff) + K4·Diffraction + K5·log(d)·log(H_Txeff)
     + K6·H_Rxeff + K7·log(H_Rxeff) + K_clutter·f(clutter) + K_hill (hilly-terrain correction)
```

* **K1** (constant offset, frequency dependent) and **K2** (distance slope, i.e. path-loss exponent × 10) matter most.
* **K_clutter** weights the per-clutter-class losses.
* **Calibration:** import CW drive-test measurements, compare predicted vs measured along the route, tune K1, K2,
  clutter losses. A common target is mean error ≈ 0 dB and standard deviation ≤ 8 dB. That standard deviation
  then feeds the shadowing margin.

**Link to your experience:** you already live with drive tests and KPIs. Calibration is exactly "make the
prediction agree with what the network measures."

---

## 4. A typical Atoll workflow for a private-network RF design (what Nokia ECE will expect)

1. **Collect inputs** from Technical Sales: coverage polygon, site candidates, antenna database, morphology /
   clutter, hotspots, service targets (e.g. RSRP ≥ −105 dBm at 95%, UL 5 Mbps per camera), regulatory limits.
2. **Project setup:** coordinate system, DTM, clutter, clutter heights, vectors (buildings for ports/mines).
3. **Global parameters:** frequency band (e.g. n78, n40, B48/CBRS), bandwidth, numerology, TDD pattern (UL-heavy
   for video/AGVs is common in industry).
4. **Templates:** site / transmitter / cell templates with power, antennas, heights, tilts, equipment.
5. **Propagation model:** assign SPM (or 3GPP UMi/InH for small cells), set matrix radius and resolution;
   calibrate if measurements exist.
6. **Terminals, services, mobility, reception equipment** (bearers / thresholds).
7. **Predictions:** best server, RSRP / SS-RSRP coverage, C/(I+N), overlapping zones (pilot pollution),
   DL/UL throughput.
8. **Traffic and capacity:** traffic maps → Monte Carlo simulations → cell loads → re-run C/(I+N).
9. **Optimise:** azimuths, tilts, power, site positions (manually or with ACP — Automatic Cell Planning).
10. **Report:** coverage plots, statistics per polygon, site list, BOM, design assumptions → customer document.

**Where our notebook fits:** steps 1 and 3–6 start from a link budget. Our MAPL gives the RSRP / signal-level
threshold and a first estimate of site count *before* opening Atoll.

---

## 5. Converting our link budget into an Atoll threshold

Atoll's coverage-by-signal-level uses received level per resource element (RSRP) for LTE/NR. To move from our
whole-carrier budget to an RSRP threshold:

```
RSRP_threshold [dBm] ≈ Sensitivity_DL(per RE) + DL margins that Atoll does not compute itself
Sensitivity per RE  = −174 + 10·log10(SCS_Hz) + NF_UE + SINR_req
```

Example (LTE, SCS 15 kHz, NF 7 dB, SINR −5 dB):
`−174 + 41.8 + 7 − 5 = −130.2 dBm` per RE. If shadowing and indoor loss are handled by Atoll (clutter σ and
indoor loss ticked), leave them out of the threshold to avoid counting them twice. *(Stage 3 of the notebook
adds an RSRP-threshold helper that does this conversion.)*

---

## 6. Study resources

**Official (authoritative)**
- Forsk — *Atoll User Manual* (LTE and 5G NR chapters) and *Atoll Technical Reference Guide* (the formulas):
  available through the Forsk customer support portal / with an Atoll licence. Ask Nokia, a previous employer
  or a colleague with a licence for access.
- Forsk Atoll training courses (LTE / NR radio planning, SPM calibration).
- Forsk's website: <https://www.forsk.com>

**Books**
- H. Holma, A. Toskala — *LTE for UMTS: Evolution to LTE-Advanced* (link budget chapter; source of the LTE
  validation case in our notebook).
- H. Holma, A. Toskala, T. Nakamura (eds.) — *5G Technology: 3GPP New Radio* (NR coverage and link budget).
- S. Saunders, A. Aragón-Zavala — *Antennas and Propagation for Wireless Communication Systems* (shadowing,
  Hata models, coverage probability).
- M. Tolstrup — *Indoor Radio Planning: A Practical Guide* (indoor link budgets; the iBwave side of the job).

**Standards used in this project**
- 3GPP TR 36.942 (attenuated Shannon, Annex A), TR 38.901 (NR channel models, O2I), TS 36.101 / TS 38.101-1
  (bandwidths and PRB counts).

**Videos**
- I could not verify a specific official Atoll video course to recommend. Searching YouTube for
  "Atoll LTE coverage prediction" or "Atoll SPM calibration" gives community tutorials; check their Atoll
  version against what Nokia uses.

---

## 7. Five Atoll questions to rehearse

1. How do you calibrate SPM, and what statistics tell you it's good? *(Mean error ≈ 0, σ ≤ ~8 dB, per clutter.)*
2. What is the difference between a path loss matrix and a coverage prediction? *(Matrix = pure path loss per
   transmitter; prediction = matrix + powers, antennas, thresholds, interference.)*
3. How does Atoll account for shadowing? *(Clutter σ + cell-edge coverage probability → per-pixel margin.)*
4. How do you model interference realistically? *(Cell loads from Monte Carlo traffic simulations.)*
5. How would you design a port with stacked containers? *(High sites, clutter heights/vectors for containers,
   UL-heavy TDD for cameras/cranes, redundancy on critical areas, verify with drive/walk test.)*
