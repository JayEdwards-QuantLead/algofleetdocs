# Universe Selection

## Futures Universes

### Introduction

A Future Universe Selection model selects contracts for a set of Futures.

### Future Universe Selection

The `FutureUniverseSelectionModel` selects all the contracts for a set of Futures you specify. To use this model, provide a `refreshInterval` `refresh_interval` and a selector function. The `refreshInterval` `refresh_interval` defines how frequently LEAN calls the selector function. The selector function receives a `DateTime` `datetime` object that represents the current Coordinated Universal Time (UTC) and returns a list of `Symbol` objects. The `Symbol` objects you return from the selector function are the Futures of the universe.

Select Language: C#Python

```
// Run universe selection asynchronously to speed up your algorithm.
// This setting means you cannot rely on the method or algorithm state between filter calls.
UniverseSettings.Asynchronous = true;
// Add a universe of E-mini S&P 500 Futures contracts.
AddUniverseSelection(
    new FutureUniverseSelectionModel(
        // Refresh the universe daily.
        TimeSpan.FromDays(1),
        _ => new List<Symbol> {{ QuantConnect.Symbol.Create(Futures.Indices.SP500EMini, SecurityType.Future, Market.CME) }}
    )
);
```

```
from Selection.FutureUniverseSelectionModel import FutureUniverseSelectionModel
# Run universe selection asynchronously to speed up your algorithm.
# This setting means you cannot rely on the method or algorithm state between filter calls.
self.universe_settings.asynchronous = True
# Add a universe of E-mini S&P 500 Futures contracts.
self.add_universe_selection(
    FutureUniverseSelectionModel(
        # Refresh the universe daily.
        timedelta(1),
        lambda _: [Symbol.create(Futures.Indices.SP500E_MINI, SecurityType.FUTURE, Market.CME)]
    )
)
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `refreshInterval` `refresh_interval` | `TimeSpan` `timedelta` | Time interval between universe refreshes |  |
| `futureChainSymbolSelector` `future_chain_symbol_selector` | `Func<DateTime, IEnumerable<Symbol>>` `Callable[[datetime], list[Symbol]]` | A function that selects the Future symbols for a given Coordinated Universal Time (UTC). To view the supported assets in the US Futures dataset, see [Supported Assets](https://www.quantconnect.com/docs/v2/writing-algorithms/datasets/algoseek/us-futures#09-Supported-Assets). |  |
| `universeSettings` `universe_settings` | `UniverseSettings` | The [universe settings](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/universe-settings). If you don't provide an argument, the model uses the `algorithm.UniverseSettings` `algorithm.universe_settings` by default. | `null` `None` |

The following example shows how to define the Future chain Symbol selector as an isolated method:

Select Language: C#Python

```
// In the Initialize method, add the FutureUniverseSelectionModel with a custom selection function.
public override void Initialize()
{
    AddUniverseSelection(
        new FutureUniverseSelectionModel(TimeSpan.FromDays(1), SelectFutureChainSymbols)
    );
}

private static IEnumerable<Symbol> SelectFutureChainSymbols(DateTime utcTime)
{
    // Add E-mini S&P 500 and Gold Futures to the universe.
    return new[] {
        QuantConnect.Symbol.Create(Futures.Indices.SP500EMini, SecurityType.Future, Market.CME),
        QuantConnect.Symbol.Create(Futures.Metals.Gold, SecurityType.Future, Market.COMEX)
    };
}
```

```
from Selection.FutureUniverseSelectionModel import FutureUniverseSelectionModel

# In the initialize method, add the FutureUniverseSelectionModel with a custom selection function.
def initialize(self) -> None:
    self.set_universe_selection(
        FutureUniverseSelectionModel(timedelta(days=1), self.select_future_chain_symbols)
    )

def select_future_chain_symbols(self, utc_time: datetime) -> list[Symbol]:
    # Add E-mini S&P 500 and Gold Futures to the universe.
    return [\
        Symbol.create(Futures.Indices.SP500E_MINI, SecurityType.FUTURE, Market.CME),\
        Symbol.create(Futures.Metals.GOLD, SecurityType.FUTURE, Market.COMEX)\
    ]
```

This model uses the default Future contract filter, which doesn't select any Futures contracts. To use a different filter, subclass the `FutureUniverseSelectionModel` and define a `Filter` `filter` method. The `Filter` `filter` method accepts and returns a `FutureFilterUniverse` object to select the Futures contracts. The following table describes the filter methods of the `FutureFilterUniverse` class:

|     |
| --- |
| `StandardsOnly()` `standards_only()` <br>Selects standard contracts |
| `FrontMonth()` `front_month()` <br>Selects the front month contract |
| `BackMonths()` `back_months()` <br>Selects the non-front month contracts |
| `BackMonth()` `back_month()` <br>Selects the back month contracts |
| `Expiration(TimeSpan minExpiry, TimeSpan maxExpiry)` `expiration(min_expiry: timedelta, max_expiry: timedelta)` <br>Selects contracts that expire within a range of dates relative to the current day |
| `Expiration(int minExpiryDays, int maxExpiryDays)` `expiration(min_expiry_days: int, max_expiry_days: int)` <br>Selects contracts that expire within a range of dates relative to the current day |
| `Contracts(IEnumerable<Symbol> contracts)` `contracts(contracts: list[Symbol])` <br>Selects a list of contracts |
| `Contracts(Func<IEnumerable<Symbol>, IEnumerable< Symbol>> contractSelector)` `contracts(contractSelector: Callable[[list[Symbol]], list[Symbol]])` <br>Selects contracts that a selector function selects |

The contract filter runs at the first time step of each day.

To move the Future chain Symbol selector and the contract selection function outside of the algorithm class, create a universe selection model that inherits the FundamentalUniverseSelectionModel class and override its Select method.

Select Language: C#Python

```
// In the Initialize method, define the universe settings and add data.
UniverseSettings.Asynchronous = true;
AddUniverseSelection(new FrontMonthFutureUniverseSelectionModel());

// Outside of the algorithm class, define the universe selection model.
class FrontMonthFutureUniverseSelectionModel : FutureUniverseSelectionModel
{
    public FrontMonthFutureUniverseSelectionModel()
        // Refresh the universe daily.
        : base(TimeSpan.FromDays(1), SelectFutureChainSymbols) {}

    private static IEnumerable<Symbol> SelectFutureChainSymbols(DateTime utcTime)
    {
        // Add E-mini S&P 500 and Gold Futures to the universe.
        return new List<Symbol> {
            QuantConnect.Symbol.Create(Futures.Indices.SP500EMini, SecurityType.Future, Market.CME),
            QuantConnect.Symbol.Create(Futures.Metals.Gold, SecurityType.Future, Market.COMEX)
        };
    }

    protected override FutureFilterUniverse Filter(FutureFilterUniverse filter)
    {
        // Select the front month contracts.
        return filter.FrontMonth();
    }
}
```

```
# In the initialize method, define the universe settings and add data.
self.universe_settings.asynchronous = True
self.add_universe_selection(FrontMonthFutureUniverseSelectionModel())

# Outside of the algorithm class, define the universe selection model.
class FrontMonthFutureUniverseSelectionModel(FutureUniverseSelectionModel):
    def __init__(self) -> None:
        # Refresh the universe daily.
        super().__init__(timedelta(1), self.select_future_chain_symbols)

    def select_future_chain_symbols(self, utc_time: datetime) -> list[Symbol]:
        # Add E-mini S&P 500 and Gold Futures to the universe.
        return [\
            Symbol.create(Futures.Indices.SP500E_MINI, SecurityType.FUTURE, Market.CME),\
            Symbol.create(Futures.Metals.GOLD, SecurityType.FUTURE, Market.COMEX)\
        ]

    def filter(self, filter: FutureFilterUniverse) -> FutureFilterUniverse:
        # Select the front month contracts.
        return filter.front_month()
```

Some of the preceding filter methods only set an internal enumeration in the `FutureFilterUniverse` that it uses later on in the filter process. This subset of filter methods don't immediately reduce the number of contract `Symbol` objects in the `FutureFilterUniverse`.

The `AddUniverseSelection` `add_universe_selection` method doesn't return a `Future` object like the [AddFutureadd\_future](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/futures#11-Create-Universes) method.
The `Future` object contains `Symbol` and `Mapped` `mapped` properties, which reference the [continuous contract](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/futures#12-Continous-Contracts) and the currently selected contract in the continuous contract series, respectively.
To get the `Future` object, define the `OnSecuritiesChanged` `on_securities_changed` method in your algorithm class or framework models and check the result of the `IsCanonical` method.

Select Language: C#Python

```
// Save the Future object if the security added to the universe is the canonical asset.
public override void OnSecuritiesChanged(QCAlgorithm algorithm, SecurityChanges changes)
{
    foreach (var security in changes.AddedSecurities)
    {
        if (security.Symbol.IsCanonical() && security.Type == SecurityType.Future)
        {
            _future = security as Future;
        }
    }
}
```

```
# Save the Future object if the security added to the universe is the canonical asset.
def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
    for security in changes.added_securities:
        if security.Symbol.IsCanonical():
            self.future = security
```

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/FutureUniverseSelectionModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/FutureUniverseSelectionModel.py).

### Open Interest Future Universe Selection

The `OpenInterestFutureUniverseSelectionModel` is an extension of the `FutureUniverseSelectionModel` that selects the contract with the greatest open interest on a daily basis.

Select Language: C#Python

```
// Enable asynchronous universe settings for faster performance.
UniverseSettings.Asynchronous = true;
// Add an OpenInterestFutureUniverseSelectionModel for E-mini S&P 500 Futures, incorporating contracts with high
// open interest into the trading universe.
AddUniverseSelection(
    new OpenInterestFutureUniverseSelectionModel(
        this,
        utcTime => new[] { QuantConnect.Symbol.Create(Futures.Indices.SP500EMini, SecurityType.Future, Market.CME) }
    )
);
```

```
# Enable asynchronous universe settings for faster performance.
self.universe_settings.asynchronous = True
# Add an OpenInterestFutureUniverseSelectionModel for E-mini S&P 500 Futures, incorporating contracts with high
# open interest into the trading universe.
self.add_universe_selection(
    OpenInterestFutureUniverseSelectionModel(
        self,
        lambda utc_time: [Symbol.create(Futures.Indices.SP500E_MINI, SecurityType.FUTURE, Market.CME)]
    )
)
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `algorithm` | `IAlgorithm` | Algorithm |  |
| `futureChainSymbolSelector` `future_chain_symbol_selector` | `Func<DateTime, IEnumerable<Symbol>>` `Callable[[datetime], list[Symbol]]` | A function that selects the Future symbols for a given Coordinated Universal Time (UTC). To view the supported assets in the US Futures dataset, see [Supported Assets](https://www.quantconnect.com/docs/v2/writing-algorithms/datasets/algoseek/us-futures#09-Supported-Assets). |  |
| `chainContractsLookupLimit` `chain_contracts_lookup_limit` | `int?` `int/None` | Limit on how many contracts to query for open interest | 6 |
| `resultsLimit` `results_limit` | `int?` `int/None` | Limit on how many contracts will be part of the universe | 1 |

The following example shows how to define the Future chain Symbol selector as an isolated method:

Select Language: C#Python

```
// In the Initialize method, define the universe settings and add a universe.
public override void Initialize()
{
    UniverseSettings.Asynchronous = true;
    AddUniverseSelection(
        new OpenInterestFutureUniverseSelectionModel(this, SelectFutureChainSymbols)
    );
}

// Define the selection function, which returns Symbol objects.
private static IEnumerable<Symbol> SelectFutureChainSymbols(DateTime utcTime)
{
    return new[] {
        QuantConnect.Symbol.Create(Futures.Indices.SP500EMini, SecurityType.Future, Market.CME),
        QuantConnect.Symbol.Create(Futures.Metals.Gold, SecurityType.Future, Market.COMEX)
    };
}

```

```
# In the Initialize method, define the universe settings and add a universe.
def initialize(self) -> None:
    self.universe_settings.asynchronous = True
    self.add_universe_selection(
        OpenInterestFutureUniverseSelectionModel(self, self.select_future_chain_symbols)
    )

# Define the selection function, which returns Symbol objects.
def select_future_chain_symbols(self, utc_time: datetime) -> list[Symbol]:
    return [\
        Symbol.create(Futures.Indices.SP500E_MINI, SecurityType.FUTURE, Market.CME),\
        Symbol.create(Futures.Metals.GOLD, SecurityType.FUTURE, Market.COMEX)\
    ]
```

To move the Future chain Symbol selector outside of the algorithm class, create a universe selection model that inherits the `OpenInterestFutureUniverseSelectionModel` class.

Select Language: C#Python

```
// In the Initialize method, define the universe settings and add a universe.
UniverseSettings.Asynchronous = true;
AddUniverseSelection(new GoldOpenInterestFutureUniverseSelectionModel(this));

// Outside of the algorithm class, define the universe selection model.
class GoldOpenInterestFutureUniverseSelectionModel : OpenInterestFutureUniverseSelectionModel
{
    public GoldOpenInterestFutureUniverseSelectionModel(QCAlgorithm algorithm,
        int? chainContractsLookupLimit = 6, int? resultsLimit = 1)
        : base(algorithm, SelectFutureChainSymbols, chainContractsLookupLimit, resultsLimit) {}

    private static IEnumerable<Symbol> SelectFutureChainSymbols(DateTime utcTime)
    {
        return new List<Symbol> {
            QuantConnect.Symbol.Create(Futures.Metals.Gold, SecurityType.Future, Market.COMEX)
        };
    }
}
```

```
# In the Initialize method, define the universe settings and add a universe.
self.universe_settings.asynchronous = True
self.add_universe_selection(GoldOpenInterestFutureUniverseSelectionModel(self))

# Outside of the algorithm class, define the universe selection model.
class GoldOpenInterestFutureUniverseSelectionModel(OpenInterestFutureUniverseSelectionModel):
    def __init__(self, algorithm: QCAlgorithm, chain_contracts_lookup_limit: int=6, results_limit: int=1):
        super().__init__(algorithm, self.select_future_chain_symbols, chain_contracts_lookup_limit, results_limit)

    def select_future_chain_symbols(self, utcTime: datetime) -> list[Symbol]:
        return [Symbol.Create(Futures.Metals.GOLD, SecurityType.FUTURE, Market.COMEX)]
```

The `AddUniverseSelection` `add_universe_selection` method doesn't return a `Future` object like the [AddFutureadd\_future](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/futures#11-Create-Universes) method.
The `Future` object contains `Symbol` and `Mapped` `mapped` properties, which reference the [continuous contract](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/futures#12-Continous-Contracts) and the currently selected contract in the continuous contract series, respectively.
To get the `Future` object, define the `OnSecuritiesChanged` `on_securities_changed` method in your algorithm class or framework models and check the result of the `IsCanonical` method.

Select Language: C#Python

```
// Save the Future object if the security added to the universe is the canonical asset.
public override void OnSecuritiesChanged(QCAlgorithm algorithm, SecurityChanges changes)
{
    foreach (var security in changes.AddedSecurities)
    {
        if (security.Symbol.IsCanonical() && security.Type == SecurityType.Future)
        {
            _future = security as Future;
        }
    }
}
```

```
# Save the Future object if the security added to the universe is the canonical asset.
def on_securities_changed(self, algorithm: QCAlgorithm, changes: SecurityChanges) -> None:
    for security in changes.added_securities:
        if security.Symbol.IsCanonical():
            self.future = security
```

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/OpenInterestFutureUniverseSelectionModel.cs).

### Examples

The following examples demonstrate some common practices for implementing a Futures universe selection model.

#### Example 1: Front Month Contracts

The following algorithm selects a list of Futures to buy and hold equally. To ensure liquidity and efficiency, we will select only their front-month contracts. We can do so in the algorithm framework through a child class inheriting the `FutureUniverseSelectionModel` while providing the list of Futures and overriding the `Filter` `filter` method to select the front month contracts.

Select Language: C#Python

```
public class FrameworkFutureUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2022, 1, 1);
        SetEndDate(2022, 2, 1);
        SetCash(100000);

        // It is usual to trade Futures during extended market hours.
        UniverseSettings.ExtendedMarketHours = true;
        // Add a fundamental universe with custom selection rules for filtering.
        AddUniverseSelection(new FrontMonthFutureUniverseSelectionModel(10));

        // Sent insights on buying and holding the selected securities.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(1)));
        // Evenly dissipate the capital risk among selected securities.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }

    private class FrontMonthFutureUniverseSelectionModel : FutureUniverseSelectionModel
    {
        /// Creates futures chain universes that select the front-month contract and run a user-defined
        /// futureChainSymbolSelector every day to enable choosing different futures chains
        public FrontMonthFutureUniverseSelectionModel(int rebalancePeriod)
            : base(TimeSpan.FromDays(rebalancePeriod), SelectFutureChainSymbols)
        {
        }

        protected override FutureFilterUniverse Filter(FutureFilterUniverse filter)
        {
            // Defines the futures chain universe filter to select only the front-month contracts at market open.
            return filter.FrontMonth().OnlyApplyFilterAtMarketOpen();
        }

        private static IEnumerable<Symbol> SelectFutureChainSymbols(DateTime utcTime)
        {
            return new List<Symbol>()
            {
                QuantConnect.Symbol.Create(Futures.Indices.VIX, SecurityType.Future, Market.CFE),
                QuantConnect.Symbol.Create(Futures.Indices.SP500EMini, SecurityType.Future, Market.CME),
                QuantConnect.Symbol.Create(Futures.Indices.NASDAQ100EMini, SecurityType.Future, Market.CME),
                QuantConnect.Symbol.Create(Futures.Indices.Dow30EMini, SecurityType.Future, Market.CBOT),
                QuantConnect.Symbol.Create(Futures.Energies.Gasoline, SecurityType.Future, Market.NYMEX),
                QuantConnect.Symbol.Create(Futures.Energies.HeatingOil, SecurityType.Future, Market.NYMEX),
                QuantConnect.Symbol.Create(Futures.Energies.NaturalGas, SecurityType.Future, Market.NYMEX),
                QuantConnect.Symbol.Create(Futures.Grains.Corn, SecurityType.Future, Market.CBOT),
                QuantConnect.Symbol.Create(Futures.Grains.Oats, SecurityType.Future, Market.CBOT),
                QuantConnect.Symbol.Create(Futures.Grains.Soybeans, SecurityType.Future, Market.CBOT),
                QuantConnect.Symbol.Create(Futures.Grains.Wheat, SecurityType.Future, Market.CBOT),
            };
        }
    }
}
```

```
from Selection.FutureUniverseSelectionModel import FutureUniverseSelectionModel

class FrameworkFutureUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2022, 1, 1)
        self.set_end_date(2022, 2, 1)
        self.set_cash(100000)

        # It is usual to trade Futures during extended market hours.
        self.universe_settings.extended_market_hours = True
        # Add a universe with custom selection rules for filtering.
        self.add_universe_selection(FrontMonthFutureUniverseSelectionModel(7))

        # Sent insights on buying and holding the selected securities.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(7)))
        # Evenly dissipate the capital risk among selected securities.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

class FrontMonthFutureUniverseSelectionModel(FutureUniverseSelectionModel):
    '''Creates futures chain universes that select the front month contract and run a user-defined
    futureChainSymbolSelector every day to enable choosing different futures chains'''
    def __init__(self, rebalance_period: int = 7) -> None:
        super().__init__(timedelta(rebalance_period), self.select_future_chain_symbols)

    def filter(self, filter: FutureFilterUniverse) -> FutureFilterUniverse:
        # Defines the futures chain universe filter to select only the front-month contracts at market open.
        return (filter.front_month()
                      .only_apply_filter_at_market_open())

    def select_future_chain_symbols(self, utc_time: datetime) -> list[Symbol]:
        return [\
            Symbol.create(Futures.Indices.VIX, SecurityType.FUTURE, Market.CFE),\
            Symbol.create(Futures.Indices.SP_500_E_MINI, SecurityType.FUTURE, Market.CME),\
            Symbol.create(Futures.Indices.NASDAQ_100_E_MINI, SecurityType.FUTURE, Market.CME),\
            Symbol.create(Futures.Indices.DOW_30_E_MINI, SecurityType.FUTURE, Market.CBOT),\
            Symbol.create(Futures.Energies.GASOLINE, SecurityType.FUTURE, Market.NYMEX),\
            Symbol.create(Futures.Energies.HEATING_OIL, SecurityType.FUTURE, Market.NYMEX),\
            Symbol.create(Futures.Energies.NATURAL_GAS, SecurityType.FUTURE, Market.NYMEX),\
            Symbol.create(Futures.Grains.CORN, SecurityType.FUTURE, Market.CBOT),\
            Symbol.create(Futures.Grains.OATS, SecurityType.FUTURE, Market.CBOT),\
            Symbol.create(Futures.Grains.SOYBEANS, SecurityType.FUTURE, Market.CBOT),\
            Symbol.create(Futures.Grains.WHEAT, SecurityType.FUTURE, Market.CBOT),\
        ]
```

#### Example 2: Seasonal Contracts

Some Futures exhibit a seasonality effect, i.e., a yearly cycle in its price fluctuation. Studies show that the Natural Gas contract has a yearly cycle of maximizing its price in early December and minimizing it in early April. It might be due to the high demand for natural gas during winter warming. The following algorithm will hold Natural Gas contracts from summer to winter with a similar universe selection model in Example 1, but using a custom selection function with a time argument to select the contract.

Select Language: C#Python

```
public class FrameworkFutureUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2018, 1, 1);
        SetEndDate(2024, 12, 1);
        SetCash(100000);

        // It is usual to trade Futures during extended market hours.
        UniverseSettings.ExtendedMarketHours = true;
        // We hold the Future and will not frequently trade it; Daily resolution is sufficient for computational efficiency.
        UniverseSettings.Resolution = Resolution.Daily;
        // Add a fundamental universe with custom selection rules for filtering.
        // Rescan every week to ensure we are on our plan.
        AddUniverseSelection(new FrontMonthFutureUniverseSelectionModel(7));

        // Sent insights on buying and holding the selected securities.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(7)));
        // Place orders for a single contract to control risk exposure and avoid over-leveraging.
        SetPortfolioConstruction(new SingleSharePortfolioConstructionModel());
    }

    private class FrontMonthFutureUniverseSelectionModel : FutureUniverseSelectionModel
    {
        /// Creates futures chain universes that select the front-month contract and run a user-defined
        /// futureChainSymbolSelector every day to enable choosing different futures chains
        public FrontMonthFutureUniverseSelectionModel(int rebalancePeriod)
            : base(TimeSpan.FromDays(rebalancePeriod), SelectFutureChainSymbols)
        {
        }

        protected override FutureFilterUniverse Filter(FutureFilterUniverse filter)
        {
            // Defines the futures chain universe filter to select only the front-month contracts at market open.
            return filter.FrontMonth().OnlyApplyFilterAtMarketOpen();
        }

        private static IEnumerable<Symbol> SelectFutureChainSymbols(DateTime utcTime)
        {
            // We hold Natural Gas from summer to winter since it is in an upward cycle.
            if (utcTime.Month >= 4 && utcTime.Month <= 11)
            {
                return new List<Symbol>()
                {
                    QuantConnect.Symbol.Create(Futures.Energies.NaturalGas, SecurityType.Future, Market.NYMEX)
                };
            }
            // We do not hold any contracts during the time of the downward cycle.
            return new List<Symbol>();
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
from Selection.FutureUniverseSelectionModel import FutureUniverseSelectionModel

class FrameworkFutureUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2018, 1, 1)
        self.set_end_date(2024, 12, 1)
        self.set_cash(100000)

        # It is usual to trade Futures during extended market hours.
        self.universe_settings.extended_market_hours = True
        # We hold the Future and will not frequently trade it; using Daily resolution is sufficient for computational efficiency.
        self.universe_settings.resolution = Resolution.DAILY
        # Add a universe with custom selection rules for filtering.
        # Rescan every week to ensure we are on our plan.
        self.add_universe_selection(FrontMonthFutureUniverseSelectionModel(7))

        # Sent insights on buying and holding the selected securities.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(7)))
        # Place orders for a single contract to control risk exposure and avoid over-leveraging.
        self.set_portfolio_construction(SingleSharePortfolioConstructionModel())

class FrontMonthFutureUniverseSelectionModel(FutureUniverseSelectionModel):
    '''Creates futures chain universes that select the front month contract and run a user-defined
     futureChainSymbolSelector every day to enable choosing different futures chains'''
    def __init__(self, rebalance_period: int = 7) -> None:
        super().__init__(timedelta(rebalance_period), self.select_future_chain_symbols)

    def filter(self, filter: FutureFilterUniverse) -> FutureFilterUniverse:
        # Defines the futures chain universe filter to select only the front-month contracts at market open.
        return (filter.front_month().only_apply_filter_at_market_open())

    def select_future_chain_symbols(self, utc_time: datetime) -> list[Symbol]:
        # We hold Natural Gas from summer to winter since it is in an upward cycle.
        if 4 <= utc_time.month <= 11:
            return [\
                Symbol.create(Futures.Energies.NATURAL_GAS, SecurityType.FUTURE, Market.NYMEX)\
            ]
        # We do not hold any contracts during the time of the downward cycle.
        return []

class SingleSharePortfolioConstructionModel(PortfolioConstructionModel):
    def create_targets(self, algorithm: QCAlgorithm, insights: list[Insight]) -> list[PortfolioTarget]:
        targets = []
        for insight in insights:
            if algorithm.securities[insight.symbol].is_tradable:
                # Use integer target to create a portfolio target to trade a single contract
                targets.append(PortfolioTarget(insight.symbol, insight.direction))
        return targets
```

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/02%20Universe%20Selection/07%20Futures%20Universes)