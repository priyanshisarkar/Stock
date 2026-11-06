# NIFTY 50 Portfolio Optimization: MVP vs HRP

A Streamlit dashboard that compares two portfolio allocation strategies using daily **Close** prices for a user-selected group of NIFTY 50 stocks:

- **MVP (Minimum Variance Portfolio):** finds long-only weights that minimize estimated portfolio variance.
- **HRP (Hierarchical Risk Parity):** uses correlation-based hierarchical clustering and recursive bisection to allocate portfolio weights.
- **Equal Weight:** included as a simple benchmark for out-of-sample comparison.

The app displays return, risk and Sharpe ratio metrics, performance charts, allocation pie charts, an allocation table, and a CSV download.

## 1. Project files

Keep these files in the same folder:

```text
portfolio-optimization/
├── portfolio_optimizer_app.py
├── requirements.txt
└── README.md
```

## 2. Requirements

- Python 3.9 or later recommended
- Internet connection to download market data from Yahoo Finance
- The Python packages listed in `requirements.txt`

Install the dependencies:

```bash
pip install -r requirements.txt
```

## 3. Run the Streamlit app

Open a terminal in the project folder and run:

```bash
streamlit run portfolio_optimizer_app.py
```

Streamlit will start the app and provide a local browser address, usually:

```text
http://localhost:8501
```

Open that address if the browser does not launch automatically.

## 4. Default stock universe

The app starts with the following editable Yahoo Finance symbols:

| Company | Yahoo Finance ticker |
|---|---|
| Reliance Industries | `RELIANCE.NS` |
| HDFC Bank | `HDFCBANK.NS` |
| ICICI Bank | `ICICIBANK.NS` |
| Bharti Airtel | `BHARTIARTL.NS` |
| Infosys | `INFY.NS` |
| Tata Consultancy Services | `TCS.NS` |
| State Bank of India | `SBIN.NS` |
| Larsen & Toubro | `LT.NS` |
| ITC | `ITC.NS` |
| Axis Bank | `AXISBANK.NS` |

These are editable example constituents, not a dynamically verified top-10-by-market-cap list. Confirm the NIFTY 50 membership and market-cap ranking for the reference date required by your project. Yahoo Finance NSE symbols commonly use the `.NS` suffix.

## 5. Analysis period and train/test split

The default requested date range is:

- **Full data period:** 1 January 2025 to 31 October 2026
- **Training data:** the historical observations before the final six-month test window
- **Testing data:** the final six calendar months of available observations

The app determines the test window relative to the last available trading date returned by Yahoo Finance. The end date supplied to Yahoo Finance is made exclusive-safe so that the selected end date can be included.

Portfolio weights are estimated using training-period returns only. The resulting weights are then held fixed as target weights during the test-period calculation. No test-period observations are used to estimate the allocations.

> **Date availability note:** The requested period ends on 31 October 2026. Data for future dates cannot be downloaded before those dates have occurred and Yahoo Finance has published the observations. If you run the app before the full period is available, the actual downloaded end date and six-month test window will be earlier. Check the dates displayed by the dashboard.

## 6. Price data and return calculation

The application downloads daily Yahoo Finance data and explicitly selects the **`Close`** field. It does not use Open, High, Low, Adjusted Close, or Volume for the analysis.

Daily simple returns are calculated as:

```text
Daily return = (Today's Close / Previous Trading Day's Close) - 1
```

The app aligns the price series to dates shared by all selected stocks before calculating returns. Stocks with no available data are dropped; dates with missing prices across the selected universe are excluded.

**Important:** Unadjusted Close prices do not account for dividend reinvestment and may be affected by stock splits. This choice follows the Close-only requirement, but it is not the same as a total-return analysis.

## 7. Portfolio methodologies

### 7.1 Minimum Variance Portfolio (MVP)

MVP minimizes estimated portfolio variance:

```text
Minimize: w' Σ w

Subject to:
    Sum of weights = 1
    0 <= each weight <= 1
```

Where:

- `w` is the vector of stock weights.
- `Σ` is the covariance matrix estimated from training-period daily returns and annualized using 252 trading days.

The optimization uses SciPy's SLSQP solver. The portfolio is long-only, fully invested, and does not use leverage or short selling.

### 7.2 Hierarchical Risk Parity (HRP)

HRP uses the training-period correlation matrix to:

1. Convert correlations into a distance matrix.
2. Perform hierarchical clustering.
3. Order stocks according to the resulting cluster tree.
4. Recursively divide the ordered clusters.
5. Allocate weights using cluster variances.

HRP is designed to account for the correlation structure between assets without directly solving the same global variance-minimization problem used by MVP.

### 7.3 Equal-weight benchmark

The dashboard also calculates an equal-weight portfolio, assigning the same initial weight to every included stock. It is a reference benchmark, not an optimized strategy.

## 8. Performance measures

The dashboard reports the following out-of-sample measures:

| Measure | Meaning |
|---|---|
| Total Return | Cumulative portfolio return over the test period |
| CAGR | Annualized compound growth rate over the test period |
| Annualized Volatility | Standard deviation of daily portfolio returns multiplied by the square root of 252 |
| Sharpe Ratio | Annualized excess return divided by annualized volatility |
| Maximum Drawdown | Largest peak-to-trough decline in cumulative portfolio value |

The annual risk-free rate defaults to **6%** and can be edited in the sidebar.

The Sharpe ratio is calculated as:

```text
Sharpe Ratio = (Annualized portfolio return - Annual risk-free rate)
               / Annualized portfolio volatility
```

For the out-of-sample results, annualized return is calculated from the realized test-period CAGR. The risk-free rate and volatility are annualized on a consistent basis.

## 9. Dashboard visualizations

The app provides:

1. **Cumulative return chart:** growth of ₹1 for MVP, HRP and Equal Weight during testing.
2. **Rolling risk chart:** rolling 21-trading-day annualized volatility.
3. **Risk-return chart:** test-period CAGR against annualized volatility.
4. **MVP allocation pie chart:** percentage allocation to each stock.
5. **HRP allocation pie chart:** percentage allocation to each stock.
6. **Allocation table:** exact MVP and HRP portfolio weights.
7. **Training estimates:** annualized return, volatility and Sharpe ratio estimated from training data.

The app also provides a CSV download of the allocation table.

## 10. Interpretation guide

When comparing the strategies:

- A higher Sharpe ratio indicates more excess return per unit of measured risk.
- Lower annualized volatility indicates a less volatile return series.
- A higher CAGR indicates stronger annualized compound growth.
- A less negative maximum drawdown indicates a smaller observed peak-to-trough loss.
- More balanced allocations do not automatically mean better performance.
- Results can change when the stock universe, training window, risk-free rate or market-data source changes.

Do not conclude that one strategy is universally superior based on a single historical test window.

## 11. Methodological assumptions and limitations

- The analysis uses daily unadjusted Close prices only.
- The default stock list is editable and should be verified for the intended historical reference date.
- The training/test split is chronological.
- Portfolio weights are estimated only from training data.
- Test-period portfolio returns are calculated as the weighted sum of stock returns using the estimated weights. This represents a constant-weight, daily-rebalanced return series.
- Transaction costs, brokerage, taxes, slippage, liquidity constraints and market impact are not included.
- The risk-free rate is a user-supplied assumption, not a live market yield.
- Yahoo Finance data may be revised, delayed, incomplete or unavailable for some symbols.
- Historical performance does not guarantee future results.

This application is intended for educational and research purposes. It is not investment advice.

## 12. Troubleshooting

### No data was downloaded

- Check your internet connection.
- Confirm that the Yahoo Finance ticker symbols are correct.
- Confirm that the requested dates are supported by the data provider.

### A stock is missing

- Check the ticker spelling and `.NS` suffix.
- Confirm that the stock has data for the selected date range.
- The app removes columns with no observations and uses dates with complete data across the remaining stocks.

### Insufficient training or testing observations

- Use a longer date range.
- Check whether all selected stocks have overlapping Close-price observations.
- Try a smaller or corrected ticker universe.

### Streamlit command not found

Install Streamlit in the active Python environment:

```bash
python -m pip install streamlit
```

Then run:

```bash
python -m streamlit run portfolio_optimizer_app.py
```

---

**Project:** NIFTY 50 Portfolio Optimization  
**Strategies:** Minimum Variance Portfolio (MVP) and Hierarchical Risk Parity (HRP)  
**Interface:** Streamlit  
**Data source:** Yahoo Finance via `yfinance`


## 13. Compatibility fix

The HRP distance matrix is converted using `dist.to_numpy(dtype=float, copy=True)` before its diagonal is set to zero. This creates a writable NumPy array and avoids `ValueError: underlying array is read-only` in newer Pandas/NumPy combinations.
