# Behavioral Augmented CNN-LSTM Forecasting of the Bahrain All Share Index

A deep learning forecaster that beats its own price only baseline by 26.1 percent RMSE by adding a single behavioral feature derived from how retail traders misread candlestick patterns.

**Live showcase:** [https://pratmo.github.io/ai-time-series-contra](https://pratmo.github.io/ai-time-series-contra)

## What is in this repo

```
.
├── index.html               (showcase webpage served by GitHub Pages)
├── assets/                  (diagrams and forecast plots)
├── notebooks/
│   ├── baseline.ipynb       (Hybrid CNN-LSTM, price only)
│   └── contrarian.ipynb     (Hybrid CNN-LSTM with 11 candlestick pattern channels)
├── data/
│   └── df_bax_cleaned_till_outliers.csv
└── docs/
    └── project_report.pdf    (full project report)
```

The two notebooks contain all original outputs from the Kaggle runs that produced the metrics on the showcase page. They render directly on GitHub. Click either one to see the full code with plots and printed results inline.

## The idea

Retail traders interpret candlestick patterns in a textbook way. A Doji means reversal coming, sell. A Hammer means bullish reversal. According to the SEBI 2024 report, 93 percent of individual F&O traders lose money despite having access to the same patterns and the same charts. When a large, predictable group acts on the same signal in the same direction, markets often move the opposite way.

The model in this project does not hard code that intuition. It one hot encodes the pattern label for each day and lets the neural network learn from data what the pattern actually predicts. If retail readings are systematically wrong, the network picks up on that on its own.

## How it works

| Step | What happens |
|------|--------------|
| 1 | Detect candlestick pattern from days *t minus 2* and *t minus 1* using rule based logic. There are 11 possible labels. |
| 2 | One hot encode the pattern. |
| 3 | Build sliding window samples of shape `(60, 12)`. That is 60 days of price and 11 pattern flags tiled across the window. |
| 4 | Pass through `Conv1D` then `MaxPool1D` then `LSTM(64)` then `Dropout(0.2)` then `Dense(1)`. |
| 5 | Run walk forward cross validation on the first 80 percent of dates to pick hyperparameters. |
| 6 | Train final model on the full 80 percent. Recursively forecast 15 days into the held out 20 percent test block. |
| 7 | Compare against the same pipeline with pattern channels disabled. |

## Results

Test window is 15 days starting March 20, 2022.

| Model | MAE | RMSE | MAPE | vs. baseline |
|-------|-----:|-----:|-----:|-------------:|
| Baseline (price only) | 127.57 | 136.93 | 6.15% | Reference |
| **Contrarian (price plus patterns)** | **87.33** | **101.17** | **4.19%** | **26.1% lower RMSE** |

## Honest limitations

* Single test window. Validation across multiple rolling windows and market regimes is the recommended next step.
* GPU non determinism. TensorFlow on GPU is not perfectly reproducible across runs even with fixed seeds. The reported numbers come from the saved Kaggle notebook outputs and match the project report. Other runs of the same code may show different magnitudes of improvement.
* Naive baseline. A trivial "tomorrow equals today" predictor scores RMSE 8.7 on this window, far better than either CNN-LSTM. Recursive multi step forecasting compounds errors, while the naive predictor only ever has to be right one step ahead.

## Stack

Python, TensorFlow, Keras, scikit-learn, pandas, NumPy, matplotlib. Trained on Kaggle free NVIDIA P100 GPU.

## Author

Prathik Mohan. M.Sc. Data Science (Symbiosis International University).
