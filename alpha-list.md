# Passed Alphas

### Price Volume:

USA, TOP3000, Decay 20, Delay 1, Truncation 0.01, Neutralization Subindustry
```
-ts_delta(close, 4)
```


USA, TOP3000, Decay 4, Delay 1, Truncation 0.1, Neutralization Subindustry
```
-ts_delta(close, 5) / ts_delay(close, 5)
```


USA, TOP3000, Decay 4, Delay 1, Truncation 0.05, Neutralization Subindustry
```
close_signal = -(close - low) / (high - low);
volume_signal = volume / ts_mean(volume, 20);
smoothed_raw = ts_decay_linear(rank(close_signal) * rank(volume_signal), 40);

rank(group_neutralize(smoothed_raw, subindustry))
```
> Main hypothesis:  mean reversal alpha, identified when the asset closes at the bottom of its daily trading range could be effective in identifying short term directional changes
>
> Testing impovements: 
> - verifying the mean reversal signal through the closing price with a significant spike in institutional volume (volume signal) to justify that the price drop is short term, rather than a permanent change
> - inceasing linear decay lookback window to control the daily portfolio turnover
> - using a relatively smaller lookback window for volume comparions (20 days) to identify consistent relatively fast volume growth
> - using lower trucation values of 0.05 - 0.08 rather than >= 0.1 to ensure that weight concentration does not spike dring market shocks which may cause outliers between the closing price and the trading range


USA, TOP2000, Decay 0, Delay 1, Truncation 0.08, Neutralization Sector (unsubmitted, too high correlation)
```
slow_anchor = rank(ts_rank(high, 30));

fast_trigger = rank(-ts_delta(close, 2) / ts_std_dev(close, 5));

combined_alpha = (slow_anchor * 0.13) + (fast_trigger * 0.87);

clamped_output = min(max(group_zscore(combined_alpha, subindustry), -5), 5);
rank(group_neutralize(ts_decay_linear(clamped_output, 5), sector))
```
> Main hypothesis:  a mean reversal alpha using short-term 5-day directional price can be paired with a slower price anchor to create an alpha less susceptible to fluctuations
>
> Testing impovements: 
> - providing greater weight to the fast reversal trigger (0.87) than the slow anchor so that the reflected weights prioritise the price reversal aspect of the alpha
> - the mean reversion engine has a much lower lookback window as compared to the long term price anchor (5 vs 30), so that the alpha can identify short-term price reversions while still being affected by a slow anchor, providing portfolio stability
> - using a linear decay on the raw values to smooth the raw data and reduce turnover, protecting the weights from sudden, single-day momentum shifts


USA, TOP3000, Decay 0, Delay 1, Truncation 0.01, Neutralization Market
```
rank(ts_decay_linear(ts_corr(vwap, close, 7) * 0.6 + 0.4 * rank((vwap - close) / close), 17))
```

> Main hypothesis:  Pairing the short-term rolling 7-day time-series correlation between VWAP and closing prices with an intraday basis spread ratio, could create a model that exploits temporary liquidity-driven price dislocations
>
> Testing impovements: 
> - using the spread between vwap and closing prices to capture late-day order-book imbalances
> - finding that the ratio of 60% to 40% of the correlation model to the ratio model produced the highest overall sharpe
> - using a 17-day linear decay window to dampen the high-frequency trading friction and reduce turnover
> - finding that using market group neutralisation produced the highest fitness values


<br>

### Fundamental data:

USA, TOP3000, Decay 4, Delay 1, Truncation 0.08, Neutralization Subindustry
```
ts_rank(operating_income/cap,250)
```


USA, TOP1000, Decay 0, Delay 1, Truncation 0.1, Neutralization Subindustry
```
ts_rank(operating_income/cap,252)
```


USA, TOP3000, Decay 0, Delay 1, Truncation 0.1, Neutralization Subindustry
```
book_to_market = book_value_per_share_avg / close;
relative_value = rank(group_zscore(book_to_market, subindustry));
rank(group_neutralize(ts_decay_linear(min(max(relative_value, 0.02), 0.98), 5), subindustry))
```


USA, TOP3000, Decay 20, Delay 1, Truncation 0.08, Neutralization Subindustry
```
over_leveraged = group_zscore(liabilities / assets, industry);
fcf_decay = ts_mean(free_cash_flow_per_share, 50) - ts_mean(free_cash_flow_per_share, 130);
raw_short_signal = rank(over_leveraged) * rank(-fcf_decay);

rank(group_neutralize(ts_decay_linear(min(max(raw_short_signal, 0.05), 0.95), 50), subindustry))
```


USA, TOP3000, Decay 0, Delay 1, Truncation 0.08, Neutralization Subindustry

```
raw = -ts_rank(fn_liab_fair_val_l1_a, 100);
ts_decay_linear(raw, 10)
```

> Main hypothesis: Sell stocks when there is an increase in the fair value of liabilities, which serves as an indicator for lower profitability 
> 
> Testing impovements: 
> - ranking the stocks based on an observation period of 100 days instead of a year to allow for more accurate financial health prediction
> - use of a decay horizon to smoothly distribute weight transitions, reducing turnover


USA, TOP3000, Decay 50, Delay 1, Truncation 0.2, Neutralization Sector
```
-ts_corr(fnd6_fopo, debt_lt, 23)
```

> Main hypothesis: Companies that accumulate high funds from operations compared to their long term debt, can be considered sustainable in the future
> 
> Testing impovements: 
> - using a time series correlation of FFO to long-term debt instead of a ratio, to identify increasing financial health overtime
> - using a lookback period of just short of a month (23 days) to match the speed at which companies accumulate funds, to create a more accurate signal
> - using higher truncation values to improve sharpe while maintaining turnover, thus improving fitness in the process
> - use of a long decay horizon of 50 days to reduce instances of overly high weight concentrations


USA, TOP3000, Decay 4, Delay 1, Truncation 0.08, Neutralization Sector
```
cash_quality = ts_decay_linear(ts_scale(est_cashflow_op,252),4)-ts_decay_linear(ts_scale(est_capex,252),4);
inventory_quality = ts_decay_linear(inventory/sales, 100);
rank(cash_quality * inventory_quality)
```

> Main hypothesis: Companies with persistently high estimated operating cash flow relative to their capital expenditure are expected to outperform
> 
> Testing impovements: 
> - using the change in the inventory-to-sales ratio to identify companies experience a dramatic improvement in inventory turnover, amplifying the signal
> - using sector neutralisation instead of industry, to improve sharpe as inventory of a company can differ greatly between sectors
> - using a long decay window of 100 days on the inventory sales ratio to isolate the top tier of improvers over a longer period
> - slightly smoothing the free cash flow signal with a linear decay to reduce turnover, but not so much as to minimise reduction on returns and sharpe


USA, TOP3000, Decay 0, Delay 1, Truncation 0.08, Neutralization Industry
```
group_rank(-ts_zscore(enterprise_value/cashflow_op, 63),industry)
```

> Main hypothesis: A lower EV/CF usually suggests the company is becoming cheaper relative to its cash-generating ability; a higher multiple suggests it’s getting more expensive
> 
> Testing impovements: 
> - using Net Cash Flow in operating activities rather than total cashflow to improve sharpe
> - using industry neutralisation instead of subindustry, as fundamentals of a company can affect stock price in a different way depending on the industry
> - using the z score over a lookback period of 2 months to control the turnover
> - considered increasing decay to 4 days, resutling in a increase in fitness - however there was also a non-negligible decrease in sharpe


USA, TOP3000, Decay 10, Delay 1, Truncation 0.08, Neutralization Sector
```
group_rank(-ts_zscore(mdl177_deepvaluefactor_cashsev_alt/fnd6_newa1v1300_dv, 17),industry)
```

> Main hypothesis: A lower EV/CF usually suggests the company is becoming cheaper relative to its cash-generating ability; a higher multiple suggests it’s getting more expensive
> 
> Testing impovements: 
> - This model is a fork of the above a submission, with the goal of using more refined datasets and experimenting with other neutralisations
> - The raw interaction divides a proprietary deep-value cash security factor (mdl177_deepvaluefactor_cashsev_alt) by an enterprise asset baseline volume (fnd6_newa1v1300_dv), isolating a highly refined enterprise-to-cash yield
> - a shorter lookback period of 17 days in the zscore showed significant improvement in identifying intense institutional pricing inefficiencies
> - to control the turnover, a decay of 10 days is used, this also helped to boost the fitnesss
> - neutralising the data by sector also showed slight improvement over industry, ultimaytely corporate cash generation structures differ wildly across macro segments, supporting the idea that sector, industry and market neutralisations perform better than subindustry


USA, TOP3000, Decay 4, Delay 1, Truncation 0.08, Neutralization Subindustry (unsubmitted, too high correlation)
```
cum_rel_return = (1+ts_delay(rel_ret_all,4))*(1+ts_delay(rel_ret_all,3))*(1+ts_delay(rel_ret_all,2))*(1+ts_delay(rel_ret_all,1))*(1+rel_ret_all);
cum_return = (1+ts_delay(returns,4))*(1+ts_delay(returns,3))*(1+ts_delay(returns,2))*(1+ts_delay(returns,1))*(1+returns);
cum_rel_return - cum_return
```

> Main hypothesis: If peers have done much better than the stock, the stock may be a short-term laggard that could mean-revert up
> 
> Testing impovements: 
> - neutralising the data by subindustry instead of sector showed improvement in sharpe, indicating that the magnitude of the difference in returns between peers of a given stock, differs across subcategories
> - extending the decay period to 4 days to reduce turnover, hence improving fitness, while maintaining high sharpe




<br>

### News:

USA, TOP3000, Decay 20, Delay 1, Truncation 0.1, Neutralization Subindustry
```
avg_news = vec_avg(nws12_afterhsz_sl);
rank((2/3) * ts_sum(avg_news, 30) + (2/3) * ts_sum(avg_news, 60)) > 0.5 ? 1 : rank(-ts_delta(close, 2))
```


USA, TOP3000, Decay 20, Delay 1, Truncation 0.15, Neutralization Subindustry
```
slope = ts_regression(ts_backfill(news_pct_1min, 20), ts_step(1), 2, rettype=2);
winsorize(-ts_backfill(news_max_up_ret, 20) * abs(slope),std=4)
```

> Main hypothesis:  When the multi-day slope of first-minute reactions is deteriorating but a large up-spike occurs today, it flags potential trap.
>
> Testing impovements: 
> - reducing the backfill lookback window to 20 days to prevent older news from replacing NaN values, when they may be irrelevant
> - having a high decay lookback period (20 days) to reduce magnitude of weights placed on the stocks, reducing turnover


USA, TOP3000, Decay 0, Delay 1, Truncation 0.08, Neutralization Subindustry
```
ts_regression(ts_sum(ts_backfill(fnd6_newqv1300_ivltq,30),300),ts_step(1),756,rettype = 2) * group_rank(ts_rank(est_eps/close, 90),industry)
```

> Main hypothesis:  The firms that invest more in the long-term may get more profit in the future than those who do not
>
> Testing impovements: 
> - reducing the backfill lookback window to 30 days to prevent older news from replacing NaN values, when they may be irrelevant in recent times
> - multiplying the main news weight with an earnings yield momentum model, adding more weight to firms that also have recently increasing earnings in the past quarter - preventing weight from being placed on firms that carelessly invest more
> - removing the decay to increase turnover, allowing more weight to be places on the stocks which in turn also increases sharpe


<br>

### Volatility:

USA, TOP3000, Decay 10, Delay 1, Truncation 0.05, Neutralization Subindustry
```
long_regime = rank(ts_decay_linear(implied_volatility_call_270 - implied_volatility_put_270, 40));
short_regime = rank(-ts_delta(close, 2) / ts_std_dev(close, 10));
raw_signal = (long_regime * 0.4) + (short_regime * 0.6);
smoothed = min(max(raw_signal, 0.05), 0.95);
rank(group_neutralize(smoothed, subindustry))
```


USA, TOP2000, Decay 0, Delay 1, Truncation 0.08, Neutralization Subindustry
```
raw = (implied_volatility_call_120 - implied_volatility_put_120)/implied_volatility_mean_120;
rank(ts_decay_linear(ts_backfill(raw, 10), 20))
```


USA, TOP3000, Decay 20, Delay 1, Truncation 0.1, Neutralization Subindustry
```

raw = rank(liabilities/assets);
rank(group_neutralize(raw * rank((high + low)/2 - close), industry))
```

<br>

### Analyst factor model:

USA, TOP3000, Decay 0, Delay 20, Truncation 0.1, Neutralization Subindustry
```
main_signal = ts_decay_linear(rank(mdl177_garpanalystmodel_qgp_relgrowth), 10);
price_catalyst = rank(-ts_delta(close, 3) / ts_std_dev(returns, 20));
raw = min(max(main_signal * price_catalyst, 0.02), 0.98);

rank(group_neutralize(raw, subindustry))
```


USA, TOP3000, Decay 0, Delay 1, Truncation 0.1, Neutralization Subindustry
```
rank(-mdl177_2_5yearrelativevaluefactor_rel5yfwdep)
```


USA, TOP3000, Decay 20, Delay 1, Truncation 0.1, Neutralization Subindustry
```
social_sentiment = ts_decay_linear(rank(snt_social_value), 10);
fundamental_value = rank(-mdl177_2_5yearrelativevaluefactor_rel5yfwdep);
raw_sentiment = (social_sentiment - 0.5) * 2;
raw = fundamental_value + (0.3 * raw_sentiment);
raw_neutral = group_neutralize(raw, subindustry);
rank(min(max(raw_neutral, 0.02), 0.98))
```


USA, TOP3000, Decay 0, Delay 1, Truncation 0.12, Neutralization Subindustry
```
solvency_anchor = rank(group_zscore(mdl77_altmanz, subindustry));

market_dispersion = ts_mean(ts_std_dev(returns, 10), 60);
uncertainty_regime = rank(market_dispersion); 

reversion_engine = rank(ts_mean(-ts_delta(close, 3), 10));

raw = (solvency_anchor * uncertainty_regime) + (reversion_engine * (1.0 - uncertainty_regime));

rank(group_neutralize(min(max(raw, 0.02), 0.98), subindustry))
```
> Main hypothesis:  valuing stable stocks higher in periods of market uncertainty, and going back to mean reversion during calm periods could results in the best of both worlds
>
> Testing impovements: 
> - using the bankruptcy risk indicator (mdl77_altmanz) to identify 'safe' companies that are less vulnerable to economic downturns, especially in times of market uncertainty
> - the reversion engine has a much lower lookback window as compared to the uncertainty regime (10 vs 60), so that the alpha identifies short-term market overreactions during quiet regimes, and long-term stable stocks in times of volatility
> - having a low decay lookback window to allow the alpha to change weights quickly given changing market conditions


USA, TOP3000, Decay 20, Delay 1, Truncation 0.01, Neutralization Subindustry
```
-ts_corr(est_ptp,est_fcf,26)
```

> Main hypothesis:  When analyst price target estimates and free cashflow estimates  have high positive correlation in the past few weeks, it may signal that the market has already fully priced in the cash flow expectations into price targets — leaving little room for further upside
>
> Testing impovements: 
> - reducing the correlation lookback window from a year to less than month to provide a more indicative window to react on the price correction
> - using a low truncation value of 0.01 to prevent overly high weight concentrations, indirectly improving fitness
> - having a high decay lookback period (20 days) to reduce magnitude of weights placed on the stocks, reducing turnover and improving fitness

<br>

### Social media sentiment:

USA, TOP3000, Decay 5, Delay 1, Truncation 0.1, Neutralization Subindustry
```
sent = rank(ts_decay_linear(snt_social_value * snt_social_volume, 60));
short = rank(-ts_delta(close, 2) / ts_std_dev(close, 5));
raw_signal = (sent * 0.3) + (short * 0.7);
rank(group_neutralize(raw_signal, subindustry))
```


USA, TOP3000, Decay 0, Delay 1, Truncation 0.08, Neutralization Subindustry
```
-ts_std_dev(scl12_buzz, 20)
```


USA, TOP3000, Decay 10, Delay 1, Truncation 0.1, Neutralization Subindustry
```
signal = -rank(close / vwap);
rank(ts_decay_linear(snt_social_value, 100)) > 0.9 ? 0 : signal
```

