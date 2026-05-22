# CO2 Forecasting ARIMA Pipeline

An end-to-end time series analysis and forecasting pipeline for atmospheric $CO_2$ data. This project bypasses automated brute-force grid searches to implement a structured, diagnostic-driven engineering workflow—utilizing manual statistical identification, information criteria optimization, strict residual auditing, and out-of-sample chronological validation.

---

## 🚀 Workflow Architecture

The modeling pipeline follows a strict sequence of cause, effect, and statistical verification:


[Raw Data] ➔ [Trend/Seasonal Differencing] ➔ [ACF/PACF Plots] ➔ [AIC Showdown] ➔ [Residual Auditing] ➔ [Blind Out-of-Sample Forecast]


# CO2 Forecasting ARIMA Pipeline

An end-to-end time series analysis and forecasting pipeline for atmospheric $CO_2$ data. This project bypasses automated brute-force grid searches to implement a structured, diagnostic-driven engineering workflow—utilizing manual statistical identification, information criteria optimization, strict residual auditing, and out-of-sample chronological validation.

---

## 🚀 Workflow Architecture

The modeling pipeline follows a strict sequence of cause, effect, and statistical verification:


```

[Raw Data] ➔ [Trend/Seasonal Differencing] ➔ [ACF/PACF Plots] ➔ [AIC Showdown] ➔ [Residual Auditing] ➔ [Blind Out-of-Sample Forecast]

```

### 1. Achieving Stationarity
Visual inspection of the raw historical monthly data showed an obvious upward trend and a strong annual cyclic wave. To stabilize the mean and variance before modeling, the time series was transformed using:
* One round of regular differencing ($d=1$) to remove the long-term trend.
* One round of seasonal differencing ($D=1$) with a period of 12 months ($s=12$) to strip away the annual cycle.

### 2. Structural Identification (ACF & PACF)
Autocorrelation Function (ACF) and Partial Autocorrelation Function (PACF) plots were generated for the differenced, stationary data to expose the underlying memory structure:
* **Observations:** The plots revealed a sharp short-term cutoff alongside a prominent, significant negative spike at Lag 1. This negative correlation indicated a powerful inverse dependency between consecutive months, where an increase in $CO_2$ in one month is typically followed by a balancing decrease the next month.
* **Avoiding Over-Differencing:** A lingering seasonal shock was visible at Lag 12. Rather than applying a second round of seasonal differencing, which would cause over-differencing, introduce artificial noise, and destroy the structural signal, the model's math was leveraged instead. This lingering spike served as direct evidence to introduce a Seasonal Autoregressive parameter ($P=1$) to mathematically absorb the annual memory.

### 3. Parameter Selection via AIC
The visual signatures narrowed the architecture down to a direct face-off between short-term Autoregressive (AR) and Moving Average (MA) configurations. To penalize parameter complexity and maximize efficiency, the Akaike Information Criterion (AIC) was introduced to score the top candidate frameworks:
* **AR-Heavy Candidate** [$\text{ARIMA}(1,1,0) \times (1,1,0)_{12}$]: $\text{AIC} = -104.53$
* **MA-Heavy Candidate** [$\text{ARIMA}(0,1,1) \times (1,1,0)_{12}$]: $\text{AIC} = -123.41$

**The Winner:** The MA-heavy configuration secured the lower mathematical score, finalizing the architecture. The choice was validated by a final `ma.L1` coefficient of **-0.4524**, confirming that treating the month-to-month negative dependency as an immediate, self-correcting moving average shock was the most parameter-efficient engine.

---

## 📊 Model Evaluation & Performance Metrics

### 1. Residual Diagnostics (The Statistical Audit)
To guarantee that the parameters successfully extracted all time-dependent relationships from the data, the model's tracking mistakes (residuals) were put on trial:

| Metric Group | Key Indicator | Preferred Range | Model Score | Status / Verdict |
| :--- | :--- | :--- | :--- | :--- |
| **Independence** | `Prob(Q)` (Ljung-Box) | $> 0.05$ | **0.63** | **PASS** (Leftover errors are pure, independent white noise) |
| **Stable Variance** | `Prob(H)` (Heteroskedasticity) | $> 0.05$ | **0.26** | **PASS** (Tracking volatility remains stable across the timeline) |
| **Normality** | `Prob(JB)` (Jarque-Bera) | $> 0.05$ | **0.02** | **Mild Fail** (Driven by isolated initialization anomalies) |

#### Investigating the Normality Fail
Rather than accepting the Jarque-Bera failure blindly, a deeper look into the residual distribution shape showed an excellent, balanced Skew of **0.11** but an elevated Kurtosis of **4.10**. Analysis of the *Error Timeline* and *Normal Q-Q Plot* isolated the root cause: the error variance was perfectly flat and tightly bound to zero for over a decade, but caught two extreme outlier spikes in the very first year while the ARIMA equation was establishing its lookback memory. This confirmed an isolated initialization artifact rather than a systemic modeling defect.

### 2. Out-of-Sample Validation (The Blind Test)
To prove true predictive robustness, the data was split chronologically. The final 21 months of data were completely hidden from the model during the training phase. The model was then forced to generate a blind forecast into this unseen horizon:
* **Mean Absolute Error (MAE):** `0.3711`
* **Root Mean Squared Error (RMSE):** `0.4338`

**Engineering Takeaway:** On baseline data values averaging 350, an MAE of 0.3711 proves that the typical month-to-month tracking error is under **0.1%**. Because the RMSE sits tightly beside the MAE, it mathematically confirms that the forecast remained highly stable and entirely free of variance inflation or extreme guessing disasters across the entire 21-month projection.

---

## 🛠️ Tech Stack & Libraries
* **Language:** Python
* **Environment:** Jupyter Notebook / Google Colab
* **Core Libraries:** `statsmodels` (ARIMA, ACF/PACF diagnostics), `pandas` (time series manipulation), `numpy`, `matplotlib` (residual visualization), `sklearn` (validation metrics).

```
