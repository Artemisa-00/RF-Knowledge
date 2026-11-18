[README.md](https://github.com/user-attachments/files/35035667/README.md)
# ran-link-budget — LTE & 5G NR Link Budget Engine

A transparent, fully documented radio link budget tool for **LTE** and **5G NR (FR1)**, built step by step
in Jupyter notebooks and validated against published references.

**Author:** Carolina — Senior RF/RAN Engineer (LTE/5G NR optimisation, O-RAN, C-Band rollout, multi-vendor
Ericsson / Nokia / Huawei).

## Why this project

Planning tools such as Atoll and iBwave are powerful but opaque. This project rebuilds the link budget from
first principles so every dB can be explained and defended, then connects each term to how Atoll computes
coverage. It is both a study tool and a working dimensioning calculator.

## What makes it different from typical online calculators

- One engine for **LTE and NR**.
- **Required SINR derived from a target cell-edge throughput** (3GPP TR 36.942 attenuated Shannon), not typed by hand.
- **Shadow fade margin computed from a coverage probability**, for cell edge or cell area (Jakes).
- Explicit **TMA gain, control-channel overhead, cable loss at each end**.
- **Double-count warning** for massive-MIMO array gain + Rx combining gain.
- Operator-style **lettered rows** with formulas, so the output can be presented.
- **Automatic validation** against published examples.

## Validation

| Reference | Quantity | Published | This engine |
|---|---|---|---|
| cafetele.com 5G calculator (n78, 100 MHz) | MAPL DL | 145.6 dB | 145.6 dB |
| cafetele.com 5G calculator (n78, 100 MHz) | MAPL UL | 148.6 dB | 148.6 dB* |
| Holma & Toskala, *LTE for UMTS* (UL 64 kbps, 2 PRB) | MAPL UL | 163.4 dB | 163.4 dB |
| Holma & Toskala, *LTE for UMTS* (DL 1 Mbps, 10 MHz) | MAPL DL | 165.5 dB | 165.4 dB** |

\* The reference ignores receive-side feeder loss on UL; the engine reports both values.
\** The book rounds thermal noise to −104.5 dBm.

## Roadmap

| Stage | Notebook | Content | Status |
|---|---|---|---|
| 1 | `01_core_link_budget.ipynb` | dB toolkit, PRBs, thermal noise, NF, SINR from throughput, sensitivity, EIRP, shadow margins (edge + Jakes area), MAPL DL/UL, validation, sensitivity analysis | ✅ |
| 2 | `02_propagation_models.ipynb` | Free space, Okumura-Hata, COST-231 Hata, 3GPP TR 38.901 UMa/UMi/RMa, MAPL → cell radius (numerical inversion), plots | ⏳ |
| 3 | `03_o2i_sites_ui.ipynb` | O2I penetration (TR 38.901), site count and ISD, RSRP threshold for Atoll, interactive sliders | ⏳ |
| 4 | package + tests | `linkbudget/` Python package, pytest suite, LTE vs NR band comparison report | ⏳ |

## Documentation

- [`docs/GLOSSARY.md`](docs/GLOSSARY.md) — every term explained: formula, typical values, interview angle.
- [`docs/ATOLL_LINK_BUDGET.md`](docs/ATOLL_LINK_BUDGET.md) — how Atoll computes coverage per pixel and where each
  link-budget parameter is set.

## Run it

```bash
pip install -r requirements.txt
jupyter lab notebooks/01_core_link_budget.ipynb
```

## References

- 3GPP TR 36.942 (Annex A, attenuated Shannon), TR 38.901 (channel models, O2I), TS 36.101, TS 38.101-1.
- H. Holma, A. Toskala, *LTE for UMTS*.
- W. C. Jakes, *Microwave Mobile Communications* (cell-area coverage).
- cafetele.com, 5G Link Budget Calculator — <https://www.cafetele.com/5g-tools/5g-link-budget/>
