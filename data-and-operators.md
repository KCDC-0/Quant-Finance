## Time Series operators

- ts_rank(x,d): Ranks x values for each stock over the past d days and distributes them on a 0-1 scale, similar to rank()
- ts_zscore(x,d): Shows how far today's x is from the d-day average in standard deviation units (Z-score)
- ts_mean(x, d): Returns the average of x values over the past d days
- ts_regression(x, y): Returns various parameters related to regression between x and y

## Cross-Sectional operators

- rank(x): Returns a uniformly distributed value between 0.0 and 1.0 based on ranking among all stocks
- zscore(x): Shows how far an instrument's x value is from the mean in standard deviation units (Z-score)
- winsorize(x, std=4): Limits extreme values so all x values fall between upper and lower bounds set by standard deviation multiples

## Vector operators

Unlike matrix data with one value per day per instrument, vector data can store many values

Thus vector operators are needed to convert vector data to matrix form before applying matrix operators

- vec_avg(x): Returns the mean of all elements in a vector field for each instrument and date, converting vector data to a single matrix value
- vec_sum(x): Returns the sum of all values in a vector field

## Conditional operators

- if_else(condition, if_true, if_false)
- condition? if_true:if_false
- trade_when(entry_condition, Alpha, exit_condition)

## Neutralisation operators

- bucket(x, range="{start},{end},{step}"): Divides data into buckets (ranges) based on ranked values of any data field
- densify(x): Converts a grouping field of many buckets into lesser number of only available buckets
- group_neutralize: Neutralizes Alpha values within each specified group by subtracting the group mean from each value

<br>
<br>

## Options

Option: The right to buy or sell a stock at a specific price at a future date

Call Option: Right to buy (used when expecting price increase)

Put Option: Right to sell (used when expecting price decrease)

Option Premium: Cost to buy the option

- Strike Price: The price agreed to buy or sell in the future
- Breakeven Price: The point where neither profit nor loss occurs
- Expiration: The last date to exercise the right
- Premium: The price of the option
- Volatility: Indicator of stock price volatility
- Historical Volatility: Actual past price volatility
- Implied Volatility: Expected future volatility derived from option prices


## Price-Volume

- close: closing (end) price of the day
- open: opening (start) price of the day
- high: daily high price
- low: daily low price
- volume: daily volume
- vwap: daily volume weighted average price


## Fundamentals
Comes from balance sheet (snapshot of the company's financial health: assets, liabilities, and equity), income statement (profitability) and cash flow statement (liquidity). Features low turnover as it's updated quarterly, semi-annually, or annually


- ebit: Earnings before interests and taxes
- ebitda: Earnings Before interest, taxes, depreciation, and amortization
- capex: Capital expenditures
- assets: total assets of a company
- retained_earnings: Retained Earnings


## Sentiment

Pre session: 4:00 am to 9:30 am

Main session: 9:30 am to 4:00 pm

Post session: 4:00 pm to 8:00 pm

News is often released in pre and post sessions

- scl12_buzz: Amount of sentiment (frequency of mentions on social media)
- nws12_afterhsz_sl: whether the overall news sentiment for a company is positive or negative
