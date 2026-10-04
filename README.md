# Stock Quantitative Analysis & LINE Bot

A quantitative analysis system for the Taiwan stock market integrating
feature engineering, machine learning, deep learning, LINE Bot,
FastAPI, and cloud deployment.

本專案建立一套台股短線量化分析系統，從資料蒐集、特徵工程、
模型比較與 Transformer 預測，到 LINE Bot 實際應用，
完成一套 End-to-End 的量化分析流程。

系統使用過去 **20 個交易日 × 16 個特徵**，
預測個股未來 **5 個交易日**的價格方向，
並將模型輸出轉換為多空訊號。

---

## Demo

### Single Stock Analysis

輸入股票代號後，系統會回傳模型方向、走多 / 走空機率、
目前價格，以及進場、停損與停利參考資訊。

![Single Stock Analysis](linebot_single_stock.png)

### Daily Quantitative Ranking

系統於每個開盤日下午5：00自動收集完法人資料跑完模型自動推播，依照模型預測機率產生
每日做多與做空 Top 5。

![Daily Top 5](linebot_top5.png)

---

## Try the LINE Bot

Scan the QR code below to add the LINE Bot and test the system.

![LINE Bot QR Code](linebot_QRcode.png)

> For academic and project demonstration purposes only.

---

## Project Overview

```text
Stock Price Data
        +
TAIEX Market Data
        +
Institutional Trading Data
        ↓
Feature Engineering
        ↓
20-Day Time Series
        ↓
Model Training & Comparison
        ↓
Transformer
        ↓
Probability Prediction
        ↓
LONG / NEUTRAL / SHORT
        ↓
FastAPI
        ↓
LINE Bot
        ↓
Cloud Deployment
```

本專案不只建立預測模型，也將模型整合至實際可使用的 LINE Bot，
並完成全市場股票掃描與雲端部署。

---

## Prediction Task

模型使用：

```text
Input:
20 trading days × 16 features

Prediction Horizon:
5 trading days

Output:
Probability of upward movement
```

交易方向設定：

```text
prob_up >= 0.70  → LONG
prob_up <= 0.30  → SHORT
otherwise        → NEUTRAL
```

最終模型的輸入設定為 20 個交易日、16 個特徵，
並預測未來 5 個交易日方向。

---

## Feature Engineering

最終模型使用 **16 個特徵**，整合價格、趨勢、成交量、
大盤與法人籌碼資訊：

```text
return_1d
return_5d
high_low_range
MA5_MA20_gap
MA20_MA60_gap
MACD_hist
BB_position
volatility_5d
volume_ratio_5
volume_ratio_20
TAIEX_return_1d
TAIEX_return_5d
foreign_net_ratio
trust_net_ratio
foreign_net_ratio_5d
trust_net_ratio_5d
```

透過這些特徵，使模型同時考量個股技術面、
市場整體走勢與法人交易行為。

---

## Model Comparison

本專案比較傳統 Machine Learning 與 Deep Learning 模型：

| Model | Validation Accuracy | Validation AUC | Test Accuracy | Test AUC |
|---|---:|---:|---:|---:|
| Logistic Regression | 0.6935 | 0.7409 | 0.5238 | 0.5589 |
| XGBoost | 0.5323 | 0.5330 | 0.3968 | 0.5305 |
| GRU | 0.6774 | **0.8216** | 0.5238 | 0.5979 |
| GRU + Attention | 0.6613 | 0.6466 | 0.4444 | 0.4032 |
| **Transformer** | 0.6129 | 0.7523 | 0.4921 | **0.7516** |

Although GRU achieved the highest Validation AUC,
its performance decreased on the independent Test Set.

The Transformer achieved the highest **Test AUC = 0.7516**,
therefore it was selected as the final model. 

> Test AUC = 0.7516 does not mean 75.16% prediction accuracy.
> AUC measures the model's ability to distinguish and rank outcomes.

---

## Final Transformer Model

```text
20-Day Sequence
        ↓
Multi-Head Self-Attention
        ↓
Residual Connection
        ↓
Layer Normalization
        ↓
Feed Forward Network
        ↓
Global Average Pooling
        ↓
Sigmoid
        ↓
Probability of Upward Movement
```

Transformer 能透過 Self-Attention 學習不同交易日之間的關係，
並輸出未來價格方向的機率。

---

## Full-Market Screening

系統除了單一股票預測，也能進行全市場掃描。

一次完整測試結果：

```text
Stock Pool:             1,879
Successful Prediction:  1,837
```

模型會將股票分類為：

```text
LONG
NEUTRAL
SHORT
```

並依預測機率選出：

```text
Top 5 LONG Candidates
Top 5 SHORT Candidates
```

最後透過 LINE Bot 自動產生每日短線量化結果。

---

## LINE Bot Integration

### Single Stock Query

User input:

```text
8114
```

Example output:

```text
8114 振樺電

模型方向：建議做多
模型走多：98.30%

目前價：178.00
進場：176.27
停損：171.07
停利：186.66
```

### Daily Ranking

```text
每日短線量化 Top 5

【做多 Top 5】

Stock
Current Price
Model Probability
Entry
Stop Loss
Take Profit

【做空 Top 5】

Stock
Current Price
Model Probability
Entry
Stop Loss
Take Profit
```

這使模型不只停留在 Notebook，
而是能透過實際介面提供量化分析結果。

---

## System Architecture

```text
Market Data
    ↓
Feature Engineering
    ↓
Transformer
    ↓
Probability Prediction
    ↓
LONG / NEUTRAL / SHORT
    ↓
FastAPI
    ↓
LINE Bot
    ↓
Google Cloud
```

---

## Technologies

`Python` · `Pandas` · `NumPy` · `Scikit-learn` · `XGBoost`  
`TensorFlow / Keras` · `GRU` · `Transformer`  
`FastAPI` · `LINE Messaging API` · `Docker`  
`Google Cloud Run` · `Google Cloud Storage` · `yfinance`

---

## Repository Structure

```text
stock-quant-linebot/
│
├── README.md
├── .gitignore
├── quant_analy.ipynb
├── linebot_single_stock.png
├── linebot_top5.png
└── linebot_QRcode.png
```

`quant_analy.ipynb` contains the complete development process, including
data collection, feature engineering, model comparison, Transformer modeling,
full-market prediction, LINE Bot integration, and cloud deployment.

---

## What I Learned

Through this project, I integrated financial time-series analysis,
machine learning, deep learning, and software deployment into one system.

The key learning outcome was moving from a standalone prediction model to
an **End-to-End quantitative analysis application**:

```text
Data
→ Feature Engineering
→ Model
→ Evaluation
→ API
→ LINE Bot
→ Cloud Deployment
```

---

## Future Improvements

Future work will focus on both model performance and system usability.

Planned improvements include:

- Fixing remaining bugs in the analysis and LINE Bot workflow
- Improving UI/UX and message readability
- Improving system stability and error handling
- Walk-forward validation
- Longer out-of-sample testing
- Transaction cost and slippage simulation
- Probability calibration
- Portfolio-level backtesting
- Automatic model retraining
- Improving prediction robustness across different market conditions

---

## Disclaimer

This project is developed for academic, research, and portfolio demonstration
purposes only.

The model probabilities and trading signals do not constitute investment advice.
Past model performance does not guarantee future investment results.

---

## Project Report Link

[View Full Project Report on Google Drive](https://drive.google.com/drive/folders/1FG0RotUS1tanY0KZiaMs6lOMX-_vSlx?dmr=1&ec=wgc-drive-hero-goto)
