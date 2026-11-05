# MATH840 Final project: 48 hours of generation

Liliia Yaroshenko. This repository produces an hourly forecast with a 95% prediction interval for the
48 hours after 27 October 2026, 00:00.

## Files

| File | What it is |
|---|---|
| `YAROSHENKO_project.ipynb` | the whole pipeline and the report: EDA, models, a 26-window backtest, diagnostics, the submission file |
| `YAROSHENKO_project.csv` | the final submission written by the notebook |
| `train.csv`, `weather_forecast.csv` | the data from Moodle |
| `requirements.txt` | the package versions the notebook was run with |
| `_project.ipynb` | the course template |

## How to run

**Google Colab (easiest)**
1. Choose File → Upload notebook, and pick `YAROSHENKO_project.ipynb`.
2. Optional: choose Runtime → Change runtime type → GPU, so the backtest takes about 2 minutes instead of about 15.
3. Choose Run all. The first cell installs `timestral` from GitHub. The data cell asks you to upload
   `train.csv` and `weather_forecast.csv`.

**Locally**
```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace YAROSHENKO_project.ipynb
```
Keep `train.csv` and `weather_forecast.csv` next to the notebook.

The first run downloads the Timestral-209M weights (about 0.8 GB) from Hugging Face. The notebook writes
`YAROSHENKO_project.csv`.

## Final model

- **Point forecast, daylight hours:** 0.75 × the median of **Timestral-209M** + 0.25 × a LightGBM/seasonal-naive
  blend. Timestral is a pretrained, open-weight time-series model from Mistral AI, and it is conditioned on
  the weather as known covariates.
- **Point forecast, night hours:** Timestral's 45% quantile.
- **95% interval:** Timestral's own 2.5% and 97.5% quantiles, shifted with the point forecast.

## Results (backtest, last 26 windows of 48 hours)

Mean of the per-window scores. Lower is better, except coverage, whose target is 0.95. Skill is the RMSE
divided by the seasonal naive's RMSE.

| Model | Part | RMSE | MAPE | Winkler | Coverage | Skill | RMSE wins |
|---|---|---|---|---|---|---|---|
| Seasonal naive (benchmark) | 48 h | 306.9 | 203.8 | 2391.8 | 0.75 | 1.00 | – |
| LightGBM blend (Session 1) | 48 h | 250.2 | 150.0 | – | – | 0.84 | 24/26 |
| Timestral-209M + weather | 48 h | 188.1 | 62.8 | 841.3 | 0.96 | 0.67 | 24/26 |
| **Final** | 48 h | **190.8** | **56.1** | **843.8** | 0.96 | **0.66** | 24/26 |
| Seasonal naive | day 2 | 319.7 | 156.0 | 2242.8 | 0.81 | 1.00 | – |
| **Final** | day 2 | **198.8** | **55.0** | **910.5** | 0.96 | **0.68** | 22/26 |

The backtest uses the actual weather over each window. In the submission the weather is a forecast, so
expect larger errors. The notebook's report gives a stress test with simulated forecast errors.

The report in section 7 of the notebook explains the choices, the checks and what did not work.

## Use of LLMs

I used Claude (Anthropic), through Claude Code, as a coding and analysis assistant:
- **Session 1:** it drafted the LightGBM blend and the backtest code. Its first claim that the blend
  "never loses" came from 20 windows. Extending the test to 30 windows showed 3 losses, and I kept that
  correction.
- **After the public leaderboard:** it helped me diagnose that my interval was the weak point (Winkler rank
  ~25 of 27), set up the 26-window backtest and test alternatives: LightGBM variants, interval methods,
  Timestral with and without the weather, context lengths, ensembles and a weather-noise stress test.
- **The final notebook:** it generated the notebook from that work.

I chose which approaches to try, checked the numbers by running the code (every number here comes from an
executed notebook or script), and made the final choices. 
