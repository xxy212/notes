# 投资组合优化

[理论讲解](https://docs.mosek.com/MOSEKPortfolioCookbook-a4paper.pdf)

[课程笔记](https://faculty.washington.edu/ezivot/econ424/portfolioTheoryMatrix.pdf)

假设我们有一组风险资产，它们有期望收益 `mu` 和协方差结构 `sigma`。如何构造投资组合权重 `x`，使其满足：
  * 最大化收益 / 方差 = `x^T mu / x^T S x`
  * 最小化方差 = `x^T S x`
  * 最大化 收益 - lambda * 方差 = `x^T mu - l x^T S x`

同时保持 `x^T 1 = 1`。

带线性约束的二次目标很容易求解。更一般的情况，比如风险有上界，需要使用锥规划形式。可以用 MOSEK 这样的专业软件求解。

在对协方差矩阵做 Cholesky 分解（或因子建模）时，会出现锥问题：`S = G G^T`。因此，最小化 `x^T S x = x^T G G^T x = |G^T x|^2`，这时可以把解理解为落在一个锥截面上。

这个详细推导的 [问答](https://quant.stackexchange.com/questions/59202/derivation-of-mean-variance-portfolio-weights-as-closed-form-analytical-solution) 展示了：
  * 从拉格朗日函数 `L = x^T mu - l x^T S x - g (x^T 1 - 1)` 开始
  * 对 `x` 和 `g` 求最小值
  * 这会导出一个线性方程组：`[[lS, 1],[1^T, 0]] [x, g] = [mu, 1]`
  * 应用 Schur 补可以得到最优 `x` 和 `g` 的漂亮形式
  * `xOPT = xMVP + 1/l muMVP / sMVP^2 (xTANG - xMVP)`，其中 TANG 是切点组合 `xTANG = S^-1 mu / 1^T S^-1 mu`，MVP 是最小方差组合 `xMVP = S^-1 1 / 1^T S^-1 1`
