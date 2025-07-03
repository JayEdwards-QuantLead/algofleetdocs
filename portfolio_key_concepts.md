# Portfolio Construction

## Key Concepts

### Introduction

![](https://cdn.quantconnect.com/web/i/docs/algorithm-framework/portfolio-construction.png)The Portfolio Construction model receives `Insight` objects from the Alpha model and creates `PortfolioTarget` objects for the Risk Management model. A `PortfolioTarget` provides the number of units of an asset to hold.

### Set Models

To set a Portfolio Construction model, in the `Initialize` `initialize` method, call the `SetPortfolioConstruction` `set_portfolio_construction` method.

Select Language: C#Python

```
// Create PortfolioTarget objects to form an equal weighted portfolio based on the Insight objects.
SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
```

```
# Create PortfolioTarget objects to form an equal weighted portfolio based on the Insight objects.
self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())
```

To view all the pre-built Portfolio Construction models, see [Supported Models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/supported-models).

### Model Structure

Portfolio Construction models should extend the `PortfolioConstructionModel` class or one of the [supported models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/supported-models). Extensions of the `PortfolioConstructionModel` class should implement the `CreateTargets` `create_targets` method, which receives an array of `Insight` objects from the Alpha model at every [time step](https://www.quantconnect.com/docs/v2/writing-algorithms/key-concepts/time-modeling/timeslices) and returns an array of `PortfolioTarget` objects. The Portfolio Construction model seeks to answer the question, "how many units should I buy based on the insight predictions I've been presented?".

If you don't override the `CreateTargets` `create_targets` method, the base class implementation calls the model's `IsRebalanceDue` `is_rebalance_due`, `DetermineTargetPercent` `determine_target_percent`, and `GetTargetInsights` `get_target_insights` helper methods. The `GetTargetInsights` `get_target_insights` method, in turn, calls the model's `ShouldCreateTargetForInsight` `should_create_target_for_insight` method. You can override any of these helper methods. If you don't override the `CreateTargets` `create_targets` method from the `PortfolioConstructionModel` class, your class must at least override the `DetermineTargetPercent` `determine_target_percent` method.

Select Language: C#Python

```
// Portfolio construction scaffolding class; basic method arguments.
class MyPortfolioConstructionModel : PortfolioConstructionModel
{
    // Create list of PortfolioTarget objects from Insights.
    public override List<PortfolioTarget> CreateTargets(QCAlgorithm algorithm, Insight[] insights)
    {
        return (List<PortfolioTarget>) base.CreateTargets(algorithm, insights);
    }

    // Determine if the portfolio should rebalance based on the provided rebalancing function.
    protected override bool IsRebalanceDue(Insight[] insights, DateTime algorithmUtc)
    {
        return base.IsRebalanceDue(insights, algorithmUtc);
    }

    // Determine the target percent for each insight.
    protected override Dictionary<Insight, double> DetermineTargetPercent(List<Insight> activeInsights)
    {
        return new Dictionary<Insight, double>();
    }

    // Get the target insights to calculate a portfolio target percent.
    // They will be piped to the DetermineTargetPercent method.
    protected override List<Insight> GetTargetInsights()
    {
        return base.GetTargetInsights();
    }

    // Determine if the portfolio construction model should create a target for this insight.
    protected override bool ShouldCreateTargetForInsight(Insight insight)
    {
        return base.ShouldCreateTargetForInsight(insight);
    }

    // Track universe changes.
    public override void OnSecuritiesChanged(QCAlgorithm algorithm, SecurityChanges changes)
    {
        base.OnSecuritiesChanged(algorithm, changes);
    }
}
```

```
 # Portfolio construction scaffolding class; basic method arguments.
class MyPortfolioConstructionModel(PortfolioConstructionModel):
    # Create list of PortfolioTarget objects from Insights.
    def create_targets(self, algorithm: QCAlgorithm, insights: list[Insight]) -> list[PortfolioTarget]:
        return super().create_targets(algorithm, insights)

    # Determine if the portfolio should rebalance based on the provided rebalancing function.
    def is_rebalance_due(self, insights: list[Insight], algorithmUtc: datetime) -> bool:
        return super().is_rebalance_due(insights, algorithmUtc)

    # Determine the target percent for each insight.
    def determine_target_percent(self, activeInsights: list[Insight]) -> dict[Insight, float]:
        return {}

    # Get the target insights to calculate a portfolio target percent.
    # They will be piped to the determine_target_percent method.
    def get_target_insights(self) -> list[Insight]:
        return super().get_target_insights()

    # Determine if the portfolio construction model should create a target for this insight.
    def should_create_target_for_insight(self, insight: Insight) -> bool:
        return super().should_create_target_for_insight(insight)

    # Track universe changes.
    def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
        super().on_securities_changed(algorithm, changes)
```

The Portfolio Construction model should [remove expired insights](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/insight-manager#08-Remove-Insights) from the [Insight Manager](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/insight-manager). The `CreateTargets` `create_targets` definition of the base `PortfolioConstructionModel` class already removes them during each rebalance. Therefore, if you override the `CreateTargets` `create_targets` method and don't call the `CreateTargets` `create_targets` definition of the base class, your new method definition should remove expired insights from the Insight Manager.

The model should also remove all a security's insights from the Insight Manager when the security is removed from the [universe](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/key-concepts). The `OnSecuritiesChanged` `on_securities_changed` definition of the base `PortfolioConstructionModel` class already does this. Therefore, if you override the `OnSecuritiesChanged` `on_securities_changed` method and don't call the `OnSecuritiesChanged` `on_securities_changed` definition of the base class, your new method definition should remove the security's insights from the Insight Manager.

The `algorithm` argument that the methods receive is an instance of the base `QCAlgorithm` class, not your subclass of it.

You may use the `PortfolioBias` enumeration in the definition of Portfolio Construction model methods. The `PortfolioBias` enumeration has the following members:

PortfolioBias
Select Language: C#Python

Portfolio can only have short positions (-1)

- SHORT: PortfolioBias

Portfolio can have both long and short positions (0)

- LONG\_SHORT: PortfolioBias

Portfolio can only have long positions (1)

- LONG: PortfolioBias

Portfolio can only have short positions (-1)

- Short: PortfolioBias

Portfolio can have both long and short positions (0)

- LongShort: PortfolioBias

Portfolio can only have long positions (1)

- Long: PortfolioBias

To view a full example of a `PortfolioConstructionModel` subclass, see the [EqualWeightingPortfolioConstructionModel](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/EqualWeightingPortfolioConstructionModel.cs)[EqualWeightingPortfolioConstructionModel](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/EqualWeightingPortfolioConstructionModel.py) in the LEAN GitHub repository.

### Multi-Alpha Algorithms

If you add [multiple Alpha models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/alpha/key-concepts#03-Multi-Alpha-Algorithms), each Alpha model receives the current slice in the order that you add the Alphas. The combined stream of Insight objects is passed to the Portfolio Construction model.

Each Portfolio Construction model has a unique method to combine Insight objects. The base `PortfolioConstructionModel` that most PCM's inherit from doesn't combine information from Insight objects with the same `Symbol` \- but just gets the most recent active insight. To combine the active insights differently, override the `GetTargetInsights` `get_target_insights`, and return all active insights. The `DetermineTargetPercent` `determine_target_percent` method implements the combination criteria and determines the target for each `Symbol`.

Select Language: C#Python

```
// Implement MultipleAlphaPortfolioConstructionModel to handle and utilize insights from multiple Alpha models.
// The get_target_insights method retrieves current active insights, and determine_target_percent allocates portfolio weights accordingly for integrating and balancing multiple Alpha signals within the portfolio.
public class MultipleAlphaPortfolioConstructionModel : PortfolioConstructionModel
{
    protected override List<Insight> GetTargetInsights()
    {
        return Algorithm.Insights.GetActiveInsights(Algorithm.UtcTime).ToList();
    }

    protected override Dictionary<Insight, double> DetermineTargetPercent(List<Insight> activeInsights)
    {
        return new Dictionary<Insight, double>();
    }
}
```

```
# Implement MultipleAlphaPortfolioConstructionModel to handle and utilize insights from multiple Alpha models.
# The get_target_insights method retrieves current active insights, and determine_target_percent allocates portfolio weights accordingly for integrating and balancing multiple Alpha signals within the portfolio.
class MultipleAlphaPortfolioConstructionModel(PortfolioConstructionModel):
    def get_target_insights(self) -> list[Insight]:
        return self.algorithm.insights.get_active_insights(self.algorithm.utc_time)

    def determine_target_percent(self, activeInsights: list[Insight]) -> dict[Insight, float]:
        return {}
```

### Portfolio Targets

The Portfolio Construction model returns `PortfolioTarget` objects, which are passed to the Risk Management model.

To create a `PortfolioTarget` object based on a quantity, pass the `Symbol` and quantity to the `PortfolioTarget` constructor.

Select Language: C#Python

```
// Create a new portfolio target for 1200 IBM shares.
var target = new PortfolioTarget("IBM", 1200);
```

```
# Create a new portfolio target for 1200 IBM shares.
target = PortfolioTarget("IBM", 1200)
```

To create a `PortfolioTarget` object based on a portfolio weight, call the `Percent` `percent` method. This method is only available for margin accounts.

Select Language: C#Python

```
// Calculate target equivalent to 10% of portfolio value
var target = PortfolioTarget.Percent(algorithm, "IBM", 0.1);
```

```
# Calculate target equivalent to 10% of portfolio value
target = PortfolioTarget.percent(algorithm, "IBM", 0.1)
```

To include more information in the `PortfolioTarget` object, pass a `tag` argument to the constructor or the `Percent` `percent` method.

The `CreateTargets` `create_targets` method of your Portfolio Construction model must return an array of `PortfolioTarget` objects.

Select Language: C#Python

```
return new PortfolioTarget[] {  new PortfolioTarget("IBM", 1200)  };
```

```
return [ PortfolioTarget("IBM", 1200) ]
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

### Rebalance Frequency

If you use a Portfolio Construction model that is a subclass of the `PortfolioConstructionModel` class, you can set the rebalancing frequency of the model with a function. The rebalancing function receives the Coordinated Universal Time (UTC) of the algorithm and should return the next rebalance UTC time or `None` `null`. If the function returns `None` `null`, the model doesn't rebalance unless the [rebalance settings](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/key-concepts#08-Rebalance-Settings) trigger a rebalance. For a full example of a custom rebalance function, see the [PortfolioRebalanceOnCustomFuncRegressionAlgorithm](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Python/PortfolioRebalanceOnCustomFuncRegressionAlgorithm.py)[PortfolioRebalanceOnCustomFuncRegressionAlgorithm](https://github.com/QuantConnect/Lean/blob/master/Algorithm.CSharp/PortfolioRebalanceOnCustomFuncRegressionAlgorithm.cs).

If you use a Portfolio Construction model with the following characteristics, you can also set the rebalancing frequency of the model with a `timedelta` `TimeSpan`, `Resolution`, or [DateRules](https://www.quantconnect.com/docs/v2/writing-algorithms/scheduled-events#04-Date-Rules):

- The model is a subclass of the `EqualWeightingPortfolioConstructionModel` class.
- The model constructor calls the `EqualWeightingPortfolioConstructionModel` constructor.
- The model doesn't override the `CreateTargets` `create_targets` method.

To check which of the pre-built Portfolio Construction models support this functionality, see [Supported Models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/supported-models).

### Rebalance Settings

By default, portfolio construction models create `PortfolioTarget` objects to rebalance the portfolio when any of the following events occur:

- The model's rebalance function signals it's time to rebalance
- The Alpha model emits new insights
- The universe changes

To disable rebalances when the Alpha model emits insights or when insights expire, set `RebalancePortfolioOnInsightChanges` `rebalance_portfolio_on_insight_changes` to false.

Select Language: C#Python

```
// Disable automatic portfolio rebalancing upon insight change, allowing for manual control over when portfolio adjustments are made based on insights.
Settings.RebalancePortfolioOnInsightChanges = false;
```

```
# Disable automatic portfolio rebalancing upon insight change, allowing for manual control over when portfolio adjustments are made based on insights.
self.settings.rebalance_portfolio_on_insight_changes = False
```

To disable rebalances when security changes occur, set `RebalancePortfolioOnSecurityChanges` `rebalance_portfolio_on_security_changes` to false.

Select Language: C#Python

```
// Disable automatic portfolio rebalancing upon security change, allowing for manual control over when portfolio adjustments are made based on security additions or removals.
Settings.RebalancePortfolioOnSecurityChanges = false;
```

```
# Disable automatic portfolio rebalancing upon security change, allowing for manual control over when portfolio adjustments are made based on security additions or removals.
self.settings.rebalance_portfolio_on_security_changes = False
```

### Portfolio Optimizer Structure

Some portfolio construction models contain an optimizer that accepts the historical returns of each security and returns a list of optimized portfolio weights. Portfolio optimizer models must implement the `IPortfolioOptimizer` interface, which has an `Optimize` `optimize` method.

Select Language: C#Python

```
// Implement an equal-weighted portfolio optimizer to assign equal weights to all securities, providing basic diversification to reduce risk compared to a concentrated portfolio.
public class MyPortfolioOptimizer : IPortfolioOptimizer
{
    public double[] Optimize(double[,] historicalReturns, double[] expectedReturns = null, double[,] covariance = null)
    {
        // Create weights
        //  For example, equal-weighting:
        int numAssets = historicalReturns.GetLength(1);
        var weights = Enumerable.Repeat(1.0 / numAssets, numAssets).ToArray();

        return weights;
    }
}
```

```
# Implement an equal-weighted portfolio optimizer to assign equal weights to all securities, providing basic diversification to reduce risk compared to a concentrated portfolio.
class MyPortfolioOptimizer:

    def optimize(self, historicalReturns: pd.DataFrame, expectedReturns: pd.Series = None, covariance: pd.DataFrame = None) -> pd.Series:
        # Create weights
        #  For example, equal-weighting:
        num_assets = historical_returns.shape[1]
        weights = [1/num_assets] * num_assets

        return weights
```

The following table describes the arguments the `Optimize` `optimize` method accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `historicalReturns` `historical_returns` | `double[,]` `DataFrame` | Matrix of historical returns where each column represents a security and each row returns for the given date/time (size: K x N) |  |
| `expectedReturns` `expected_returns` | `double[]` `Series` | Array of double with the portfolio annualized expected returns (size: K x 1) | `null` `None` |
| `covariance` | `double[,]` `DataFrame` | Multi-dimensional array of double with the portfolio covariance of annualized returns (size: K x K) | `null` `None` |

The method should return a K x 1 array of double objects that represent the portfolio weights.

To view all the pre-built portfolio optimization algorithms, see [Supported Optimizers](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/supported-optimizers).

To view a full example of an `IPortfolioOptimizer` implementation, see the [MaximumSharpeRatioPortfolioOptimizer](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/MaximumSharpeRatioPortfolioOptimizer.cs)[MaximumSharpeRatioPortfolioOptimizer](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Portfolio/MaximumSharpeRatioPortfolioOptimizer.py) in the LEAN GitHub repository.

If you define a custom optimizer and want to use it as the `optimizer` argument for one of the [pre-built Portfolio Construction models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/supported-models), import the Python version of the Portfolio Construction model into your project file. For example, to pair your optimizer with the [Black Litterman Optimization Model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/supported-models#09-Black-Litterman-Optimization-Model), add the following line:

Python

```
from Portfolio.black_litterman_optimization_portfolio_construction_model import BlackLittermanOptimizationPortfolioConstructionModel
```

### Track Security Changes

The [Universe Selection model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/key-concepts) may select a dynamic universe of assets, so you should not assume a fixed set of assets in the Portfolio Construction model. When the Universe Selection model adds and removes assets from the universe, it triggers an `OnSecuritiesChanged` `on_securities_changed` event. In the `OnSecuritiesChanged` `on_securities_changed` event handler, you can initialize the security-specific state or load any history required for your Portfolio Construction model. If you need to save data for individual securities, add custom members to the respective `Security` objectcast the `Security` object to a `dynamic` object and then save custom members to it.

Select Language: C#Python

```
class MyPortfolioConstructionModel : PortfolioConstructionModel{
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
class MyPortfolioConstructionModel(PortfolioConstructionModel):
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

### Universe Timing Considerations

If the Portfolio Construction model manages some indicators or [consolidators](https://www.quantconnect.com/docs/v2/writing-algorithms/consolidating-data/getting-started) for securities in the universe and the universe selection runs during the indicator sampling period or the consolidator aggregation period, the indicators and consolidators might be missing some data. For example, take the following scenario:

- The security resolution is minute
- You have a consolidator that aggregates the security data into daily bars to update the indicator
- The universe selection runs at noon

In this scenario, you create and [warm-up the indicator](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/manual-indicators#06-Warm-Up-Indicators) at noon. Since it runs at noon, the history request that gathers daily data to warm up the indicator won't contain any data from the current day and the consolidator that updates the indicator also won't aggregate any data from before noon. This process doesn't cause issues if the indicator only uses the close price to calculate the indicator value (like the [simple moving average](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/simple-moving-average) indicator) because the first consolidated bar that updates the indicator will have the correct close price. However, if the indicator uses more than just the close price to calculate its value (like the [True Range](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/true-range) indicator), the open, high, and low values of the first consolidated bar may be incorrect, causing the initial indicator values to be incorrect.

### Examples

The following examples demonstrate common practices for implementing the framework portfolio construction model.

#### Example 1: All-Weather Portfolio

The following algorithm uses `RiskParityPortfolioConstructionModel` to construct an all-weather portfolio that dissipates 1-year daily variance equally among ETFs that represent different asset classes, including stock market (SPY), bond (TLT), gold (GLD), oil (USO), and agricultural(DBA).

Select Language: C#Python

```
public class FrameworkPortfolioConstructionModelAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2024, 1, 1);
        SetEndDate(2024, 12, 1);

        // Add a universe of selected lists of ETFs that represent various asset classes.
        var etfs = new[] {
            QuantConnect.Symbol.Create("SPY", SecurityType.Equity, Market.USA),     // stock market
            QuantConnect.Symbol.Create("TLT", SecurityType.Equity, Market.USA),     // bond
            QuantConnect.Symbol.Create("GLD", SecurityType.Equity, Market.USA),     // gold
            QuantConnect.Symbol.Create("USO", SecurityType.Equity, Market.USA),     // oil
            QuantConnect.Symbol.Create("DBA", SecurityType.Equity, Market.USA)      // agricultural
        };
        AddUniverseSelection(new ManualUniverseSelectionModel(etfs));
        // Emit insights for all selected ETFs.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(1)));
        // Using risk parity PCM to construct an all-weather portfolio to dissipate risk.
        SetPortfolioConstruction(new RiskParityPortfolioConstructionModel());
    }
}
```

```
class FrameworkPortfolioConstructionModelAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2024, 1, 1)
        self.set_end_date(2024, 12, 1)

        # Add a universe of selected lists of ETFs that represent various asset classes.
        etfs = [\
            Symbol.create("SPY", SecurityType.EQUITY, Market.USA),  # stock market\
            Symbol.create("TLT", SecurityType.EQUITY, Market.USA),  # bond\
            Symbol.create("GLD", SecurityType.EQUITY, Market.USA),  # gold\
            Symbol.create("USO", SecurityType.EQUITY, Market.USA),  # oil\
            Symbol.create("DBA", SecurityType.EQUITY, Market.USA)   # agricultural\
        ]
        self.add_universe_selection(ManualUniverseSelectionModel(etfs))
        # Emit insights for all selected ETFs.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(1)))# Using risk parity PCM to construct an all-weather portfolio to dissipate risk.
        self.set_portfolio_construction(RiskParityPortfolioConstructionModel())
```

#### Example 2: Protective Position

The following algorithm implements an equal-weighting portfolio on the given insights. To hedge catastrophic downside risk, each position would order a protective call/put 10% OTM.

Select Language: C#Python

```
public class FrameworkPortfolioConstructionModelAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2024, 5, 1);
        SetEndDate(2024, 7, 1);
        SetCash(1000000);

        // Set to raw data normalization for the strike and underlying price fair comparison.
        UniverseSettings.DataNormalizationMode = DataNormalizationMode.Raw;
        // Add a universe of the market representative SPY.
        var spy = new[] {
            QuantConnect.Symbol.Create("SPY", SecurityType.Equity, Market.USA)
        };
        AddUniverseSelection(new ManualUniverseSelectionModel(spy));
        // Emit insights for all selected ETFs. Each insight lasts for 7 days for the option hedge expiry.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(7)));
        // Using a custom PCM to control the order size and hedge option position order with a 10% OTM threshold.
        SetPortfolioConstruction(new ProtectivePositionPortfolioConstructionModel(0.1m));
    }

    private class ProtectivePositionPortfolioConstructionModel : PortfolioConstructionModel
    {
        private readonly decimal _threshold;

        public ProtectivePositionPortfolioConstructionModel(decimal threshold)
        {
            _threshold = threshold;
        }

        public override List<PortfolioTarget> CreateTargets(QCAlgorithm algorithm, Insight[] insights)
        {
            var targets = new List<PortfolioTarget>();
            if (insights.Length == 0)
            {
                return targets;
            }

            // Equally invest in each position group to dissipate the capital risk.
            var fundPerInsight = algorithm.Portfolio.TotalPortfolioValue / insights.Length;

            foreach (var insight in insights)
            {
                var underlying = insight.Symbol;
                // Hedge position option right: long position should purchase OTM put, while short should purchase OTM call.
                var right = insight.Direction == InsightDirection.Up ? OptionRight.Put : OptionRight.Call;
                var threshold = insight.Direction == InsightDirection.Up ? 1m - _threshold : 1m + _threshold;

                // Select a protective option position that expires in 7 days.
                var hedge = algorithm.OptionChain(underlying)
                    .Where(x => x.Expiry < algorithm.Time.AddDays(7) && x.Right == right)
                    .OrderByDescending(x => x.Expiry)
                    .ThenBy(x => Math.Abs(x.Strike - x.UnderlyingLastPrice * threshold))
                    .First();
                // Request the protective position data for trading.
                var hedgeSymbol = algorithm.AddOptionContract(hedge).Symbol;

                // Each insight will be ordered by the position group of the underlying and the hedging option positions.
                var contractMultiplier = algorithm.Securities[underlying].SymbolProperties.ContractMultiplier;
                var quantity = Math.Floor(fundPerInsight / contractMultiplier / (hedge.BidPrice + hedge.UnderlyingLastPrice));
                targets.Add(new PortfolioTarget(underlying, (int)(quantity * contractMultiplier)));
                targets.Add(new PortfolioTarget(hedgeSymbol, (int)quantity));
            }

            return targets;
        }
    }
}
```

```
class FrameworkPortfolioConstructionModelAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2024, 5, 1)
        self.set_end_date(2024, 7, 1)
        self.set_cash(1000000)

        # Set to raw data normalization for a strike and the underlying price fair comparison.
        self.universe_settings.data_normalization_mode = DataNormalizationMode.RAW
        # Add a universe of the market representative SPY.
        spy = [Symbol.create("SPY", SecurityType.EQUITY, Market.USA)]
        self.add_universe_selection(ManualUniverseSelectionModel(spy))
        # Emit insights for all selected ETFs. Each insight lasts for 7 days for the option hedge expiry.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(7)))
        # Using a custom PCM to control the order size and hedge option position ordering with a 10% OTM threshold.
        self.set_portfolio_construction(ProtectivePositionPortfolioConstructionModel(0.1))

class ProtectivePositionPortfolioConstructionModel(PortfolioConstructionModel):
    def __init__(self, threshold: float) -> None:
        self.threshold = threshold

    def create_targets(self, algorithm: QCAlgorithm, insights: list[Insight]) -> list[PortfolioTarget]:
        targets = []
        if not insights:
            return targets

        # Equally invest in each position group to dissipate the capital risk.
        fund_per_insight = algorithm.portfolio.total_portfolio_value / len(insights)

        for insight in insights:
            underlying = insight.symbol
            # Hedge position option right: long position should purchase OTM put, while short should purchase OTM call.
            right = OptionRight.PUT if insight.direction == InsightDirection.UP else OptionRight.CALL
            threshold = 1 - self.threshold if insight.direction == InsightDirection.UP else 1 + self.threshold

            # Select a protective option position that expires in 7 days.
            hedge = sorted([x for x in algorithm.option_chain(underlying) if x.expiry < algorithm.time + timedelta(7) and x.right == right],
                           key=lambda x: (x.expiry, -abs(x.strike - x.underlying_last_price * threshold)),
                           reverse=True)[0]
            # Request the protective position data for trading.
            hedge_symbol = algorithm.add_option_contract(hedge).symbol

            # Each insight will be ordered by the position group of the underlying and the hedging option positions.
            contract_multiplier = algorithm.securities[underlying].symbol_properties.contract_multiplier
            quantity = fund_per_insight // (contract_multiplier * (hedge.bid_price + hedge.underlying_last_price))
            targets.append(PortfolioTarget(underlying, int(quantity * contract_multiplier)))
            targets.append(PortfolioTarget(hedge_symbol, int(quantity)))

        return targets
```

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/04%20Portfolio%20Construction/01%20Key%20Concepts)