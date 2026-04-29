# Time-series Forecasting Webapp

Final project for **AIE 1902 — Quantitative Methods with AI tools for Financial Markets**.

- **Webapp URL**: `<paste Streamlit Cloud URL here after deploy>`
- **GitHub repo**: `<paste GitHub URL here>`

## What it does

User uploads an `.xlsx`. The app automatically (no method selection, no extra
buttons) does two things:

1. **Part 1 — One-step-ahead forecast**: predicts `y_{N+1}` using only
   `y_1 ... y_N`.
2. **Part 2 — Walk-forward backtest**: refits the model on every prefix
   `y[:t]` and forecasts `y[t]` for each `t` in the last 20% of the series.
   Reports RMSE / MAE / MAPE and offers the test-segment forecasts as a
   downloadable `forecast.xlsx`.

## Input schema

| Column | Required | Notes |
|---|---|---|
| `y` | yes | Numeric, ordered oldest → newest, **no missing values** |
| `date` | optional | Used only for display / output alignment |

The first sheet of the workbook is used regardless of name.

## Output schema

`forecast.xlsx` contains a single sheet named `forecast` with:

- `y` — predicted values for the test segment (`0.2 * N` rows)
- `date` — only present if the input had a `date` column

## Method

- **Model**: `ARIMA(1, 1, 1)` from `statsmodels.tsa.arima.model`, refit at every
  walk-forward step.
- **Fallback**: naive (`y_t = y_{t-1}`) if the optimizer fails to converge.
- **Selection**: chosen via a bake-off on the sample dataset
  (`naive / MA(5) / SES / ARIMA(0,1,1) / ARIMA(1,1,1) / ARIMA(2,1,0) / ARIMA(2,1,2)`),
  ARIMA(1,1,1) had the lowest RMSE.
- **Data leakage**: the function `walk_forward` slices `y[:t]` for each step
  with an explicit `assert` to guarantee no future information is used.

## Local development

```powershell
pip install -r requirements.txt
streamlit run app.py
```

Open <http://localhost:8501> and upload an `.xlsx`.

## Deployment (Streamlit Community Cloud)

1. Push this repo to GitHub (public repo for the free tier).
2. Go to <https://share.streamlit.io>, sign in with GitHub, click **Create app**.
3. Repository = your repo, Branch = `main`, Main file path = `app.py`.
4. Click **Deploy**; wait 3–5 minutes for the first build.

## Files

```
app.py                 # single-page upload-driven exam flow
requirements.txt       # exact runtime deps
.streamlit/config.toml # theme + server options
utils/
  __init__.py
  forecast.py          # ARIMA(1,1,1) + walk-forward + naive fallback
README.md              # this file
```
