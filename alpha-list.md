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

<br>

### News:

USA, TOP3000, Decay 20, Delay 1, Truncation 0.1, Neutralization Subindustry
```
avg_news = vec_avg(nws12_afterhsz_sl);
rank((2/3) * ts_sum(avg_news, 30) + (2/3) * ts_sum(avg_news, 60)) > 0.5 ? 1 : rank(-ts_delta(close, 2))
```

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

