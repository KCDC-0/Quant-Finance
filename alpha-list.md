# Passed Alphas

USA, TOP3000, Decay 20, Delay 1, Truncation 0.01, Neutralization Subindustry
```
-ts_delta(close, 4)
```

USA, TOP3000, Decay 4, Delay 1, Truncation 0.08, Neutralization Subindustry
```
ts_rank(operating_income/cap,250)
```

USA, TOP1000, Decay 0, Delay 1, Truncation 0.1, Neutralization Subindustry
```
ts_rank(operating_income/cap,252)
```

USA, TOP3000, Decay 4, Delay 1, Truncation 0.1, Neutralization Subindustry
```
-ts_delta(close, 5) / ts_delay(close, 5)
```

USA, TOP3000, Decay 20, Delay 1, Truncation 0.1, Neutralization Subindustry
```
avg_news = vec_avg(nws12_afterhsz_sl);
rank((2/3) * ts_sum(avg_news, 30) + (2/3) * ts_sum(avg_news, 60)) > 0.5 ? 1 : rank(-ts_delta(close, 2))
```

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


# Alpha ideas


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