# Explanation of the Multi-Factor Equity Alpha Model Notebook

The notebook is a complete research system for building and testing a stock-picking investment strategy. Its purpose is to determine whether a combination of several investment factors can identify stocks that may outperform the broader market. Rather than relying on one measure, such as recent price performance or company profitability, the notebook combines multiple types of information into one overall ranking.

The model uses two main sources of information. The first is historical market data, including stock prices and returns. The second is financial information collected from company filings submitted to the Securities and Exchange Commission, or SEC. These sources are combined to create an investment dataset, rank stocks, build a portfolio, and evaluate the results through historical backtesting.

## Project Setup

The first part of the notebook prepares the research environment. It installs and imports several Python packages used for financial data, statistical analysis, machine learning, and visualization. Libraries such as pandas and NumPy are used to organize and calculate data. SciPy and scikit-learn support statistical analysis, while matplotlib, seaborn, and Plotly create charts.

The notebook also creates folders for raw data, cached SEC information, processed data, charts, and final tables. This organization is important because it keeps the research outputs separate from the original data. It also makes the project easier to reproduce and maintain.

The model is configured to study a group of 20 large and well-known companies between 2015 and 2025. It rebalances the portfolio at the end of each month. At each rebalance, it selects the highest-ranked 20% of stocks. Because the universe contains 20 companies, the strategy normally selects about five holdings.

The model also includes a transaction-cost assumption of 10 basis points. Including this cost makes the backtest more realistic because actual investing involves trading expenses.

## Data Collection

The notebook collects stock prices using Yahoo Finance. Adjusted prices are used so that stock splits and dividends are handled more appropriately. These prices are necessary for calculating momentum, volatility, future returns, and portfolio performance.

The notebook also connects to the SEC’s Company Facts service. This service contains financial information reported by public companies. The model collects data such as revenue, net income, assets, shareholders’ equity, operating income, cash, and shares outstanding.

Because SEC information is downloaded from an external website, the notebook includes a small SEC client. This client identifies itself properly, handles temporary errors, respects request delays, and stores downloaded data locally. The local cache prevents the same information from being downloaded repeatedly.

Before requesting company financial information, the notebook converts stock tickers into SEC identification numbers called CIKs. This allows it to match each stock ticker with the correct company filing data.

## Point-in-Time Financial Data

One of the most important parts of the notebook is its treatment of financial information. The model attempts to use only information that would have been available to investors at the time.

For every monthly rebalance date, the notebook checks the filing date of each financial observation. It only uses observations that had been filed with the SEC on or before that date. It then selects the most recent eligible observation.

This approach helps reduce look-ahead bias. Look-ahead bias occurs when a backtest uses information from the future to make an earlier investment decision. For example, using a company’s annual results before those results were officially released would make the historical test look better than it could have been in reality.

The notebook also calculates year-over-year revenue growth using quarterly revenue observations. This compares a company’s most recent quarterly revenue with revenue from the same quarter in the previous year.

## Creating Investment Factors

After collecting the data, the notebook creates several investment factors. Each factor represents a different characteristic that may be related to future stock returns.

### Momentum

Momentum measures how strongly a stock has performed over a recent historical period. The model uses 12-month and 6-month price performance while excluding the most recent month. Excluding the latest month helps reduce the influence of short-term price reversals.

A stock with stronger historical price performance receives a higher momentum score.

### Value

Value measures whether a stock appears inexpensive relative to its financial fundamentals. The model uses earnings yield and book-to-market.

Earnings yield compares net income with market capitalization. A higher value means the company generates more earnings relative to the price investors are paying.

Book-to-market compares shareholders’ equity with market capitalization. A higher value generally indicates that the stock is cheaper compared with the accounting value of the company.

### Quality

Quality evaluates the strength and efficiency of a company. The notebook combines return on equity, return on assets, and operating margin.

Return on equity measures how effectively a company uses shareholder capital to generate earnings. Return on assets measures how effectively it uses its total assets. Operating margin measures how much operating profit remains from each dollar of revenue.

Companies with stronger profitability and operating performance receive higher quality scores.

### Growth

Growth is represented by year-over-year revenue growth. Companies with faster revenue growth receive higher growth scores.

Revenue growth is useful because it captures the expansion of a company’s business. However, growth by itself does not guarantee good investment performance, which is why it is combined with value, quality, and other factors.

### Low Volatility

Volatility measures how much a stock’s price has moved historically. The notebook calculates annualized volatility using approximately one year of daily returns.

The low-volatility factor reverses the volatility score. Therefore, stocks with lower historical volatility receive higher low-volatility scores.

### Size

Size is based on market capitalization. The notebook uses the logarithm of market capitalization and reverses the score, giving smaller companies higher size-factor scores.

Because the selected universe consists mostly of large companies, this factor may have limited variation compared with a broader stock universe.

## Cleaning and Standardizing the Data

Financial and market data can contain extreme values. A very small denominator, unusual accounting event, or data-entry problem can create an unrealistic ratio. To reduce the influence of these values, the notebook winsorizes factor data at the 1st and 99th percentiles.

The model then converts each factor into a cross-sectional z-score. This means each stock is compared with the other stocks available on the same date.

Standardization is important because the different factors use different units. Earnings yield, revenue growth, and volatility are not directly comparable until they are converted into a common scale.

The signs of the factors are aligned so that a higher score always represents a preferred characteristic. For example, lower volatility is desirable, so the volatility score is multiplied by negative one.

## Combining the Factors

The notebook combines the six factor scores into one composite alpha score. Each factor receives a predetermined weight:

- Momentum contributes 25%.
- Value contributes 20%.
- Quality contributes 20%.
- Growth contributes 15%.
- Low volatility contributes 10%.
- Size contributes 10%.

The composite score is calculated by multiplying each factor score by its weight and adding the results together.

A stock with a high composite score performs well across several characteristics. The notebook then calculates each stock’s percentile and rank within the universe. This creates a clear ordering from the most attractive stock to the least attractive stock according to the model.

## Measuring Predictive Power

The notebook calculates 21-day forward returns for every stock. These are the returns that occur after the model’s ranking date.

It then calculates the information coefficient, or IC. The IC measures the relationship between the model’s ranking and future stock returns. Specifically, it uses the Spearman correlation between alpha scores and subsequent returns.

A positive IC means that higher-ranked stocks tended to perform better during that period. A negative IC means the ranking worked in the opposite direction.

The notebook reports the average IC and the information coefficient information ratio, or ICIR. The ICIR compares the average predictive signal with its variability. A higher value suggests a more consistent signal.

The rolling IC chart is useful because it shows how the model behaves through different market conditions. The signal is not always positive. It moves above and below zero, showing that the model works better during some periods than others.

## Building the Portfolio

The model selects the top 20% of stocks according to the alpha percentile. The selected securities receive equal weights.

This is a simple and transparent portfolio-construction method. It avoids giving excessive influence to one stock based solely on its score. With 20 stocks in the universe, the portfolio generally contains five positions, each with a weight of approximately 20%.

The holdings are rebalanced monthly. Existing holdings may remain in the portfolio, while lower-ranked stocks can be replaced by higher-ranked stocks.

## Backtesting the Strategy

The backtest converts the monthly portfolio weights into daily portfolio weights. Each month’s holdings are carried forward until the next rebalance.

The notebook uses lagged weights when calculating returns. This prevents the model from using the portfolio decision at the same time as the return it is trying to explain.

It calculates gross returns, turnover, transaction costs, and net returns. Turnover measures how much the portfolio changes from one period to the next. Higher turnover generally results in higher trading costs.

The net asset value is calculated by compounding the daily net returns. The notebook also builds an equal-weight benchmark using the same stock universe. This allows the multi-factor strategy to be compared with a simpler alternative.

## Performance and Risk Analysis

The notebook calculates several performance measures:

- Total return measures the overall growth of the investment.
- CAGR measures the average annual growth rate.
- Annual volatility measures the variability of returns.
- The Sharpe ratio compares return with total risk.
- The Sortino ratio focuses specifically on downside risk.
- Maximum drawdown measures the largest decline from a previous high.
- The Calmar ratio compares return with maximum drawdown.
- Average turnover estimates trading activity.

These measures are important because a strategy should not be judged only by its final return. A strategy with high returns but very large drawdowns may be difficult for investors to hold.

## Charts and Research Outputs

The notebook creates several charts. The performance dashboard compares the strategy with the benchmark, shows drawdowns, and displays turnover.

The factor correlation matrix shows whether the factors provide different information. Lower correlations are generally useful because they indicate that the model is not relying on several versions of the same signal.

The factor-exposure chart shows the characteristics of the current portfolio. In the available results, momentum, value, quality, and growth are positive exposures, while low volatility and size are negative exposures.

The notebook also creates a momentum quintile chart. This divides stocks into five groups based on momentum and compares their future performance. The chart is intended to test whether stronger momentum corresponds to stronger returns.

However, the extremely large value shown near the end of that chart should be investigated. It may result from overlapping forward returns, outliers, missing data, or an issue in how returns are compounded. It should not automatically be interpreted as realistic investment performance.

Finally, the notebook saves processed datasets, portfolio weights, backtest results, performance metrics, current holdings, factor exposures, and charts. These files make the research easier to review and reproduce.

## Overall Importance

The notebook is important because it connects raw information to an actual investment decision. It explains which stocks are selected, why they are selected, what characteristics the portfolio has, and how the strategy performed historically.

Its strongest features are the use of multiple factors, point-in-time financial data, transaction-cost assumptions, benchmark comparison, and risk analysis.

At the same time, the results should be treated as research rather than proof of a ready-to-trade strategy. The universe is small, the stock list may contain survivorship bias, and some charts require further validation. Nevertheless, the notebook provides a strong framework for studying systematic equity investing and for improving the model through additional testing.
