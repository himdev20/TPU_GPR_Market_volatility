# Data

All variables used in the project are harmonised to monthly frequency (dated to the first of the month) and compiled
into a single panel, `Compiled.xlsx` (sheet `X_var`, April 1999 – December 2025), so that every model loads the same
data from one file. The NIFTY 50 prices are kept separately in `NSE_monthly_final.csv`, and the three IIP base-year
series used to construct the output gap are in `IIP/`.

## Data availability

The Indian macroeconomic series were downloaded from **CEIC**, a paid subscription database whose terms do not allow
redistribution. **These files are therefore not included in the public repository.** They are listed in `.gitignore`,
so a local copy can keep them without their being pushed. This applies to:

- `IIP/IPI_94.xlsx`, `IIP/IPI_5.xlsx`, `IIP/IPI_11.xlsx` (CEIC exports)
- `Compiled.xlsx`, which contains the CEIC series alongside the public ones
- `raw/ceic/` (original CEIC downloads)
- `../outputs/master_dataset.xlsx`, which contains the same series

Users with CEIC access can download the series below and place them in the folders shown to reproduce every result.

| Variable | CEIC series | File / folder |
|---|---|---|
| IIP, 1993-94 = 100 (MOSPI) | Series ID 18989701 | `IIP/IPI_94.xlsx` |
| IIP, 2004-05 = 100 (MOSPI) | Series ID 314398401 | `IIP/IPI_5.xlsx` |
| IIP, 2011-12 = 100 (MOSPI) | Series ID 386587797 | `IIP/IPI_11.xlsx` |
| CPI inflation, % y/y | Consumer Price Index, year-on-year growth | `raw/ceic/` and column `CPI_GR` of `Compiled.xlsx` |
| INR per USD, monthly average | RBI reference rate | `raw/ceic/` and column `FX_USD` |
| Weighted average call money rate | RBI | `raw/ceic/` and column `Int_Rate` |

Each IIP file is a CEIC export: sheet `My Series`, metadata rows at the top, then `Date | series on its own base |
CEIC linked series on the 2011-12 base`. The notebook (Appendix A) reads the second column, splices the three bases and
checks the result against CEIC's linked series.

## Public series

| Variable | Source | File / folder |
|---|---|---|
| NIFTY 50 monthly close | National Stock Exchange of India | `NSE_monthly_final.csv` |
| Trade Policy Uncertainty (Caldara et al., 2020) | https://www.matteoiacoviello.com/tpu.htm | `raw/uncertainty/` |
| Geopolitical Risk (Caldara & Iacoviello, 2022) | https://www.matteoiacoviello.com/gpr.htm | `raw/uncertainty/` |
| US Monetary Policy Uncertainty | https://www.policyuncertainty.com | `raw/uncertainty/` |
| US Economic Policy Uncertainty | https://www.policyuncertainty.com | `raw/uncertainty/` |
| CBOE VIX (VIXCLS) | https://fred.stlouisfed.org/series/VIXCLS | `raw/fred/` |

## Folder layout

```
data/
├── Compiled.xlsx            merged monthly panel (not public: contains CEIC series)
├── NSE_monthly_final.csv    NIFTY 50 monthly prices
├── IIP/                     CEIC IIP exports (not public)
└── raw/
    ├── ceic/                original CEIC downloads (not public)
    ├── uncertainty/         TPU, GPR, MPU, US EPU downloads
    └── fred/                VIX download
```

## Cleaning applied

One observation was corrected: the call money rate for March 2007 is recorded as 54.32%, whereas the RBI
*Handbook of Statistics on the Indian Economy* reports a monthly weighted average of 14.07%. The notebook applies the
correction (switch `FIX_INT_RATE_MAR2007`); `Compiled.xlsx` itself is left as downloaded.
