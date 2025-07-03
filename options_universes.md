# Universe Selection

## Options Universes

### Introduction

An Option Universe Selection model selects contracts for a set of Options.

### Options Universe Selection

The `OptionUniverseSelectionModel` selects all the available contracts for the Equity Options, Index Options, and Future Options you specify. To use this model, provide a `refreshInterval` `refresh_interval` and a selector function. The `refreshInterval` `refresh_interval` defines how frequently LEAN calls the selector function. The selector function receives a `DateTime` `datetime` object that represents the current Coordinated Universal Time (UTC) and returns a list of `Symbol` objects. The `Symbol` objects you return from the selector function are the Options of the universe.

Select Language: C#Python

```
// Run universe selection asynchronously to speed up your algorithm.
// In this case, you can't rely on the method or algorithm state between filter calls.
UniverseSettings.Asynchronous = true;
// Add a universe of SPY Options.
AddUniverseSelection(
    new OptionUniverseSelectionModel(
        // Refresh the universe daily.
        TimeSpan.FromDays(1),
        _ => new [] { QuantConnect.Symbol.Create("SPY", SecurityType.Option, Market.USA) }
    )
);
```

```
from Selection.OptionUniverseSelectionModel import OptionUniverseSelectionModel

# Run universe selection asynchronously to speed up your algorithm.
# In this case, you can't rely on the method or algorithm state between filter calls.
self.universe_settings.asynchronous = True
# Add a universe of SPY Options.
self.set_universe_selection(
    OptionUniverseSelectionModel(
        # Refresh the universe daily.
        timedelta(1), lambda _: [Symbol.create("SPY", SecurityType.OPTION, Market.USA)]
    )
)
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `refreshInterval` `refresh_interval` | `TimeSpan` `timedelta` | Time interval between universe refreshes |  |
| `optionChainSymbolSelector` `option_chain_symbol_selector` | `Func<DateTime, IEnumerable<Symbol>>` `Callable[[datetime], list[Symbol]]` | A function that selects the Option symbols |  |
| `universeSettings` `universe_settings` | `UniverseSettings` | The [universe settings](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/universe-settings). If you don't provide an argument, the model uses the `algorithm.UniverseSettings` `algorithm.universe_settings` by default. | `null` `None` |

The following example shows how to define the Option chain Symbol selector as an isolated method:

Select Language: C#Python

```
// In the Initialize method, add the OptionUniverseSelectionModel with a custom selection function.
public override void Initialize()
{
    AddUniverseSelection(
        new OptionUniverseSelectionModel(TimeSpan.FromDays(1), SelectOptionChainSymbols)
    );
}

// Define the selection function.
private IEnumerable<Symbol> SelectOptionChainSymbols(DateTime utcTime)
{
    // Equity Options example:
    //var tickers = new[] {"SPY", "QQQ", "TLT"};
    //return tickers.Select(ticker => QuantConnect.Symbol.Create(ticker, SecurityType.Option, Market.USA));

    // Index Options example:
    //var tickers = new[] {"VIX", "SPX"};
    //return tickers.Select(ticker => QuantConnect.Symbol.Create(ticker, SecurityType.IndexOption, Market.USA));

    // Future Options example:
    var futureSymbol = QuantConnect.Symbol.Create(Futures.Indices.SP500EMini, SecurityType.Future, Market.CME);
    foreach (var contract in FuturesChain(futureSymbol))
    {
        yield return QuantConnect.Symbol.CreateCanonicalOption(contract.Symbol);
    }
}
```

```
from Selection.OptionUniverseSelectionModel import OptionUniverseSelectionModel

# In the initialize method, add the OptionUniverseSelectionModel with a custom selection function.
def initialize(self) -> None:
    self.add_universe_selection(
        OptionUniverseSelectionModel(timedelta(days=1), self.select_option_chain_symbols)
    )

# Define the selection function.
def select_option_chain_symbols(self, utc_time: datetime) -> list[Symbol]:
    # Equity Options example:
    #tickers = ["SPY", "QQQ", "TLT"]
    #return [Symbol.create(ticker, SecurityType.OPTION, Market.USA) for ticker in tickers]

    # Index Options example:
    #tickers = ["VIX", "SPX"]
    #return [Symbol.create(ticker, SecurityType.INDEX_OPTION, Market.USA) for ticker in tickers]

    # Future Options example:
    future_symbol = Symbol.create(Futures.Indices.SP_500_E_MINI, SecurityType.FUTURE, Market.CME)
    return [Symbol.create_canonical_option(contract.symbol) for contract in self.futures_chain(future_symbol)]
```

This model uses the default Option filter, which selects all of the available Option contracts at the current time step. To use a different filter for the contracts, subclass the `OptionUniverseSelectionModel` and define a `Filter` `filter` method. The `Filter` `filter` method accepts and returns an `OptionFilterUniverse` object to select the Option contracts. The following table describes the methods of the `OptionFilterUniverse` class:

The following table describes the filter methods of the `OptionFilterUniverse` class:

|     |
| --- |
| `Strikes(int minStrike, int maxStrike)` `strikes(min_strike: int, max_strike: int)` <br>Selects contracts that are within `minStrike` `m_strike` strikes below the underlying price and `maxStrike` `max_strike` strikes above the underlying price. |
| `CallsOnly()` `calls_only()` <br>Selects call contracts. |
| `PutsOnly()` `puts_only()` <br>Selects put contracts. |
| `StandardsOnly()` `standards_only()` <br>Selects standard contracts. |
| `IncludeWeeklys()` `include_weeklys()` <br>Selects non-standard weeklys contracts. |
| `WeeklysOnly()` `weeklys_only()` <br>Selects weekly contracts. |
| `FrontMonth()` `front_month()` <br>Selects the front month contract. |
| `BackMonths()` `back_months()` <br>Selects the non-front month contracts. |
| `BackMonth()` `back_month()` <br>Selects the back month contracts. |
| `Expiration(int minExpiryDays, int maxExpiryDays)` `expiration(min_expiryDays: int, max_expiryDays: int)` <br>Selects contracts that expire within a range of dates relative to the current day. |
| `Contracts(IEnumerable<Symbol> contracts)` `contracts(contracts: list[Symbol])` <br>Selects a list of contracts. |
| `Contracts(Func<IEnumerable<Symbol>, IEnumerable< Symbol>> contractSelector)` `contracts(contract_selector: Callable[[list[Symbol]], list[Symbol]])` <br>Selects contracts that a selector function selects. |

The preceding methods return an `OptionFilterUniverse`, so you can chain the methods together.

The contract filter runs at the first time step of each day.

To move the Option chain Symbol selector outside of the algorithm class, create a universe selection model that inherits the `OptionUniverseSelectionModel` class.

Select Language: C#Python

```
// In the Initialize method, define the universe settings and add data.
UniverseSettings.Asynchronous = true;
AddUniverseSelection(new EarliestExpiringAtTheMoneyCallOptionUniverseSelectionModel(this));

// Outside of the algorithm class, define the universe selection model.
class EarliestExpiringAtTheMoneyCallOptionUniverseSelectionModel : OptionUniverseSelectionModel
{
    public EarliestExpiringAtTheMoneyCallOptionUniverseSelectionModel(QCAlgorithm algorithm)
            : base(TimeSpan.FromDays(1), utcTime => SelectOptionChainSymbols(algorithm, utcTime)) {}

    private static IEnumerable<Symbol> SelectOptionChainSymbols(QCAlgorithm algorithm, DateTime utcTime)
    {
        // Equity Options example:
        //var tickers = new[] {"SPY", "QQQ", "TLT"};
        //return tickers.Select(ticker => QuantConnect.Symbol.Create(ticker, SecurityType.Option, Market.USA));

        // Index Options example:
        //var tickers = new[] {"VIX", "SPX"};
        //return tickers.Select(ticker => QuantConnect.Symbol.Create(ticker, SecurityType.IndexOption, Market.USA));

        // Future Options example:
        var futureSymbol = QuantConnect.Symbol.Create(Futures.Indices.SP500EMini, SecurityType.Future, Market.CME);
        foreach (var contract in algorithm.FuturesChain(futureSymbol))
        {
            yield return QuantConnect.Symbol.CreateCanonicalOption(contract.Symbol);
        }
    }

    // Create a filter to select contracts that have the strike price within 1 strike level and expire within 7 days.
    protected override OptionFilterUniverse Filter(OptionFilterUniverse filter)
    {
        return filter.Strikes(-1, -1).Expiration(0, 7).CallsOnly();
    }
}
```

```
# In the initialize method, define the universe settings and add data.
self.universe_settings.asynchronous = True
self.add_universe_settings(EarliestExpiringAtTheMoneyCallOptionUniverseSelectionModel(self))

# Outside of the algorithm class, define the universe selection model.
class EarliestExpiringAtTheMoneyCallOptionUniverseSelectionModel(OptionUniverseSelectionModel):
    def __init__(self, algorithm):
        self.algo = algorithm
        super().__init__(timedelta(1), self.select_option_chain_symbols)

    def select_option_chain_symbols(self, utc_time: datetime) -> list[Symbol]:
        # Equity Options example:
        #tickers = ["SPY", "QQQ", "TLT"]
        #return [Symbol.create(ticker, SecurityType.OPTION, Market.USA) for ticker in tickers]

        # Index Options example:
        #tickers = ["VIX", "SPX"]
        #return [Symbol.create(ticker, SecurityType.INDEX_OPTION, Market.USA) for ticker in tickers]

        # Future Options example:
        future_symbol = Symbol.create(Futures.Indices.SP_500_E_MINI, SecurityType.FUTURE, Market.CME)
        return [Symbol.create_canonical_option(contract.symbol) for contract in self.algo.futures_chain(future_symbol)]

    # Create a filter to select contracts that have the strike price within 1 strike level and expire within 7 days.
    def Filter(self, option_filter_universe: OptionFilterUniverse) -> OptionFilterUniverse:
        return option_filter_universe.strikes(-1, -1).expiration(0, 7).calls_only()
```

Some of the preceding filter methods only set an internal enumeration in the `OptionFilterUniverse` that it uses later on in the filter process. This subset of filter methods don't immediately reduce the number of contract `Symbol` objects in the `OptionFilterUniverse`.

To override the default [pricing model](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/options-models/pricing) of the Options, [set a pricing model](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/options-models/pricing#03-Set-Models) in a security initializer.

To override the [initial guess of implied volatility](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/options-models/pricing#96-What-Is-Implied-Volatility3F), set and warm up the underlying [volatility model](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/options-models/volatility/key-concepts).

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/OptionUniverseSelectionModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/OptionUniverseSelectionModel.py).

### Option Chained Universe Selection

An Option chained universe subscribes to Option contracts on the constituents of a [US Equity universe](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/equity).

Select Language: C#Python

```
// Configure the universe to use price data unadjusted for splits and dividends ("raw") into the algorithm.
// Options require raw Equity prices.
UniverseSettings.DataNormalizationMode = DataNormalizationMode.Raw;
UniverseSettings.Asynchronous = true;
AddUniverseSelection(
    new OptionChainedUniverseSelectionModel(
        // Add a universe of the 10 most liquid US Equities.
        AddUniverse(Universe.DollarVolume.Top(10)),
        // Select call Option contracts on the underlying Equities that have the strike price within 2 strike levels.
        optionFilterUniverse => optionFilterUniverse.Strikes(-2, +2).FrontMonth().CallsOnly()
    )
);
```

```
# Configure the universe to use price data unadjusted for splits and dividends ("raw") into the algorithm.
# Options require raw Equity prices.
self.universe_settings.data_normalization_mode = DataNormalizationMode.RAW
self.universe_settings.asynchronous = True
self.add_universe_selection(
    OptionChainedUniverseSelectionModel(
        # Add a universe of the 10 most liquid US Equities.
        self.add_universe(self.universe.dollar_volume.top(10)),
        # Select call Option contracts on the underlying Equities that have the strike price within 2 strike levels.
        lambda option_filter_universe: option_filter_universe.strikes(-2, +2).front_month().calls_only()
    )
)
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `universe` | `Universe` | The universe to chain onto the Option Universe Selection model |  |
| `optionFilter` `option_filter` | `Func<OptionFilterUniverse, OptionFilterUniverse>` `Callable[[OptionFilterUniverse], OptionFilterUniverse]` | The Option filter universe to use |  |
| `universeSettings` `universe_settings` | `UniverseSettings` | The [universe settings](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/universe-settings). If you don't provide an argument, the model uses the `algorithm.UniverseSettings` `algorithm.universe_settings` by default. | `null` `None` |

The `optionFilter` `option_filter` function receives and returns an `OptionFilterUniverse` to select the Option contracts. The following table describes the methods of the `OptionFilterUniverse` class:

The following table describes the filter methods of the `OptionFilterUniverse` class:

|     |
| --- |
| `Strikes(int minStrike, int maxStrike)` `strikes(min_strike: int, max_strike: int)` <br>Selects contracts that are within `minStrike` `m_strike` strikes below the underlying price and `maxStrike` `max_strike` strikes above the underlying price. |
| `CallsOnly()` `calls_only()` <br>Selects call contracts. |
| `PutsOnly()` `puts_only()` <br>Selects put contracts. |
| `StandardsOnly()` `standards_only()` <br>Selects standard contracts. |
| `IncludeWeeklys()` `include_weeklys()` <br>Selects non-standard weeklys contracts. |
| `WeeklysOnly()` `weeklys_only()` <br>Selects weekly contracts. |
| `FrontMonth()` `front_month()` <br>Selects the front month contract. |
| `BackMonths()` `back_months()` <br>Selects the non-front month contracts. |
| `BackMonth()` `back_month()` <br>Selects the back month contracts. |
| `Expiration(int minExpiryDays, int maxExpiryDays)` `expiration(min_expiryDays: int, max_expiryDays: int)` <br>Selects contracts that expire within a range of dates relative to the current day. |
| `Contracts(IEnumerable<Symbol> contracts)` `contracts(contracts: list[Symbol])` <br>Selects a list of contracts. |
| `Contracts(Func<IEnumerable<Symbol>, IEnumerable< Symbol>> contractSelector)` `contracts(contract_selector: Callable[[list[Symbol]], list[Symbol]])` <br>Selects contracts that a selector function selects. |

The preceding methods return an `OptionFilterUniverse`, so you can chain the methods together.

The following example shows how to define the Option filter as an isolated method:

Select Language: C#Python

```
// In the Initialize method, define the universe settings and add the universe selection model.
public override void Initialize()
{
    UniverseSettings.DataNormalizationMode = DataNormalizationMode.Raw;
    UniverseSettings.Asynchronous = true;
    AddUniverseSelection(
        new OptionChainedUniverseSelectionModel(
            AddUniverse(Universe.DollarVolume.Top(10)), OptionFilterFunction
        )
    );
}

// Define the contract filter function to select front month call contracts with a strike price within 2 strike levels.
private OptionFilterUniverse OptionFilterFunction(OptionFilterUniverse optionFilterUniverse)
{
    return optionFilterUniverse.Strikes(-2, +2).FrontMonth().CallsOnly();
}
```

```
# In the initialize method, define the universe settings and add the universe selection model.
def initialize(self) -> None:
    self.universe_settings.data_normalization_mode = DataNormalizationMode.RAW
    self.universe_settings.asynchronous = True
    self.add_universe_selection(
        OptionChainedUniverseSelectionModel(
            self.add_universe(self.universe.dollar_volume.top(10)), self.option_filter_function
        )
    )

# Define the contract filter function to select front month call contracts with a strike price within 2 strike levels.
def option_filter_function(self, option_filter_universe: OptionFilterUniverse) -> OptionFilterUniverse:
    return option_filter_universe.strikes(-2, +2).front_month().calls_only()
```

Some of the preceding filter methods only set an internal enumeration in the `OptionFilterUniverse` that it uses later on in the filter process. This subset of filter methods don't immediately reduce the number of contract `Symbol` objects in the `OptionFilterUniverse`.

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm/Selection/OptionChainedUniverseSelectionModel.cs).

### Examples

The following examples demonstrate some common practices for implementing the framework Option Universe Selection Model.

#### Example 1: Horizontal Jelly Roll

The following algorithm selects SPX index options to construct a Jelly Roll strategy. It filters for ATM calls and puts with 30 days and 90 days till expiration. Using the SMA indicator to predict the interest rate cycle, it longs Jelly Roll if the cycle is considered uprising, otherwise selling the Jelly Roll.

Select Language: C#Python

```
public class FrameworkOptionUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2023, 1, 1);
        SetEndDate(2023, 8, 1);

        // Add a universe that selects the needed option contracts.
        AddUniverseSelection(new AtmOptionHorizontalSpreadUniverseSelectionModel());
        // Add Alpha model to trade Jelly Roll, using interest rate data.
        AddAlpha(new JellyRollAlphaModel(this));
        // Invest in the same number of contracts per leg in the Jelly Roll.
        SetPortfolioConstruction(new SingleSharePortfolioConstructionModel());
    }

    private class AtmOptionHorizontalSpreadUniverseSelectionModel : OptionUniverseSelectionModel
    {
        // 30d update with the SelectOptionChainSymbols function since the filter returns at least 30d expiry options.
        public AtmOptionHorizontalSpreadUniverseSelectionModel()
                : base(TimeSpan.FromDays(30), SelectOptionChainSymbols) {}

        private static IEnumerable<Symbol> SelectOptionChainSymbols(DateTime utcTime)
        {
            // We will focus only on SPX options since they have a relatively stable dividend yield, which we assume will remain the same over time.
            // Also, assignment handling is not required since it is cash-settled.
            return new[] {QuantConnect.Symbol.Create("SPX", SecurityType.IndexOption, Market.USA)};
        }

        protected override OptionFilterUniverse Filter(OptionFilterUniverse filter)
        {
            // To trade interest rates using options, Jelly Roll is one of the best strategies.
            // It is market-neutral but sensitive to interest rate and dividend yield changes.
            // We target to trade the market speculation between 30d and 90d options interest rate.
            return filter.JellyRoll(0, 30, 90);
        }
    }

    private class JellyRollAlphaModel : AlphaModel
    {
        private QCAlgorithm _algorithm;
        private Symbol _symbol = QuantConnect.Symbol.Create("SPX", SecurityType.IndexOption, Market.USA);
        private bool _wasRising;
        // Use a 365d SMA indicator of daily interest rate to estimate if the interest rate cycle is upward or downward.
        private SimpleMovingAverage _sma = new(365);

        public JellyRollAlphaModel(QCAlgorithm algorithm)
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

            // Set a schedule to update the interest rate trend indicator daily.
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

            // Hold one position group at a time.
            if (algorithm.Portfolio.Invested || !slice.OptionChains.TryGetValue(_symbol, out var chain))
            {
                return insights;
            }

            // Obtain the Jelly Roll constituents from the option chain.
            var calls = chain.Where(x => x.Right == OptionRight.Call)
                .OrderBy(x => x.Expiry)
                .ToList();
            var puts = chain.Where(x => x.Right == OptionRight.Put)
                .OrderBy(x => x.Expiry)
                .ToList();
            var nearCall = calls[0];
            var farCall = calls[^1];
            var nearPut = puts[0];
            var farPut = puts[^1];

            // Emit insight of the Jelly Roll constituents, with directions depending on the interest rate trend given by SMA.
            var rate = algorithm.RiskFreeInterestRateModel.GetInterestRate(algorithm.Time);
            List<Insight> insightGroup;
            // During the rising interest rate cycle, order a long Jelly Roll.
            if (rate > _sma)
            {
                insightGroup = new() {
                    Insight.Price(nearCall.Symbol, TimeSpan.FromDays(30), InsightDirection.Down),
                    Insight.Price(farCall.Symbol, TimeSpan.FromDays(30), InsightDirection.Up),
                    Insight.Price(nearPut.Symbol, TimeSpan.FromDays(30), InsightDirection.Up),
                    Insight.Price(farPut.Symbol, TimeSpan.FromDays(30), InsightDirection.Down)
                };
            }
            // During a downward interest rate cycle, order short Jelly Roll.
            else if (rate < _sma)
            {
                insightGroup = new() {
                    Insight.Price(nearCall.Symbol, TimeSpan.FromDays(30), InsightDirection.Up),
                    Insight.Price(farCall.Symbol, TimeSpan.FromDays(30), InsightDirection.Down),
                    Insight.Price(nearPut.Symbol, TimeSpan.FromDays(30), InsightDirection.Down),
                    Insight.Price(farPut.Symbol, TimeSpan.FromDays(30), InsightDirection.Up)
                };
            }
            // If the interest rate cycle remains steady for a long time, we expect a flip in the cycle soon.
            else if (_wasRising)
            {
                insightGroup = new() {
                    Insight.Price(nearCall.Symbol, TimeSpan.FromDays(30), InsightDirection.Up),
                    Insight.Price(farCall.Symbol, TimeSpan.FromDays(30), InsightDirection.Down),
                    Insight.Price(nearPut.Symbol, TimeSpan.FromDays(30), InsightDirection.Down),
                    Insight.Price(farPut.Symbol, TimeSpan.FromDays(30), InsightDirection.Up)
                };
            }
            else
            {
                insightGroup = new() {
                    Insight.Price(nearCall.Symbol, TimeSpan.FromDays(30), InsightDirection.Down),
                    Insight.Price(farCall.Symbol, TimeSpan.FromDays(30), InsightDirection.Up),
                    Insight.Price(nearPut.Symbol, TimeSpan.FromDays(30), InsightDirection.Up),
                    Insight.Price(farPut.Symbol, TimeSpan.FromDays(30), InsightDirection.Down)
                };
            }
            insights.AddRange(insightGroup);

            return insights;
        }
    }

    private class SingleSharePortfolioConstructionModel : PortfolioConstructionModel
    {
        public override IEnumerable<PortfolioTarget> CreateTargets(QCAlgorithm algorithm, Insight[] insights)
        {
            var targets = new List<PortfolioTarget>();
            foreach (var insight in insights)
            {
                if (algorithm.Securities[insight.Symbol].IsTradable)
                {
                    // Use integer target to create a portfolio target to trade a single contract
                    targets.Add(new PortfolioTarget(insight.Symbol, (int) insight.Direction));
                }
            }
            return targets;
        }
    }
}
```

```
from Selection.OptionUniverseSelectionModel import OptionUniverseSelectionModel

class FrameworkOptionUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2023, 1, 1)
        self.set_end_date(2023, 8, 1)

        # Add a universe that selects the needed option contracts.
        self.add_universe_selection(AtmOptionHorizontalSpreadUniverseSelectionModel())
        # Add Alpha model to trade Jelly Roll, using interest rate data.
        self.add_alpha(JellyRollAlphaModel(self))
        # Invest in the same number of contracts per leg in the Jelly Roll.
        self.set_portfolio_construction(SingleSharePortfolioConstructionModel())

class AtmOptionHorizontalSpreadUniverseSelectionModel(OptionUniverseSelectionModel):
    # 30d update with the SelectOptionChainSymbols function since the filter returns at least 30d expiry options.
    def __init__(self) -> None:
        super().__init__(timedelta(30), self.selection_option_chain_symbols)

    def selection_option_chain_symbols(self, utc_time: datetime) -> list[Symbol]:
        # We will focus only on SPX options since they have a relatively stable dividend yield, which we assume will remain the same over time.
        # Also, assignment handling is not required since it is cash-settled.
        return [Symbol.create("SPX", SecurityType.INDEX_OPTION, Market.USA)]

    def filter(self, filter: OptionFilterUniverse) -> OptionFilterUniverse:
        # Jelly Roll is one of the best strategies for trading interest rates using options.
        # It is market-neutral but sensitive to interest rate and dividend yield changes.
        # We target to trade the market speculation between 30d and 90d options interest rate.
        return filter.jelly_roll(0, 30, 90)

class JellyRollAlphaModel(AlphaModel):
    _symbol = Symbol.create("SPX", SecurityType.INDEX_OPTION, Market.USA)
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

        # Hold one position group at a time.
        chain = slice.option_chains.get(self._symbol)
        if algorithm.portfolio.invested or not chain:
            return insights

        # Obtain the Jelly Roll constituents from the option chain.
        calls = sorted([x for x in chain if x.right == OptionRight.CALL], key=lambda x: x.expiry)
        puts = sorted([x for x in chain if x.right == OptionRight.PUT], key=lambda x: x.expiry)
        near_call = calls[0]
        far_call = calls[-1]
        near_put = puts[0]
        far_put = puts[-1]

        # Emit insight of the Jelly Roll constituents, with directions depending on the interest rate trend given by SMA.
        rate = algorithm.risk_free_interest_rate_model.get_interest_rate(algorithm.time)
        # During the rising interest rate cycle, order a long Jelly Roll.
        if rate > self._sma.current.value:
            insights.extend([\
                Insight.price(near_call.symbol, timedelta(30), InsightDirection.DOWN),\
                Insight.price(far_call.symbol, timedelta(30), InsightDirection.UP),\
                Insight.price(near_put.symbol, timedelta(30), InsightDirection.UP),\
                Insight.price(far_put.symbol, timedelta(30), InsightDirection.DOWN)\
            ])
        # During the downward interest rate cycle, order short Jelly Roll.
        elif rate < self._sma.current.value:
            insights.extend([\
                Insight.price(near_call.symbol, timedelta(30), InsightDirection.UP),\
                Insight.price(far_call.symbol, timedelta(30), InsightDirection.DOWN),\
                Insight.price(near_put.symbol, timedelta(30), InsightDirection.DOWN),\
                Insight.price(far_put.symbol, timedelta(30), InsightDirection.UP)\
            ])
        # If the interest rate cycle is steady for a long, we expect a flip in the cycle coming up.
        elif self._was_rising:
            insights.extend([\
                Insight.price(near_call.symbol, timedelta(30), InsightDirection.UP),\
                Insight.price(far_call.symbol, timedelta(30), InsightDirection.DOWN),\
                Insight.price(near_put.symbol, timedelta(30), InsightDirection.DOWN),\
                Insight.price(far_put.symbol, timedelta(30), InsightDirection.UP)\
            ])
        else:
            insights.extend([\
                Insight.price(near_call.symbol, timedelta(30), InsightDirection.DOWN),\
                Insight.price(far_call.symbol, timedelta(30), InsightDirection.UP),\
                Insight.price(near_put.symbol, timedelta(30), InsightDirection.UP),\
                Insight.price(far_put.symbol, timedelta(30), InsightDirection.DOWN)\
            ])

        return insights

class SingleSharePortfolioConstructionModel(PortfolioConstructionModel):
    def create_targets(self, algorithm: QCAlgorithm, insights: list[Insight]) -> list[PortfolioTarget]:
        targets = []
        for insight in insights:
            if algorithm.securities[insight.symbol].is_tradable:
                # Use integer target to create a portfolio target to trade a single contract
                targets.append(PortfolioTarget(insight.symbol, insight.direction))
        return targets
```

The following example chains a [fundamental universe](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/equity/fundamental-universes) and an [Equity Options universe](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/equity-options).
It first selects 10 stocks with the lowest PE ratio and then selects their front-month call Option contracts.
It buys one front-month call Option contract every day.

To override the default [pricing model](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/options-models/pricing) of the Options, [set a pricing model](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/options-models/pricing#03-Set-Models) in a security initializer.

To override the [initial guess of implied volatility](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/options-models/pricing#96-What-Is-Implied-Volatility3F), set and warm up the underlying [volatility model](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/options-models/volatility/key-concepts).

Select Language: C#Python

```
// Example code to chain a fundamental universe and an Equity Options universe by selecting top 10 stocks with lowest PE, indicating potentially undervalued stocks and then selecting their from-month call Option contracts to target contracts with high liquidity.
public class ETFUniverseOptions : QCAlgorithm
{
    private int _day;
    public override void Initialize()
    {
        SetStartDate(2023, 2, 2);
        SetCash(100000);
        UniverseSettings.Asynchronous = true;
        UniverseSettings.DataNormalizationMode = DataNormalizationMode.Raw;
        SetSecurityInitializer(new CustomSecurityInitializer(this));

        var universe = AddUniverse(FundamentalFunction);
        AddUniverseOptions(universe, OptionFilterFunction);
    }

    private IEnumerable<Symbol> FundamentalFunction(IEnumerable<Fundamental> fundamental)
    {
        return fundamental
            .Where(f => !double.IsNaN(f.ValuationRatios.PERatio))
            .OrderBy(f => f.ValuationRatios.PERatio)
            .Take(10)
            .Select(x => x.Symbol);
    }

    private OptionFilterUniverse OptionFilterFunction(OptionFilterUniverse optionFilterUniverse)
    {
        return optionFilterUniverse.Strikes(-2, +2).FrontMonth().CallsOnly();
    }

    public override void OnData(Slice data)
    {
        if (IsWarmingUp || _day == Time.Day) return;

        foreach (var (symbol, chain) in data.OptionChains)
        {
            if (Portfolio[chain.Underlying.Symbol].Invested)
                Liquidate(chain.Underlying.Symbol);

            var spot = chain.Underlying.Price;
            var contract = chain.OrderBy(x => Math.Abs(spot-x.Strike)).FirstOrDefault();
            var tag = $"IV: {contract.ImpliedVolatility:F3} Δ: {contract.Greeks.Delta:F3}";
            MarketOrder(contract.Symbol, 1, true, tag);
            _day = Time.Day;
        }
    }
}

internal class CustomSecurityInitializer : BrokerageModelSecurityInitializer
{
    private QCAlgorithm _algorithm;
    public CustomSecurityInitializer(QCAlgorithm algorithm)
        : base(algorithm.BrokerageModel, new FuncSecuritySeeder(algorithm.GetLastKnownPrices))
    {
        _algorithm = algorithm;
    }

    public override void Initialize(Security security)
    {
        // First, call the superclass definition
        // This method sets the reality models of each security using the default reality models of the brokerage model
        base.Initialize(security);

        // Next, overwrite the price model
        if (security.Type == SecurityType.Option) // Option type
        {
            (security as Option).PriceModel = OptionPriceModels.CrankNicolsonFD();
        }

        // Overwrite the volatility model and warm it up
        if (security.Type == SecurityType.Equity)
        {
            security.VolatilityModel = new StandardDeviationOfReturnsVolatilityModel(30);
            var tradeBars = _algorithm.History(security.Symbol, 30, Resolution.Daily);
            foreach (var tradeBar in tradeBars)
                security.VolatilityModel.Update(security, tradeBar);
        }
    }
}
```

```
# Example code to chain a fundamental universe and an Equity Options universe by selecting top 10 stocks with lowest PE, indicating potentially undervalued stocks and then selecting their from-month call Option contracts to target contracts with high liquidity.
from AlgorithmImports import *

    class ChainedUniverseAlgorithm(QCAlgorithm):
    def initialize(self):
        self.set_start_date(2023, 2, 2)
        self.set_cash(100000)
        self.universe_settings.asynchronous = True
        self.universe_settings.data_normalization_mode = DataNormalizationMode.RAW
        self.set_security_initializer(CustomSecurityInitializer(self))

        universe = self.add_universe(self.fundamental_function)
        self.add_universe_options(universe, self.option_filter_function)
        self.day = 0

    def fundamental_function(self, fundamental: list[Fundamental]) -> list[Symbol]:
        filtered = (f for f in fundamental if not np.isnan(f.valuation_ratios.pe_ratio))
        sorted_by_pe_ratio = sorted(filtered, key=lambda f: f.valuation_ratios.pe_ratio)
        return [f.symbol for f in sorted_by_pe_ratio[:10]]

    def option_filter_function(self, option_filter_universe: OptionFilterUniverse) -> OptionFilterUniverse:
        return option_filter_universe.strikes(-2, +2).front_month().calls_only()

    def on_data(self, data: Slice) -> None:
        if self.is_warming_up or self.day == self.time.day:
            return

        for symbol, chain in data.option_chains.items():
            if self.portfolio[chain.underlying.symbol].invested:
                self.liquidate(chain.underlying.symbol)

            spot = chain.underlying.price
            contract = sorted(chain, key=lambda x: abs(spot-x.strike))[0]
            tag = f"IV: {contract.implied_volatility:.3f} Δ: {contract.greeks.delta:.3f}"
            self.market_order(contract.symbol, 1, True, tag)
            self.day = self.time.day

class CustomSecurityInitializer(BrokerageModelSecurityInitializer):
    def __init__(self, algorithm: QCAlgorithm) -> None:
        super().__init__(algorithm.brokerage_model, FuncSecuritySeeder(algorithm.get_last_known_prices))
        self.algorithm = algorithm

    def initialize(self, security: Security) -> None:
        # First, call the superclass definition
        # This method sets the reality models of each security using the default reality models of the brokerage model
        super().initialize(security)

        # Overwrite the price model
        if security.type == SecurityType.OPTION: # Option type
            security.price_model = OptionPriceModels.crank_nicolson_fd()

        # Overwrite the volatility model and warm it up
        if security.type == SecurityType.EQUITY:
            security.volatility_model = StandardDeviationOfReturnsVolatilityModel(30)
            trade_bars = self.algorithm.history[TradeBar](security.symbol, 30, Resolution.DAILY)
            for trade_bar in trade_bars:
                security.volatility_model.update(security, trade_bar)
```

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/02%20Universe%20Selection/08%20Options%20Universes)