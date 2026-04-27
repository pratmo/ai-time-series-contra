# Contrarian CNN-LSTM Forecasting for the Bahrain All Share Index

> M.Sc. Data Science thesis project · 2026

A hybrid CNN-LSTM model for forecasting the BAX index, augmented with a contrarian behavioral feature derived from candlestick pattern misinterpretations among retail traders. Adding the behavioral feature reduced test-set RMSE by **26.1%** vs the same architecture without it.

**Live showcase:** [https://pratmo.github.io/ai-time-series-contra](https://pratmo.github.io/ai-time-series-contra)

---

## What's in this repo

```
.
├── index.html               # Showcase webpage (served by GitHub Pages)
├── assets/                  # Diagrams and forecast plots
├── notebooks/
│   ├── baseline.ipynb       # Hybrid CNN-LSTM, price only
│   └── contrarian.ipynb     # Hybrid CNN-LSTM + 11 candlestick pattern channels
├── data/
│   └── df_bax_cleaned_till_outliers.csv
└── docs/
    └── thesis_report.pdf
```

The two notebooks contain all original outputs from the Kaggle runs that produced the metrics on the showcase page. They render directly on GitHub — click either one to see the full code with plots and printed results inline.

---

## The idea

Retail traders interpret candlestick patterns in a textbook way: a Doji means "reversal coming, sell," a Hammer means "bullish reversal." Per the SEBI 2024 study, **93% of individual F&O traders lose money** despite having access to the same patterns and the same charts. When a large, predictable group acts on the same signal in the same direction, markets often move the opposite way.

The model in this project doesn't hard-code that intuition. It one-hot encodes the pattern label for each day and lets the neural network learn from data what the pattern actually predicts. If retail readings are systematically wrong, the network picks up on that on its own.

## How it works

| Step | What happens |
|------|--------------|
| 1 | Detect candlestick pattern from days *t−2* and *t−1* using rule-based logic — 11 possible labels. |
| 2 | One-hot encode the pattern. |
| 3 | Build sliding-window samples of shape `(60, 12)`: 60 days of price + 11 pattern flags tiled across the window. |
| 4 | Pass through `Conv1D → MaxPool1D → LSTM(64) → Dropout(0.2) → Dense(1)`. |
| 5 | Walk-forward cross-validation on the first 80% of dates to pick hyperparameters. |
| 6 | Train final model on full 80%; recursively forecast 15 days into the held-out 20% test block. |
| 7 | Compare against the same pipeline with pattern channels disabled. |

## Results

Test window: 15 days starting 2022-03-20.

| Model | MAE | RMSE | MAPE | vs. baseline |
|-------|-----:|-----:|-----:|-------------:|
| Baseline (price only) | 127.57 | 136.93 | 6.15% | — |
| **Contrarian (price + patterns)** | **87.33** | **101.17** | **4.19%** | **−26.1% RMSE** |

## Honest limitations

- **Single test window.** Validation across multiple rolling windows and market regimes is the recommended next step.
- **GPU non-determinism.** TensorFlow on GPU is not perfectly reproducible across runs even with fixed seeds. The reported numbers come from the saved Kaggle notebook outputs and match the thesis report; other runs of the same code may show different magnitudes of improvement.
- **Naive baseline.** A "tomorrow = today" predictor scores RMSE 8.7 on this window — far better than either CNN-LSTM. Recursive multi-step forecasting compounds errors, while the naive predictor only ever has to be right one step ahead.

## Stack

Python, TensorFlow / Keras, scikit-learn, pandas, NumPy, matplotlib. Trained on Kaggle's free NVIDIA P100 GPU.

## Author

Prathik · M.Sc. Data Science (Symbiosis International University, 2026)
