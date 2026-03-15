# Replication Package

**Paper:** FDI and Growth: Panel Evidence with Income-Group Heterogeneity  
**Author:** Lorenzo Dufour
**Course:** Applied Economics, CATÓLICA Lisbon  
**Date:** March 2026

---

## Overview

This package contains all code and instructions needed to reproduce the paper from scratch. The main analysis is a single Quarto document
(`paper.qmd`) that pulls data live from the World Bank API and produces
the PDF paper in one command. No pre-processed datasets are required.

---

## Repository structure

```
replication/
├── data/                      # data for reproducability if API data changes
  ├── API_BX.KLT.DINV.WD.GD.ZS_DS2_en_csv_v2_397.csv            # Foreign direct investment, net inflows (% of GDP)
  ├── API_NY.GDP.PCAP.KD.ZG_DS2_en_csv_v2_94752.csv             # GDP per capita growth (annual %)
  ├── API_NY.GDP.PCAP.PP.KD_DS2_en_csv_v2_4572.csv              # GDP per capita, PPP (constant 2021 international $)
├── README.md                  # this file
├── paper.pdf                  # rendered paper
├── paper.qmd                  # paper source (Quarto + R)
├── references.bib             # bibliography (BibTeX)
```

---

## Requirements

### R packages

Install all required packages with:

```r
install.packages(c("WDI", "tidyverse", "fixest", "kableExtra"))
```

Tested with:

| Package    | Version  |
|------------|----------|
| R          | ≥ 4.3.0  |
| WDI        | ≥ 2.7.8  |
| tidyverse  | ≥ 2.0.0  |
| fixest     | ≥ 0.12.0 |
| kableExtra | ≥ 1.3.4  |

### System dependencies

- **Quarto** ≥ 1.4 — to render the paper: <https://quarto.org>
- **A LaTeX distribution** (e.g. TinyTeX, TeX Live, MiKTeX) — required by Quarto

Install TinyTeX from R if needed:

```r
install.packages("tinytex")
tinytex::install_tinytex()
```

---

## Reproducing the paper

```bash
quarto render final_paper.qmd
```

This will:
1. Pull the three WDI series from the World Bank API (cached after first run)
2. Run the full analysis (winsorisation, panel models, robustness checks)
3. Produce `paper.pdf`

The `load-data` chunk is cached (`cache: true`), so subsequent renders are fast.
Delete `final_paper_cache/` to force a fresh API pull.

---

## Data sources

All data come from the World Bank World Development Indicators (WDI):

| Series code              | Description                                    |
|--------------------------|------------------------------------------------|
| BX.KLT.DINV.WD.GD.ZS     | FDI net inflows (% of GDP)                     |
| NY.GDP.PCAP.KD.ZG        | GDP per capita growth (annual %)               |
| NY.GDP.PCAP.PP.KD        | GDP per capita, PPP (constant 2021 intl. $)    |

Data are pulled via the `WDI` R package (Arel-Bundock, 2019). Regional aggregates
are removed; only individual economies are retained.

---

## Key design choices

- **IHS transformation** for FDI: 1.3% of raw FDI observations are exactly zero
  and 8.7% are negative (disinvestment/profit repatriation). `log` is undefined
  for these values; dropping them induces selection bias. `asinh(x)` is used
  instead — it is defined for all reals and approximates `log(2x)` for large `x`.
- **Winsorisation** at 1st/99th percentiles: FDI before lagging, growth after.
- **Two-way fixed effects** (country + year) via `feols()` in the `fixest` package.
- **Income classification**: static dummy based on each country's time-averaged
  GDP per capita relative to the cross-country sample median.
- **Standard errors** clustered by country throughout.

---

## References

- Arel-Bundock, V. (2019). WDI: World Development Indicators. R package.
- Bellemare, M. F., & Wichman, C. J. (2020). Elasticities and the inverse
  hyperbolic sine transformation. *Oxford Bulletin of Economics and Statistics*, 82(1), 50–61.
- Berge, L. (2018). Efficient estimation of maximum likelihood models with
  multiple fixed-effects. *Econometrics*, 9(3), 1–15. (`fixest` package)
- Findlay, R. (1978). Relative backwardness, direct foreign investment, and
  the transfer of technology. *Quarterly Journal of Economics*, 92(1), 1–16.
