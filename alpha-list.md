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

<br>
<br>
<br>
<br>


# Alpha ideas

List of possible ideas to explore / alphas that are close but do not meet requirements to pass
<br>
<br>


EV/EBITDA ratio:
```
ts_rank(ebitda/enterprise_value, 100)
```

Dividing operating income by cap (OEY):

Short term cash to debt ratio:
```
zscore(cash_st / debt_st)
```

```
group_zscore(cash_st / debt_st, sector)
```

```
asset_group = bucket(rank(operating_income/assets), range=“0.1, 1, 0.1”)
alpha = group_zscore(cap/income, densify(asset_group))
```

```
(high + low)/2 - close
```

```
ts_rank(open - (high + low)/2, 10)
```

```
ts_rank(implied_volatility_mean_120,20)
```

```
iv_difference = rank(implied_volatility_call_120 - implied_volatility_put_120);
cap_group = bucket(rank(cap),range="0.1,1,0.1");
group_neutralize(iv_difference,cap_group)
```

```
rank(implied_volatility_call_270 - implied_volatility_put_270) > 0.5 ? 1 : rank(-ts_delta(close, 2))

```

```
long_regime = rank(ts_decay_linear(implied_volatility_call_270 - implied_volatility_put_270, 40));
short_regime = rank(-ts_delta(close, 2) / ts_std_dev(close, 10));
raw_signal = (long_regime * 0.4) + (short_regime * 0.6);
rank(group_neutralize(raw_signal, subindustry))
```

```
raw = rank(-scl12_buzz);
smoothed = min(max(raw, 0.1), 0.9);
rank(group_neutralize(smoothed * rank((high + low)/2 - close), industry))
```

```
trade_when(0.8 >= scl12_buzz, rank((high + low)/2 - close) , -1)
```

```
-ts_std_dev(scl12_buzz, 10)
```

```
liabilities/assets
```

```
stab = -ts_std_dev(scl12_buzz, 20);
sent = ts_mean(scl12_buzz, 20);

rank(group_neutralize(rank(stab) + rank(sent), subindustry))
```

```
rank(-ts_std_dev(volume, 20))
```

```
grow = ts_delta(ebitda, 252) / ts_delay(assets, 252);
valuation = rank(enterprise_value / sales);
rank(group_neutralize(rank(grow) / (valuation + 0.01), subindustry))
```

```
-rank(close / vwap)
```

```
signal = rank(-ts_delta(close, 2));
rank(ts_decay_linear(snt_social_value, 60)) > 0.8 ? 1 : signal
```

```
social_sentiment = ts_decay_linear(rank(snt_social_value), 10);
fundamental_value = rank(-mdl177_2_5yearrelativevaluefactor_rel5yfwdep);
combined_signal = rank(social_sentiment) * rank(fundamental_value);
rank(group_neutralize(min(max(combined_signal, 0.02), 0.98), subindustry))
```

```
rp_css_ratings
```