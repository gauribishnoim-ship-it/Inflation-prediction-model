# Methodology and notes

## What this model does

It studies thirteen years of monthly Consumer Price Index inflation for
India and produces a multi month forecast, along with a confidence band
so you know how much to trust each part of that forecast.

Two learning algorithms are used, Random Forest and XGBoost, and their
outputs are combined into a single ensemble figure. Both belong to a
family called gradient boosted or bagged decision trees, which are
good at finding nonlinear patterns in noisy economic data without
requiring you to specify the exact shape of the relationship in
advance, the way a classical regression would.

## Why the built in data looks the way it does

This project has no live connection to MOSPI or the Reserve Bank of
India. So the default historical series in `src/inflation_forecast/data.py`
was rebuilt from turning points in India's inflation history that are
widely documented: the double digit inflation of 2013, the low point
near one and a half percent in mid 2017, the onion price spike at the
close of 2019, the pandemic supply disruption of 2020, the oil and war
driven spike of 2022, the tomato price shock of mid 2023, and the
gradual disinflation toward the Reserve Bank's four percent target
through 2024 and 2025. Monthly values were produced by connecting these
points and adding a small amount of noise, since real monthly CPI
prints never sit exactly on a smooth line.

This means the shape of the history is trustworthy, the direction and
rough scale of every major swing match the real record, but the exact
number for any single month is an approximation, not an official
figure. Before using this model for any real decision, run it with

```
python run.py --data path/to/official_cpi.csv
```

where the CSV has exactly two columns, `date` and `cpi_inflation_yoy`,
sourced from MOSPI (mospi.gov.in) or the Reserve Bank's data warehouse
(dbie.rbi.org.in). Nothing else in the code needs to change.

## How the model reads the past

Inflation is not a coin flip each month. It has memory. A model that
only looked at this month's value in isolation would ignore three
things that matter:

**Trend.** What has inflation been doing on average over the last
quarter, half year, and full year. Captured by rolling averages.

**Momentum.** Is inflation accelerating or decelerating right now,
compared to three and six months ago. Captured by simple differences
between recent lagged values.

**Season.** Indian food prices move with the agricultural calendar,
so December often differs systematically from July regardless of the
underlying trend. Captured by encoding the calendar month as a pair of
sine and cosine values, so the model can learn that December and
January are close together numerically, the way they are close in
time.

All twelve of the past month's readings are also given to the model
directly as separate inputs, called lags, so it can find patterns a
human eye might miss. See `src/inflation_forecast/features.py` for the
exact feature list.

## How the model is judged

A model that is simply asked to reproduce data it has already seen
will look excellent and mean very little. Instead, this model is
tested with a technique called time ordered cross validation, in
`src/inflation_forecast/model.py`. The model is trained only on the
earliest part of the history, then tested on the months right after
that boundary. The boundary is then moved forward and the process
repeated five times. This mirrors exactly the situation the model faces
in real use, forecasting months it has never seen, using only what
came before.

On that honest test, both models typically average an error of about
half a percentage point to three quarters of a percentage point per
month, reasonable for a pattern based model with no outside economic
information.

## How multiple months are forecast from a model that predicts one month

The model is fundamentally a one month ahead predictor. To reach
further out, in `src/inflation_forecast/forecast.py`, it predicts month
one, treats that prediction as if it were a real observation, and uses
it to predict month two, and continues in this way through the full
horizon. This is the standard approach for this kind of model, but it
means errors can accumulate. The forecast for month one rests on solid
ground. The forecast for the final month rests partly on the model's
own earlier guesses, so it deserves noticeably less confidence.

## Where the confidence band comes from

A Random Forest is built from four hundred individual decision trees,
each trained on a slightly different resample of the data. Rather than
just taking their average, this project lets every one of those four
hundred trees produce its own full forecast path. The spread across all
four hundred paths, specifically the tenth and ninetieth percentile at
each future month, becomes the shaded band on the chart. A wide band in
a given month means the trees disagree with each other about that
month, which is itself useful information, it tells you the forecast is
genuinely uncertain there, rather than giving a false sense of
precision.

## What the model cannot do

It does not know about anything that has not already shown up as a
pattern in the historical numbers. A surprise monsoon failure, a change
in fuel subsidy policy, a shock to global commodity prices, a change in
the Reserve Bank's policy stance, none of these are visible to the
model until they actually appear in the CPI print. A forecaster who
also tracks these factors directly, through a structural or
econometric model with real exogenous variables such as crude oil
prices, the rupee dollar rate, and the repo rate, will usually do
better over a full year than a pattern only model like this one.

The right way to think about this model is as one useful, honestly
validated input among several, not as a replacement for economic
judgment.
