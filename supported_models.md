# Alpha

## Supported Models

### Introduction

This page describes the pre-built Alpha models in LEAN. The number of models grows over time. To add a model to LEAN, make a pull request to the [GitHub repository](https://github.com/QuantConnect/Lean/tree/master/Algorithm.Framework/Alphas). If none of these models perform exactly how you want, create a [custom Alpha model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/alpha/key-concepts#04-Model-Structure).

### Null Model

The `NullAlphaModel` doesn't emit any insights. It's the default Alpha model.

Select Language: C#Python

```
// NullAlphaModel doesn't emit insights.
AddAlpha(new NullAlphaModel());
```

```
# NullAlphaModel doesn't emit insights.
self.add_alpha(NullAlphaModel())
```

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm/Alphas/NullAlphaModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm/Alphas/NullAlphaModel.py).

### Constant Model

The `ConstantAlphaModel` always returns the same insight for each security.

Select Language: C#Python

```
// Emit up insights that expire in 30 days for all assets.
AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(30)));
```

```
# Emit up insights that expire in 30 days for all assets.
self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(30)))
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `type` | `InsightType` | The type of insight |  |
| `direction` | `InsightDirection` | The direction of the insight |  |
| `period` | `TimeSpan` `timedelta` | The period over which the insight will come to fruition |  |
| `magnitude` | `double?` `float/NoneType` | The predicted change in magnitude as a +/- percentage | `null` `None` |
| `confidence` | `double?` `float/NoneType` | The confidence in the insight | `null` `None` |
| `weight` | `double?` `float/NoneType` | The portfolio weight of the insights | `null` `None` |

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/ConstantAlphaModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/ConstantAlphaModel.py).

### Historical Returns Model

The `HistoricalReturnsAlphaModel` buys securities that have a positive trailing return and sells securities that have a negative trailing return. It sets the magnitude of the Insight objects to the trailing rate of change.

Select Language: C#Python

```
// Add HistoricalReturnsAlphaModel to leverage historical return data for generating alpha signals, identifying trends based on past performance.
AddAlpha(new HistoricalReturnsAlphaModel());
```

```
# Add HistoricalReturnsAlphaModel to leverage historical return data for generating alpha signals, identifying trends based on past performance.
self.add_alpha(HistoricalReturnsAlphaModel())
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `lookback` | `int` | Historical return lookback period | 1 |
| `resolution` | `Resolution` | The resolution of historical data | `Resolution.Daily` `Resolution.DAILY` |

This model cancels all the active insights it has emit for a security when the security has a 0% historical return.

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/HistoricalReturnsAlphaModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/HistoricalReturnsAlphaModel.py).

### EMA Cross Model

The `EmaCrossAlphaModel` uses an [exponential moving average](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/exponential-moving-average) (EMA) cross to create insights. When the fast EMA crosses above the slow EMA, it emits up insights. When the fast EMA crosses below the slow EMA, it emits down insights. It sets the duration of Insight objects to be the product of the `resolution` and `fastPeriod` `fast_period` arguments.

Select Language: C#Python

```
// Add EmaCrossAlphaModel to generate trading insights based on the crossing of fast and slow exponential moving averages identifying trends and potential reversals. The duration of insights is set by the resolution and fast_period to ensure timely and relevant trading signals.
AddAlpha(new EmaCrossAlphaModel());
```

```
# Add EmaCrossAlphaModel to generate trading insights based on the crossing of fast and slow exponential moving averages identifying trends and potential reversals. The duration of insights is set by the resolution and fast_period to ensure timely and relevant trading signals.
self.add_alpha(EmaCrossAlphaModel())
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `fastPeriod` `fast_period` | `int` | The fast EMA period | 12 |
| `slowPeriod` `slow_period` | `int` | The slow EMA period | 26 |
| `resolution` | `Resolution` | The resolution of data sent into the EMA indicators | `Resolution.Daily` `Resolution.DAILY` |

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/EmaCrossAlphaModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/EmaCrossAlphaModel.py).

### MACD Model

The `MacdAlphaModel` emits insights based on moving average convergence divergence (MACD) crossovers. If the MACD signal line is 1% above the security price, the model emits an up insight. If the MACD signal line is 1% below the security price, the model emits a down insight. If the MACD signal line is within 1% of the security price, the model cancels all the active insights it has emitted for the security.

Select Language: C#Python

```
// Use MacdAlphaModel to generate insights based on MACD crossovers, emitting up signals when the MACD signal line is 1% above the security price and down signals when it is 1% below, while canceling insights if within 1% capturing significant trend changes while avoiding noise from minor fluctuations.
AddAlpha(new MacdAlphaModel());
```

```
# Use MacdAlphaModel to generate insights based on MACD crossovers, emitting up signals when the MACD signal line is 1% above the security price and down signals when it is 1% below, while canceling insights if within 1% capturing significant trend changes while avoiding noise from minor fluctuations.
self.add_alpha(MacdAlphaModel())
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `fastPeriod` `fast_period` | `int` | The MACD fast period | 12 |
| `slowPeriod` `slow_period` | `int` | The MACD slow period | 26 |
| `signalPeriod` `signal_period` | `int` | The smoothing period for the MACD signal | 9 |
| `movingAverageType` `moving_average_type` | `MovingAverageType` | The type of moving average to use in the MACD | `MovingAverageType.ExponentialEXPONENTIAL` |
| `resolution` | `Resolution` | The resolution of data sent into the MACD indicator | `Resolution.Daily` `Resolution.DAILY` |

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/MacdAlphaModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/MacdAlphaModel.py).

### RSI Model

The `RsiAlphaModel` generates insights based on the relative strength index (RSI) indicator values. When the RSI value passes above 70, the model emits a down insight. When the RSI value passes below 30, the model emits an up insight. The model uses the Wilder moving average type and sets the duration of Insight objects to be the product of the `resolution` and `period` arguments.

Select Language: C#Python

```
// The RsiAlphaModel generates insights based on the RSI indicator: emits a down insight when RSI exceeds 70 (overbought) and an up insight when RSI falls below 30 (oversold). This helps identify potential price reversals based on momentum.
AddAlpha(new RsiAlphaModel());
```

```
# The RsiAlphaModel generates insights based on the RSI indicator: emits a down insight when RSI exceeds 70 (overbought) and an up insight when RSI falls below 30 (oversold). This helps identify potential price reversals based on momentum.
self.add_alpha(RsiAlphaModel())
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `period` | `int` | The RSI indicator period | 14 |
| `resolution` | `Resolution` | The resolution of data sent into the RSI indicator | `Resolution.Daily` `Resolution.DAILY` |

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/RsiAlphaModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/RsiAlphaModel.py).

### Base Pairs Trading Model

The `BasePairsTradingAlphaModel` analyzes every possible pair combination from securities that the Universe Selection model selects. This model calculates a ratio between the two securities by dividing their historical prices over a lookback window. It then calculates the mean of this ratio by taking the 500-period EMA of the quotient. When the ratio diverges far enough from the mean ratio, this model emits generates alternating long ratio/short ratio insights emitted as a group to capture the reversion of the ratio.

Select Language: C#Python

```
// Use BasePairsTradingAlphaModel to analyze security pairs from the universe, generating long/short insights based on deviations from a 500-period EMA of their price ratio to capture mean-reversion opportunities.
AddAlpha(new BasePairsTradingAlphaModel());
```

```
# Use BasePairsTradingAlphaModel to analyze security pairs from the universe, generating long/short insights based on deviations from a 500-period EMA of their price ratio to capture mean-reversion opportunities.
self.add_alpha(BasePairsTradingAlphaModel())
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `lookback` | `int` | Lookback period of the analysis | 1 |
| `resolution` | `Resolution` | Analysis resolution | `Resolution.Daily` `Resolution.DAILY` |
| `threshold` | `decimal` `float` | The percent \[0, 100\] deviation of the ratio from the mean before emitting an insight | 1 |

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/BasePairsTradingAlphaModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/BasePairsTradingAlphaModel.py).

### Pearson Correlation Pairs Trading Model

The `PearsonCorrelationPairsTradingAlphaModel` ranks every pair combination by its Pearson correlation coefficient and trades the pair with the highest correlation. This model follows the same insight logic as the `BasePairsTradingModel`.

Select Language: C#Python

```
// Add PearsonCorrelationPairsTradingAlphaModel to rank and trade the most correlated security pairs based on Pearson correlation coefficients, using a similar insight logic as BasePairsTradingModel to capture mean-reversion opportunities.
AddAlpha(new PearsonCorrelationPairsTradingAlphaModel());
```

```
# Add PearsonCorrelationPairsTradingAlphaModel to rank and trade the most correlated security pairs based on Pearson correlation coefficients, using a similar insight logic as BasePairsTradingModel to capture mean-reversion opportunities.
self.add_alpha(PearsonCorrelationPairsTradingAlphaModel())
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `lookback` | `int` | Lookback period of the analysis | 15 |
| `resolution` | `Resolution` | Analysis resolution | `Resolution.Minute` `Resolution.MINUTE` |
| `threshold` | `decimal` `float` | The percent \[0, 100\] deviation of the ratio from the mean before emitting an insight | 1 |
| `minimumCorrelation` `minimum_correlation` | `double` `float` | The minimum correlation to consider a tradable pair | 0.5 |

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/PearsonCorrelationPairsTradingAlphaModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/PearsonCorrelationPairsTradingAlphaModel.py).

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/03%20Alpha/02%20Supported%20Models)