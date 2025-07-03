# Universe Selection

## Key Concepts

### Introduction

![](https://cdn.quantconnect.com/web/i/docs/algorithm-framework/universe-selection.png)The Universe Selection model creates `Universe` objects, which select the assets for your algorithm. As the universe changes, we notify your algorithm through the `OnSecuritiesChanged` `on_securities_changed` event handler. With this event handler, you can track the current universe constituents in other parts of your algorithm without breaking the separation of concerns design principle.

### Types of Universe Selection

We have identified several types of universes that cover most people's requirements and built helper classes to make their implementation easier. The following table describes the types of pre-built Universe Selection models:

| Universe Type | Description |
| --- | --- |
| [Manual Universes](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/manual-universes) | Universes that use a fixed, static set of assets |
| [Fundamental Universes](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/fundamental-universes) | Universes for US Equities that are based on coarse price or fundamental data |
| [Scheduled Universes](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/scheduled-universes) | Universes that trigger on regular, custom intervals |
| [Futures Universes](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/futures-universes) | Universes that subscribe to Future chains |
| [Option Universes](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/options-universes) | Universes that subscribe to Option chains |

### Add Models

To add a Universe Selection model, in the `Initialize` `initialize` method, call the `AddUniverseSelection` method.

Select Language: C#Python

```
var symbols = new [] {QuantConnect.Symbol.Create("SPY", SecurityType.Equity, Market.USA)};
AddUniverseSelection(new ManualUniverseSelectionModel(symbols));
```

```
# Adds a universe selection model to the algorithm, manually defining the set of securities to include in the algorithm.
symbols = [Symbol.create("SPY", SecurityType.EQUITY, Market.USA)]
self.add_universe_selection(ManualUniverseSelectionModel(symbols))
```

To view all the pre-built Universe Selection models, see [Types of Universe Selection](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/key-concepts#02-Types-of-Universe-Selection).

### Multi-Universe Algorithms

You can add multiple Universe Selection models to a single algorithm.

Select Language: C#Python

```
// Configures asynchronous universe settings for faster updates and includes both manual and EMA cross universe selection models for dynamic and static selection.
UniverseSettings.Asynchronous = true;
var symbols = new [] {QuantConnect.Symbol.Create("SPY", SecurityType.Equity, Market.USA)};
AddUniverseSelection(new ManualUniverseSelectionModel(symbols));

AddUniverseSelection(new EmaCrossUniverseSelectionModel());
```

```
# Configures asynchronous universe settings for faster updates and includes both manual and EMA cross universe selection models for dynamic and static selection.
self.universe_settings.asynchronous = True
symbols = [Symbol.create("SPY", SecurityType.EQUITY, Market.USA)]
self.add_universe_selection(ManualUniverseSelectionModel(symbols))

self.add_universe_selection(EmaCrossUniverseSelectionModel())
```

If you add multiple Universe Selection models, the algorithm subscribes to the constituents of all the models.

### Model Structure

Universe Selection models should extend the `UniverseSelectionModel` class. Extensions of the `UniverseSelectionModel` class must implement the `CreateUniverses` `create_universes` method, which receives an `algorithm` object and returns an array of `Universe` objects.

Select Language: C#Python

```
// Extend UniverseSelectionModel to define a custom security universe tailored to the algorithm.
public class MyUniverseSelectionModel : UniverseSelectionModel
{
    // Creates the universes for this algorithm
    public override IEnumerable<Universe> CreateUniverses(QCAlgorithm algorithm)
    {
        return new List<Universe>();
    }

    // Gets the next time the framework should invoke the `CreateUniverses` method to refresh the set of universes.
    public override DateTime GetNextRefreshTimeUtc()
    {
        return DateTime.MaxValue;
    }
}
```

```
# Extend UniverseSelectionModel to define a custom security universe tailored to the algorithm.
class MyUniverseSelectionModel(UniverseSelectionModel):

    # Creates the universes for this algorithm
    def create_universes(self, algorithm: QCAlgorithm) -> list[Universe]:
        universes = []
        return universes

    # Gets the next time the framework should invoke the `CreateUniverses` method to refresh the set of universes.
    def get_next_refresh_time_utc(self):
        return datetime.max
```

The `algorithm` argument that the methods receive is an instance of the base `QCAlgorithm` class, not your subclass of it.

Generally, you should be able to extend one of the pre-built Universe Selection types. If you need to do something that doesn't fit into the pre-built types, let us know and we'll create a new foundational type of Universe Selection model.

### Track Security Changes

When the Universe Selection model adjusts the universe constituents, we notify your algorithm through the `OnSecuritiesChanged` `on_securities_changed` event handler on your other framework components. The method receives `QCAlgorithm` and `SecurityChanges` objects. The `QCAlgorithm` object is an instance of the base QCAlgorithm class, not a reference to your algorithm object. To access the added securities, check the `changes.AddedSecurities` `changes.added_securities` method property. To access the removed securities, check the `changes.RemovedSecurities` `changes.removed_securities` method property.

Select Language: C#Python

```
// Log added and removed securities to update the universe and adapt the strategy.
public override void OnSecuritiesChanged(QCAlgorithm algorithm, SecurityChanges changes)
{
    foreach (var security in changes.AddedSecurities)
    {
        Log($"Added {security.Symbol}");
    }

    foreach (var security in changes.RemovedSecurities)
    {
        Log($"Removed {security.Symbol}");
    }
}
```

```
# Log added and removed securities to update the universe and adapt the strategy.
def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
    for security in changes.added_securities::
        self.log(f"Added {security.symbol}")

    for security in changes.removed_securities:
        self.log(f"Removed {security.symbol}")
```

### Live Trading Considerations

In live trading, the securities in your brokerage account are added to your user-defined universe. If your Universe Selection model doesn't select these securities, they are not removed from your user-defined universe. When you liquidate the positions, you can remove them from the user-defined universe. If you call the `RemoveSecurity` `remove_security` method, it automatically liquidates the position.

To see the securities in your user-defined universe that were loaded from your brokerage account, check your algorithm's [ActiveSecuritiesactive\_securities](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/key-concepts#10-Active-Securities) in the [OnWarmupFinishedon\_warmup\_finished](https://www.quantconnect.com/docs/v2/writing-algorithms/key-concepts/event-handlers#07-Warmup-Finished-Event) event handler.

### Examples

The following examples demonstrate some common practices for implementing the universe selection model.

#### Example 1: Liquid Universe Selection Model

The following algorithm selects a universe of the top 20 liquid equity through the framework universe selection model. Then, buying and holding these popular equities equally to bet on their high demand will drive up the price.

Select Language: C#Python

```
public class FrameworkUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2019, 1, 1);
        SetEndDate(2024, 12, 1);
        SetCash(100000);

        // Rebalance weekly to allow time to capitalize on the trend
        UniverseSettings.Schedule.On(DateRules.WeekStart());
        // Since the selection is based on daily data, we only require daily data to trade daily.
        UniverseSettings.Resolution = Resolution.Daily;
        // Add universe selection model to select the most liquid stocks to trade with.
        AddUniverseSelection(new FundamentalUniverseSelectionModel(Selection));

        // Sent insights on buying and holding the most liquid stocks for a week.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(7)));
        // Evenly dissipate the capital risk among selected stocks.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }

    private IEnumerable<Symbol> Selection(IEnumerable<Fundamental> fundamental)
    {
        // Select the top 20 liquid equity to trade the popularity, suggesting a higher demand to drive the price.
        return fundamental.OrderByDescending(x => x.DollarVolume)
            .Take(20)
            .Select(x => x.Symbol);
    }
}
```

```
class FrameworkUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2019, 1, 1)
        self.set_end_date(2024, 12, 1)
        self.set_cash(100000)

        # Rebalance weekly to allow time to capitalize on the trend
        self.universe_settings.schedule.on(self.date_rules.week_start())
        # Since the selection is based on daily data, we only require daily data to trade daily.
        self.universe_settings.resolution = Resolution.DAILY
        # Add a universe selection model to select the most liquid stocks to trade with.
        self.add_universe_selection(FundamentalUniverseSelectionModel(self.selection))

        # Sent insights on buying and holding the most liquid stocks weekly.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(7)))
        # Evenly dissipate the capital risk among selected stocks.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

    def selection(self, fundamental: list[Fundamental]) -> list[Symbol]:
        # Select the top 20 liquid equity to trade the popularity, suggesting a higher demand to drive the price.
        sorted_by_dollar_volume = sorted(fundamental, key=lambda x: x.dollar_volume, reverse=True)
        return [x.symbol for x in sorted_by_dollar_volume[:20]]
```

#### Example 2: Multiple Universe Selection Models

The following algorithm implements two framework universe selection models to manually select the most liquid market ETFs and those with EMA crossing events to trade on trend changes. Then, buy and hold these equities equally to bet on the insights and dissipate the capital risk.

Select Language: C#Python

```
public class FrameworkUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2019, 1, 1);
        SetEndDate(2024, 12, 1);
        SetCash(100000);

        // Rebalance weekly to allow time to capitalize on the trend
        UniverseSettings.Schedule.On(DateRules.WeekStart());
        // Since the selection is based on daily data, we only require daily data to trade daily.
        UniverseSettings.Resolution = Resolution.Daily;
        // Add a universe selection model to select the most liquid ETFs manually.
        var symbols = new [] {
            QuantConnect.Symbol.Create("SPY", SecurityType.Equity, Market.USA),
            QuantConnect.Symbol.Create("QQQ", SecurityType.Equity, Market.USA)
        };
        AddUniverseSelection(new ManualUniverseSelectionModel(symbols));
        // Add another universe selection model to select the ones with EMA crosses.
        AddUniverseSelection(new EmaCrossUniverseSelectionModel());

        // Sent insights on buying and holding the most liquid stocks for a week.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(7)));
        // Evenly dissipate the capital risk among selected stocks.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }
}
```

```
class FrameworkUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2019, 1, 1)
        self.set_end_date(2024, 12, 1)
        self.set_cash(100000)

        # Rebalance weekly to allow time to capitalize on the trend
        self.universe_settings.schedule.on(self.date_rules.week_start())
        # Since the selection is based on daily data, we only require daily data to trade daily.
        self.universe_settings.resolution = Resolution.DAILY
        # Add a universe selection model to select the most liquid ETFs manually.
        symbols = [\
            Symbol.create("SPY", SecurityType.EQUITY, Market.USA),\
            Symbol.create("QQQ", SecurityType.EQUITY, Market.USA)\
        ]
        self.add_universe_selection(ManualUniverseSelectionModel(symbols))
        # Add another universe selection model to select the ones with EMA crosses.
        self.add_universe_selection(EmaCrossUniverseSelectionModel())

        # Sent insights on buying and holding the most liquid stocks weekly.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(7)))
        # Evenly dissipate the capital risk among selected stocks.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())
```

#### Other Examples

For more examples, see the following algorithms:

Demonstration Algorithms

[Manual Selection - BasicTemplateFrameworkAlgorithm.py Python](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Python/BasicTemplateFrameworkAlgorithm.py) [Fundamental Selection - QC500UniverseSelectionModel.py Python](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/QC500UniverseSelectionModel.py) [Scheduled Selection - ScheduledUniverseSelectionModelRegressionAlgorithm.py Python](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Python/ScheduledUniverseSelectionModelRegressionAlgorithm.py) [Manual Selection - BasicTemplateFrameworkAlgorithm.cs C#](https://github.com/QuantConnect/Lean/blob/master/Algorithm.CSharp/BasicTemplateFrameworkAlgorithm.cs) [Fundamental Selection - QC500UniverseSelectionModel.cs C#](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/QC500UniverseSelectionModel.cs) [Scheduled Selection - ScheduledUniverseSelectionModelRegressionAlgorithm.cs C#](https://github.com/QuantConnect/Lean/blob/master/Algorithm.CSharp/ScheduledUniverseSelectionModelRegressionAlgorithm.cs)

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/02%20Universe%20Selection/01%20Key%20Concepts)