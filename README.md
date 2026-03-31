# Replication Package

**Paper:** FDI and Growth: Panel Evidence with Income-Group Heterogeneity  
**Author:** Lorenzo Dufour  
**Course:** Applied Economics, CATÓLICA Lisbon  
**Date:** March 2026

---

## Overview

Everything needed to reproduce the paper. The whole analysis lives in a single Quarto document (`paper.qmd`) — it fetches the data from the World Bank API and renders out the PDF in one go. You don't need any pre-downloaded datasets, though raw CSVs are included in `data/` in case the API results ever shift.

---

## Structure

```
replication/
├── data/                      # raw CSVs, kept as a fallback
│   ├── API_BX.KLT.DINV.WD.GD.ZS_DS2_en_csv_v2_397.csv      # FDI net inflows (% of GDP)
│   ├── API_BM.KLT.DINV.WD.GD.ZS_DS2_en_csv_v2_316.csv      # FDI net outflows (% of GDP)
│   ├── API_NY.GDP.PCAP.KD.ZG_DS2_en_csv_v2_94752.csv       # GDP per capita growth (annual %)
│   ├── API_NY.GDP.PCAP.PP.KD_DS2_en_csv_v2_4572.csv        # GDP per capita, PPP (const. 2021 intl. $)
│   └── edit_bit_data.csv                                   # Database of BITs
├── README.md                  # this file
├── paper.pdf                  # rendered output
├── paper.qmd                  # source: Quarto + R
└── references.bib             # BibTeX bibliography
```

---

## Requirements

### R packages

```r
install.packages(c("WDI", "tidyverse", "fixest", "kableExtra"))
```

| Package    | Version tested |
|------------|----------------|
| R          | ≥ 4.3.0        |
| WDI        | ≥ 2.7.8        |
| tidyverse  | ≥ 2.0.0        |
| fixest     | ≥ 0.12.0       |
| kableExtra | ≥ 1.3.4        |

### System

- **Quarto** ≥ 1.4: <https://quarto.org>
- **LaTeX** (TinyTeX works fine):

```r
install.packages("tinytex")
tinytex::install_tinytex()
```

---

## How to reproduce

```bash
quarto render paper.qmd
```

That's it. Pulls the WDI series, run the full analysis (winsorisation, panel regressions, robustness checks), and write `paper.pdf`.

The `load-data` chunk is cached, so re-renders are fast. If you want to force a fresh API pull, just delete `paper_cache/`.

---

## Data

All series are from the World Bank WDI, pulled via the `WDI` R package (Arel-Bundock, 2019):

| Series code           | Description                                 |
|-----------------------|---------------------------------------------|
| BX.KLT.DINV.WD.GD.ZS  | FDI net inflows (% of GDP)                  |
| BM.KLT.DINV.WD.GD.ZS  | FDI net outflows (% of GDP)                 |
| NY.GDP.PCAP.KD.ZG     | GDP per capita growth (annual %)            |
| NY.GDP.PCAP.PP.KD     | GDP per capita, PPP (constant 2021 intl. $) |

Regional and aggregate entries are dropped; only individual economies are kept.

---

## A few notes on design choices

- **IHS instead of log for FDI**: about 1.3% of raw FDI observations are exactly zero and 8.7% are negative (disinvestment, profit repatriation). Taking logs would drop all of those. `asinh(x)` handles the full real line and behaves like `log(2x)` for large values (Bellemare & Wichman, 2020).
- **Winsorisation**: FDI at 1st/99th percentiles before lagging; growth after. Extreme outliers bias the estimates.
- **Two-way FE**: country and year fixed effects via `feols()` from `fixest`. Standard errors clustered by country throughout.
- **Income classification**: a static high/low dummy based on whether a country's time-averaged GDP per capita (PPP) is above or below the cross-country median. Simple but consistent with what the literature does.

---

## References

- Arel-Bundock, V. (2019). WDI: World Development Indicators. R package.
- Bellemare, M. F., & Wichman, C. J. (2020). Elasticities and the inverse hyperbolic sine transformation. *Oxford Bulletin of Economics and Statistics*, 82(1), 50–61.
- Berge, L. (2018). Efficient estimation of maximum likelihood models with multiple fixed-effects. *Econometrics*, 9(3), 1–15.
- Findlay, R. (1978). Relative backwardness, direct foreign investment, and the transfer of technology. *Quarterly Journal of Economics*, 92(1), 1–16.
