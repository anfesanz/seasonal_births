# Great Britain Monthly Births Forecasting

This project analyzes monthly births and total fertility rate data for Great Britain. It includes an R workflow for seasonal adjustment and classical forecasting, plus a Python workflow combining statistical, neural, and hybrid models.

## Publication status

Only the study abstract is currently public in this repository:
[Seasonality_Births_abst.pdf](https://github.com/anfesanz/seasonal_births/blob/main/Seasonality_Births_abst.pdf).
The complete data, code, results, and supporting materials will be made publicly
available upon request or acceptance of the associated manuscript, whichever
comes first.

## Layout

```text
.
├── config.yaml
├── data/                 # Input CSV files
├── docs/                 # Notes and supporting documentation
├── output/               # Generated tables, plots, and forecasts
├── scripts/
│   ├── run_analysis.py   # Python forecasting workflow
│   ├── seasonality_analysis.R
│   └── install.sh
├── src/                  # Reusable Python modules
└── requirements.txt
```

## Setup

From this directory:

```bash
make install
```

The Python workflow requires Python 3.10 or newer. The R workflow is optional
and additionally requires the `forecast`, `seasonal`, `ggplot2`, `dplyr`,
`tidyr`, and `readr` packages plus an X-13ARIMA-SEATS installation.

The tested neural dependencies are pinned to NeuralForecast 3.2.2 and the
official `xlstm` 2.0.6 backend. Run `make install` again when upgrading an older
checkout. The upgrade from NeuralForecast 2 changes the LSTM implementation,
so regenerate all comparisons together instead of mixing old and new metrics.

## Run

```bash
make run
```

`make install` creates a local `.venv` and installs the Python dependencies there.
`make run` (also available as `make run-python`) reads from `data/` and writes
Python results to `output/`. Settings are in `config.yaml`.

The legacy R analysis is still available explicitly with `make run-r`. It now
writes to `output/r/`, keeping its results separate from the Python files. The
R-specific analyses have not yet been ported to Python. Existing files are not
moved automatically; rerun the relevant command to refresh them.

Run `make help` to see all available project commands.

Run the Python tests with:

```bash
make test
```

The holdout period is configured in `config.yaml`; forecasts and metrics must
not be compared across runs without recording the configuration and package
versions used.

## Accuracy Table

`make run` writes `forecast_accuracy_table.csv` and `forecast_accuracy_table.png`
using the Python results. The first column is `Category`, not `Rank`, and the
rows follow this fixed order: **Non-AI**, **AI**, **Hybrid**. Statistical models
and naive baselines are Non-AI; raw LSTM and xLSTM are AI; statistical/neural
combinations and Fourier-assisted neural models are Hybrid. Within each group,
models follow the explicit order in `src/reporting.py`, not their error scores.

The presentation table includes the holdout period and two-decimal MAE/RMSE.
The full-precision, RMSE-ranked `forecast_accuracy_summary.csv` is unchanged.
To refresh only the presentation table from a completed run, without retraining:

```bash
.venv/bin/python scripts/update_accuracy_table.py --run-dir output
```

Use `--run-dir output/ets-hybrids-check --output-dir output` to publish the
latest 14-model verification run's table in the main output folder. This changes
only the table CSV/PNG, not the main folder's forecasts, summary, or manifest.
The rendered table records its source run. Legacy R results remain separate.

## Run Diagnostics

The same enabled models are evaluated on the holdout and refitted on the full
series for future forecasts, including ETS, standalone STL and Fourier models,
and the three xLSTM variants:
`xLSTM (raw)`, `SARIMA residual + xLSTM`, and `Fourier + xLSTM`.

The standalone `STL` model decomposes the series with STL and forecasts the
decomposition using ARIMA. The standalone `Fourier` model fits a linear trend
with configurable Fourier seasonal terms. These are separate from the
`STL + LSTM + Fourier` hybrid.

`ETS + LSTM` fits the same additive Holt-Winters model as the ETS baseline,
then trains LSTM on its in-sample residuals (`observed - fitted`). Its forecast
is the ETS forecast plus the LSTM residual forecast. `ETS + LSTM + Fourier`
also supplies calendar sine/cosine terms to that residual LSTM as known-future
inputs; it does not add another seasonal forecast on top of ETS.

Both hybrids use the `neural` settings, `seasonal.stl_period` for the ETS seasonal
period, and (for the Fourier variant) `seasonal.fourier_order`. They remain
enabled when only xLSTM is disabled. All fitting uses the training portion for
holdout evaluation and the full series for future forecasts, without selecting
weights on the holdout. Residual correction is experimental and may not improve
on ETS alone; compare their holdout metrics from the same run.

xLSTM uses NeuralForecast's
[official xLSTM integration](https://nixtlaverse.nixtla.io/neuralforecast/models.xlstm.html)
with the mLSTM backbone (matrix memory). The tested default is CPU, including
on Apple Silicon; no NVIDIA GPU is needed for this configuration. Its settings
are in the `xlstm` section of `config.yaml`. When `xlstm.enabled` is true, a
missing model or backend stops the run with an installation message. Set it to
false to explicitly exclude all three variants; exclusions are recorded in
`model_status.csv`. No LSTM substitute is labelled xLSTM.

The Fourier variant supplies its sine/cosine columns through `futr_exog_list`,
with consistent calendar phase across training and forecast dates. The SARIMA
residual variant remains an experimental residual-based method, not X-13
seasonal adjustment; its label reflects that distinction.

SARIMA retries a failed optimization using `sarima.retry_maxiter` from the last
parameter estimates. If it still fails to converge, its forecast is excluded
and recorded as failed. This also applies to SARIMA preprocessing inside the
residual xLSTM hybrid. Other unexpected model errors stop the run and are
recorded before being raised.

`run_manifest.json` records completion status, configuration, package versions,
and the input file's SHA-256 hash. It is marked `running` before the analysis
starts and updated on success or failure. Check this file when using existing outputs,
since an interrupted run can leave results from an earlier run in the folder.

Set `neural.enabled: false` for classical models only. Neural progress bars and
automatic Lightning experiment logs are disabled by default; set
`neural.enable_progress_bar: true` to show training progress. Dependency warnings
remain visible. Reaching `max_steps` is the configured training stop condition.

To write a separate run without replacing the default outputs:

```bash
.venv/bin/python scripts/run_analysis.py --config config.yaml --output-dir output/check
```
