# Alpha testing parameters

## Regions
United States (USA) and China (CHN), etc.

## Universe
Subset of stocks ranked by liquidity, smaller numbers are more liquid

TOP3000, TOP2000, TOP1000, TOP500, TOP200, TOPSP500

## Decay
Linearly decays data over specified number of days to smooth signal

## Delay
Data delay

Delay = 1 alphas trade in the morning using data from yesterday

Delay = 0 alphas trade in the evening using data from today

## Truncation
Maximum daily weight of each stock. USe to avoid over-weighting one company

## Neutralization
Adjust alpha weights to sum to zero within each group of the selected type

Market, Sector, Industry, Subindustry, or None

## Pasteurization
whether to include stocks not in the universe in calculations or leave them as NaN

## Other features
Unit Handling (warns about mismatched data fields before running), NaN Handling (replaces missing values with 0)


<br>
<br>
<br>

# Alpha performance metrics

## Sharpe
Average measure of risk-adjusted returns

> Sharpe = Average Annualized Returns / Annualized Std Dev of Returns

Cutoff: >1.25 (Universe dependent)

Best pratice: >1.50

Improve: Information Content / Delay Reduction


## Sub-universe sharpe

Alpha’s performance in the immediate, smaller set of tradable stocks, or sub universe

Cutoff: >0.50 to >0.80

Best pratice: Match Main Sharpe

Improve: Granular group neutralization (subindustry)


## Turnover
Measure of daily trading activity

> Turnover = Value Traded / Value Held

Cutoff: 1.0%−70.0%

Best pratice: 15.0%−35.0%

Improve: Temporal Smoothing (eg: ts_decay_linear)


## Fitness
Hybrid metric for overall performance 

> Fitness = Sharpe * sqrt(abs(Returns) / Max(Turnover,0.125))

Cutoff: >1.00

Best pratice: >1.30

Improve: Low Turnover + High Sharpe


## Returns
Annualized average gain or loss as a fraction of the invested amount.

> Invested amount is equal to half the book size


## Drawdown
Largest reduction in PnL during a given period, as a percentage.

Best practice: return-to-drawdown ratio greater than one

Improve: Filtering and risk management


## Margin
Average gain or loss per dollar traded

> PnL divided by total dollars traded in a given time period

Cutoff: >0.00

Best pratice: Higher is better

Improve: Volatility scaling (eg: / ts_std)


## Weight
Measure if alpha weight is evenly distributed across stocks, not concentrated on few

Cutoff: <10.0% for each single stock

Best pratice: <5.0%

Improve: Uniform distribution via outer rank()


<br>
<br>
<br>

# Alpha creation
Layer 1: Raw predictive edge

Layer 2: Cross-sectional & Temporal filtering (smoothing and scaling)

Layer 3: Risk management & Scaling (neutralization)



<br>
<br>
<br>

# Alpha modification

## Tweaking ideas
- Change one variable at a time
- Experiment with lookback windows (5, 10, 20, 60)
- Add smoothing using decay_linear
- Apply rank or zscore transformations
- Neutralize by sector or industry
- Combine time-series and cross-sectional effects

## Tweaking operators
- lower turnover: linear decay smoothing or increase lookback windows
- lower weight concentrtion: use lower truncation values or clamp on outliers (normalize, rank, log, zscore), use min max to remove outliers
- improve sub-universe sharpe: normalize the signal magnitude by group volatility


## Expansion ideas
- Different Universes
- Different Neutralizations
- Different Position Distribution through Operators
- Combining Alphas
- Conditionals and filters
- Hard binary transitions vs weighted continuous blends