# Taiwan Stock Quantitative Analysis & LINE Bot

A quantitative analysis system for the Taiwan stock market integrating
technical indicators, institutional trading data, machine learning,
deep learning, LINE Bot, FastAPI, and cloud deployment.

本專案建立一套台股短線量化分析系統，從市場資料蒐集、特徵工程、
模型比較、Transformer 預測，到 LINE Bot 實際應用與雲端部署，
完成一套 End-to-End 的量化分析流程。

系統使用過去 **20 個交易日 × 16 個特徵**，
預測個股未來 **5 個交易日**的價格方向，
並將模型輸出轉換為 LONG、NEUTRAL、SHORT 訊號。

---

## Demo

### Single Stock Analysis

輸入股票代號後，系統會回傳：

- 股票名稱
- 模型方向
- 模型走多 / 走空機率
- 目前價格
- 參考進場價
- 停損價
- 停利價
- 資料日期

![Single Stock Analysis](linebot_single_stock.png)

### Daily Quantitative Ranking

系統可掃描台股市場，依照模型預測結果產生每日：

- 做多 Top 5
- 做空 Top 5
- 模型方向機率
- 進場價格
- 停損價格
- 停利價格

並透過 LINE Bot 自動推播。

![Daily Top 5](linebot_top5.png)

---

## Try the LINE Bot

Scan the QR code below to add the LINE Bot and test the quantitative analysis system.

![LINE Bot QR Code](linebot_QRcode.png)

> The LINE Bot is provided for project demonstration and research purposes only.

---

## Project Overview

本專案的目標並非只建立單一預測模型，而是完成一套可實際操作的
台股量化分析系統。

整體流程：

```text
Taiwan Stock Market Data
        +
Institutional Trading Data
        ↓
Data Cleaning
        ↓
Feature Engineering
        ↓
Model Training & Comparison
        ↓
Transformer
        ↓
Probability Prediction
        ↓
LONG / NEUTRAL / SHORT
        ↓
Risk Management Levels
        ↓
FastAPI
        ↓
LINE Bot
        ↓
Cloud Deployment
```

專案涵蓋：

- 台股歷史行情資料處理
- 技術指標建構
- 大盤資訊整合
- 法人籌碼資料整合
- 時間序列特徵工程
- Machine Learning 模型比較
- Deep Learning 模型比較
- Transformer 模型建立
- Out-of-sample Test
- 全市場股票掃描
- LINE Bot 串接
- FastAPI API
- Google Cloud 部署

---

## Prediction Task

The prediction task is formulated as a binary classification problem.

模型使用：

```text
Past 20 trading days
        ↓
16 features per day
        ↓
20 × 16 time-series input
        ↓
Predict next 5 trading days
```

預測目標：

```text
Target = 1 → Future price direction is upward
Target = 0 → Future price direction is downward
```

最終模型輸出：

```text
prob_up   = Probability of upward movement
prob_down = Probability of downward movement
```

交易方向設定：

```text
prob_up >= 0.70  → LONG
prob_up <= 0.30  → SHORT
otherwise        → NEUTRAL
```

---

## Feature Engineering

最終模型使用 **16 個特徵**，分成四大類：

### 1. Price & Return Features

```text
return_1d
return_5d
high_low_range
```

用於描述短期報酬與每日價格波動範圍。

### 2. Trend & Technical Features

```text
MA5_MA20_gap
MA20_MA60_gap
MACD_hist
BB_position
volatility_5d
```

捕捉：

- 短中期均線趨勢
- MACD 動能
- Bollinger Band 相對位置
- 短期波動程度

### 3. Volume Features

```text
volume_ratio_5
volume_ratio_20
```

比較目前成交量與近期平均成交量，
用來衡量市場交易活躍程度。

### 4. Market & Institutional Features

```text
TAIEX_return_1d
TAIEX_return_5d
foreign_net_ratio
trust_net_ratio
foreign_net_ratio_5d
trust_net_ratio_5d
```

將個股資訊與：

- 台灣加權指數
- 外資買賣超
- 投信買賣超

進行整合，使模型同時考慮技術面、整體市場環境與法人籌碼。

---

## Model Development

專案並非直接採用單一模型，而是比較多種 Machine Learning
與 Deep Learning 架構。

實驗模型包括：

- Logistic Regression
- XGBoost
- GRU
- GRU + Attention
- Transformer

模型以 Validation Set 進行調整，
並另外保留 Test Set 評估模型在未見資料上的表現。

---

## Model Comparison

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
showing better out-of-sample ranking ability among the evaluated models.

Therefore, Transformer was selected as the final model.

> **Note:** Test AUC = 0.7516 does not mean a 75.16% prediction accuracy.
> AUC measures the model's ability to distinguish and rank positive and
> negative outcomes across probability thresholds.

---

## Final Transformer Model

Final model configuration:

```text
Model: Transformer

Input:
20 trading days × 16 features

Output:
1 probability

Prediction Horizon:
5 trading days
```

Architecture:

```text
Input
  ↓
Multi-Head Self-Attention
  ↓
Residual Connection
  ↓
Layer Normalization
  ↓
Feed Forward Network
  ↓
Residual Connection
  ↓
Layer Normalization
  ↓
Global Average Pooling
  ↓
Dense
  ↓
Sigmoid
  ↓
Probability of Upward Movement
```

The Transformer uses self-attention to learn relationships between
different time steps in the 20-day sequence.

---

## Model Evaluation

Final Transformer performance:

| Metric | Validation | Test |
|---|---:|---:|
| Accuracy | 0.6129 | 0.4921 |
| AUC | 0.7523 | **0.7516** |

Because the final system uses probability outputs to rank stocks,
AUC is an important evaluation metric in addition to classification accuracy.

The project therefore focuses on:

```text
Probability Ranking
        +
Directional Discrimination
        +
Out-of-Sample Evaluation
```

rather than interpreting a single classification threshold as the entire
model performance.

---

## Full-Market Screening

The system was extended from single-stock prediction to full-market screening.

In a full-market system test:

```text
Stock Pool:          1,879
Market Data Loaded:  1,879
Successful Prediction: 1,837
```

The model generates:

```text
LONG
NEUTRAL
SHORT
```

predictions for individual stocks.

Stocks with the strongest probabilities are then ranked to generate:

```text
Top 5 LONG Candidates
Top 5 SHORT Candidates
```

The final ranking can be automatically sent through LINE Bot.

---

## LINE Bot Integration

The system supports two main LINE Bot functions.

### Single Stock Query

The user sends a Taiwan stock code:

```text
8114
```

The system automatically returns information such as:

```text
8114 振樺電

模型方向：建議做多
模型走多：98.30%

目前價：178.00
進場：176.27
停損：171.07
停利：186.66
```

### Daily Market Ranking

The system can also generate a market-wide ranking:

```text
每日短線量化 Top 5

【做多 Top 5】

Stock
Current Price
Probability
Entry
Stop Loss
Take Profit

【做空 Top 5】

Stock
Current Price
Probability
Entry
Stop Loss
Take Profit
```

This allows model predictions to be delivered through a practical
user interface instead of remaining only inside a Jupyter Notebook.

---

## Risk Management

In addition to model direction probabilities,
the system calculates reference trading levels such as:

```text
Entry Price
Stop Loss
Take Profit
```

The purpose is to extend the model from pure classification toward
a more complete quantitative decision-support workflow.

These values are reference outputs for model demonstration and are not
intended to represent guaranteed trading returns.

---

## Data Pipeline

The production workflow combines multiple data sources.

```text
Individual Stock Prices
        +
TAIEX Market Data
        +
Institutional Trading Data
        ↓
Date Alignment
        ↓
Feature Engineering
        ↓
Missing Value Handling
        ↓
Feature Scaling
        ↓
20-Day Sequence
        ↓
Transformer Prediction
```

The system also checks the latest available common trading date to reduce
date mismatches between price data and institutional data.

---

## System Architecture

```text
             ┌───────────────────────┐
             │ Taiwan Stock Market   │
             │ Price Data            │
             └──────────┬────────────┘
                        │
             ┌──────────▼────────────┐
             │ TAIEX Market Data     │
             └──────────┬────────────┘
                        │
             ┌──────────▼────────────┐
             │ Institutional Data    │
             │ Foreign / Investment  │
             │ Trust                 │
             └──────────┬────────────┘
                        │
             ┌──────────▼────────────┐
             │ Feature Engineering   │
             │ 16 Features           │
             └──────────┬────────────┘
                        │
             ┌──────────▼────────────┐
             │ Standardization       │
             └──────────┬────────────┘
                        │
             ┌──────────▼────────────┐
             │ 20-Day Sequence       │
             └──────────┬────────────┘
                        │
             ┌──────────▼────────────┐
             │ Transformer Model     │
             └──────────┬────────────┘
                        │
             ┌──────────▼────────────┐
             │ Probability Output    │
             │ LONG / NEUTRAL / SHORT│
             └──────────┬────────────┘
                        │
             ┌──────────▼────────────┐
             │ Risk Management       │
             │ Entry / SL / TP       │
             └──────────┬────────────┘
                        │
             ┌──────────▼────────────┐
             │ FastAPI               │
             └──────────┬────────────┘
                        │
             ┌──────────▼────────────┐
             │ LINE Bot              │
             └───────────────────────┘
```

---

## Cloud Deployment

The project was designed for containerized deployment.

Technologies used in the deployment workflow include:

```text
FastAPI
Uvicorn
Docker
Google Cloud Run
Google Cloud Storage
LINE Messaging API
```

The API can be started using:

```bash
uvicorn main:app --host 0.0.0.0 --port 8080
```

Google Cloud Storage is used to support data persistence required by the
deployed analysis workflow.

---

## Security

Sensitive credentials are not intended to be stored directly in
the public source code.

The application reads sensitive configuration from environment variables,
for example:

```python
import os

token = os.getenv("LINE_CHANNEL_ACCESS_TOKEN")
```

Sensitive information such as the following should never be committed:

```text
LINE Channel Access Token
LINE Channel Secret
API Keys
Cloud Credentials
.env
```

---

## Technologies

### Programming & Data Analysis

- Python
- Pandas
- NumPy
- Matplotlib
- Scikit-learn
- Joblib

### Machine Learning

- Logistic Regression
- XGBoost

### Deep Learning

- TensorFlow / Keras
- GRU
- Attention
- Transformer

### Financial Data

- yfinance
- Taiwan stock market data
- TAIEX
- Institutional trading data

### Backend & Deployment

- FastAPI
- Uvicorn
- LINE Messaging API
- Docker
- Google Cloud Run
- Google Cloud Storage

---

## Repository Structure

```text
stock-quant-linebot/
│
├── README.md
│
├── .gitignore
│
├── quant_analy.ipynb
│
├── linebot_single_stock.png
│
├── linebot_top5.png
└── linebot_QRcode.png
```

### `quant_analy.ipynb`

The notebook contains the complete project development process, including:

```text
Data Collection
Technical Analysis
Feature Engineering
Institutional Data Processing
Backtesting
Logistic Regression
XGBoost
GRU
GRU + Attention
Transformer
Model Comparison
Test Evaluation
Full-Market Prediction
LINE Bot Integration
FastAPI
Cloud Deployment
```

---

## Key Challenges

Several practical issues were addressed during project development.

### 1. Time-Series Data Leakage

Financial models are sensitive to future information leakage.

Therefore, the project uses chronological train / validation / test
splitting rather than random shuffling for time-series evaluation.

### 2. Multiple Data Sources

Price data, market index data, and institutional trading data may not
always share the exact same available date.

The pipeline therefore aligns data dates before prediction.

### 3. Model Generalization

A model with strong validation performance does not necessarily maintain
the same performance on unseen test data.

This was observed during the comparison between GRU and Transformer,
which motivated the use of an independent Test Set for final model selection.

### 4. Model-to-Application Integration

Instead of ending the project after model training,
the model was integrated into:

```text
Prediction
→ API
→ LINE Bot
→ Cloud Deployment
```

to build a usable end-to-end system.

---

## What I Learned

This project allowed me to integrate concepts from statistics,
machine learning, deep learning, finance, and software engineering.

Key learning outcomes include:

- Designing features for financial time-series data
- Constructing prediction targets without future-data leakage
- Evaluating models with chronological validation
- Comparing traditional machine learning and deep learning models
- Understanding the difference between Accuracy and ROC-AUC
- Working with sequential models such as GRU and Transformer
- Integrating technical and institutional market information
- Building a full-market screening pipeline
- Developing REST APIs with FastAPI
- Connecting machine learning predictions to LINE Bot
- Managing sensitive credentials using environment variables
- Containerizing applications with Docker
- Deploying a quantitative analysis workflow to the cloud

The most important outcome of this project was moving from a standalone
prediction model to a complete quantitative analysis system.

---

## Future Improvements

Possible future extensions include:

- Expanding the training universe
- Longer out-of-sample testing periods
- Walk-forward validation
- Probability calibration
- Additional market and fundamental features
- Transaction cost and slippage simulation
- Portfolio-level backtesting
- Position sizing
- Improved risk management
- Model monitoring and periodic retraining

---

## Disclaimer

This project is developed for academic, research, and portfolio demonstration
purposes only.

The model probabilities, LONG / SHORT signals, entry prices, stop-loss levels,
and take-profit levels do **not** constitute investment advice.

Past model performance does not guarantee future investment results.
