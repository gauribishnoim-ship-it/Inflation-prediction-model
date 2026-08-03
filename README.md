# India CPI Inflation Forecast

A machine learning pipeline that studies thirteen years of India's CPI
inflation, learns the underlying pattern, trend, momentum, and seasonal
rhythm, and forecasts the months ahead using Random Forest and
XGBoost, validated honestly with time ordered cross validation rather
than an ordinary random split.

## What you get

* A clean, tested Python package, not a notebook full of loose cells
* Two models compared on a fair, chronological test, plus an ensemble
* A confidence band built from the four hundred trees inside the
  forest, not an assumed or invented range
* A chart of the full history next to the forecast
* A full explanation of the reasoning in `docs/METHODOLOGY.md`, and of
  the limits of what a pattern based model can honestly claim

## A note on the data before you start

This project has no live connection to MOSPI or the Reserve Bank of
India, so the built in historical series is a reconstruction from
widely documented turning points in India's inflation history, not the
official monthly print. It is directionally accurate but not exact.
Read the data section of `docs/METHODOLOGY.md` before drawing any real
conclusion from the default run, and see below for how to plug in the
official series in one step.

## Installation

```bash
git clone https://github.com/your-username/india-inflation-forecast.git
cd india-inflation-forecast
pip install -r requirements.txt
```

Requires Python 3.9 or newer.

## Usage

Run with the built in illustrative data:

```bash
python run.py
```

Choose a different forecast horizon:

```bash
python run.py --horizon 6
```

Use your own data, for example the official MOSPI or RBI series, saved
as a two column CSV with headers `date` and `cpi_inflation_yoy`:

```bash
python run.py --data path/to/official_cpi.csv --horizon 12
```

Each run prints the cross validated accuracy of both models, the
forecast table, and the leading feature importances, then saves
`outputs/forecast.csv` and `outputs/forecast_chart.png`.

## Project layout

```
india-inflation-forecast/
├── run.py                        entry point, wires the pipeline together
├── src/
│   └── inflation_forecast/
│       ├── data.py               historical series and CSV loader
│       ├── features.py           lag, rolling, momentum, seasonal features
│       ├── model.py              Random Forest and XGBoost, cross validation
│       ├── forecast.py           recursive multi month forecast, confidence band
│       └── plotting.py           the history and forecast chart
├── tests/
│   └── test_pipeline.py          sanity tests, run with pytest
├── docs/
│   └── METHODOLOGY.md            the full reasoning and known limits
├── outputs/                      forecast.csv and forecast_chart.png land here
├── requirements.txt
└── LICENSE
```

## Running the tests

```bash
pytest
```

The tests check that the pipeline runs end to end and that the feature
engineering has no lookahead leak, not that the forecast is accurate.
Accuracy is reported honestly by the cross validation step at run time
instead, since a fixed test cannot tell you how good a forecast of the
future will be.

## What this model is not

It is a pattern based forecaster. It has no idea about a monsoon that
has not happened yet, a policy change under discussion, or a shock to
global commodity prices. It should sit alongside economic judgment and
structural models, not replace them. `docs/METHODOLOGY.md` covers this
in more depth, including exactly why the far end of any forecast
deserves less trust than the near end.

## License

MIT, see `LICENSE`.
