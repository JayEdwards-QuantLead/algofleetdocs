# Risk Management

## Key Concepts

### Introduction

![](https://cdn.quantconnect.com/web/i/docs/algorithm-framework/risk-management.png)The Risk Management model seeks to manage risk on the `PortfolioTarget` collection it receives from the Portfolio Construction model before the targets reach the Execution model. There are many creative ways to manage risk. Some examples of risk management include the following:

- _"Trailing Stop Risk Management Model"_

Create and manage trailing stop-loss orders for open positions.

- _"Option Hedging Risk Management Model"_

Purchase options to hedge large equity exposures.

- _"Sector Exposure Risk Management Model"_

Reduce position sizes when overexposed to sectors or individual assets, keeping the portfolio within diversification requirements.

- _"Flash Crash Detection Risk Management Model"_

Scan for strange market situations that might be precursors to a flash crash and attempt to protect the portfolio when they are detected.


### Add Models

To set a Risk Management model, in the `Initialize` `initialize` method, call the `AddRiskManagement` `add_risk_management` method.

Select Language: C#Python

```
# Add the null risk management model, which doesn't affect the portfolio targets.
self.add_risk_management(NullRiskManagementModel())
```

```
// Add the null risk management model, which doesn't affect the portfolio targets.
AddRiskManagement(new NullRiskManagementModel());
```

To view all the pre-built Risk Management models, see [Supported Models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/risk-management/supported-models).

### Multi-Model Algorithms

To add multiple Risk Management models, in the `Initialize` `initialize` method, call the `AddRiskManagement` method multiple times.

Select Language: C#Python

```
// Add multiple Risk Management models to sequentially adjust portfolio targets, with each model refining the targets passed from the previous model, managing risk across individual securities and sectors.
AddRiskManagement(new MaximumDrawdownPercentPerSecurity());
AddRiskManagement(new MaximumSectorExposureRiskManagementModel());
```

```
# Add multiple Risk Management models to sequentially adjust portfolio targets, with each model refining the targets passed from the previous model, managing risk across individual securities and sectors.
self.add_risk_management(MaximumDrawdownPercentPerSecurity())
self.add_risk_management(MaximumSectorExposureRiskManagementModel())
```

If you add multiple Risk Management models, the original collection of `PortfolioTarget` objects from the Portfolio Construction model is passed to the first Risk Management model. The risk-adjusted targets from the first Risk Management model are passed to the second Risk Management model. The process continues sequentially until all of the Risk Management models have had an opportunity to adjust the targets.

### Model Structure

Risk Management models should extend the `RiskManagementModel` class. Extensions of the `RiskManagementModel` class must implement the `ManageRisk` `manage_risk` method, which receives an array of `PortfolioTarget` objects from the Portfolio Construction model at every [time step](https://www.quantconnect.com/docs/v2/writing-algorithms/key-concepts/time-modeling/timeslices) and should return an array of risk-adjusted `PortfolioTarget` objects. The method should only return the adjusted targets, not all of targets. If the method creates a `PortfolioTarget` object to liquidate a security, [cancel the security's insights](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/insight-manager#10-Cancel-Insights) to avoid re-entering the position.

Select Language: C#Python

```
// Extend the RiskManagementModel class by implementing the ManageRisk method, which receives PortfolioTarget objects from the Portfolio Construction model and returns only the risk-adjusted targets.
class MyRiskManagementModel : RiskManagementModel
{
    // Adjust the portfolio targets and return them. If no changes emit nothing.
    public override List<PortfolioTarget> ManageRisk(QCAlgorithm algorithm, PortfolioTarget[] targets)
    {
        return new List<PortfolioTarget>();
    }

    // Optional: Be notified when securities change
    public override void OnSecuritiesChanged(QCAlgorithm algorithm, SecurityChanges changes)
    {
        // Security additions and removals are pushed here.
        // This can be used for setting up algorithm state.
        // changes.AddedSecurities
        // changes.RemovedSecurities
    }
}
```

```
# Extend the RiskManagementModel class by implementing the manage_risk method, which receives PortfolioTarget objects from the Portfolio Construction model and returns only the risk-adjusted targets.
class MyRiskManagementModel(RiskManagementModel):
    # Adjust the portfolio targets and return them. If no changes emit nothing.
    def manage_risk(self, algorithm: QCAlgorithm, targets: list[PortfolioTarget]) -> list[PortfolioTarget]:
        return []

    # Optional: Be notified when securities change
    def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
        # Security additions and removals are pushed here.
        # This can be used for setting up algorithm state.
        # changes.added_securities
        # changes.removed_securities
        pass
```

The `algorithm` argument that the methods receive is an instance of the base `QCAlgorithm` class, not your subclass of it.

To view a full example of a `RiskManagementModel` subclass, see the [MaximumDrawdownPercentPerSecurity](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Risk/MaximumDrawdownPercentPerSecurity.cs)[MaximumDrawdownPercentPerSecurity](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Risk/MaximumDrawdownPercentPerSecurity.py) in the LEAN GitHub repository.

### Track Security Changes

The [Universe Selection model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/key-concepts) may select a dynamic universe of assets, so you should not assume a fixed set of assets in the Risk Management model. When the Universe Selection model adds and removes assets from the universe, it triggers an `OnSecuritiesChanged` `on_securities_changed` event. In the `OnSecuritiesChanged` `on_securities_changed` event handler, you can initialize the security-specific state or load any history required for your Risk Management model. If you need to save data for individual securities, add custom members to the respective `Security` objectcast the `Security` object to a `dynamic` object and then save custom members to it.

Select Language: C#Python

```
class MyRiskManagementModel : RiskManagementModel{
    private List<Security> _securities = new List<Security>();

    public override void OnSecuritiesChanged(QCAlgorithm algorithm, SecurityChanges changes)
    {
        base.OnSecuritiesChanged(algorithm, changes);
        foreach (var security in changes.AddedSecurities)
        {
            // Store and manage Symbol-specific data
            var dynamicSecurity = security as dynamic;
            dynamicSecurity.Sma = SMA(security.Symbol, 20);

            _securities.Add(security);
        }

        foreach (var security in changes.RemovedSecurities)
        {
            if (_securities.Contains(security))
            {
                algorithm.DeregisterIndicator((security as dynamic).Sma);

                _securities.Remove(security);
            }
        }
    }
}
```

```
class MyRiskManagementModel(RiskManagementModel):
    _securities = []

    def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
        super().on_securities_changed(algorithm, changes)
        for security in changes.added_securities::
            # Store and manage Symbol-specific data
            security.indicator = algorithm.sma(security.symbol, 20)
            algorithm.warm_up_indicator(security.symbol, security.indicator)

            self._securities.append(security)

        for security in changes.removed_securities:
            if security in self.securities:
                algorithm.deregister_indicator(security.indicator)
                self._securities.remove(security)
```

### Portfolio Target Collection

The `PortfolioTargetCollection` class is a helper class to manage `PortfolioTarget` objects. The class manages an internal dictionary that has the security `Symbol` as the key and a `PortfolioTarget` as the value.

#### Add Portfolio Targets

To add a `PortfolioTarget` to the `PortfolioTargetCollection`, call the `Add` `add` method.

Select Language: C#Python

```
_targetsCollection.Add(portfolioTarget);
```

```
self.targets_collection.add(portfolio_target)
```

To add a list of `PortfolioTarget` objects, call the `AddRange` `add_range` method.

Select Language: C#Python

```
_targetsCollection.AddRange(portfolioTargets);
```

```
self.targets_collection.add_range(portfolio_targets)
```

#### Check Membership

To check if a `PortfolioTarget` exists in the `PortfolioTargetCollection`, call the `Contains` `contains` method.

Select Language: C#Python

```
var targetInCollection = _targetsCollection.Contains(portfolioTarget);
```

```
target_in_collection = self.targets_collection.contains(portfolio_target)
```

To check if a Symbol exists in the `PortfolioTargetCollection`, call the `ContainsKey` `contains_key` method.

Select Language: C#Python

```
var symbolInCollection = _targetsCollection.ContainsKey(symbol);
```

```
symbol_in_collection = self.targets_collection.contains_key(symbol)
```

To get all the Symbol objects, use the `Keys` `keys` property.

Select Language: C#Python

```
var symbols = _targetsCollection.Keys;
```

```
symbols = self.targets_collection.keys
```

#### Access Portfolio Targets

To access the `PortfolioTarget` objects for a Symbol, index the `PortfolioTargetCollection` with the Symbol.

Select Language: C#Python

```
var portfolioTarget = _targetsCollection[symbol];
```

```
portfolio_target = self.targets_collection[symbol]
```

To iterate through the `PortfolioTargetCollection`, call the `GetEnumerator` `get_enumerator` method.

Select Language: C#Python

```
var enumerator = _targetsCollection.GetEnumerator();
```

```
enumerator = self.targets_collection.get_enumerator()
```

To get all the `PortfolioTarget` objects, use the `Values` `values` property

Select Language: C#Python

```
var portfolioTargets = _targetsCollection.Values;
```

```
portfolio_targets = self.targets_collection.values
```

#### Order Portfolio Targets by Margin Impact

To get an enumerable where position reducing orders are executed first and the remaining orders are executed in decreasing order value, call the `OrderByMarginImpact` `order_by_margin_impact` method.

Select Language: C#Python

```
foreach (var target in _targetsCollection.OrderByMarginImpact(algorithm))
{
    // Place order
}
```

```
for target in self.targets_collection.order_by_margin_impact(algorithm):
    # Place order
```

This method won't return targets for securities that have no data yet. This method also won't return targets for which the sum of the current holdings and open orders quantity equals the target quantity.

#### Remove Portfolio Targets

To remove a `PortfolioTarget` from the `PortfolioTargetCollection`, call the `Remove` `remove` method.

Select Language: C#Python

```
removeSuccessful = _targetsCollection.Remove(symbol);
```

```
remove_successful = self.targets_collection.remove(symbol)
```

To remove all the `PortfolioTarget` objects, call the `Clear` `clear` method.

Select Language: C#Python

```
_targetsCollection.Clear();
```

```
self.targets_collection.clear()
```

To remove all the `PortfolioTarget` objects that have been fulfilled, call the `ClearFulfilled` `clear_fulfilled` method.

Select Language: C#Python

```
_targetsCollection.ClearFulfilled(algorithm);
```

```
self.targets_collection.clear_fulfilled(algorithm)
```

### Universe Timing Considerations

If the Risk Management model manages some indicators or [consolidators](https://www.quantconnect.com/docs/v2/writing-algorithms/consolidating-data/getting-started) for securities in the universe and the universe selection runs during the indicator sampling period or the consolidator aggregation period, the indicators and consolidators might be missing some data. For example, take the following scenario:

- The security resolution is minute
- You have a consolidator that aggregates the security data into daily bars to update the indicator
- The universe selection runs at noon

In this scenario, you create and [warm-up the indicator](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/manual-indicators#06-Warm-Up-Indicators) at noon. Since it runs at noon, the history request that gathers daily data to warm up the indicator won't contain any data from the current day and the consolidator that updates the indicator also won't aggregate any data from before noon. This process doesn't cause issues if the indicator only uses the close price to calculate the indicator value (like the [simple moving average](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/simple-moving-average) indicator) because the first consolidated bar that updates the indicator will have the correct close price. However, if the indicator uses more than just the close price to calculate its value (like the [True Range](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/true-range) indicator), the open, high, and low values of the first consolidated bar may be incorrect, causing the initial indicator values to be incorrect.

### Examples

The following examples demonstrate some common practices for implementing a risk management model.

#### Example 1: Take Profit Stop Loss

The following algorithm trades 20-60 EMA crosses on the top 500 liquid US stocks in equal weighting. To minimize risk and capture early profit, we can implement both `MaximumUnrealizedProfitPercentPerSecurity` to take profit and `TrailingStopRiskManagementModel` to stop loss by trailing high price.

Select Language: C#Python

```
public class FrameworkRiskManagementAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2024, 8, 12);
        SetEndDate(2024, 10, 12);
        SetCash(1000000);

        // Add a universe of the most liquid stocks since their trend is more capital-supported.
        AddUniverseSelection(new QC500UniverseSelectionModel());
        // Emit insights on EMA cross, indicating the trend changes. We use short-term versus medium-term for more trade opportunities.
        AddAlpha(new EmaCrossAlphaModel(20, 60, Resolution.Daily));
        // Equal weighting on each insight is needed to dissipate capital risk evenly.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());

        // Take profit at 10%.
        AddRiskManagement(new MaximumUnrealizedProfitPercentPerSecurity(0.1m));
        // Trailing stop loss at 5%.
        AddRiskManagement(new TrailingStopRiskManagementModel(0.05m));
    }
}
```

```
class FrameworkRiskManagementAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2024, 8, 12)
        self.set_end_date(2024, 10, 12)
        self.set_cash(1000000)

        # Add a universe of the most liquid stocks since their trend is more capital-supported.
        self.add_universe_selection(QC500UniverseSelectionModel())
        # Emit insights on EMA cross, indicating the trend changes. We use short-term versus medium-term for more trade opportunities.
        self.add_alpha(EmaCrossAlphaModel(20, 60, Resolution.DAILY))
        # Equal weighting on each insight to dissipate capital risk evenly.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

        # Take profit at 10%.
        self.add_risk_management(MaximumUnrealizedProfitPercentPerSecurity(0.1))
        # Trailing stop loss at 5%.
        self.add_risk_management(TrailingStopRiskManagementModel(0.05))
```

#### Example 2: Tail Value At Risk

The following algorithm implements a custom risk management model that liquidates the position if an asset has PnL lower than the tail value-at-risk (TVaR) throughout the insight to avoid large catastrophic losses. To calculate the TVaR, we create a log return indicator and use the following formula for the calculation:
$$
\\textrm{TVaR}\_{\\alpha} = \\mu + \\sigma \\times \\phi\[\\Phi^{-1}(\\alpha)\] / (1 - \\alpha)
$$

Select Language: C#Python

```
using Accord.Statistics;
using MathNet.Numerics.Distributions;

public class FrameworkRiskManagementAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2020, 1, 1);
        SetEndDate(2020, 2, 1);
        SetCash(1000000);

        // Add a universe of the most liquid stocks since their trend is more capital-supported.
        AddUniverseSelection(new QC500UniverseSelectionModel());
        // Emit insights all for selected stocks, rebalancing every 2 weeks.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(14)));
        // Equal weighting on each insight is needed to dissipate capital risk evenly.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());

        // Liquidate on extreme catastrophic event.
        AddRiskManagement(new TailValueAtRiskRiskManagementModel(0.05d, 14d));
    }

    private class TailValueAtRiskRiskManagementModel : RiskManagementModel
    {
        // The alpha level on TVaR calculation.
        private readonly double _alpha;
        // The number of days that the insight signal lasts for.
        private readonly double _days;
        private Dictionary<Symbol, SymbolData> _logRetBySymbol = new();

        public TailValueAtRiskRiskManagementModel(double alpha = 0.05d, double numDays = 14d)
        {
            _alpha = alpha;
            _days = numDays;
        }

        // Adjust the portfolio targets and return them. If there are no changes, emit nothing.
        public override IEnumerable<PortfolioTarget> ManageRisk(QCAlgorithm algorithm, IPortfolioTarget[] _)
        {
            var targets = new List<PortfolioTarget>();

            foreach (var (symbol, security) in algorithm.Securities)
            {
                if (!security.Invested)
                {
                    continue;
                }

                var pnl = security.Holdings.UnrealizedProfitPercent;
                // If the %PnL is worse than the preset level TVaR, we liquidate it.
                if (pnl < GetTVaR(symbol))
                {
                    // Cancel insights to avoid reordering afterward.
                    algorithm.Insights.Cancel(new[] { symbol });
                    // Liquidate.
                    targets.Add(new PortfolioTarget(symbol, 0, tag: "Liquidate due to TVaR"));
                }
            }

            return targets;
        }

        private decimal GetTVaR(Symbol symbol)
        {
            if (!_logRetBySymbol.TryGetValue(symbol, out var symbolData))
            {
                return 0m;
            }

            // TVaR = \mu + \sigma * \phi[\Phi^{-1}(p)] / (1 - p)
            // Scale up to the days of the signal. By stochastic calculus, we multiply by sqrt(days).
            var dailyTVaR = symbolData.MeanLogRet + symbolData.SdLogRet * Math.Sqrt(_days) * Normal.PDF(0d, 1d, Normal.InvCDF(0d, 1d, _alpha)) / (1d - _alpha);
            // We want the left side of the symmetric distribution.
            return -Convert.ToDecimal(dailyTVaR);
        }

        public override void OnSecuritiesChanged(QCAlgorithm algorithm, SecurityChanges changes)
        {
            foreach (var added in changes.AddedSecurities)
            {
                // Add SymbolData class to handle log returns.
                _logRetBySymbol[added.Symbol] = new SymbolData(algorithm, added.Symbol);
            }

            foreach (var removed in changes.RemovedSecurities)
            {
                // Stop subscription on the data to release computational resources.
                if (_logRetBySymbol.Remove(removed.Symbol, out var symbolData))
                {
                    symbolData.Dispose();
                }
            }
        }
    }

    private class SymbolData
    {
        private readonly QCAlgorithm _algorithm;
        private readonly Symbol _symbol;
        // Since the return is assumed log-normal, we use the log return indicator to calculate TVaR later.
        private LogReturn _logRet = new(1);
        // Set up a rolling window to save the log return for calculating the mean and SD for TVaR calculation.
        private RollingWindow<double> _window = new(252);

        public bool IsReady => _window.IsReady;

        public double MeanLogRet => _window.Average();

        public double SdLogRet => Measures.StandardDeviation(_window.ToArray());

        public SymbolData(QCAlgorithm algorithm, Symbol symbol)
        {
            _algorithm = algorithm;
            // Register the indicator for automatic updating for daily log returns.
            algorithm.RegisterIndicator(symbol, _logRet, Resolution.Daily);
            // Add a handler to save the log return to the rolling window.
            _logRet.Updated += (_algorithm, point) => _window.Add((double)point.Value);
            // Warm up the rolling window.
            var history = algorithm.History<TradeBar>(symbol, 253, Resolution.Daily);
            foreach (var bar in history)
            {
                _logRet.Update(bar.EndTime, bar.Close);
            }
        }

        public void Dispose()
        {
            // Stop subscription on the data to release computational resources.
            _algorithm.DeregisterIndicator(_logRet);
        }
    }
}
```

```
from scipy.stats import norm

class FrameworkRiskManagementAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2020, 1, 1)
        self.set_end_date(2020, 2, 1)
        self.set_cash(1000000)

        # Add a universe of the most liquid stocks since their trend is more capital-supported.
        self.add_universe_selection(QC500UniverseSelectionModel())
        # Emit insights all for selected stocks, rebalancing every 2 weeks.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(14)))
        # Equal weighting on each insight to dissipate capital risk evenly.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

        # Liquidate on extreme catastrophic events.
        self.add_risk_management(TailValueAtRiskRiskManagementModel(0.05, 14))

class TailValueAtRiskRiskManagementModel(RiskManagementModel):
    _log_ret_by_symbol = {}

    def __init__(self, alpha: float = 0.05, num_days: float = 7) -> None:
        # The alpha level on TVaR calculation.
        self.alpha = alpha
        # The number of days that the insight signal lasts for.
        self.days = num_days

    # Adjust the portfolio targets and return them. If there are no changes, emit nothing.
    def manage_risk(self, algorithm: QCAlgorithm, targets: list[PortfolioTarget]) -> list[PortfolioTarget]:
        targets = []
        for kvp in algorithm.securities:
            security = kvp.value
            if not security.invested:
                continue

            pnl = security.holdings.unrealized_profit_percent
            symbol = security.symbol
            # If the %PnL is worse than the preset level TVaR, we liquidate it.
            if pnl < self.get_tvar(symbol):
                # Cancel insights to avoid reordering afterward.
                algorithm.insights.cancel([symbol])
                # Liquidate.
                targets.append(PortfolioTarget(symbol, 0, tag="Liquidate due to TVaR"))

        return targets

    def get_tvar(self, symbol: Symbol) -> float:
        symbol_data = self._log_ret_by_symbol.get(symbol)
        if not symbol_data:
            return 0

        # TVaR = \mu + \sigma * \phi[\Phi^{-1}(p)] / (1 - p)
        # Scale up to the days of the signal. By stochastic calculus, we multiply by sqrt(days).
        daily_tvar = symbol_data.mean_log_ret + symbol_data.sd_log_ret * np.sqrt(self.days) * norm.pdf(norm.ppf(self.alpha)) / (1 - self.alpha)
        # We want the left side of the symmetric distribution.
        return -daily_tvar

    def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
        for added in changes.added_securities:
            # Add SymbolData class to handle log returns.
            self._log_ret_by_symbol[added.symbol] = SymbolData(algorithm, added.symbol)

        for removed in changes.removed_securities:
            # Stop subscription on the data to release computational resources.
            symbol_data = self._log_ret_by_symbol.pop(removed.symbol, None)
            if symbol_data:
                symbol_data.dispose()

class SymbolData:
    def __init__(self, algorithm: QCAlgorithm, symbol: Symbol) -> None:
        self.algorithm = algorithm
        self.symbol = symbol

        # Since the return is assumed log-normal, we use the log return indicator to calculate TVaR later.
        self.log_ret = LogReturn(1)
        # Register the indicator for automatic updating for daily log returns.
        algorithm.register_indicator(symbol, self.log_ret, Resolution.DAILY)
        # Set up a rolling window to save the log return for calculating the mean and SD for TVaR calculation.
        self.window = RollingWindow(252)
        # Add a handler to save the log return to the rolling window.
        self.log_ret.updated += lambda _, point: self.window.add(point.value)
        # Warm up the rolling window.
        history = algorithm.history[TradeBar](symbol, 253, Resolution.DAILY)
        for bar in history:
            self.log_ret.update(bar.end_time, bar.close)

    @property
    def is_ready(self) -> bool:
        return self.window.is_ready

    @property
    def mean_log_ret(self) -> float:
        # Mean log return for TVaR calculation.
        return np.mean(list(self.window))

    @property
    def sd_log_ret(self) -> float:
        # SD of log return for TVaR calculation.
        return np.std(list(self.window), ddof=1)

    def dispose(self) -> None:
        # Stop subscription on the data to release computational resources.
        self.algorithm.deregister_indicator(self.log_ret)
```

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/05%20Risk%20Management/01%20Key%20Concepts)