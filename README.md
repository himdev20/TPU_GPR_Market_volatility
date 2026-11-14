# Trade Policy Uncertainty, Geopolitical Risk and Stock Market Volatility: Evidence from India

Undergraduate project in economics (supervisor: Prof. Sanjiv Kumar). The project asks whether global trade policy
uncertainty (TPU) and geopolitical risk (GPR) affect the volatility of the Indian stock market, using monthly NIFTY 50
data from April 1999 to December 2025.

**Paper:** [`paper/main.pdf`](paper/main.pdf) (LaTeX source in `paper/`)
**Analysis:** [`UGP_analysis.ipynb`](UGP_analysis.ipynb) reproduces every table and figure from the files in `data/`.

---

## Summary of findings

1. **TPU does not raise volatility.** Its coefficient in the GARCH-X variance equation is small and negative
   (δ₁ = −0.0053, p = 0.095), and it is insignificant in both ARDL models, in the SVAR and in every month of the
   time-varying-parameter regression, including the 2025 tariff episode.
2. **GPR has a small negative long-run effect.** In the log-volatility ARDL its long-run multiplier is −0.00125
   (p = 0.026), about −6% per standard deviation of GPR. The negative effect is concentrated in 2020–22.
3. **A long-run level relationship exists**, but it runs through the exchange rate (negative) and US monetary policy
   uncertainty (positive), not through TPU. Bounds-test F = 3.63 (ARDL(2,2)) and 6.75 (ARDL(1,0) in logs), both above
   the 5% upper bound.
4. **Volatility is driven by its own past.** Own shocks explain more than 99% of the forecast-error variance at every
   horizon up to 24 months; the half-life of a shock is about 3.8 months.
5. **Machine learning does not beat persistence.** Out of sample (Dec 2020 – Dec 2025), an AR(1) forecast has the lowest
   RMSE; LASSO and the elastic net are statistically indistinguishable from it, and tree ensembles are significantly worse.
6. **TPU and GPR do not reinforce each other.** Their interaction is never significant.

| Model | Key result |
|---|---|
| GARCH-X(1,1) | α + β = 0.937; LR test vs GARCH(1,1): χ²(4) = 3.52, p = 0.475 |
| Johansen (TPU, GPR, volatility) | VAR(1) in levels; rank 3 of 3 at lags 0–2 → system stationary in levels |
| ARDL(2,2), volatility in levels | R² = 0.932; bounds F = 3.63 (5% bounds 2.06 / 3.24) |
| ARDL(1,0), log volatility | bounds F = 6.75 (5% bounds 2.22 / 3.40); GPR LR −0.00125**, FX −0.0192***, MPU +0.0021*** |
| SVAR (Cholesky TPU → GPR → vol) | 24-month FEVD of volatility: TPU 0.91%, GPR 0.01%, own 99.08% |
| TVP regression (Kalman) | TPU never significant; GPR significantly negative Jul 2020 – Jan 2022 |
| ML, test RMSE | AR(1) 0.0244 · naive 0.0246 · elastic net 0.0257 · LASSO 0.0259 · RF 0.0286–0.0399 · XGBoost 0.0421 |

**What this means.** Headline trade-policy and geopolitical uncertainty are not important drivers of monthly volatility
in Indian equities; the null result for TPU is consistent across five methods and holds even at the index's 2025 peak.
External shocks reach Indian markets mainly through US monetary policy uncertainty and global risk aversion (VIX).
Once a volatile period begins, it runs its course over many months largely regardless of the uncertainty indices,
which is also why flexible ML models find no signal beyond persistence. The estimates are associations rather than
causal effects, and they apply to monthly rolling volatility; the paper's conclusion discusses these limits and the
extensions (daily realised volatility, India-specific uncertainty, event studies) that would test them.

![Rolling volatility](outputs/figures/F01_nse_price_returns_rollvol.png)

---

## Data

Monthly, April 1999 – December 2025 (321 months). Indian series from **CEIC**; the other series from the institutions
that publish them.

| Variable | Description | Source |
|---|---|---|
| NIFTY 50 | Monthly closing level | National Stock Exchange of India |
| IIP | Index of Industrial Production, 1993-94, 2004-05 and 2011-12 bases (MOSPI) | CEIC |
| CPI_GR | CPI inflation, % y/y | CEIC |
| FX_USD | INR per USD, monthly average | CEIC |
| Int_Rate | Weighted average call money rate, % | CEIC (RBI) |
| TPU | Trade Policy Uncertainty index (Caldara et al., 2020) | Caldara et al. data site |
| GPR | Geopolitical Risk index (Caldara & Iacoviello, 2022) | Caldara & Iacoviello data site |
| MPU | US Monetary Policy Uncertainty index | policyuncertainty.com |
| US_EPU | US Economic Policy Uncertainty index | policyuncertainty.com |
| US_VIX | CBOE VIX (FRED series VIXCLS), monthly mean of daily closes | FRED |

**Data preparation.** All series are dated to the first of the month, harmonised to monthly frequency and compiled
into a single panel, `data/Compiled.xlsx` (sheet `X_var`), so that every model loads the same data from one file. One data error was corrected: the March 2007 call money rate (54.32%) is replaced
with the monthly average published in the RBI *Handbook of Statistics* (14.07%).

**Output gap.** The IIP is published on three base years whose levels are not comparable. The 2004-05 series is linked
to the 2011-12 base by its ratio in April 2012, and the 1993-94 series to the linked 2004-05 series by its ratio in
April 2005. The spliced index is logged and HP-filtered (λ = 14,400); the gap is 100 × the cycle. The rebuilt gap
reproduces the series in `Compiled.xlsx` to within 10⁻⁹ ([`data_preparation.ipynb`](data_preparation.ipynb) and Appendix A of the analysis notebook).

![IIP splicing and output gap](outputs/figures/FA1_iip_splicing_output_gap.png)

**Data availability.** The CEIC series (IIP, CPI, exchange rate, call money rate) come from a paid subscription
database whose terms do not allow redistribution, so they are **not included in this public repository**. This also
applies to `data/Compiled.xlsx` and `outputs/master_dataset.xlsx`, which contain them. These files are listed in
`.gitignore`. [`data/README.md`](data/README.md) gives the CEIC series (including the IIP series IDs) and where to
place the files; with them in place, the notebook reproduces every result. All other series are public.

**Volatility measure.** 12-month rolling standard deviation of monthly log returns, annualised (× √12). It is the
dependent variable of every model after the GARCH stage; the estimation sample is April 2000 – December 2025 (N = 309).

---

## Methods

| Stage | Method | Purpose |
|---|---|---|
| 1 | GARCH-X(1,1) on monthly returns (MLE) | Do TPU and GPR enter the conditional mean or variance? |
| 2 | ADF, PP, KPSS, Zivot–Andrews | Orders of integration (mixture of I(0) and I(1), nothing I(2)) |
| 2 | Johansen trace test (lag = VAR lag − 1) | Specification of the SVAR / TVP system |
| 2 | ARDL bounds tests (Pesaran, Shin & Smith 2001) | Long-run relationship with mixed I(0)/I(1) regressors |
| 2 | Structural VAR, Cholesky identification | Impulse responses and variance decomposition |
| 2 | Time-varying-parameter regression (Kalman filter) | Changes in the effects over time |
| 3 | LASSO, ridge, elastic net, random forest, XGBoost + SHAP | Out-of-sample nowcasting vs naive and AR(1) benchmarks, Diebold–Mariano tests |

---

## Repository structure

```
├── UGP_analysis.ipynb          final, cleaned analysis notebook (executed)
├── data_preparation.ipynb      data sources, cleaning, IIP splicing and HP-filter output gap (runs on its own, e.g. in Colab)
├── data/
│   ├── Compiled.xlsx           merged monthly panel, sheet X_var (not public: contains CEIC data)
│   ├── NSE_monthly_final.csv   NIFTY 50 monthly prices
│   ├── IIP/                    CEIC IIP exports, 1993-94 / 2004-05 / 2011-12 bases (not public)
│   ├── raw/                    original downloads by source (CEIC files not public)
│   └── README.md               data sources, CEIC series IDs, where to place each file
├── outputs/
│   ├── figures/                all figures (PNG, 200 dpi)
│   ├── tables/                 all tables (CSV) and key_results.json
│   ├── results_tables.xlsx     all tables, one sheet each
│   └── master_dataset.xlsx     panel with returns, rolling volatility and GARCH-X volatility
├── paper/                      two-column paper: LaTeX source, bibliography, figures and compiled PDF
├── src/                        notebook source (part1–4.py) and build_notebook.py
└── archive/original_notebooks/ the original working notebooks (raw work)
```

**Raw work.** The notebooks in [`archive/original_notebooks/`](archive/original_notebooks/) (`UGP.ipynb`,
`UGP_1.ipynb`, `UGP_2.ipynb`, `UGP_3.ipynb`) contain the original exploratory work: data download and merging,
early volatility measures, first versions of the models and the machine-learning experiments. They are kept for
reference. `UGP_analysis.ipynb` is the final cleaned version, and all results in the paper come from it.

---

## Reproducing the results

Place the CEIC files as described in `data/README.md`, then:

```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace UGP_analysis.ipynb   # about 5–8 minutes on 2 CPUs
```

Figures and tables are written to `outputs/`. To rebuild the paper, copy `outputs/figures/*.png` to `paper/figures/`
and run `latexmk -pdf main.tex` in `paper/`.

The notebook can also be regenerated from its source with `python src/build_notebook.py` (this overwrites
`UGP_analysis.ipynb` with an unexecuted copy).
