# Great Britain Monthly Births Forecasting

This project analyzes monthly births and total fertility rate data for Great Britain. It includes an R workflow for seasonal adjustment and classical forecasting, plus a Python workflow combining statistical, neural, and hybrid models.

## Publication status

Only the study abstract is currently public: [Seasonality_Births_abst.pdf](Seasonality_Births_abst.pdf).
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

## Availability

The current public release is intentionally limited to the abstract. The full
data, source code, analysis outputs, and supporting materials are retained by
the authors and will be made available upon request or acceptance of the
associated manuscript.
