# WorldQuant 研讨会

## 价格成交量 Alpha

### 算子

算子是构建模块。
  * 每个算子都有详细解释 / 数学定义。
  * 加、减、乘、除——注意单位！
  * if/else、三元表达式。
  * 逻辑 and、or、not。
  * 小于/大于比较、相等/不等。

横截面算子是在某一时点，对股票池中的所有股票执行操作。
  * `rank`、`zscore`、`scale`、`sign`。

时间序列算子是在多个时间点上对单只股票执行操作。
  * `tsdelta`、`tsdelay`、`tssum`、`tsmean`、`tsrank`、`tszscore`、`tsstddev`、`tsregression`。

分组算子是更强大的横截面算子。
  * 选择 sector、industry、subindustry 等分组。

时间序列回归：
  * `tsregression(y, x, window, lag, retval)` —— 对过去 window 天中的 `t`，拟合 `y(t) = a + b * x(t-lag)`。返回 error、a、b、estimate。

不能做横截面相关性，只能做时间序列相关性。

`tradewhen` 算子在入场和退出条件之间激活。
  * 降低换手率。
  * 适合在高波动时期交易。

`Humpdecay` —— 根据变化是否超过 hump 值，返回今天或昨天的价格。
  * 降低换手率。

较好的参数值：
  * Truncation value = 0.01，用于分散化。
  * Reversion threshold = 0.55，因为研究显示 50% 时间为反转、10% 为动量、40% 为随机。

### 数据

Learn tab --> Datasets 中有所有可用数据集。
  * 可以为 USA 和 China 创建 alpha。

股票价格随需求上升。
  * Open、high、low、close。
  * 总日收益——从第 1 天 close 到第 2 天 close。

Volume（重要！）
  * 日频。
  * 在一定时间范围内取平均，通常约 1 个月，即 20 个交易日。
  * !!!成交量加权平均价 = `sum(price * shares traded at that price / total shares traded that day)`。

Frequency —— 数据更新频率，除了 sharesout、cap、split、dividend。
  * 通常每日更新。

Coverage —— 可用股票数据的覆盖程度。
  * 100% coverage —— 对 universe 中所有股票都有数据。

### 技术指标

技术指标更适合识别条件，而不是直接作为买/卖信号。
  * 例如：超买、超卖、交易、趋势。

RSI 衡量价格变化幅度，表示超买/超卖。
  * 公式可在线查。Brain 平台中必须手动实现。
  * 范围从 0（超卖）到 100（超买）。

### 思路

动量 / 趋势跟随：
  * 买入上涨股票，做空下跌股票。

趋势反转：
  * 做空超买股票，买入便宜股票。
  * 在多空框架中有用。
  * 在极端价格运动（过度反应）期间效果较好。
    * 可考虑用标准差衡量极端价格变化。
    * 在高波动时期交易。

尝试 alpha：`-(close - ts_mean(close, 5))`
  * 回归到周度价格，并进行横截面比较。
  * 使用绝对差异，因此高价格股票被不公平地赋予更高权重。
  * 多空中性化用于平衡多头和空头。
  * 用 `rank` 包裹 alpha 获得百分位？有助于改善权重。
    * 这是提高分散性和表现的好技巧。

降低换手率 = 每日组合交易比例，约等于交易成本！
  * 增加 alpha decay。
  * 组合多个算子。
    * 也会降低相关性。

尝试 alpha：`vwap / close`
  * 回归到成交量加权平均价，并做横截面比较。
  * vwap 代表“公允价格”，所以次日应发生回归。

尝试 alpha：`group_zscore( -(close - open) / (high - low), subindustry )`
  * 简单的 K 线 alpha。
  * 如果收盘低于开盘则买入，并按整体振幅缩放。

尝试 alpha：`group_rank( ts_zscore(vwap/close, 250), subindustry )`
  * zscore 类似 rank，但分布是正态而非均匀。
  * 在子行业内比较不同股票，选择远离自身时间均价的股票。

尝试 alpha：`-tsregression( returns, ts_delay(returns,1), 252 )`
  * 使用回归残差作为权重，因此价格应向回归预测值回归。

尝试 alpha：
```
trade_when( ts_rank( ts_std_dev(returns, 22), 252 ) > 0.55, 
            -ts_regression(returns, ts_delay(returns,1), 252), -1
          )
```
  * 效果非常好！
  * 在波动率高时交易回归反转思路。
  * 否则使用前一天的值。
  * 用 Delay 防止高换手。

查看 Wilmott Magazine 的研究论文 “101 Formulaic Alphas”。
  * 改进实现方式，找到 6-8 个可提交 alpha！
  * 理解它们在真实市场中如何运作。

IncredibleCharts.com 上有指标。
  * Bollinger bands 等。

### 作业

尝试简单移动平均反转。
  * `signal = ts_mean(close, 20) / ts_mean(close, 5);`
  * `signed_power(ts_zscore(signal, 480), 2)`

尝试 Bollinger Bands。
  * 检查收盘价是否低于下轨 / 高于上轨。
  * `window = 20; average = (high + low + close + open) / 4;`
  * `MA = ts_mean(average, window);`
  * `BOLU = MA + ts_std_dev(average, window);`
  * `BOLD = MA - ts_std_dev(average, window);`
  * `signal = ts_sum(close < BOLD, window) - ts_sum(close > BOLU, window);`
  * `ts_zscore(signal, 240)`

尝试 Stochastic Oscillator。
  * `window = 10; lo = ts_min(low, window); hi = ts_max(high_window);`
  * `k = (close - lo) / (hi - lo); d = ts_decay_exp_window(k, 3, factor=0.5);`
  * `-d`

通过阅读数据集描述产生想法。
  * 例如，Fundamental 数据可能表示价值。使用比率。

用多样化数据集降低自相关。

## 基本面数据集

数据来自资产负债表、利润表和现金流量表。
  * 利润表显示基本盈利和成本。
  * 资产负债表显示股东权益（账面价值）。
  * 现金流量表显示公司再投资和流动性。
  * 按季度更新，取决于交易所要求。
  * 有助于理解业务和未来前景。

创建财务画像，识别内在价值，帮助提出建议。
  * 洞察商业决策。
  * 关注关键价值驱动因素，如现金流和 EPS。
  * 横截面比较和时间序列分析。
    * 使用财务比率。
  * 价格成交量 + 基本面最好。
    * 基于财务或经济思想组合。

用时间序列算子和 PV 数据比率可以提高换手率。

用 `keep` 和 `tradewhen` 降低换手率。

例子：
  * `ts_rank(equity / cap, 20)` —— 类似账面/市值比。
    * TOP3000，neutralization industry，decay 5，truncation 0.1。
    * Sharpe 1.6，turnover 25%，fitness 0.87，returns 7%。
    * 尝试 `ts_rank` 之外的算子，使用分组。
  * `-ts_zscore(enterprise_value / ebitda, 63)` —— 做空估值上升。
    * TOP3000，neutralization industry，decay 5，truncation 0.01。
    * Sharpe 1.7，turnover 15%，fitness 1.08，returns 7%。

Model data 包含来自专有数据集的预测基本面、价格和比率数据。
  * 包括日频、季度和年度数据。

例子：
  * `ts_rank(mdf_oey, 250)` —— 做多经营收益率上升。
    * TOP3000，neutralization industry，decay 0，truncation 0.01。
    * Sharpe 1.9，turnover 15%，fitness 1.14，returns 5.5%。
  * `group_zscore( ts_zscore(mdf_gry, 20), sector )` —— 做多增长率和股息率上升。
    * TOP3000，neutralization industry，decay 0，truncation 0.1。
    * Sharpe 1.9，turnover 33%，fitness 0.86，returns 7%。
    * 通过增加 decay 降低换手率。

查看 Financial Ratio 视频、“The Power of Firm Fundamentals in Explaining Stock Returns”（abstract ID 3090626）、“Earnings and Price Momentum”（abstract ID 342581）。

## 创建可提交 Alpha

Documentation 页面新增了 19 个 alpha 示例。

上传头像，让排行榜看起来更有参与感。

同一思路，不同数据字段——尝试所有可能组合。
  * 在 TOP3000、neutralization subindustry、0 decay、0.08 truncation、1 delay 下尝试 `ts_rank(operating_income/cap, 252)` 和 `ts_rank(mdf_oey, 252)`。
  * 搜索所有数据集。

同一思路，不同算子。
  * `ts_rank(mdf_oey, 252)` 和 `ts_quantile(mdf_oey, 252)`。
  * 可以先加横截面的分组 rank，再做时间序列 rank，例如：

切换设置，尤其是 decay 和 neutralization。
  * `ts_av_diff(mdf_nps, 500)`，neutralization market 或 subindustry，TOP3000，decay 0，truncation 0.08，delay 1。
  * Subindustry 通常表现好。

新闻数据——财务报告、分析师建议、公司产品、法律事件。
  * 影响可持续数天到数个季度。
  * 与股票价格数据结合。
  * 必须使用 `vec_avg` 或类似算子。
  * 3 个交易时段——盘前（4-9:30）、主交易（9:30-4）、盘后（4-8）。
    * 新闻通常在盘前或盘后发布，这会导致主交易时段出现跳跃。
    * Delay 是新闻数据的重要因素——尝试 0。
  * `rank( ts_sum( vec_avg(nws12_afterhsz_sl), 60 ) ) > 0.5 ? 1 : rank( -ts_delta(close, 2) )*1`，在 TOP3000 上，neutralization subindustry，decay 3，truncation 0.01。
  * 换手率通常很高。

情绪数据——来自新闻网站、社交媒体和论坛的 aggregated buzz/mood。
  * 帮助企业理解消费者观点。
  * 必须使用 `vec_avg` 或类似算子。
  * `-ts_regression(returns, ts_delay(snt_buzz_ret, 1), 120)`，在 TOP3000 上，neutralization subindustry，decay 15，max weight 0.1。`snt_buzz_ret` 使用 buzz 的幅度预测收益。

## 问答

查看论坛文章。
  * Seven Tips for Creating Delay 0 Alphas。
  * 6 ways to evaluate a dataset。

上次锦标赛雇佣了 6 名实习生和 5 名全职员工。

中国市场入门——涨跌停制度阻止把策略直接取反。
  * CHN 样本内表现很高。

只有一部分 alpha 在 delay-1 和 delay-0 下都表现良好。

使用高频数据，如价格/成交量、新闻、情绪和期权数据，创建 delay-0 alpha。在流动性好的 universe 上交易。

`zscore(vwap / close) * (1 - rank(high / low))`
  * Neutralization market，truncation 0.01，decay 15，TOP1000，delay 0。
  * 可用更流动的 universe 改进。

`zscore(implied_volatility_call_720 - implied_volatility_put_720)`
  * Neutralization market，truncation 0.01，decay 3，TOP3000，delay 0。
  * 可结合历史波动率、交易量和未平仓量改进。

`-ts_corr(ts_rank(volume, 10), ts_rank(vwap, 10), 20)`
  * Neutralization subindustry，truncation 0.1，decay 3，TOP3000 China，delay 0。
  * 当相关性低于阈值时，可通过降低噪声改进。
  * 中国市场短期趋势环境中通常存在价格反转。

`when = ts_arg_max(volume, 5) == 0; trade_when(when, -rank(ts_delta(close, 3)), -1)`
  * Neutralization subindustry，truncation 0.1，decay 3，TOP3000 China，delay 1。
  * 当股票触及中国涨跌停（10%）时退出，可进一步改进。

`rank(ts_delta(retained_earnings / sharesout, 90))`
  * Neutralization subindustry，truncation 0.01，decay 5，TOP3000 China，delay 1。
  * 可通过降低 `ts_delta` 的天数改进。

TOP3000 的 Truncation 设为 0.01。
  * 用 `rank` 防止过度加权。

## Consultant/Researcher 问答

主持人：Nicole Goldstein。

You-Lin Wu 在台湾读金融本科。现在是研究员。
  * 用 Brain 做免费研究，并把课堂理论用于实践。
  * 赢得台湾区域赛，去新加坡参赛。
  * 日常工作：类似 IQC，但有更多工具和数据。
  * 提示：重要的是复盘 alpha，并思考改进。
  * 建议：提交很多 alpha。

Yash Zanwar 在印度读本科。现在是研究员。
  * 逐渐学习 Brain，然后和朋友一起参加 IQC。
  * 赢得印度区域赛，去新加坡参赛（和 You-Lin 一起！）。
  * 日常工作：研究、平台改进、社区互动。
  * 提示：想法可以来自任何地方，例如新闻。尝试不同主题。
  * 建议：注意细节。让 alpha 在许多设置下都表现良好。

Jayden Lee 先是 consultant，然后实习生，现在是研究员。
  * 只进入第 2 轮，没有进入后续轮次。
  * 提示：避免过拟合。使用少量参数。分散 alpha 池。
  * Alpha 思路：做多低 beta，做空高 beta（被高估）。用 CAPM 模型计算 beta——更多信息查看 Data Explorer。
  * 建议：避免过拟合，因为你投资的是未来，不是过去。

Yu-Xiang (Simon) Mao 是计算机本科。做过实习和 consultant，现在是研究员。
  * 台湾区域赛第 2 名。
  * 把 alpha 创作看成游戏。什么都试。
  * 建议：演示时准备原创 alpha。

Ishan Shandilya 在本科专业中接触量化金融。
  * 了解 Brain 平台后做了实习。
  * 量化金融很神秘且缺乏协作。Brain 促进互动。
  * 建议：简单带来质量。

Aditya Chaturvedi 成绩不好，后来发现 WorldQuant 竞赛。
  * 获得实习，然后成为 consultant。现在是研究员。
  * 背景是材料科学！
  * 建议：持续提交 alpha。每天都试！

Data Explorer 有用于 alpha generation 的标签——可以直接使用那些数据！

第 2 轮可以重复使用第 1 轮的 alpha。

## 期权、向量算子和中国市场

向量数据每天有多个点（我认为更应称为 tensor）。

股票只指定方向，并使用当前价格。期权指定方向、时间和价格。

标的属性（call 响应）（put 响应）：
  * 价格 `+-`，执行价 `-+`，到期时间 `++`，波动率 `++`，无风险利率 `+-`，股息 `-+`。

IV-vs-strike 图形成 smile。

`ts_delay(call_breakeven_10, 7) / close`
  * 可用其他 call 或 breakeven 字段改进。统计 close 达到 breakeven 的频率。
  * `a = above; trade_when(ts_arg_max(pcr_vol_10, 7) < 1, a, -1)`
  * Neutralization industry，truncation 0.03，decay 0，TOP3000。

`ts_decay_linear(ts_delta(implied_volatility_call_60, 25) > 0, 20)`
  * 可用 `vector_neut()` 算子改进。
  * `a = above; vector_neut(a, (cap))`
  * Neutralization subindustry，truncation 0.03，decay 0，TOP3000。
  * 查看 [论文](https://papers.ssrn.com/sol3/paper.cfm?abstract_id=2008902)。
    * 过去一个月 call（put）IV 大幅上升的股票未来收益高（低）。

`vector_neut()` 得到 `a - (a*b)b`（相对 `b` 的正交分量）——在另一个风险因子上中性化 alpha 向量。

`-mdl175_volatility` —— 中国市场的新数据字段。
  * Neutralization market，truncation 0.1，decay 3，TOP3000 China。
  * 可通过在高成交量时期减少对高波动股票的暴露来改进。
  * 也可以提高过去一年高收入股票的权重。
  * `rank(-mdl175_volatility * log(volume)) * (1 + group_rank(mdl175_revenuettm, sector))`
  * `vector_neut(above, ts_mean(mdl175_02amvt, 240))`
  * `group_vector_neut(above, ts_mean(mdl175_02amvt, 240), sector)`

`buzz = ts_backfill(-vec_sum(scl12_alltype_buzzvec), 20); ts_av_diff(buzz, 60)`
  * Neutralization industry/subindustry，truncation 0.08，decay 15，TOP3000。

`ts_backfill(-vec_avg(nws12_afterhsz_maxupamt), 20)`
  * Neutralization industry，truncation 0.01，decay 10，TOP1000。
  * 可通过相对动量中性化来改进。设置 no neutralization 和 decay 0。
  * `a = above; neut_a = vector_neut(a, ts_mean(returns, 250)); decay_a = ts_decay_exp_window(neut_a, 20, factor=0.4); group_neutralize(decay_a, densify((industry+1)*10 + exchange))`

`avg_ret = power(ts_product(returns+1, 5), 1/5); comp_avg = power(ts_product(rel_ret_comp+1, 5), 1/5); a = zscore(comp_avg / avg_ret); when = ts_rank(ts_std_dev(returns, 60), 126) > 0.55; trade_when(when, a, -1)`
  * Neutralization subindustry，truncation 0.01，decay 3，TOP3000。
  * 几何均值对金融数据比算术均值好得多。
    * 可用 `exp(1/N sum(log(x)))` 做数值稳定。
  * 可通过相对动量中性化改进。
  * `vector_neut(above, ts_mean(returns, 120))`

`rank(-(mdf_pbk - ts_max(mdf_pbk, 10)))`
  * Neutralization subindustry，truncation 0.01，decay 3，TOP3000。
  * 可通过降低换手率改进。
  * `rank(ts_sum(vec_avg(nws12_afterhsz_s1), 60)) > 0.5 ? 1 : above`
    * 在这里把负号移到 `rank()` 算子外面不好。
  * 可进一步用 `pv13` 模型分组改进。
  * `group_neutralize(above, densify(pv13_r2_min20_3000_sector))`

查看 Data Explorer 寻找 farming ideas——模拟使用率高的数据字段。

中国市场提示：
  * 使用 `group_rank` 而不是 `rank`。
  * 使用我们自己的风险因子做向量中性化。使用 `group_vector_neut`！

低 coverage 也可以——例如，TOP1000 上的数据覆盖率只有 33%。

把数学翻译成 alpha 表达式——变化是 `ts_delta` 或 `ts_zscore`，依赖性是 `ts_corr` 或 `ts_regression`，预测是 `ts_regression(... rettype=3)`。
