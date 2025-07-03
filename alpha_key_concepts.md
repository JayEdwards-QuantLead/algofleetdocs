# Alpha

## Key Concepts

### Introduction

![](https://cdn.quantconnect.com/web/i/docs/algorithm-framework/alpha-creation.png)The Alpha model predicts market trends and signals the best moments to trade. These signals, or `Insight` objects, contain the `Direction`, `Magnitude`, and `Confidence` of a market prediction and the suggested portfolio `Weight`. You should generate insights on the set of assets provided by the Universe Selection model and only generate them when your predictions change.

### Add Models

To add an Alpha model, in the `Initialize` `initialize` method, call the `AddAlpha` method.

Select Language: C#Python

```
// Add EmaCrossAlphaModel to generate market trend insights and trading signals based on EMA crossovers, providing Direction, Magnitude, and Confidence for the universe-selected assets.
AddAlpha(new EmaCrossAlphaModel());
```

```
# Add EmaCrossAlphaModel to generate market trend insights and trading signals based on EMA crossovers, providing Direction, Magnitude, and Confidence for the universe-selected assets.
self.add_alpha(EmaCrossAlphaModel())
```

To view all the pre-built Alpha models, see [Supported Models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/alpha/supported-models).

### Multi-Alpha Algorithms

You can add multiple Alpha models to a single algorithm and generate Insight objects with all of them.

Select Language: C#Python

```
// Add RsiAlphaModel and EmaCrossAlphaModel to generate trading signals based on RSI and EMA crossovers, utilizing multiple alpha models to capture different market signals and trends for robust predictions.
AddAlpha(new RsiAlphaModel());
AddAlpha(new EmaCrossAlphaModel());
```

```
# Add RsiAlphaModel and EmaCrossAlphaModel to generate trading signals based on RSI and EMA crossovers, utilizing multiple alpha models to capture different market signals and trends for robust predictions.
self.add_alpha(RsiAlphaModel())
self.add_alpha(EmaCrossAlphaModel())
```

Each Alpha model has a unique name. The Insight objects they generate are automatically named according to the Alpha model name.

If you add multiple Alpha models, each alpha model receives the current slice in the order that you add the Alphas. The combined stream of Insight objects is passed to the [Portfolio Construction model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/key-concepts#04-Multi-Alpha-Algorithms) that defines how the Insight collection is combined. If you have a hybrid algorithm, the combined stream of Insight objects is passed to the [event handler](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/hybrid-algorithms#03-Alpha).

### Model Structure

Alpha models should extend the `AlphaModel` class. Extensions of the `AlphaModel` class must implement the `Update` `update` method, which receives a [Slice](https://www.quantconnect.com/docs/v2/writing-algorithms/key-concepts/time-modeling/timeslices) object and returns an array of `Insight` objects. Extensions should also implement the `OnSecuritiesChanged` `on_securities_changed` method to track security changes in the universe.

Select Language: C#Python

```
// Algorithm framework model that produces insights
class MyAlphaModel : AlphaModel
{
    // Updates this Alpha model with the latest data from the algorithm.
    // This is called each time the algorithm receives data for subscribed securities
    public override IEnumerable<Insight> Update(QCAlgorithm algorithm, Slice data)
    {
        var insights = new List<Insight>();
        return insights;
    }

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
# Algorithm framework model that produces insights
class MyAlphaModel(AlphaModel):

    def update(self, algorithm: QCAlgorithm, data: Slice) -> list[Insight]:
        # Updates this Alpha model with the latest data from the algorithm.
        # This is called each time the algorithm receives data for subscribed securities
        # Generate insights on the securities in the universe.
        insights = []
        return insights

    def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
        # Security additions and removals are pushed here.
        # This can be used for setting up algorithm state.
        # changes.added_securities
        # changes.removed_securities
        pass
```

#### Method Parameters

The `algorithm` parameter that the methods receive is an instance of the base `QCAlgorithm` class, not your subclass of it. To access members of your algorithm object, either use a global variable or pass an algorithm reference to Alpha model constructor. Both of these approaches violate the separation of concerns principle.

The `data` parameter contains the latest data available to the algorithm.

The `changes` parameter contains the universe changes.

#### Model Names

To ensure your Alpha model is compatible with all [Portfolio Construction models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/supported-models), assign a unique name to your Alpha model. Some Portfolio Construction models can combine multiple Alpha models together, so it can be important to distinguish between the Alpha models.

Select Language: C#Python

```
# Assign a unique name to RsiAlphaModel to ensure compatibility and distinguish it when combining with other Alpha models in portfolio construction.
class RsiAlphaModel(AlphaModel):
    name = "RsiAlphaModel"
```

```
// Assign a unique name to RsiAlphaModel to ensure compatibility and distinguish it when combining with other Alpha models in portfolio construction.
public class RsiAlphaModel : AlphaModel
{
    // Give your alpha a name (perhaps based on its constructor args?)
    public override string Name { get; }
}
```

By default, LEAN uses the string representation of a [Guid object](https://learn.microsoft.com/en-us/dotnet/api/system.guid.newguid?view=net-7.0) to set the Alpha model name. An example default name is "0f8fad5b-d9cb-469f-a165-70867728950e". This default name is different for every backtest you run and every live algorithm you deploy. For consistent behavior, manually set the Alpha model name.

#### Example

To view a full example of an `AlphaModel` subclass, see the [ConstantAlphaModel](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/ConstantAlphaModel.cs)[ConstantAlphaModel](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Alphas/ConstantAlphaModel.py) in the LEAN GitHub repository.

### Track Security Changes

The [Universe Selection model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/key-concepts) may select a dynamic universe of assets, so you should not assume a fixed set of assets in the Alpha model. When the Universe Selection model adds and removes assets from the universe, it triggers an `OnSecuritiesChanged` `on_securities_changed` event. In the `OnSecuritiesChanged` `on_securities_changed` event handler, you can initialize the security-specific state or load any history required for your Alpha model. If you need to save data for individual securities, add custom members to the respective `Security` objectcast the `Security` object to a `dynamic` object and then save custom members to it.

Select Language: C#Python

```
// Handle dynamic asset changes by initializing indicators for new securities and removing them for removed securities to keep the Alpha model adaptive and accurate.
class MyAlphaModel : AlphaModel
{
    private List<Security> _securities = new List<Security>();

    public override void OnSecuritiesChanged(QCAlgorithm algorithm, SecurityChanges changes) {
        foreach (var security in changes.AddedSecurities)
        {
            var dynamicSecurity = security as dynamic;
            dynamicSecurity.Sma = algorithm.SMA(security.Symbol, 20);
            algorithm.WarmUpIndicator(security.Symbol, dynamicSecurity.Sma);
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
# Handle dynamic asset changes by initializing indicators for new securities and removing them for removed securities to keep the Alpha model adaptive and accurate.
class MyAlphaModel(AlphaModel):
    securities = []

    def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
        for security in changes.added_securities:
            security.indicator = algorithm.SMA(security.symbol, 20)
            algorithm.warm_up_indicator(security.symbol, security.indicator)
            self.securities.append(security)

        for security in changes.removed_securities:
            if security in self.securities:
                algorithm.deregister_indicator(security.indicator)
                self.securities.remove(security)
```

### Insights

An Insight is a single prediction for an asset. The `Update` `update` method returns an array of Insight objects. You can think of these as actionable trading signals, indicating the asset direction, magnitude, and confidence in the near future. All insights can take a weight parameter to set the desired weighting for the insight. If you change the cash of the algorithm, it will affect the orders, but not necessarily the insights.

The Portfolio Construction model consumes the Insight objects from the Alpha model. It's up to the Portfolio Construction model to define how Insight objects are converted into `PortfolioTarget` objects. In the pre-built Portfolio Construction models, down insights translate to short-biased trades, up insights translate to long-biased trades, and flat insights usually close open positions, but this doesn't have to be the case in [custom Portfolio Construction models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/key-concepts#03-Model-Structure).

#### Insight Properties

`Insight` objects have the following properties:

Insight
Select Language: C#Python

Gets the unique identifier for this insight

- id: Guid

Gets the group id this insight belongs to, null if not in a group

- group\_id: Guid

Gets an identifier for the source model that generated this insight.

- source\_model: string

Gets the utc time this insight was generated

- generated\_time\_utc: DateTime

Gets the insight's prediction end time. This is the time when this insight prediction is expected to be fulfilled. This time takes into account market hours, weekends, as well as the symbol's data resolution

- close\_time\_utc: DateTime

Gets the symbol this insight is for

- symbol: Symbol

Gets the type of insight, for example, price insight or volatility insight

- type: InsightType

Gets the initial reference value this insight is predicting against. The value is dependent on the specified [QuantConnect.Algorithm.Framework.Alphas.InsightType](https://www.lean.io/docs/v2/lean-engine/class-reference/index.html)

- reference\_value: decimal

Gets the final reference value, used for scoring, this insight is predicting against. The value is dependent on the specified [QuantConnect.Algorithm.Framework.Alphas.InsightType](https://www.lean.io/docs/v2/lean-engine/class-reference/index.html)

- reference\_value\_final: decimal

Gets the predicted direction, down, flat or up

- direction: InsightDirection

Gets the period over which this insight is expected to come to fruition

- period: TimeSpan

Gets the predicted percent change in the insight type (price/volatility)

- magnitude: Double

Gets the confidence in this insight

- confidence: Double

Gets the portfolio weight of this insight

- weight: Double

Gets the most recent scores for this insight

- score: InsightScore

Gets the estimated value of this insight in the account currency

- estimated\_value: decimal

The insight's tag containing additional information

- tag: string

Gets the unique identifier for this insight

- Id: Guid

Gets the group id this insight belongs to, null if not in a group

- GroupId: Guid

Gets an identifier for the source model that generated this insight.

- SourceModel: string

Gets the utc time this insight was generated

- GeneratedTimeUtc: DateTime

Gets the insight's prediction end time. This is the time when this insight prediction is expected to be fulfilled. This time takes into account market hours, weekends, as well as the symbol's data resolution

- CloseTimeUtc: DateTime

Gets the symbol this insight is for

- Symbol: Symbol

Gets the type of insight, for example, price insight or volatility insight

- Type: InsightType

Gets the initial reference value this insight is predicting against. The value is dependent on the specified [QuantConnect.Algorithm.Framework.Alphas.InsightType](https://www.lean.io/docs/v2/lean-engine/class-reference/index.html)

- ReferenceValue: decimal

Gets the final reference value, used for scoring, this insight is predicting against. The value is dependent on the specified [QuantConnect.Algorithm.Framework.Alphas.InsightType](https://www.lean.io/docs/v2/lean-engine/class-reference/index.html)

- ReferenceValueFinal: decimal

Gets the predicted direction, down, flat or up

- Direction: InsightDirection

Gets the period over which this insight is expected to come to fruition

- Period: TimeSpan

Gets the predicted percent change in the insight type (price/volatility)

- Magnitude: Double

Gets the confidence in this insight

- Confidence: Double

Gets the portfolio weight of this insight

- Weight: Double

Gets the most recent scores for this insight

- Score: InsightScore

Gets the estimated value of this insight in the account currency

- EstimatedValue: decimal

The insight's tag containing additional information

- Tag: string

#### Create Insights

To create an `Insight`, call the `Insight` constructor.

Select Language: C#Python

```
# Insight(symbol, period, type, direction, magnitude=None, confidence=None, source_model=None, weight=None, tag='')
insight = Insight("IBM", timedelta(minutes=20), InsightType.PRICE, InsightDirection.UP)
```

```
// new Insight(symbol, period, type, direction, magnitude=null, confidence=null, sourceModel=null, weight=null, tag="");
var insight = new Insight("IBM", TimeSpan.FromMinutes(20), InsightType.Price, InsightDirection.Up);
```

In the `Insight` constructor, the `period` argument can be a `timedelta` `TimeSpan` object or a function that receives a `DateTime` `datetime` object and returns the expiry `DateTime` `datetime`. Some of the constructor arguments are optional to create the `Insight` object, but some of the Portfolio Construction models may require these properties. To check which `Insight` properties a Portfolio Construction model requires, see [Supported Models](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/supported-models).

You can also use the helper method to create Insight objects of the Price type.

Select Language: C#Python

```
# Insight.price(symbol, period, direction, magnitude=None, confidence=None, source_model=None, weight=None, tag='')
insight = Insight.price("IBM", timedelta(minutes = 20), InsightDirection.UP)

# Insight.price(symbol, resolution, barCount, direction, magnitude=None, confidence=None, source_model=None, weight=None, tag='')
insight = Insight.price("IBM", Resolution.MINUTE, 20, InsightDirection.UP)
```

```
// new Insight(symbol, period, direction, magnitude=null, confidence=null, sourceModel=null, weight=null, tag="")
var insight = Insight.Price("IBM", TimeSpan.FromMinutes(20), InsightDirection.Up);

// new Insight(symbol, resolution, barCount, direction, magnitude=null, confidence=null, sourceModel=null, weight=null, tag="")
var insight = Insight.Price("IBM", Resolution.Minute, 20, InsightDirection.Up);
```

In the `Price` `price` method, the `period` argument can be a `timedelta` `TimeSpan` object, a `DateTime` `datetime` object, or a function that receives a `DateTime` `datetime` object and returns the expiry `DateTime` `datetime`.

#### Group Insights

Sometimes an algorithm's performance relies on multiple insights being traded together, like in pairs trading and options straddles. These types insights should be grouped. Insight groups signal to the Execution model that the insights need to be acted on as a single unit to maximize the alpha created.

To mark insights as a group, call the `Insight.Group` method.

Select Language: C#Python

```
// Generate and group insights for a set of tickers with a 20-minute prediction horizon; each Insight object provides directional signals, magnitudes, and confidence for price movement.
return Insight.Group(
	Insight.price("IBM", TimeSpan.FromMinutes(20), InsightDirection.UP),
	Insight.price("SPY", TimeSpan.FromMinutes(20), InsightDirection.UP),
	Insight.price("AAPL", TimeSpan.FromMinutes(20), InsightDirection.UP)
);
```

```
# Generate and group insights for a set of tickers with a 20-minute prediction horizon; each Insight object provides directional signals, magnitudes, and confidence for price movement.
tickers = ["IBM", "SPY", "AAPL"]
insights = [Insight.price(ticker, timedelta(minutes = 20), InsightDirection.UP) for ticker in tickers]
return Insight.group(insights)
```

#### Cancel Insights

If an Alpha model in your algorithm emits an Insight to enter a position but it determines the trading opportunity has pre-maturely ended, you should cancel the Insight. For example, say you want your algorithm to enter a long position in a security when its [Relative Strength Index](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/relative-strength-index) (RSI) moves below 20 and then liquidate the position when the security's RSI moves above 30. If you emit an Insight that has a duration of 30 days when the RSI moves below 20 but the RSI moves above 30 in 10 days, you should cancel the Insight.

To cancel an Insight, call its `Cancel` `cancel`/ `Expire` method with the algorithm's Coordinated Universal Time (UTC).

Select Language: C#Python

```
# Cancel any outstanding insights to ensure only current predictions are considered and to avoid outdated signals influencing trading decisions.
self.insight.cancel(algorithm.utc_time)
```

```
// Cancel any outstanding insights to ensure only current predictions are considered and to avoid outdated signals influencing trading decisions.
_insight.Cancel(algorithm.UtcTime);
```

If you don't have a reference to the Insight you want to cancel, [get it from the InsightManager](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/insight-manager#04-Get-Insights).

When you cancel an insight, it's `CloseTimeUtc` `close_time_utc` property is set to one second into the past.

### Stop Loss Orders Workaround

In some cases, if you add a Risk Management model that uses stop loss logic, the Risk Management model generates `PortfolioTarget` objects with a 0 quantity to make the Execution model liquidate your positions, but then the Portfolio Construction model generates `PortfolioTarget` objects with a non-zero quantity to make the Execution model re-enter the same position. This issue can occur if your Portfolio Construction model rebalances and your Alpha model still has an active insight for the liquidated securities. To avoid this issue, add the stop loss order logic to the Alpha model. When the stop loss is hit, emit a flat insight for the security. Note that this is a workaround, but it violates the separation of concerns principle since the Alpha Model shouldn't react to open positions.

### Universe Timing Considerations

If the Alpha model manages some indicators or [consolidators](https://www.quantconnect.com/docs/v2/writing-algorithms/consolidating-data/getting-started) for securities in the universe and the universe selection runs during the indicator sampling period or the consolidator aggregation period, the indicators and consolidators might be missing some data. For example, take the following scenario:

- The security resolution is minute
- You have a consolidator that aggregates the security data into daily bars to update the indicator
- The universe selection runs at noon

In this scenario, you create and [warm-up the indicator](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/manual-indicators#06-Warm-Up-Indicators) at noon. Since it runs at noon, the history request that gathers daily data to warm up the indicator won't contain any data from the current day and the consolidator that updates the indicator also won't aggregate any data from before noon. This process doesn't cause issues if the indicator only uses the close price to calculate the indicator value (like the [simple moving average](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/simple-moving-average) indicator) because the first consolidated bar that updates the indicator will have the correct close price. However, if the indicator uses more than just the close price to calculate its value (like the [True Range](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/true-range) indicator), the open, high, and low values of the first consolidated bar may be incorrect, causing the initial indicator values to be incorrect.

### Examples

The following examples demonstrate common practices for implementing the framework alpha model.

#### Example 1: EMA Cross On Liquid Stocks

The following algorithm selects the top 500 liquid US stocks traded in popular exchanges using `QC500UniverseSelectionModel`. Then, emit trade signals based on trade changes using 60-day versus 200-day EMA cross through `EmaCrossAlphaModel`. Each signal will weight equally.

Select Language: C#Python

```
public class FrameworkAlphaModelAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2023, 1, 1);
        SetEndDate(2023, 8, 1);

        // Add a universe of liquid equities since they have capital to support demand and trends.
        AddUniverseSelection(new QC500UniverseSelectionModel());
        // Using an EMA cross-alpha model to trade medium versus long-term trends.
        AddAlpha(new EmaCrossAlphaModel(60, 200, Resolution.Daily));
        // Equally invested in insights to dissipate capital risk evenly.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }
}
```

```
class FrameworkAlphaModelAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2023, 1, 1)
        self.set_end_date(2023, 8, 1)

        # Add a universe of liquid equities since they have capital to support demand and trends.
        self.add_universe_selection(QC500UniverseSelectionModel())
        # Using an EMA cross-alpha model to trade on medium versus long-term trends.
        self.add_alpha(EmaCrossAlphaModel(60, 200, Resolution.DAILY))
        # Equally invested in insights to dissipate capital risk evenly.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())
```

#### Example 2: Interest Rate Effect On Forex

The following algorithm implements a custom alpha model that uses the interest rate cycle as a signal to trade Forex. We trade the top 6 forexes: AUDUSD, EURUSD, GBPUSD, USDCAD, USDCHF, and USDJPY. For the ones with USD as quote currencies, we buy them during the US interest rate down cycle since USD is relatively weak while shorting them during the upcycle. The opposite is true for the ones with USD as the base currency.

Select Language: C#Python

```
public class FrameworkAlphaModelAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2020, 1, 1);
        SetEndDate(2024, 12, 1);

        // Use daily resolution since the interest rate signal is on a daily basis.
        UniverseSettings.Resolution = Resolution.Daily;
        // Add a universe of selected lists of Forex.
        var forex = new[] {
            QuantConnect.Symbol.Create("AUDUSD", SecurityType.Forex, Market.Oanda),
            QuantConnect.Symbol.Create("EURUSD", SecurityType.Forex, Market.Oanda),
            QuantConnect.Symbol.Create("GBPUSD", SecurityType.Forex, Market.Oanda),
            QuantConnect.Symbol.Create("USDCAD", SecurityType.Forex, Market.Oanda),
            QuantConnect.Symbol.Create("USDCHF", SecurityType.Forex, Market.Oanda),
            QuantConnect.Symbol.Create("USDJPY", SecurityType.Forex, Market.Oanda)
        };
        AddUniverseSelection(new ManualUniverseSelectionModel(forex));
        // Using a custom alpha model to trade forex based on interest rate.
        AddAlpha(new InterestRateAlphaModel(this));
        // Equally invested in insights to dissipate capital risk evenly.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }

    private class InterestRateAlphaModel : AlphaModel
    {
        private QCAlgorithm _algorithm;
        private bool _wasRising;
        // Use a 365d SMA indicator of daily interest rate to estimate if the interest rate cycle is upward or downward.
        private SimpleMovingAverage _sma = new(365);

        public InterestRateAlphaModel(QCAlgorithm algorithm)
        {
            _algorithm = algorithm;

            // Warm up the SMA indicator.
            var current = algorithm.Time;
            var provider = algorithm.RiskFreeInterestRateModel;
            for (DateTime dt = current.AddDays(-365); dt <= current; dt = dt.AddDays(1))
            {
                var rate = provider.GetInterestRate(dt);
                _sma.Update(dt, rate);
                _wasRising = rate > _sma;
            }

            // Set schedule to update the interest rate trend indicator daily.
            algorithm.Schedule.On(
                algorithm.DateRules.EveryDay(),
                algorithm.TimeRules.At(0, 1),
                UpdateInterestRate
            );
        }

        private void UpdateInterestRate()
        {
            // Update the interest rate on the SMA indicator to estimate its trend.
            var rate = _algorithm.RiskFreeInterestRateModel.GetInterestRate(_algorithm.Time);
            _sma.Update(_algorithm.Time, rate);
            _wasRising = rate > _sma;
        }

        public override IEnumerable<Insight> Update(QCAlgorithm algorithm, Slice slice)
        {
            var insights = new List<Insight>();

            // Split the forexes by whether the quote currency is USD.
            var quoteUSD = new List<Symbol>();
            var baseUSD = new List<Symbol>();
            foreach (var (symbol, security) in algorithm.Securities)
            {
                if (security.QuoteCurrency.Symbol == Currencies.USD)
                {
                    quoteUSD.Add(symbol);
                }
                else
                {
                    baseUSD.Add(symbol);
                }
            }

            var rate = algorithm.RiskFreeInterestRateModel.GetInterestRate(algorithm.Time);
            // During the rising interest rate cycle, long the forexes with USD as the base currency and short the ones with USD as the quote currency.
            if (rate > _sma)
            {
                insights.AddRange(
                    baseUSD.Select(symbol => Insight.Price(symbol, TimeSpan.FromDays(1), InsightDirection.Up)).ToList()
                );
                insights.AddRange(
                    quoteUSD.Select(symbol => Insight.Price(symbol, TimeSpan.FromDays(1), InsightDirection.Down)).ToList()
                );
            }
            // During a downward interest rate cycle, short the forexes with USD as the base currency and long the ones with USD as the quote currency.
            else if (rate < _sma)
            {
                insights.AddRange(
                    baseUSD.Select(symbol => Insight.Price(symbol, TimeSpan.FromDays(1), InsightDirection.Down)).ToList()
                );
                insights.AddRange(
                    quoteUSD.Select(symbol => Insight.Price(symbol, TimeSpan.FromDays(1), InsightDirection.Up)).ToList()
                );
            }
            // If the interest rate cycle remains steady for a long time, we expect a flip in the cycle soon.
            else if (_wasRising)
            {
                insights.AddRange(
                    baseUSD.Select(symbol => Insight.Price(symbol, TimeSpan.FromDays(1), InsightDirection.Down)).ToList()
                );
                insights.AddRange(
                    quoteUSD.Select(symbol => Insight.Price(symbol, TimeSpan.FromDays(1), InsightDirection.Up)).ToList()
                );
            }
            else
            {
                insights.AddRange(
                    baseUSD.Select(symbol => Insight.Price(symbol, TimeSpan.FromDays(1), InsightDirection.Up)).ToList()
                );
                insights.AddRange(
                    quoteUSD.Select(symbol => Insight.Price(symbol, TimeSpan.FromDays(1), InsightDirection.Down)).ToList()
                );
            }

            return insights;
        }
    }
}
```

```
class FrameworkAlphaModelAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2020, 1, 1)
        self.set_end_date(2024, 12, 1)

        # Use daily resolution since the interest rate signal is daily.
        self.universe_settings.resolution = Resolution.DAILY
        # Add a universe of selected list of Forex.
        forex = [\
            Symbol.create("AUDUSD", SecurityType.FOREX, Market.OANDA),\
            Symbol.create("EURUSD", SecurityType.FOREX, Market.OANDA),\
            Symbol.create("GBPUSD", SecurityType.FOREX, Market.OANDA),\
            Symbol.create("USDCAD", SecurityType.FOREX, Market.OANDA),\
            Symbol.create("USDCHF", SecurityType.FOREX, Market.OANDA),\
            Symbol.create("USDJPY", SecurityType.FOREX, Market.OANDA)\
        ]
        self.add_universe_selection(ManualUniverseSelectionModel(forex))
        # Using a custom alpha model to trade forex based on interest rate.
        self.add_alpha(InterestRateAlphaModel(self))
        # Equally invested in insights to dissipate capital risk evenly.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

class InterestRateAlphaModel(AlphaModel):
    # Use a 365d SMA indicator of daily interest rate to estimate if the interest rate cycle is upward or downward.
    _sma = SimpleMovingAverage(365)

    def __init__(self, algorithm: QCAlgorithm) -> None:
        self._algorithm = algorithm

        # Warm up the SMA indicator.
        current = algorithm.time
        provider = algorithm.risk_free_interest_rate_model
        dt = current - timedelta(365)
        while dt <= current:
            rate = provider.get_interest_rate(dt)
            self._sma.update(dt, rate)
            self._was_rising = rate > self._sma.current.value
            dt += timedelta(1)

        # Set a schedule to update the interest rate trend indicator every day.
        algorithm.schedule.on(
            algorithm.date_rules.every_day(),
            algorithm.time_rules.at(0, 1),
            self.update_interest_rate
        )

    def update_interest_rate(self) -> None:
        # Update interest rate to the SMA indicator to estimate its trend.
        rate = self._algorithm.risk_free_interest_rate_model.get_interest_rate(self._algorithm.time)
        self._sma.update(self._algorithm.time, rate)
        self._was_rising = rate > self._sma.current.value

    def update(self, algorithm: QCAlgorithm, slice: Slice) -> list[Insight]:
        insights = []

        # Split the forexes by whether the quote currency is USD.
        quote_usd = []
        base_usd = []
        for symbol, security in algorithm.securities.items():
            if security.quote_currency.symbol == Currencies.USD:
                quote_usd.append(symbol)
            else:
                base_usd.append(symbol)

        rate = algorithm.risk_free_interest_rate_model.get_interest_rate(algorithm.time)
        # During the rising interest rate cycle, long the forexes with USD as the base currency and short the ones with USD as the quote currency.
        if rate > self._sma.current.value:
            insights.extend(
                [Insight.price(symbol, timedelta(1), InsightDirection.UP) for symbol in base_usd] +
                [Insight.price(symbol, timedelta(1), InsightDirection.DOWN) for symbol in quote_usd]
            )
        # During the downward interest rate cycle, short the forexes with USD as the base currency and long the ones with USD as the quote currency.
        elif rate < self._sma.current.value:
            insights.extend(
                [Insight.price(symbol, timedelta(1), InsightDirection.DOWN) for symbol in base_usd] +
                [Insight.price(symbol, timedelta(1), InsightDirection.UP) for symbol in quote_usd]
            )
        # If the interest rate cycle is steady for a long time, we expect a flip in the cycle.
        elif self._was_rising:
            insights.extend(
                [Insight.price(symbol, timedelta(1), InsightDirection.DOWN) for symbol in base_usd] +
                [Insight.price(symbol, timedelta(1), InsightDirection.UP) for symbol in quote_usd]
            )
        else:
            insights.extend(
                [Insight.price(symbol, timedelta(1), InsightDirection.UP) for symbol in base_usd] +
                [Insight.price(symbol, timedelta(1), InsightDirection.DOWN) for symbol in quote_usd]
            )

        return insights
```

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/03%20Alpha/01%20Key%20Concepts)