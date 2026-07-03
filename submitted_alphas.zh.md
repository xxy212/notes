# WorldQuant Alphas

## 术语

### Alpha 测试参数

地区包括美国（USA）和中国（CHN）。

Universe —— 按流动性排序后的股票子集。数值越小，流动性越强。
  * 包括 TOP3000、TOP2000、TOP1000、TOP500、TOP200。

Decay —— 在指定天数内对数据做线性衰减。有助于平滑信号。

Delay —— 数据延迟。
  * `Delay = 1` 的 alpha 在早上使用昨天的数据交易。
  * `Delay = 0` 的 alpha 在晚上使用当天的数据交易。
  * Delay 0 的 alpha 表现更好，因此提交要求更严格。

Truncation —— 每只股票每日最大权重。有助于避免过度集中在某一家公司。

Neutralization —— 在所选类型的每个分组内调整 alpha 权重，使其和为零。
  * 包括 Market、Sector、Industry、Subindustry 或 None。

### Alpha 表现指标

Sharpe —— 风险调整后收益的平均度量。
  * `Sharpe = 年化平均收益 / 年化收益标准差`。

Turnover —— 每日交易活跃度的平均度量。
  * `Turnover = 交易金额 / 持有金额`。

Fitness —— 总体表现的混合指标。越高越好。
  * `Fitness = Sharpe * Sqrt( Abs( Returns ) / Max( Turnover, 0.125 ) )`。

Returns —— 以投入金额为分母的年化平均盈亏比例。
  * 投入金额等于账面规模的一半。

Drawdown —— 给定期间内 PnL 的最大下降幅度，以百分比表示。

Margin —— 每交易 1 美元带来的平均盈亏。
  * 给定时间段内的 PnL 除以总交易美元金额。

## Alphas

USA, TOP3000, Decay 3, Delay 1, Truncation 0.05, Neutralization None\
Sharpe 1.39, Turnover 17.15%, Fitness 1.11, Returns 10.87%, Drawdown 8.14%, Margin 12.67‱
```
lookback = 10;
avg_ret = power(ts_product(returns+1, lookback), 1/lookback);
comp_avg = power(ts_product(rel_ret_comp+1, lookback), 1/lookback);
a = zscore(comp_avg / avg_ret);
when = ts_rank(ts_std_dev(returns, 60), 126) > 0.55; 
b = trade_when(when, a, -1);
group_vector_neut(b, ts_mean(returns, 120), subindustry)
```

USA, TOP3000, Decay 5, Delay 1, Truncation 0.1, Neutralization Subindustry\
Sharpe 1.94, Turnover 44.52%, Fitness 1.35, Returns 21.66%, Drawdown 11.37%, Margin 9.73‱
```
buzz = ts_backfill(-vec_sum(scl12_alltype_buzzvec), 20);
ts_av_diff(buzz, 60)
```

CHN, TOP3000, Decay 3, Delay 0, Truncation 0.005, Neutralization Market\
Sharpe 5.03, Turnover 13.02%, Fitness 7.52, Returns 29.08%, Drawdown 7.09%, Margin 44.68‱
```
gp = subindustry;
a = rank(-mdl175_volatility * log(volume)) * (1 + group_rank(mdl175_revenuettm,gp));
#vector_neut(a, ts_mean(mdl175_02amvt, 240))
group_vector_neut(a, ts_mean(mdl175_02amvt, 50), gp)
```

USA, TOP3000, Decay 5, Delay 1, Truncation 0.01, Neutralization Subindustry\
Sharpe 1.55, Turnover 3.71%, Fitness 1.18, Returns 7.28%, Drawdown 7.11%, Margin 39.28‱
```
-ts_rank(retained_earnings,500)
```

USA, TOP200, Decay 10, Delay 1, Truncation 0.01, Neutralization Subindustry\
Sharpe 1.62, Turnover 2.21%, Fitness 1.52, Returns 11.06%, Drawdown 11.65%, Margin 100.21‱
```
-rank(ebit/capex)
```

USA, TOP1000, Decay 10, Delay 1, Truncation 0.01, Neutralization Market\
Sharpe 1.38, Turnover 2.00%, Fitness 1.35, Returns 12.04%, Drawdown 13.20%, Margin 120.20‱
```
group_rank(fam_est_eps_rank, sector)
```

USA, TOP3000, Decay 1, Delay 1, Truncation 0.05, Neutralization Market\
Sharpe 1.59, Turnover 2.79%, Fitness 1.70, Returns 14.33%, Drawdown 7.53%, Margin 102.66‱
```
a = ts_av_diff(mdf_eg3, 50);
b = ts_corr(mdf_eg3, mdf_sg3, 50);
-a * b
```

USA, TOP500, Decay 1, Delay 1, Truncation 0.01, Neutralization Subindustry\
Sharpe 1.28, Turnover 1.91%, Fitness 1.15, Returns 10.17%, Drawdown 8.18%, Margin 106.23‱
```
rank(mdf_rds)
```

USA, TOP500, Decay 20, Delay 1, Truncation 0.01, Neutralization Sector\
Sharpe 1.42, Turnover 8.15%, Fitness 1.06, Returns 6.94%, Drawdown 4.22%, Margin 17.03‱
```
oey_growth = ts_rank(mdf_oey, 63);
oey_growth - group_mean(oey_growth, 1, subindustry)
```

USA, TOP1000, Decay 9, Delay 0, Truncation 0.1, Neutralization Subindustry\
Sharpe 2.03, Turnover 14.25%, Fitness 1.58, Returns 8.67%, Drawdown 3.96%, Margin 12.17‱
```
rank( ts_rank( fnd6_epsfx /close, 40) )
```

USA, TOP1000, Decay 10, Delay 1, Truncation 0.1, Neutralization Subindustry\
Sharpe 1.80, Turnover 13.89%, Fitness 1.42, Returns 8.64%, Drawdown 4.48%, Margin 12.44‱
```
rank(ts_rank( ebit/sharesout /close, 40))
```

USA, TOP200, Decay 0, Delay 1, Truncation 0.01, Neutralization Subindustry\
Sharpe 1.37, Turnover 2.50%, Fitness 1.09, Returns 7.89%, Drawdown 7.61%, Margin 63.22‱
```
ts_av_diff(mdf_eg3, 600)*ts_corr(mdf_eg3, mdf_sg3, 600)
```

USA, TOP200, Decay 0, Delay 1, Truncation 0.01, Neutralization Sector\
Sharpe 1.27, Turnover 5.99%, Fitness 1.29, Returns 12.90%, Drawdown 13.40%, Margin 43.07‱
```
zscore(cash_st/debt_st)
```

USA, TOP500, Decay 0, Delay 0, Truncation 0.1, Neutralization Subindustry\
Sharpe 2.02, Turnover 2.36%, Fitness 2.30, Returns 16.15%, Drawdown 6.95%, Margin 136.98‱
```
-rank(ebit/capex)
```

USA, TOP1000, Decay 0, Delay 1, Truncation 0.01, Neutralization Sector\
Sharpe 1.62, Turnover 1.50%, Fitness 1.70, Returns 13.72%, Drawdown 6.95%, Margin 183.03‱
```
-rank(ebit/capex)
```

USA, TOP3000, Decay 0, Delay 1, Truncation 0.01, Neutralization Market\
Sharpe 1.45, Turnover 1.42%, Fitness 1.18, Returns 8.31%, Drawdown 4.89%, Margin 116.89‱
```
fam_roe_rank*rank(sales/assets)
```

USA, TOP3000, Decay 0, Delay 0, Truncation 0.1, Neutralization Subindustry\
Sharpe 2.58, Turnover 26.63%, Fitness 1.70, Returns 11.62%, Drawdown 2.99%, Margin 8.73‱
```
-ts_zscore(enterprise_value/ebitda,63)
```

USA, TOP3000, Decay 30, Delay 1, Truncation 0.01, Neutralization Subindustry\
Sharpe 1.84, Turnover 8.99%, Fitness 1.08, Returns 4.29%, Drawdown 1.93%, Margin 9.55‱
```
avg_news = vec_avg(nws12_afterhsz_sl);
rank(ts_sum(avg_news, 60)) > 0.5 ? 1 : rank(-ts_delta(close, 2))
```

USA, TOP3000, Decay 0, Delay 1, Truncation 0.08, Neutralization Subindustry\
Sharpe 1.35, Turnover 2.17%, Fitness 1.57, Returns 16.83%, Drawdown 17.79%, Margin 154.75‱
```
ts_av_diff(mdf_nps,500)
```

USA, TOP3000, Decay 8, Delay 1, Truncation 0.01, Neutralization Sector\
Sharpe 1.75, Turnover 24.34%, Fitness 1.19, Returns 11.26%, Drawdown 5.04%, Margin 9.26‱
```
group_zscore( ts_zscore(mdf_gry, 20), sector )
```

USA, TOP3000, Decay 0, Delay 1, Truncation 0.1, Neutralization Subindustry\
Sharpe 1.79, Turnover 14.71%, Fitness 1.24, Returns 7.10%, Drawdown 6.82%, Margin 9.65‱
```
ts_zscore(mdf_oey, 250)
```

USA, TOP3000, Decay 1, Delay 1, Truncation 0.01, Neutralization Industry\
Sharpe 2.00, Turnover 25.32%, Fitness 1.26, Returns 10.10%, Drawdown 3.77%, Margin 7.98‱
```
-ts_zscore(enterprise_value/ebitda,63)
```

USA, TOP500, Decay 0, Delay 1, Truncation 0.01, Neutralization Market\
Sharpe 1.44, Turnover 1.52%, Fitness 1.36, Returns 11.10%, Drawdown 12.14%, Margin 145.75‱
```
-Last_Diff_Value(sales, lookback=125) * rank(vwap)
```

USA, TOP3000, Decay 20, Delay 1, Truncation 0.01, Neutralization Market\
Sharpe 1.25, Turnover 18.36%, Fitness 1.10, Returns 14.27%, Drawdown 11.55%, Margin 15.54‱
```
-delta(close, 5)
```

USA, TOP3000, Decay 10, Delay 1, Truncation 0.01, Neutralization Market\
Sharpe 1.80, Turnover 51.95%, Fitness 1.03, Returns 17.13%, Drawdown 8.75%, Margin 6.60‱
```
(high + low)/2 - close
```

USA, TOP200, Decay 10, Delay 1, Truncation 0.05, Neutralization Market\
Sharpe 1.58, Turnover 49.64%, Fitness 1.09, Returns 23.76%, Drawdown 10.30%, Margin 9.57‱
```
decay_days = 1;
rel_days_since_max = rank(ts_arg_max(close, 30));
decline_pct = (vwap - close) / close;
decline_pct / min( ts_decay_linear(rel_days_since_max, decay_days), 0.15)
```

USA, TOP3000, Decay 10, Delay 1, Truncation 0.01, Neutralization None\
Sharpe 1.40, Turnover 9.68%, Fitness 2.86, Returns 52.02%, Drawdown 55.18%, Margin 107.45‱
```
change_day = (vwap - close) / close;
ts_decay_exp_window(rank(change_day), 4)
```

USA, TOP3000, Decay 10, Delay 1, Truncation 0.01, Neutralization Subindustry\
Sharpe 1.92, Turnover 48.92%, Fitness 0.99, Returns 13.01%, Drawdown 8.25%, Margin 5.32‱
```
decay_days = 2;
rel_days_since_max = rank(ts_arg_max(close, 30));
decline_pct = (vwap - close) / close;
decline_pct / min( ts_decay_linear(rel_days_since_max, decay_days), 0.20)
```
