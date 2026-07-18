# (In Progress) What Moves Price: A Behavioral Signal Framework

**Currently working on XLE forming a template that can be easily replicated to other ETFs in the study**

### The Core Idea
A stock's price moves when buyers and sellers disagree about what it's worth, and one side pushes harder than the other. That "pushing harder" is conviction. But conviction alone doesn't explain price, there has to be a reason behind it. This study focuses on one reason: forward-looking fundamentals ("FF"), what the market currently believes a company will earn or grow into. Everything else (liquidity, sentiment, and macro conditions) is treated as evidence of conviction, not as a separate reason to buy or sell.

### What "Success" Looks Like
The goal isn't to find a strategy that made money in the past. It's to test, with real statistical methods, whether FF predicts forward returns. A well-tested result that says "this didn't work" is reported the same way a positive one would be. If this ends up in the graveyard, that's still a win: it means fundamentals-based signals can be set aside in future studies unless a strong, fresh insight specifically justifies revisiting them. This frees up time and attention for approaches more likely to hold up.


## FLOWCHART

Data Ingestion → Data Processing → Modeling → Conclusion

.....

## DEFINITIONS

**Forward Fundamentals**: What the market currently expects a company to earn or grow into, going forward. Measured indirectly in this study through valuation multiple behavior, not analyst forecasts.

**Valuation Multiple**: A ratio like price-to-earnings (P/E) that expresses how much investors are paying for a company's earnings.

**Expansion / Contraction**: Whether a stock's multiple is rising or falling over time. This study only measures direction and rate of change, not whether a multiple is objectively high or low.

**SPDR (Standard & Poor's Depositary Receipts)**: A family of exchange-traded funds (ETFs), including sector-specific funds that track segments of the S&P 500 (e.g. technology, healthcare, financials). Used here as the source of each stock's peer group, via their fund holdings.

**S&P 500**: A market index of 500 large, publicly traded U.S. companies, maintained by S&P Dow Jones Indices. Referenced as the underlying universe SPDR sector ETFs are built from.

**GICS (Global Industry Classification Standard)**: The sector/industry classification system used by S&P, and by extension SPDR, to group companies. Referenced here only as the basis for peer grouping, not applied directly.

**VIX (Volatility Index)**: Often called the market's "fear gauge," VIX measures expected volatility in the S&P 500 over the next 30 days. Sourced from FRED. Used in this study as a filter for market-wide sentiment, so results can be checked and compared between  calm vs turbulent conditions.

**Corporate Bond Yields**: Interest rates on corporate debt, used here as a secondary gauge of broader financing conditions alongside VIX. Sourced from FRED.

**FRED (Federal Reserve Economic Data)**: A public database maintained by the Federal Reserve Bank of St. Louis, providing macroeconomic and financial time series data.

**EDGAR (Electronic Data Gathering, Analysis, and Retrieval)**: The U.S. Securities and Exchange Commission's (SEC) public database of company filings. Used both directly, via its structured XBRL companyfacts API for bulk historical financial data, and indirectly through edgartools, a library that parses the underlying filed documents for metadata (industry, SIC code, fiscal year end) not easily available through the raw API. Both are the same underlying SEC data, accessed through different extraction methods.

## DESIGN AND PROCESS

### Data Collection
Holdings are pulled from SPDR's official site for each sector ETF, giving the peer universe for relative comparisons. Metadata is then downlaoded from EDGAR via EdgarTools, financial data and actual filing dates from EDGAR's official API, FRED for macros data, and finally price from yFinance.

### Filing Type Selection
Fundamentals are sourced from 10-K (annual) and 10-Q (quarterly) filings only. These are the forms carrying standardized, audited or reviewed financial data, and the only forms consistently available through EDGAR's structured XBRL companyfacts endpoint. 8-K filings, which often carry the earliest earnings press release, are excluded. Not every filer reports financials through 8-K in a structured, retrievable way, and NVIDIA is one example where only 10-K/10-Q data is available through this endpoint despite the company filing 8-Ks around earnings.

### Defining the Peer Group
Peer groups are drawn from sector ETF holdings rather than generic sector/industry tags (e.g. from yFinance). This shifts the burden of classification to the fund manager and index provider, whose grouping methodology (GICS-based) is documented, maintained, and industry-standard, making it more defensible than an arbitrary tag pulled from a free data source. Cap-weighting within the ETF doesn't affect this study, since only the holdings list is used to define peers.

### Two Ways FF is Measured
1. Absolute: is the stock's own multiple expanding or contracting over time.
2. Relative: is the stock's multiple expanding or contracting faster than its peer group.

### Primary Multiple: Market Cap to Revenue (P/S), with Income as Fallback
This study primarily uses P/S, computed as market capitalization ("MCAP") divided by revenue, not price divided by revenue- or earnings-per-share, as its valuation multiple. MCAP is used rather than price because it represents the market's actual valuation of the entire company, while price is simply a transactional amount. Revenue is preferred over earnings as denominator because it cannot be inflated by margin improvement from cost-cutting, and directly reflects a company's ability to grow and capture market share, closer to the spirit of what this study calls forward fundamentals.

However, revenue is not always the most reliably reported figure across companies and sectors. Energy sector universe (XLE), revenue-tagged data was available for roughly 40% of filings, compared to 95%+ coverage for EPS and net income. This is not primarily a lookup or fallback-list problem; even after exhausting known revenue tag variants, coverage remained thin. The most likely explanation is that revenue is frequently affected by accounting treatment changes over time: hedging activity, commodity price accounting, and impairments can all shift how revenue is classified or reported from one filing to the next, sometimes under an entirely different XBRL tag.

**Why EPS is More Resistant to This Problem**

Earnings-per-share ("EPS"), and net income more broadly, are bottom-line figures. Even when a company amends a filing or changes an accounting treatment, EPS typically remains tagged under the same concept. What changes is the computation feeding into it, not the tag itself. Revenue does not share this property: a genuine reclassification can mean the figure moves to a different concept entirely, which is functionally indistinguishable (from a data-pulling perspective) to the figure simply not being reported at all. This is a real inconsistency in EDGAR's XBRL data. Even a well-structured, government-maintained source like EDGAR has structural inconsistencies that have to be worked around rather than assumed away.

**The Resulting Rule**

P/S remains the primary multiple by default, with P/E as the alternative. The tradeoff: this reintroduces P/E's known sensitivity to cost-cutting-driven margin expansion, accepted here as the lesser risk compared to arbitrarily reconstructing or summing fragmented revenue tags to force a P/S figure that may not be reliable.

### Indifferent to Valuation Level
This study does not judge whether a multiple is objectively "high" or "low." A stock at 100x isn't automatically expensive, and a stock at 10x isn't automatically cheap; lighter names routinely carry exaggerated multiples that mean little in isolation. What matters is direction: a multiple moving from 100x to 90x is still contractionary, regardless of how rich 90x still looks. The study measures behavior, not valuation level.

### TTM basis and pivot structure
All valuation multiples are computed on a trailing-twelve-month ("TTM") basis, giving a consistent 12-month period regardless of when in the fiscal year a filing lands, and avoiding the seasonality noise that a simple quarter-over-quarter comparison would introduce. TTM fundamentals update only at each filing date; between filings, TTM values are held constant while price continues to move, meaning the multiple itself changes daily even though its denominator only updates quarterly. Multiple expansion or contraction is measured strictly at filing dates rather than continuously in real time. This anchors every measured change to an actual confirmed fundamental update rather than to routine day-to-day price movement, avoiding the circularity risk of a purely price-driven signal.

### Why Not Analyst Forecasts
The obvious way to measure "forward-looking" is analyst estimates, forward EPS and P/E. But these update slowly, aren't always available, and reflect one analyst's opinion rather than the market's. Instead, this study measures the market's forward expectation by watching how a company's valuation multiple changes over time. When a multiple expands, the market is paying more for the same earnings (a sign it expects better things ahead). When it contracts, the opposite. This study makes no claim about why a multiple moved. This avoids guessing at market psychology and stays grounded in observable behavior.

### Why Not Technicals
Technical indicators (moving averages, oscillators, chart patterns) are commonly used to time entries and exits, but this study deliberately excludes them as a predictor of forward returns. Technicals are derived from price itself, so using them to predict future price risks measuring the same performance twice, once as the signal and another as the outcome being tested. This exclusion is by design, not an oversight: the study is built to test whether FF independently predict returns, and mixing in a price-derived predictor would blur that answer.

### Pivot: Filing, Not Release Date
The pivot point for measuring multiple expansion/contraction is the 10-Q/10-K filing dates, not earlier earnings releases (typically disclosed via 8-K). On the filing date, the reported earnings figure and the stock price are both real, and verifiable. This is a deliberate tradeoff, not an oversight: this study does not attempt to capture the earliest possible market reaction to an earnings surprise. It measures the market's fundamentals-driven repricing using confirmed numbers, accepting a lag relative to when the market may have already partially "priced in" expectations from an earlier release.

### Handling Recently Listed or Restructured Entities
Some tickers correspond to companies that recently underwent legal restructuring or spun off as new public entities (e.g. FedEx Freight, Honeywell Aerospace in 2026). In these cases, the current EDGAR history become too short to support the time-series methods this study requires, even though the underlying business has a long operating history. These entities are excluded from the study rather than patched with a predecessor company's filing history. Using an older, already legally distinct entity's data to fill the gap would misrepresent the current filer's actual track record. This exclusion is applied consistently even where it removes a heavyweight name (e.g. XOM) from a peer group, the study prioritizes correctness of the current filing entity over completeness of the universe.

## NOTEBOOK

.....

## RESULTS AND CONCLUSION

.....
