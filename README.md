# What Moves Price: A Behavioral Signal Framework

### The Core Idea
A stock's price moves when buyers and sellers disagree about what it's worth, and one side pushes harder than the other. That "pushing harder" is conviction. But conviction alone doesn't explain price, there has to be a reason behind it. This study focuses on one reason: forward-looking fundamentals ("FF"), what the market currently believes a company will earn or grow into. Everything else (liquidity, sentiment, and macro conditions) is treated as evidence of conviction, not as a separate reason to buy or sell.

### What "Success" Looks Like
The goal isn't to find a strategy that made money in the past. It's to test, with real statistical methods, whether FF predicts forward returns. A well-tested result that says "this didn't work" is reported the same way a positive one would be. If this ends up in the graveyard, that's still a win: it means fundamentals-based signals can be set aside in future studies unless a strong, fresh insight specifically justifies revisiting them. This frees up time and attention for approaches more likely to hold up.


## FLOWCHART

Data Ingestion → Data Processing → Modeling → Conclusion

## DEFINITIONS

## DESIGN AND PROCESS

### Data Collection
Holdings are pulled from SPDR's official site for each sector ETF, giving the peer universe for relative comparisons. Metadata is then downlaoded from EDGAR via EdgarTools, financial data and actual filing dates from EDGAR's official API, FRED for macros data, and finally price from yFinance.

### Defining the Peer Group
Peer groups are drawn from sector ETF holdings rather than generic sector/industry tags (e.g. from yFinance). This shifts the burden of classification to the fund manager and index provider, whose grouping methodology (GICS-based) is documented, maintained, and industry-standard, making it more defensible than an arbitrary tag pulled from a free data source. Cap-weighting within the ETF doesn't affect this study, since only the holdings list is used to define peers.

### Two Ways FF is Measured
1. Absolute: is the stock's own multiple expanding or contracting over time.
2. Relative: is the stock's multiple expanding or contracting faster than its peer group.

### Indifferent to Valuation Level
This study does not judge whether a multiple is objectively "high" or "low." A stock at 100x isn't automatically expensive, and a stock at 10x isn't automatically cheap; lighter names routinely carry exaggerated multiples that mean little in isolation. What matters is direction: a multiple moving from 100x to 90x is contracting, regardless of how rich 90x still looks. The study measures behavior, not valuation level.

### Why Not Analyst Forecasts
The obvious way to measure "forward-looking" is analyst estimates — forward EPS and P/E. But these update slowly, aren't always available, and reflect one analyst's opinion rather than the market's. Instead, this study measures the market's forward expectation by watching how a company's valuation multiple changes over time. When a multiple expands, the market is paying more for the same earnings (a sign it expects better things ahead). When it contracts, the opposite. This study makes no claim about why a multiple moved. This avoids guessing at market psychology and stays grounded in observable behavior.

### Why Not Technicals
Technical indicators (moving averages, oscillators, chart patterns) are commonly used to time entries and exits, but this study deliberately excludes them as a predictor of forward returns. Technicals are derived from price itself, so using them to predict future price risks measuring the same performance twice, once as the signal and another as the outcome being tested. This exclusion is by design, not an oversight: the study is built to test whether FF independently predict returns, and mixing in a price-derived predictor would blur that answer.


## NOTEBOOK

## RESULTS AND CONCLUSION
