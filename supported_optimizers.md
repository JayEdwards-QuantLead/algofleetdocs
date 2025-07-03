# Portfolio Construction

## Supported Optimizers

### Introduction

This page describes the pre-built Portfolio Optimizer models in LEAN. The number of models grows over time. To add a model to LEAN, make a pull request to the [GitHub repository](https://github.com/QuantConnect/Lean/tree/master/Algorithm.Framework/Portfolio). If none of these models perform exactly how you want, create a [custom Portfolio Optimizer model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/key-concepts#09-Portfolio-Optimizer-Structure).

### Maximum Sharpe Ratio Optimizer

The `MaximumSharpeRatioPortfolioOptimizer` seeks to maximize the portfolio [Sharpe Ratio](https://www.quantconnect.com/docs/v2/writing-algorithms/key-concepts/glossary#28-Sharpe-ratio).

Select Language: C#Python

```
var optimizer = new MaximumSharpeRatioPortfolioOptimizer();
```

```
optimizer = MaximumSharpeRatioPortfolioOptimizer()
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `minimum_weight` `lower` | `double` `float` | The lower bounds on portfolio weights | -1 |
| `maximum_weight` `upper` | `double` `float` | The upper bounds on portfolio weights | 1 |
| `risk_free_rate` `riskFreeRate` | `double` `float` | The risk free rate | 0 |

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/MaximumSharpeRatioPortfolioOptimizer.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/MaximumSharpeRatioPortfolioOptimizer.py).

### Minimum Variance Optimizer

The `MinimumVariancePortfolioOptimizer` seeks to minimize the portfolio variance and achieve a target return.

Select Language: C#Python

```
var optimizer = new MinimumVariancePortfolioOptimizer();
```

```
optimizer = MinimumVariancePortfolioOptimizer()
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `minimum_weight` `lower` | `double` `float` | The lower bounds on portfolio weights | -1 |
| `maximum_weight` `upper` | `double` `float` | The upper bounds on portfolio weights | 1 |
| `target_return` `targetReturn` | `double` `float` | The target portfolio return | 0.02 (2%) |

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/MinimumVariancePortfolioOptimizer.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/MinimumVariancePortfolioOptimizer.py).

### Unconstrained Mean Variance Optimizer

The `UnconstrainedMeanVariancePortfolioOptimizer` seeks to find the optimal risk-adjusted portfolio that lies on the efficient frontier.

Select Language: C#Python

```
var optimizer = new UnconstrainedMeanVariancePortfolioOptimizer();
```

```
optimizer = UnconstrainedMeanVariancePortfolioOptimizer()
```

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/UnconstrainedMeanVariancePortfolioOptimizer.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/UnconstrainedMeanVariancePortfolioOptimizer.py).

### Risk Parity Optimizer

The `RiskParityPortfolioOptimizer` seeks to equalize the individual risk contribution to the total portfolio risk from each asset.

Select Language: C#Python

```
var optimizer = new RiskParityPortfolioOptimizer();
```

```
optimizer = RiskParityPortfolioOptimizer()
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `minimum_weight` `lower` | `double` `float` | The lower bounds on portfolio weights | 1e-05 |
| `maximum_weight` `upper` | `double` `float` | The upper bounds on portfolio weights | `sys.float_info.max` `Double.MaxValue` |

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/RiskParityPortfolioOptimizer.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/RiskParityPortfolioOptimizer.py).

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/04%20Portfolio%20Construction/03%20Supported%20Optimizers)