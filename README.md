# StrataDCA — Decline Curve & Probabilistic Forecast Studio

StrataDCA is a single-file, client-side web app for oil well **decline curve analysis (DCA)**. Upload production data, and it automatically segments the history, selects the best-fit window, fits the three Arps decline models (Exponential / Harmonic / Hyperbolic), and runs a Monte Carlo residual-bootstrap to produce P10/P50/P90 EUR and remaining-life estimates.

Everything runs **entirely in the browser** — no backend, no build step, no server-side processing. Your production data never leaves your machine.

## Features

- **Automatic segmentation** of production history, with detection of shut-ins and possible workovers
- **Sliding-window QC scoring** to pick the fitting window that best represents the well's current decline regime
- **Arps model fitting** (Exponential, Harmonic, Hyperbolic) via Nelder–Mead optimization, with automatic best-model selection by RMSE
- **Probabilistic forecasting** (P10/P50/P90 EUR and remaining life) via Monte Carlo residual-bootstrap
- **Terminal decline switch** (modified hyperbolic / BDF-consistent forecasting) to avoid over-optimistic EUR from large-b hyperbolic extrapolation
- **BDF (boundary-dominated flow) diagnostic notes** — automatic commentary on how consistent the fitted b-factor is with physical BDF behavior
- **Single-well and multi-well modes**, with a cross-well comparison view and aggregated portfolio EUR
- **CSV summary export**
- Built with [Plotly.js](https://plotly.com/javascript/) for interactive charts and [SheetJS](https://sheetjs.com/) for reading `.xlsx` / `.csv` files

## Getting started

This is a static, dependency-free HTML file — there's nothing to install or build.

### Option 1 — Open locally
Just download `index.html` and open it in any modern browser.

### Option 2 — Run a local server (recommended for file uploads in some browsers)
```bash
git clone https://github.com/<your-username>/strata-dca-studio.git
cd strata-dca-studio
python3 -m http.server 8000
# then open http://localhost:8000
```

## Input data format

Upload an `.xlsx` or `.csv` file with at least these two columns (header names are matched case-insensitively):

| Column | Description |
|---|---|
| `Date` | Production date |
| `Oil bbl/d` | Oil rate in barrels per day |

In **Multi Well** mode, upload multiple files — one file per well, named however you like (the file name is used as the well name).

A small synthetic dataset is included at (sample-data.example.xlsx) 

## Methodology

1. **Cleaning & QC** — outlier flagging (IQR rule), shut-in detection (rate ≤ economic limit), possible-workover detection (rate jump above a configurable threshold).
2. **Segmentation** — the series is automatically split into regime segments based on rate-change events; short segments are merged.
3. **Tail / current-regime selection** — the most recent segment(s) with a declining trend are selected as the analysis basis.
4. **Sliding-window search** — candidate fitting windows within the tail are scored on monotonicity, noise, shut-in fraction, stability, trend, and recency; the highest-scoring window is used for fitting.
5. **Arps fitting** — Exponential, Harmonic, and Hyperbolic models are all fit via Nelder–Mead; the model with the lowest RMSE is selected.
6. **Forecast** — the selected model is extrapolated to the economic limit, optionally with a **terminal decline switch** (modified hyperbolic) once the instantaneous decline rate falls to a user-defined terminal rate.
7. **Monte Carlo uncertainty** — fit residuals are bootstrap-resampled many times, each realization is refit and forecast independently, and P10/P50/P90 are taken as percentiles of the resulting EUR distribution (SPE convention: P90 = conservative, P10 = optimistic).

## Tech stack

- Vanilla JavaScript (no framework, no build tooling)
- [Plotly.js](https://plotly.com/javascript/) for charts
- [SheetJS (xlsx)](https://sheetjs.com/) for spreadsheet/CSV parsing
- Google Fonts: Fraunces, Inter, IBM Plex Mono

## License

Released under the [MIT License](LICENSE) — see the LICENSE file for details.

## Disclaimer

This tool is provided for engineering/analytical convenience and educational purposes. Reserves and forecast figures produced by this tool are not a substitute for professional reservoir engineering judgment or formal reserves auditing (e.g. SEC/SPE-compliant reporting).
