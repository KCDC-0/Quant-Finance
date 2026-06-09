# Hypotheses worth testing

Momentum Hypotheses
- momentum persists more in low-volatility stocks
- medium-term momentum stronger than short-term
- momentum weakens after extreme gaps

Mean Reversion Hypotheses
- short-term reversals stronger after volume spikes
- reversals stronger in volatile regimes
- intraday overextensions revert

Volume Hypotheses
- abnormal volume predicts continuation
- low liquidity predicts delayed adjustment
- volume-price divergence predicts reversal

Volatility Hypotheses
- volatility compression predicts breakout
- expanding volatility predicts instability
- high volatility reduces momentum persistence


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

```
rank(group_zscore(mdl77_altmanz, subindustry))
```

```
smoothed_buys = rank(snt1_d1_buyrecpercent);
coverage_weight = rank(group_zscore(snt1_d1_analystcoverage, subindustry));

reversion_trigger = rank(-ts_delta(close, 3) / ts_std_dev(close, 10));
combined_alpha = (smoothed_buys * coverage_weight) + (0.3 * reversion_trigger);

normalized_vector = group_zscore(combined_alpha, subindustry);
rank(group_neutralize(ts_decay_linear(min(max(normalized_vector, -2.0), 2.0), 5), subindustry))
```

```
raw_mom = -rank(rank(ts_rank(close, 10)) + rank(close / ts_delay(close, 10)));
raw_mom * rank(ts_delta(close, 40) / ts_std_dev(returns, 40))
```

```
(adv20 < volume) ? ((-1 * ts_rank(abs(ts_delta(close, 3)), 60)) * sign(ts_delta(close, 3))) : (-1)
```

```
ts_corr(vwap, close, 7)
```

```
ts_decay_linear(rank((vwap - close) / close), 5)
```

```
-ts_zscore(enterprise_value/ebitda,63)
```