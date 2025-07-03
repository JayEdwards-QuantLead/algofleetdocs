# Algorithm Framework

## Hybrid Algorithms

### Introduction

Classic style algorithms can also use Algorithm Framework modules. This allows you to get the best of both styles of algorithm design, combining easily pluggable modules with the superior control of classic format.

Examples of popular use-cases of a hybrid approach are:

- Using framework universe selection models as the assets to select.
- Using portfolio construction models to decide portfolio allocations.
- Using a risk control model for free portfolio risk monitoring.
- Passing data between modules freely without concerns of the module interface.

### Universe Selection

You can add one or more Framework Universe Selection Models to your algorithm and it will operate normally. You can also combine it with the `AddUniverse` `add_universe` method of the classic approach. The following example initializes a classic algorithm with framework universe selection models:

Select Language: C#Python

```
// Use a hybrid approach by combining Framework Universe Selection Models with classic methods to leverage both the modular flexibility of Framework modules and the control of classic design. This allows for better universe selection, portfolio allocation, and risk management.
public override void Initialize()
{
    UniverseSettings.Asynchronous = true;
    SetUniverseSelection(new ManualUniverseSelectionModel(QuantConnect.Symbol.Create("SPY", SecurityType.Equity, Market.USA)));
    AddUniverseSelection(new ManualUniverseSelectionModel(QuantConnect.Symbol.Create("AAPL", SecurityType.Equity, Market.USA)));
    AddUniverse(Universe.DollarVolume.Top(5));
}
```

```
# Use a hybrid approach by combining Framework Universe Selection Models with classic methods to leverage both the modular flexibility of Framework modules and the control of classic design. This allows for better universe selection, portfolio allocation, and risk management.
def initialize(self):
    self.universe_settings.asynchronous = True
    self.set_universe_selection(ManualUniverseSelectionModel([ Symbol.create("SPY", SecurityType.EQUITY, Market.USA) ]))
    self.add_universe_selection(ManualUniverseSelectionModel([ Symbol.create("AAPL", SecurityType.EQUITY, Market.USA) ]))
    self.add_universe(self.universe.dollar_volume.top(5))
```

### Alpha

You can add one or multiple [Alpha](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/alpha/key-concepts) models to your classic algorithm and place the orders using `Insight` objects without a Portfolio Construction model. To receive the collection of `Insight` objects in a classic algorithm, implement the `InsightsGenerated` `insights_generated` event handler:

Select Language: C#Python

```
// Add one or multiple Alpha models to a classic algorithm allowing for direct trading based on generated Insight objects, bypassing the need for a Portfolio Construction model.
public override void Initialize()
{
    AddAlpha(new EmaCrossAlphaModel());
    // The InsightsGenerated event handler is used to receive and process the collection of insights, facilitating flexible and dynamic trading strategies.
    InsightsGenerated += OnInsightsGenerated;
}
private void OnInsightsGenerated(IAlgorithm algorithm, GeneratedInsightsCollection insightsCollection)
{
    var insights = insightsCollection.Insights;
}
```

```
# Add one or multiple Alpha models to a classic algorithm allowing for direct trading based on generated Insight objects, bypassing the need for a Portfolio Construction model.
def initialize(self):
    self.add_alpha(EmaCrossAlphaModel())
    # The insights_generated event handler is used to receive and process the collection of insights, facilitating flexible and dynamic trading strategies.
    self.insights_generated += self.on_insights_generated

def on_insights_generated(self, algorithm: IAlgorithm, insights_collection: GeneratedInsightsCollection) -> None:
    insights = insights_collection.insights
```

### Portfolio Construction

You can add a Portfolio Construction model to your classic algorithm and have it place orders without returning `Insight` objects from an Alpha model. To emit insights without an Alpha model, in the `OnData` `on_data` method, call the `EmitInsights` method.

The following example uses a Portfolio Construction Framework model with `EmitInsights` method in a classic algorithm:

Select Language: C#Python

```
// Add a Portfolio Construction model to place orders directly without relying on Alpha-generated insights.
public override void Initialize()
{
    SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
}
// Use the EmitInsights method in the OnData method to provide trading signals for executing trades, ensuring balanced investment with the chosen Portfolio Construction model.
public override void OnData(Slice slice)
{
	EmitInsights(new Insight("GOOG", TimeSpan.FromMinutes(20), InsightType.Price, InsightDirection.Up));
	EmitInsights(new []{
		new Insight("AAPL", TimeSpan.FromMinutes(20), InsightType.Price, InsightDirection.Up),
		new Insight("MSFT", TimeSpan.FromMinutes(20), InsightType.Price, InsightDirection.Up)
	});
}
```

```
# Add a Portfolio Construction model to place orders directly without relying on Alpha-generated insights.
def initialize(self):
    self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

# Use the emit_insights method in the on_data method to provide trading signals for executing trades, ensuring balanced investment with the chosen Portfolio Construction model.
def on_data(self, slice):
	self.emit_insights(Insight("GOOG", TimeSpan.from_minutes(20), InsightType.PRICE, InsightDirection.UP))
	self.emit_insights([\
		Insight("AAPL", TimeSpan.from_minutes(20), InsightType.PRICE, InsightDirection.UP),\
		Insight("MSFT", TimeSpan.from_minutes(20), InsightType.PRICE, InsightDirection.UP)\
	])
```

### Risk Management

Some Risk Management Models don't require a Portfolio Construction model to provide `PortfolioTarget` objects, allowing them to directly monitor the portfolio holdings and liquidate positions when neccessary. To see which pre-built Risk Management models don't need the Portfolio Construction model to provide `PortfolioTarget` objects, see [Supported Models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/execution/supported-models).

You can add one or more Risk Management Models to your algorithm and it will operate normally. The following example initializes a classic algorithm with framework risk management models:

Select Language: C#Python

```
// Add Risk Management models to monitor portfolio risk directly, without needing PortfolioTargets from a Portfolio Construction model. These models can independently manage holdings and trigger liquidations as needed.
public override void Initialize()
{
    AddRiskManagement(new MaximumDrawdownPercentPerSecurity(0.05m));
    AddRiskManagement(new MaximumUnrealizedProfitPercentPerSecurity(0.1m));
}
```

```
# Add Risk Management models to monitor portfolio risk directly, without needing PortfolioTargets from a Portfolio Construction model. These models can independently manage holdings and trigger liquidations as needed.
def initialize(self):
    self.add_risk_management(MaximumDrawdownPercentPerSecurity(0.05))
    self.add_risk_management(MaximumUnrealizedProfitPercentPerSecurity(0.1))
```

### Execution

[Execution models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/execution/supported-models) can place orders for your strategy instead of placing them manually.
LEAN routes [PortfolioTarget](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/execution/key-concepts#05-Portfolio-Target-Collection) objects to the `Execute` `execute` method of the Execution Model to place orders.

The following example uses a Framework Execution Model in a classic style algorithm:

Select Language: C#Python

```
// Use SetExecution to place orders via the Execution Model, which routes PortfolioTarget objects to its
// execute method, instead of placing orders manually. This ensures automated and consistent order execution.
public override void Initialize()
{
    // Execute the market order imediately to fill the portfolio targets.
    SetExecution(new ImmediateExecutionModel());
}
```

```
# Use set_execution to place orders via the Execution Model, which routes PortfolioTarget objects to its
# execute method, instead of placing orders manually. This ensures automated and consistent order execution.
def initialize(self):
    # Execute the market order imediately to fill the portfolio targets.
    self.set_execution(ImmediateExecutionModel())
```

### Universe Timing Considerations

If the Alpha, Portfolio Construction, Risk Management, or Execution model manages some indicators or [consolidators](https://www.quantconnect.com/docs/v2/writing-algorithms/consolidating-data/getting-started) for securities in the universe and the universe selection runs during the indicator sampling period or the consolidator aggregation period, the indicators and consolidators might be missing some data. For example, take the following scenario:

- The security resolution is minute
- You have a consolidator that aggregates the security data into daily bars to update the indicator
- The universe selection runs at noon

In this scenario, you create and [warm-up the indicator](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/manual-indicators#06-Warm-Up-Indicators) at noon. Since it runs at noon, the history request that gathers daily data to warm up the indicator won't contain any data from the current day and the consolidator that updates the indicator also won't aggregate any data from before noon. This process doesn't cause issues if the indicator only uses the close price to calculate the indicator value (like the [simple moving average](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/simple-moving-average) indicator) because the first consolidated bar that updates the indicator will have the correct close price. However, if the indicator uses more than just the close price to calculate its value (like the [True Range](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/true-range) indicator), the open, high, and low values of the first consolidated bar may be incorrect, causing the initial indicator values to be incorrect.

### Examples

The following examples demonstrate common practices for implementing hybrid algorithms.

#### Example 1: Corresponding Alpha

The following algorithm trades two separate logics:

- EMA cross on the 20 most liquid stocks
- Buy and hold the top 20 weighted SPY constituents

To do so, we can save the universe and allow an access point in the corresponding alpha model to filter the signals that are only applicable to the selected equities from each universe.

Select Language: C#Python

```
public class FrameworkHybridAlgorithm : QCAlgorithm
{
    public Universe Universe1, Universe2;

    public override void Initialize()
    {
        SetStartDate(2024, 1, 1);
        SetEndDate(2024, 4, 1);
        SetCash(1000000);

        // Add a universe of the most liquid stocks since their trend is more capital-supported to trend EMA cross.
        Universe1 = AddUniverse(LiquidSelection);
        // Emit insights if the trend of selected liquid stocks changes.
        AddAlpha(new CustomEmaCrossAlphaModel(this));

        // Add another universe set to pick the most weighted SPY constituents since they usually have excess return compared to the rest.
        Universe2 = AddUniverse(Universe.ETF("SPY", Market.USA, UniverseSettings, EtfConstituentFilter));
        // Emit insights for all selected SPY constituents.
        AddAlpha(new CustomConstantAlphaModel(this));

        // Equal weighting on each insight is needed to dissipate capital risk evenly.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }

    private IEnumerable<Symbol> LiquidSelection(IEnumerable<Fundamental> fundamentals)
    {
        // Select the top 20 liquid equities for EMA cross trading.
        return (from f in fundamentals
            orderby f.DollarVolume descending
            select f.Symbol).Take(20);
    }

    private IEnumerable<Symbol> EtfConstituentFilter(IEnumerable<ETFConstituentUniverse> constituents)
    {
        // Select the top 20 weighted stocks of SPY constituents to hold.
        return (from c in constituents
            where c.Weight.HasValue
            orderby c.Weight.Value descending
            select c.Symbol).Take(20);
    }

    private class CustomEmaCrossAlphaModel : EmaCrossAlphaModel
    {
        private FrameworkHybridAlgorithm _algorithm;
        private readonly int _fastPeriod, _slowPeriod;
        private readonly Resolution _resolution;

        public CustomEmaCrossAlphaModel(FrameworkHybridAlgorithm algorithm, int fastPeriod = 20, int slowPeriod = 60, Resolution resolution = Resolution.Daily)
            : base(fastPeriod, slowPeriod, resolution)
        {
            _algorithm = algorithm;
            _fastPeriod = fastPeriod;
            _slowPeriod = slowPeriod;
            _resolution = resolution;
        }

        public override void OnSecuritiesChanged(QCAlgorithm algorithm, SecurityChanges changes)
        {
            foreach (var added in changes.AddedSecurities)
            {
                // Only trade the liquid universe for this alpha model.
                if (_algorithm.Universe1.Selected.Contains(added.Symbol))
                {
                    SymbolData symbolData;
                    if (!SymbolDataBySymbol.TryGetValue(added.Symbol, out symbolData))
                    {
                        SymbolDataBySymbol[added.Symbol] = new SymbolData(added, _fastPeriod, _slowPeriod, algorithm, _resolution);
                    }
                    else
                    {
                        // A security that was already initialized was re-added, reset the indicators.
                        symbolData.Fast.Reset();
                        symbolData.Slow.Reset();
                    }
                }
            }

            foreach (var removed in changes.RemovedSecurities)
            {
                SymbolData symbolData;
                if (SymbolDataBySymbol.TryGetValue(removed.Symbol, out symbolData))
                {
                    // clean up our consolidators.
                    symbolData.RemoveConsolidators();
                    SymbolDataBySymbol.Remove(removed.Symbol);
                }
            }
        }
    }

    private class CustomConstantAlphaModel : ConstantAlphaModel
    {
        private FrameworkHybridAlgorithm _algorithm;

        public CustomConstantAlphaModel(FrameworkHybridAlgorithm algorithm)
            // Daily insight length due to daily signals.
            : base(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(1))
        {
            _algorithm = algorithm;
        }

        public override IEnumerable<Insight> Update(QCAlgorithm algorithm, Slice data)
        {
            var insights = base.Update(algorithm, data);
            // Only trade the selected SPY constituents universe for this alpha model.
            return insights.Where(x => _algorithm.Universe2.Selected.Contains(x.Symbol));
        }
    }
}
```

```
from Alphas.EmaCrossAlphaModel import EmaCrossAlphaModel, SymbolData
from Alphas.ConstantAlphaModel import ConstantAlphaModel

class FrameworkHybridAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2024, 1, 1)
        self.set_end_date(2024, 4, 1)

        # Add a universe of the most liquid stocks since their trend is more capital-supported to trend EMA cross.
        self.universe1 = self.add_universe(self.liquid_selection)
        # Emit insights if the trend of selected liquid stocks changes.
        self.add_alpha(CustomEmaCrossAlphaModel(self))

        # Add another universe set to pick the most weighted SPY constituents since they usually have excess return compared to the rest.
        self.universe2 = self.add_universe(self.universe.etf("SPY", Market.USA, self.universe_settings, self.etf_constituents_filter))
        # Emit insights for all selected SPY constituents.
        self.add_alpha(CustomConstantAlphaModel(self))

        # Equal weighting on each insight to dissipate capital risk evenly.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

    def liquid_selection(self, fundamentals: list[Fundamental]) -> list[Symbol]:
        # Select the top 20 liquid equities for EMA cross trading.
        return [x.symbol for x in sorted(fundamentals, key=lambda f: f.dollar_volume, reverse=True)[:20]]

    def etf_constituents_filter(self, constituents: list[ETFConstituentUniverse]) -> list[Symbol]:
        # Select the top 20 weighted stocks of SPY constituents to hold.
        return [x.symbol for x in sorted([c for c in constituents if c.weight], key=lambda c: c.weight, reverse=True)[:20]]

class CustomEmaCrossAlphaModel(EmaCrossAlphaModel):
    def __init__(self, algorithm: FrameworkHybridAlgorithm, fast_period: int = 20, slow_period: int = 60, resolution: Resolution = Resolution.DAILY) -> None:
        self.algorithm = algorithm
        super().__init__(fast_period, slow_period, resolution)

    def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
        for added in changes.added_securities:
            # Only trade the liquid universe for this alpha model.
            if added.symbol in self.algorithm.universe1.selected:
                symbol_data = self.symbol_data_by_symbol.get(added.symbol)
                if symbol_data is None:
                    symbol_data = SymbolData(added, self.fast_period, self.slow_period, algorithm, self.resolution)
                    self.symbol_data_by_symbol[added.symbol] = symbol_data
                else:
                    # A security that was already initialized was re-added, reset the indicators.
                    symbol_data.fast.reset()
                    symbol_data.slow.reset()

        for removed in changes.removed_securities:
            data = self.symbol_data_by_symbol.pop(removed.symbol, None)
            if data is not None:
                # clean up our consolidators.
                data.remove_consolidators()

class CustomConstantAlphaModel(ConstantAlphaModel):
    def __init__(self, algorithm: FrameworkHybridAlgorithm) -> None:
        self.algorithm = algorithm
        # Daily insight length due to daily signals.
        super().__init__(InsightType.PRICE, InsightDirection.UP, timedelta(1))

    def update(self, algorithm: QCAlgorithm, data: Slice) -> list[Insight]:
        insights = super().update(algorithm, data)
        # Only trade the selected SPY constituents universe for this alpha model.
        return [i for i in insights if i.symbol in self.algorithm.universe2.selected]
```

#### Example 2: Bracket Order On Insight

The following algorithm uses framework universe selection and alpha model to filter the universe and generate insights. However, the portfolio construction model cannot handle bracket orders easily. Hence, we will access the active insights to place bracket orders using the classic implementation, with stop losses at 1% and take profits at 2%.

Select Language: C#Python

```
public class FrameworkHybridAlgorithm : QCAlgorithm
{
    private List<Insight> _orderedInsights = new();

    public override void Initialize()
    {
        SetStartDate(2024, 1, 1);
        SetEndDate(2024, 4, 1);
        SetCash(1000000);

        // Add a universe of the most liquid stocks since their trend is more capital-supported to trend EMA cross.
        AddUniverse(LiquidSelection);
        // Emit insights if the trend of selected liquid stocks changes.
        AddAlpha(new EmaCrossAlphaModel(20, 60, Resolution.Daily));
    }

    private IEnumerable<Symbol> LiquidSelection(IEnumerable<Fundamental> fundamentals)
    {
        // Select the top 20 liquid equities for EMA cross trading.
        return (from f in fundamentals
            orderby f.DollarVolume descending
            select f.Symbol).Take(20);
    }

    public override void OnData(Slice slice)
    {
        // Exit positions if insight is expired.
        foreach (var insight in new List<Insight>(_orderedInsights))
        {
            // If the insight is expired or profit taken or stop loss
            if (insight.IsExpired(UtcTime) || !Portfolio[insight.Symbol].Invested)
            {
                // Discontinue the insight.
                Insights.Cancel(new[] { insight });
                // Liquidate any positions and any open (bracket) orders.
                Liquidate(insight.Symbol);
                // Remove from the cache.
                _orderedInsights.Remove(insight);
            }
        }

        // Place orders for active insights if not yet ordered.
        var activeInsights = Insights.GetActiveInsights(UtcTime);
        foreach (var insight in activeInsights)
        {
            if (!_orderedInsights.Contains(insight))
            {
                // Position sizing by given weight or equal weighting to dissipate capital risk.
                var size = 0d;
                if (insight.Weight.HasValue)
                {
                    size = insight.Weight.Value;
                }
                else
                {
                    size = 1d / activeInsights.Count;
                }
                SetHoldings(insight.Symbol, size);
                // Add to cache to avoid re-order.
                _orderedInsights.Add(insight);
            }
        }
    }

    public override void OnOrderEvent(OrderEvent orderEvent)
    {
        if (orderEvent.Status == OrderStatus.Filled)
        {
            if (orderEvent.Ticket.OrderType == OrderType.Market)
            {
                // Stop loss order at 1%.
                var stopPrice = orderEvent.FillQuantity > 0m ? orderEvent.FillPrice * 0.99m : orderEvent.FillPrice * 1.01m;
                StopMarketOrder(orderEvent.Symbol, -Portfolio[orderEvent.Symbol].Quantity, stopPrice);
                // Take profit order at 2%.
                var takeProfitPrice = orderEvent.FillQuantity > 0m ? orderEvent.FillPrice * 1.02m : orderEvent.FillPrice * 0.98m;
                LimitOrder(orderEvent.Symbol, -Portfolio[orderEvent.Symbol].Quantity, takeProfitPrice);
            }
            else if (orderEvent.Ticket.OrderType == OrderType.StopMarket || orderEvent.Ticket.OrderType == OrderType.Limit)
            {
                // Cancel any open order if stop loss or take profit order filled.
                Transactions.CancelOpenOrders();
            }
        }
    }
}
```

```
class FrameworkHybridAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2024, 1, 1)
        self.set_end_date(2024, 4, 1)

        # Add a universe of the most liquid stocks since their trend is more capital-supported to trend EMA cross.
        self.universe1 = self.add_universe(self.liquid_selection)
        # Emit insights if the trend of selected liquid stocks changes.
        self.add_alpha(EmaCrossAlphaModel(20, 60, Resolution.DAILY))

        self.ordered_insights = []

    def liquid_selection(self, fundamentals: list[Fundamental]) -> list[Symbol]:
        # Select the top 20 liquid equities for EMA cross trading.
        return [x.symbol for x in sorted(fundamentals, key=lambda f: f.dollar_volume, reverse=True)[:20]]

    def on_data(self, slice: Slice) -> None:
        # Exit positions if insight is expired.
        for insight in self.ordered_insights.copy():
            # If the insight is expired or profit or loss are taken.
            if insight.is_expired(self.utc_time) or not self.portfolio[insight.symbol].invested:
                # Discontinue the insight.
                self.insights.cancel([insight])
                # Liquidate any positions and any open (bracket) orders.
                self.liquidate(insight.symbol)
                # Remove from the cache.
                self.ordered_insights.remove(insight)

        # Place orders for active insights if it has not yet been ordered.
        active_insights = self.insights.get_active_insights(self.utc_time)
        for insight in active_insights:
            if insight not in self.ordered_insights:
                # Position sizing by given weight or equal weighting to dissipate capital risk.
                if insight.weight:
                    size = insight.weight
                else:
                    size = 1 / len(active_insights)
                self.set_holdings(insight.symbol, size)
                # Add to cache to avoid re-order.
                self.ordered_insights.append(insight)

    def on_order_event(self, order_event: OrderEvent) -> None:
        if order_event.status == OrderStatus.FILLED:
            if order_event.ticket.order_type == OrderType.MARKET:
                # Stop loss order at 1%.
                stop_price = order_event.fill_price * (0.99 if order_event.fill_quantity > 0 else 1.01)
                self.stop_market_order(order_event.symbol, -self.portfolio[order_event.symbol].quantity, stop_price)
                # Take profit order at 2%.
                take_profit_price = order_event.fill_price * (1.02 if order_event.fill_quantity > 0 else 0.98)
                self.limit_order(order_event.symbol, -self.portfolio[order_event.symbol].quantity, take_profit_price)
            elif order_event.ticket.order_type == OrderType.STOP_MARKET or order_event.ticket.order_type == OrderType.LIMIT:
                # Cancel any open order if stop loss or take profit order filled.
                self.transactions.cancel_open_orders()
```

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/07%20Hybrid%20Algorithms)