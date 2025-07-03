# Execution

## Key Concepts

### Introduction

![](https://cdn.quantconnect.com/web/i/docs/algorithm-framework/execute.png) The Execution model receives an array of risk-adjusted `PortfolioTarget` objects from the [Risk Management model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/risk-management/key-concepts) and places trades in the market to satisfy the targets. The Execution model only receives updates to the portfolio target share counts. It doesn't necessarily receive all of the targets at once.

### Set Models

To set an Execution model, in the `Initialize` `initialize` method, call the `SetExecution` `set_execution` method.

Select Language: C#Python

```
// Satisify portfolio targets by immediately sending market orders.
SetExecution(new ImmediateExecutionModel());
```

```
# Satisify portfolio targets by immediately sending market orders.
self.set_execution(ImmediateExecutionModel())
```

To view all the pre-built Execution models, see [Supported Models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/execution/supported-models).

### Model Structure

Execution models should extend the `ExecutionModel` class. Extensions of the `ExecutionModel` must implement the `Execute` `execute` method, which receives an array of `PortfolioTarget` objects at every [time step](https://www.quantconnect.com/docs/v2/writing-algorithms/key-concepts/time-modeling/timeslices) and is responsible for reaching the target portfolio as efficiently as possible. The [Portfolio Construction model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/key-concepts) creates the `PortfolioTarget` objects, the [Risk Management model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/risk-management/key-concepts) may adjust them, and then the Execution model places the orders to fulfill them.

Select Language: C#Python

```
 // Basic Execution Model Scaffolding Structure Example
class MyExecutionModel : ExecutionModel {

   // Fill the supplied portfolio targets efficiently.
   public override void Execute(QCAlgorithm algorithm, IPortfolioTarget[] targets)
   {
      // NOP
   }

   //  Optional: Securities changes event for handling new securities.
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
# Execution Model scaffolding structure example
class MyExecutionModel(ExecutionModel):

    # Fill the supplied portfolio targets efficiently
    def execute(self, algorithm: QCAlgorithm, targets: list[PortfolioTarget]) -> None:
        pass

    # Optional: Securities changes event for handling new securities.
    def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
        # Security additions and removals are pushed here.
        # This can be used for setting up algorithm state.
        # changes.added_securities
        # changes.removed_securities
        pass
```

The `algorithm` argument that the methods receive is an instance of the base `QCAlgorithm` class, not your subclass of it.

The following table describes the properties of the `PortfolioTarget` class that you may access in the Execution model:

| Property | Data Type | Description |
| --- | --- | --- |
| `Symbol` `symbol` | `Symbol` | Asset to trade |
| `Quantity` `quantity` | `decimal` `float` | Number of units to hold |

To view a full example of an `ExecutionModel` subclass, see the [ImmediateExecutionModel](https://github.com/QuantConnect/Lean/blob/master/Algorithm/Execution/ImmediateExecutionModel.cs)[ImmediateExecutionModel](https://github.com/QuantConnect/Lean/blob/master/Algorithm/Execution/ImmediateExecutionModel.py) in the LEAN GitHub repository.

### Track Security Changes

The [Universe Selection model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/key-concepts) may select a dynamic universe of assets, so you should not assume a fixed set of assets in the Execution model. When the Universe Selection model adds and removes assets from the universe, it triggers an `OnSecuritiesChanged` `on_securities_changed` event. In the `OnSecuritiesChanged` `on_securities_changed` event handler, you can initialize the security-specific state or load any history required for your Execution model. If you need to save data for individual securities, add custom members to the respective `Security` objectcast the `Security` object to a `dynamic` object and then save custom members to it.

Select Language: C#Python

```
class MyExecutionModel : ExecutionModel{
    private List<Security> _securities = new List<Security>();

    public override void OnSecuritiesChanged(QCAlgorithm algorithm, SecurityChanges changes)
    {
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
class MyExecutionModel(ExecutionModel):
    _securities = []

    def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
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

If the Execution model manages some indicators or [consolidators](https://www.quantconnect.com/docs/v2/writing-algorithms/consolidating-data/getting-started) for securities in the universe and the universe selection runs during the indicator sampling period or the consolidator aggregation period, the indicators and consolidators might be missing some data. For example, take the following scenario:

- The security resolution is minute
- You have a consolidator that aggregates the security data into daily bars to update the indicator
- The universe selection runs at noon

In this scenario, you create and [warm-up the indicator](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/manual-indicators#06-Warm-Up-Indicators) at noon. Since it runs at noon, the history request that gathers daily data to warm up the indicator won't contain any data from the current day and the consolidator that updates the indicator also won't aggregate any data from before noon. This process doesn't cause issues if the indicator only uses the close price to calculate the indicator value (like the [simple moving average](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/simple-moving-average) indicator) because the first consolidated bar that updates the indicator will have the correct close price. However, if the indicator uses more than just the close price to calculate its value (like the [True Range](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/true-range) indicator), the open, high, and low values of the first consolidated bar may be incorrect, causing the initial indicator values to be incorrect.

### Examples

The following examples demonstrate some common practices for implementing the execution model.

#### Example 1: Iceberg Execution

The following algorithm simulates an account with large capital that equally holds the most liquid stocks. To hide the footprint and avoid a large market impact that might erode the profit margin, we can set up an iceberg execution system, submitting only 10% of the volume of the bid/ask side order book to the order direction.

Select Language: C#Python

```
public class FrameworkExecutionModelAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2020, 4, 1);
        SetEndDate(2021, 1, 1);
        SetCash(100000000);

        // Add a universe of the most liquid stocks since their trend is more capital-supported.
        AddUniverseSelection(new QC500UniverseSelectionModel());
        // Emit insights all for selected stocks.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(7)));
        // Equal weighting on each insight is needed to dissipate capital risk evenly.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());

        // Iceberg ordering to hide traces and avoid market impact.
        // Since quote data will be used, make sure the asset class and resolution are compatible.
        SetExecution(new IcebergExecutionModel(0.1m));
    }

    private class IcebergExecutionModel : ExecutionModel
    {
        private readonly PortfolioTargetCollection _targetsCollection = new PortfolioTargetCollection();
        // The maximum order size taken from the order book in percentage.
        private readonly decimal _maximumOrderQuantityPercentVolume;

        public IcebergExecutionModel(decimal orderPercentVolume = 0.1m)
        {
            _maximumOrderQuantityPercentVolume = orderPercentVolume;
        }

        public override void Execute(QCAlgorithm algorithm, IPortfolioTarget[] targets)
        {
            //Update the complete set of portfolio targets with the new targets
            _targetsCollection.AddRange(targets);

            if (!_targetsCollection.IsEmpty)
            {
                foreach (var target in _targetsCollection.OrderByMarginImpact(algorithm))
                {
                    var symbol = target.Symbol;

                    // Calculate the remaining quantity to be ordered
                    var unorderedQuantity = OrderSizing.GetUnorderedQuantity(algorithm, target);
                    // Adjust order size to respect maximum order size based on a percentage of the current volume
                    var orderSize = GetOrderSizeForPercentVolume(algorithm.Securities[symbol], _maximumOrderQuantityPercentVolume, unorderedQuantity);

                    if (orderSize != 0)
                    {
                        algorithm.MarketOrder(symbol, orderSize);
                    }
                }

                _targetsCollection.ClearFulfilled(algorithm);
            }
        }

        private static decimal GetOrderSizeForPercentVolume(Security security, decimal maximumPercentCurrentVolume, decimal desiredOrderSize)
        {
            // Take N% from the order book according to the order direction.
            var maxOrderSize = maximumPercentCurrentVolume * (desiredOrderSize > 0 ? security.BidSize : security.AskSize);
            var orderSize = Math.Min(maxOrderSize, Math.Abs(desiredOrderSize));

            return Math.Sign(desiredOrderSize) * OrderSizing.AdjustByLotSize(security, orderSize);
        }
    }
}
```

```
class FrameworkExecutionModelAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2020, 4, 1)
        self.set_end_date(2021, 1, 1)
        self.set_cash(100000000)

        # Add a universe of the most liquid stocks since their trend is more capital-supported.
        self.add_universe_selection(QC500UniverseSelectionModel())
        # Emit insights all for selected stocks.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(7)))
        # Equal weighting on each insight to dissipate capital risk evenly.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

        # Iceberg ordering to hide traces and avoid market impact.
        # Since quote data will be used, ensure the asset class and resolution are compatible.
        self.set_execution(IcebergExecutionModel(0.1))

class IcebergExecutionModel(ExecutionModel):
    def __init__(self, order_percent_volume: float = 0.1) -> None:
        self.targets_collection = PortfolioTargetCollection()
        # The maximum order size is taken from the order book in percentage.
        self.maximum_order_quantity_percent_volume = order_percent_volume

    def execute(self, algorithm: QCAlgorithm, targets: list[PortfolioTarget]) -> None:
        # update the complete set of portfolio targets with the new targets
        self.targets_collection.add_range(targets)

        if not self.targets_collection.is_empty:
            for target in self.targets_collection.order_by_margin_impact(algorithm):
                symbol = target.symbol

                # Calculate the remaining quantity to be ordered
                unordered_quantity = OrderSizing.get_unordered_quantity(algorithm, target)
                # adjust order size to respect maximum order size based on a percentage of current volume
                order_size = self.get_order_size_for_percent_volume(algorithm.securities[symbol], self.maximum_order_quantity_percent_volume, unordered_quantity)

                if order_size != 0:
                    algorithm.market_order(symbol, order_size)

            self.targets_collection.clear_fulfilled(algorithm)

    def get_order_size_for_percent_volume(self, security: Security, maximum_percent_current_volume: float, desired_order_size: float) -> float:
        # Take N% from the order book according to the order direction.
        max_order_size = maximum_percent_current_volume * (security.bid_size if desired_order_size > 0 else security.ask_size)
        order_size = min(max_order_size, abs(desired_order_size))
        return np.sign(desired_order_size) * OrderSizing.adjust_by_lot_size(security, order_size)
```

#### Example 2: Stop Loss Order

The following algorithm uses a custom execution model to add stop loss orders for the base order to control downside risk. To do so, we ordered a stop-loss order of the same size but in the opposite direction of the base order. We also handled the cancellation of the respective insight to avoid repeated ordering if the stop-loss order was being filled.

Select Language: C#Python

```
public class FrameworkExecutionModelAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2020, 4, 30);
        SetEndDate(2020, 5, 5);
        SetCash(1000000);
        UniverseSettings.Resolution = Resolution.Daily;

        // Add a universe of the most liquid stocks since their trend is more capital-supported.
        AddUniverseSelection(new QC500UniverseSelectionModel());
        // Emit insights all for selected stocks.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(7)));
        // Equal weighting on each insight is needed to dissipate capital risk evenly.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());

        // To place bracket orders besides basic orders as well.
        SetExecution(new BracketExecutionModel(0.05m));
    }

    private class BracketExecutionModel : ExecutionModel
    {
        private readonly PortfolioTargetCollection _targetsCollection = new PortfolioTargetCollection();
        // The stop loss percent of the bracket order.
        private readonly decimal _stopLoss;
        private List<OrderTicket> _stopOrders = new();

        public BracketExecutionModel(decimal stopLoss = 0.05m)
            : base()
        {
            _stopLoss = stopLoss;
        }

        public override void Execute(QCAlgorithm algorithm, IPortfolioTarget[] targets)
        {
            foreach (var stopLoss in new List<OrderTicket>(_stopOrders))
            {
                // Check if any bracket orders have been filled out. If so, cancel the insight to avoid repeated ordering.
                if (stopLoss.Status == OrderStatus.Filled)
                {
                    algorithm.Insights.Cancel(new[] {stopLoss.Symbol});
                    _stopOrders.Remove(stopLoss);
                }
            }

            _targetsCollection.AddRange(targets);
            if (!_targetsCollection.IsEmpty)
            {
                foreach (var target in _targetsCollection.OrderByMarginImpact(algorithm))
                {
                    var security = algorithm.Securities[target.Symbol];

                    // Calculate the remaining quantity to be ordered
                    var quantity = OrderSizing.GetUnorderedQuantity(algorithm, target, security, true);

                    if (quantity != 0)
                    {
                        if (security.BuyingPowerModel.AboveMinimumOrderMarginPortfolioPercentage(security, quantity,
                            algorithm.Portfolio, algorithm.Settings.MinimumOrderMarginPortfolioPercentage))
                        {
                            algorithm.MarketOrder(security, quantity);
                            // Stop loss order.
                            var stopLossPrice = security.Price * (quantity > 0 ? 1m - _stopLoss : 1m + _stopLoss);
                            var stopLoss = algorithm.StopMarketOrder(security.Symbol, -quantity, stopLossPrice);
                            // Save the orders to track their filing status.
                            _stopOrders.Add(stopLoss);
                        }
                        else if (!PortfolioTarget.MinimumOrderMarginPercentageWarningSent.HasValue)
                        {
                            // will trigger the warning if it has not already been sent
                            PortfolioTarget.MinimumOrderMarginPercentageWarningSent = false;
                        }
                    }
                }

                _targetsCollection.ClearFulfilled(algorithm);
            }
        }
    }
}
```

```
class FrameworkExecutionModelAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2020, 4, 30)
        self.set_end_date(2020, 5, 5)
        self.set_cash(1000000)
        self.universe_settings.resolution = Resolution.DAILY

        # Add a universe of the most liquid stocks since their trend is more capital-supported.
        self.add_universe_selection(QC500UniverseSelectionModel())
        # Emit insights all for selected stocks.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(7)))
        # Equal weighting on each insight is needed to dissipate capital risk evenly.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

        # To place bracket orders besides basic orders as well.
        self.set_execution(StopLossExecutionModel(0.05))

class StopLossExecutionModel(ExecutionModel):
    def __init__(self, stop_loss: float = 0.05) -> None:
        self.targets_collection = PortfolioTargetCollection()
        # The stop loss percent of the bracket order.
        self.stop_loss = stop_loss
        self.stop_orders = []

    def execute(self, algorithm: QCAlgorithm, targets: list[PortfolioTarget]) -> None:
        for stop_loss in self.stop_orders.copy():
            # Check if any bracket orders have been filled out. If so, cancel the insight to avoid repeated ordering.
            if stop_loss.status == OrderStatus.FILLED:
                algorithm.insights.cancel([stop_loss.symbol])
                self.stop_orders.remove(stop_loss)

        self.targets_collection.add_range(targets)
        if not self.targets_collection.is_empty:
            for target in self.targets_collection.order_by_margin_impact(algorithm):
                security = algorithm.securities[target.symbol]
                # Calculate the remaining quantity to be ordered
                quantity = OrderSizing.get_unordered_quantity(algorithm, target, security, True)

                if quantity != 0:
                    above_minimum_portfolio = BuyingPowerModelExtensions.above_minimum_order_margin_portfolio_percentage(
                        security.buying_power_model,
                        security,
                        quantity,
                        algorithm.portfolio,
                        algorithm.settings.minimum_order_margin_portfolio_percentage)
                    # Place orders and the bracket orders.
                    if above_minimum_portfolio:
                        algorithm.market_order(security, quantity)
                        # Stop Loss order.
                        stop_loss_price = security.price * (1 - self.stop_loss if quantity > 0 else 1 + self.stop_loss)
                        stop_loss = algorithm.stop_market_order(security.symbol, -quantity, stop_loss_price)
                        self.stop_orders.append(stop_loss)
                    elif not PortfolioTarget.minimum_order_margin_percentage_warning_sent:
                        # will trigger the warning if it has not already been sent
                        PortfolioTarget.minimum_order_margin_percentage_warning_sent = False

            self.targets_collection.clear_fulfilled(algorithm)
```

#### Example 3: More Favorable Than Signal Time

Some algorithms are signaling based on daily close data. However, their orders often emit when the next market opens, making the filling price slip. To avoid this slippage negatively impacting your expectations, we can construct a custom execution model to place orders only when the price is more favorable than the last close price. Although we did not set a time limit in this example, you may consider doing so.

Select Language: C#Python

```
public class FrameworkExecutionModelAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2022, 1, 1);
        SetEndDate(2022, 3, 1);

        // Add a universe of the most liquid stocks since their trend is more capital-supported.
        AddUniverseSelection(new QC500UniverseSelectionModel());
        // Emit insights all for selected stocks.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(7)));
        // Equal weighting on each insight is needed to dissipate capital risk evenly.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());

        // Only place an order if the price is more favorable than the price at the insight signal.
        SetExecution(new FavorableExecutionModel());
    }

    private class FavorableExecutionModel : ExecutionModel
    {
        private readonly PortfolioTargetCollection _targetsCollection = new PortfolioTargetCollection();
        private Dictionary<Symbol, decimal> _lagPriceBySymbol = new();

        public override void Execute(QCAlgorithm algorithm, IPortfolioTarget[] targets)
        {
            _targetsCollection.AddRange(targets);
            if (!_targetsCollection.IsEmpty)
            {
                foreach (var target in _targetsCollection.OrderByMarginImpact(algorithm))
                {
                    var symbol = target.Symbol;
                    var security = algorithm.Securities[symbol];

                    // Calculate the remaining quantity to be ordered
                    var quantity = OrderSizing.GetUnorderedQuantity(algorithm, target, security, true);

                    if (quantity != 0)
                    {
                        if (BuyingPowerModelExtensions.AboveMinimumOrderMarginPortfolioPercentage(security.BuyingPowerModel, security, quantity,
                            algorithm.Portfolio, algorithm.Settings.MinimumOrderMarginPortfolioPercentage))
                        {
                            // Cache the price at signal emission to compare if the current price is more favorable.
                            if (!_lagPriceBySymbol.TryGetValue(symbol, out var lastPrice))
                            {
                                var history = algorithm.History<TradeBar>(symbol, 1, Resolution.Daily);
                                _lagPriceBySymbol[symbol] = lastPrice = history.Last().Close;
                            }

                            // Only order if the price is more favorable than the price at signal emission.
                            if ((security.Price <= lastPrice && quantity > 0) || (security.Price >= lastPrice && quantity < 0))
                            {
                                algorithm.MarketOrder(security, quantity);
                            }
                        }
                        else if (!PortfolioTarget.MinimumOrderMarginPercentageWarningSent.HasValue)
                        {
                            // will trigger the warning if it has not already been sent
                            PortfolioTarget.MinimumOrderMarginPercentageWarningSent = false;
                        }
                    }
                }

                _targetsCollection.ClearFulfilled(algorithm);
            }
        }
    }
}
```

```
class FrameworkExecutionModelAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2022, 1, 1)
        self.set_end_date(2022, 3, 1)

        # Add a universe of the most liquid stocks since their trend is more capital-supported.
        self.add_universe_selection(QC500UniverseSelectionModel())
        # Emit insights all for selected stocks.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(7)))
        # Equal weighting on each insight to dissipate capital risk evenly.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

        # Only place an order if the price is more favorable than the price at the insight signal.
        self.set_execution(FavorableExecutionModel())

class FavorableExecutionModel(ExecutionModel):
    def __init__(self) -> None:
        self.targets_collection = PortfolioTargetCollection()
        self.lag_price_by_symbol = {}

    def execute(self, algorithm: QCAlgorithm, targets: list[PortfolioTarget]) -> None:
        self.targets_collection.add_range(targets)
        if not self.targets_collection.is_empty:
            for target in self.targets_collection.order_by_margin_impact(algorithm):
                symbol = target.symbol
                security = algorithm.securities[symbol]
                # Calculate the remaining quantity to be ordered
                quantity = OrderSizing.get_unordered_quantity(algorithm, target, security, True)

                if quantity != 0:
                    above_minimum_portfolio = BuyingPowerModelExtensions.above_minimum_order_margin_portfolio_percentage(
                        security.buying_power_model,
                        security,
                        quantity,
                        algorithm.portfolio,
                        algorithm.settings.minimum_order_margin_portfolio_percentage)
                    if above_minimum_portfolio:
                        # Cache the price at signal emission to compare if the current price is more favorable.
                        if not symbol in self.lag_price_by_symbol:
                            history = algorithm.history[TradeBar](symbol, 1, Resolution.DAILY)
                            self.lag_price_by_symbol[symbol] = list(history)[-1].close

                        # Only order if the price is more favorable than the price at signal emission.
                        if (security.price <= self.lag_price_by_symbol[symbol] and quantity > 0)\
                        or (security.price >= self.lag_price_by_symbol[symbol] and quantity < 0):
                            algorithm.market_order(security, quantity)
                    elif not PortfolioTarget.minimum_order_margin_percentage_warning_sent:
                        # will trigger the warning if it has not already been sent
                        PortfolioTarget.minimum_order_margin_percentage_warning_sent = False

            self.targets_collection.clear_fulfilled(algorithm)
```

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/06%20Execution/01%20Key%20Concepts)