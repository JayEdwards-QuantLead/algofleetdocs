# Universe Selection

## ETF Constituents Universes

### Introduction

The `ETFConstituentsUniverseSelectionModel` selects a universe of US Equities based on the constituents of an ETF. These Universe Selection models rely on the [US ETF Constituents](https://www.quantconnect.com/datasets/quantconnect-us-etf-constituents) dataset. They run on a daily schedule by default. To adjust the selection schedule, see [Schedule](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/universe-settings#10-Schedule).

### Add ETF Constituents Universe Selection

To add an `ETFConstituentsUniverseSelectionModel` to your algorithm, in the `Initialize` `initialize` method, call the `AddUniverseSelection` `add_universe_selection` method. The `ETFConstituentsUniverseSelectionModel` constructor expects an ETF ticker.

Select Language: C#Python

```
// Run universe selection asynchronously to speed up your algorithm.
UniverseSettings.Asynchronous = true;
AddUniverseSelection(new ETFConstituentsUniverseSelectionModel("SPY"));
```

```
# Run universe selection asynchronously to speed up your algorithm.
self.universe_settings.asynchronous = True
self.add_universe_selection(ETFConstituentsUniverseSelectionModel("SPY"))
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `etfTicker` `etf_ticker` | `string` | Ticker of the ETF to get constituents for. To view the available ETFs, see [Supported ETFs](https://www.quantconnect.com/docs/v2/writing-algorithms/datasets/quantconnect/us-etf-constituents#08-Supported-ETFs). |  |
| `universeSettings` `universe_settings` | `UniverseSettings` | The [universe settings](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/universe-settings). If you don't provide an argument, the model uses the `algorithm.UniverseSettings` `algorithm.universe_settings` by default. | `None` |
| `universeFilterFunc` `universe_filter_func` | `Func<IEnumerable<ETFConstituentUniverse>, IEnumerable<Symbol>>` `Callable[[list[ETFConstituentUniverse]], list[Symbol]]` | Function to filter ETF constituents. If you don't provide an argument, the model selects all of the ETF constituents by default. | `None` `null` |

If you provide a `universeFilterFunc` `universe_filter_func` argument, you can use the following attributes of the `ETFConstituentUniverse` objects to select your universe:

The following example shows how to select the 10 Equities with the largest weight in the SPY ETF:

Select Language: C#Python

```
// Initialize asynchronous settings for speed and use the ETFConstituentsUniverseSelectionModel
// to select the top 10 SPY constituents by weight, focusing on blue-chip stocks with minimal risk.
public override void Initialize()
{
    UniverseSettings.Asynchronous = true;
    AddUniverseSelection(
        new ETFConstituentsUniverseSelectionModel("SPY", universeFilterFunc: ETFConstituentsFilter)
    );
}

private IEnumerable<Symbol> ETFConstituentsFilter(IEnumerable<ETFConstituentUniverse> constituents)
{
    // Select the 10 largest Equities in the ETF.
    return constituents.OrderByDescending(c => c.Weight).Take(10).Select(c => c.Symbol);
}
```

```
# Initialize asynchronous settings for speed and use the ETFConstituentsUniverseSelectionModel
# to select the top 10 SPY constituents by weight, focusing on blue-chip stocks with minimal risk.
def initialize(self) -> None:
    self.universe_settings.asynchronous = True
    self.add_universe_selection(
        ETFConstituentsUniverseSelectionModel("SPY", universe_filter_func=self._etf_constituents_filter)
    )

def _etf_constituents_filter(self, constituents: list[ETFConstituentUniverse]) -> list[Symbol]:
    # Select the 10 largest Equities in the ETF.
    selected = sorted(
        [c for c in constituents if c.weight],
        key=lambda c: c.weight, reverse=True
    )[:10]
    return [c.symbol for c in selected]
```

To move the ETF `Symbol` and the selection function outside of the algorithm class, create a universe selection model that inherits the `ETFConstituentsUniverseSelectionModel` class.

Select Language: C#Python

```
// Initialize asynchronous settings for speed and use the LargestWeightSPYETFUniverseSelectionModel
// to select the top 10 blue-chip SPY constituents by weight, focusing on stocks with minimal risk.
UniverseSettings.Asynchronous = true;
AddUniverseSelection(new LargestWeightSPYETFUniverseSelectionModel());

// Outside of the algorithm class
class LargestWeightSPYETFUniverseSelectionModel : ETFConstituentsUniverseSelectionModel
{
    public LargestWeightSPYETFUniverseSelectionModel(UniverseSettings universeSettings = null)
        : base("SPY", universeFilterFunc: ETFConstituentsFilter)
    {
    }

    private static IEnumerable<Symbol> ETFConstituentsFilter(IEnumerable<ETFConstituentUniverse> constituents)
    {
        // Select the 10 largest Equities in the ETF.
        return constituents.OrderByDescending(c => c.Weight).Take(10).Select(c => c.Symbol);
    }
}
```

```
# Initialize asynchronous settings for speed and use the LargestWeightSPYETFUniverseSelectionModel
# to select the top 10 blue-chip SPY constituents by weight, focusing on stocks with minimal risk.
self.universe_settings.asynchronous = True
self.add_universe_selection(LargestWeightSPYETFUniverseSelectionModel())

# Outside of the algorithm class
class LargestWeightSPYETFUniverseSelectionModel(ETFConstituentsUniverseSelectionModel):

    def __init__(self) -> None:
        super().__init__('SPY', universe_filter_func=self._etf_constituents_filter)

    def _etf_constituents_filter(self, constituents: list[ETFConstituentUniverse]) -> list[Symbol]:
        # Select the 10 largest Equities in the ETF.
        selected = sorted(
            [c for c in constituents if c.weight],
            key=lambda c: c.weight, reverse=True
        )[:10]
        return [c.symbol for c in selected]
```

To return the current universe constituents from the selection function, return `Universe.UnchangedUNCHANGED`.

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/ETFConstituentsUniverseSelectionModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/ETFConstituentsUniverseSelectionModel.py).

### Examples

The following examples demonstrate some common practices for implementing the framework ETF constituent universe selection model.

#### Example 1: Weighted ETF Constituents

A subset of the SPY constituents outperform the SPY while many underperform the overall index. In an attempt to buy the ETF constituents that outperform the index, the following algorithm buys the top 50 weighted assets in the ETF. In this example, we will pass a function to the `ETFConstituentsUniverseSelectionModel` for selection.

Select Language: C#Python

```
public class FrameworkETFConstituentsUniverseSelectionAlgorithm : QCAlgorithm
{
    public Dictionary<Symbol, double> EtfWeightBySymbol { get; set; } = new();

    public override void Initialize()
    {
        SetStartDate(2023, 6, 1);
        SetEndDate(2023, 8, 1);
        SetCash(10000000);

        // Add a universe of the SPY constituents.
        AddUniverseSelection(
            new ETFConstituentsUniverseSelectionModel("SPY", universeFilterFunc: ETFConstituentsFilter)
        );
        // Add an Alpha model to trade based on the constituent weights.
        AddAlpha(new EtfAlphaModel(this));
        // Position sizing was handled by insight weight (from ETF weight).
        SetPortfolioConstruction(new InsightWeightingPortfolioConstructionModel());
    }

    private IEnumerable<Symbol> ETFConstituentsFilter(IEnumerable<ETFConstituentUniverse> constituents)
    {
        // Cache the constituent weights in a dictionary for filtering and position sizing.
        EtfWeightBySymbol = constituents
            .Where(c => c.Weight.HasValue)
            .ToDictionary(c => c.Symbol, c => (double)c.Weight.Value);
        // Select the 50 constituents with the largest weight in the ETF.
        // They should have positive excess return.
        return EtfWeightBySymbol
            .OrderByDescending(x => x.Value)
            .Take(50)
            .Select(x => x.Key);
    }
}

public class EtfAlphaModel : AlphaModel
{
    private FrameworkETFConstituentsUniverseSelectionAlgorithm _algorithm;
    private static int _day = -1;

    public EtfAlphaModel(FrameworkETFConstituentsUniverseSelectionAlgorithm algorithm)
    {
        _algorithm = algorithm;
    }

    public override IEnumerable<Insight> Update(QCAlgorithm algorithm, Slice slice)
    {
        var insights = new List<Insight>();

        // Daily rebalance only.
        if (slice.Time.Day == _day)
        {
            return insights;
        }

        foreach (var (symbol, weight) in _algorithm.EtfWeightBySymbol)
        {
            // Get the ETF weight of all the assets currently in the universe.
            // To avoid trading errors, skip assets that have no price yet.
            if (slice.Bars.ContainsKey(symbol))
            {
                insights.Add(Insight.Price(symbol, TimeSpan.FromDays(1), InsightDirection.Up, weight: weight));
            }
        }

        _day = slice.Time.Day;

        return insights;
    }
}
```

```
class FrameworkETFConstituentsUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2023, 6, 1)
        self.set_end_date(2023, 8, 1)
        self.set_cash(10000000)

        self.etf_weight_by_symbol = {}

        # Add a universe of the SPY constituents.
        self.add_universe_selection(
            ETFConstituentsUniverseSelectionModel("SPY", universe_filter_func=self._etf_constituents_filter)
        )
        # Add an Alpha model to trade based on the constituent weights.
        self.add_alpha(EtfAlphaModel(self))
        # Position sizing was handled by insight weight (from ETF weight).
        self.set_portfolio_construction(InsightWeightingPortfolioConstructionModel())

    def _etf_constituents_filter(self, constituents: list[ETFConstituentUniverse]) -> list[Symbol]:
        # Cache the constituent weights in a dictionary for filtering and position sizing.
        self.etf_weight_by_symbol = {x.symbol: x.weight for x in constituents if x.weight}
        # Select the 50 constituents with the largest weight in the ETF.
        # They should have positive excess returns.
        return [x[0] for x in sorted(self.etf_weight_by_symbol.items(), key=lambda x: x[1], reverse=True)[:50]]

class EtfAlphaModel(AlphaModel):
    def __init__(self, algorithm: FrameworkETFConstituentsUniverseSelectionAlgorithm) -> None:
        self._algorithm = algorithm
        self._day = -1

    def update(self, algorithm: QCAlgorithm, slice: Slice) -> list[Insight]:
        insights = []

        # Daily rebalance only.
        if self._day == slice.time.day:
            return insights

        for symbol, weight in self._algorithm.etf_weight_by_symbol.items():
            # Get the ETF weight of all the assets currently in the universe.
            # To avoid trading errors, skip assets that have no price yet.
            if symbol in slice.bars:
                insights.append(Insight.price(symbol, timedelta(1), InsightDirection.UP, weight=weight))

        self._day = slice.time.day

        return insights
```

#### Example 2: Weight Trend Selection

The following algorithm trades the QQQ constituents with the upward trend of its weight constitution, indicated by an EMA indicator, suggesting its price trend is outperforming other constituents, or its weight was promoted by the NASDAQ index, which brings positive sentiment. To better utilize the EMA indicator, we inherit the `ETFConstituentsUniverseSelectionModel` superclass to create a custom universe selection model.

Select Language: C#Python

```
public class FrameworkETFConstituentsUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2023, 6, 1);
        SetEndDate(2023, 8, 1);
        SetCash(10000000);

        // Add a universe of the SPY constituents.
        AddUniverseSelection(
            new UptrendETFUniverseSelectionModel("QQQ")
        );
        // Add Alpha model to trade based on the selections.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(1)));
        // Equally invest in insights to dissipate the capital risk evenly.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());

        // Warm up the indicators.
        SetWarmUp(60, Resolution.Daily);
    }
}

class UptrendETFUniverseSelectionModel : ETFConstituentsUniverseSelectionModel
{
    private static Dictionary<Symbol, ExponentialMovingAverage> _emaBySymbol = new();

    public UptrendETFUniverseSelectionModel(string etfTicker)
        : base(etfTicker, universeFilterFunc: ETFConstituentsFilter)
    {
    }

    private static IEnumerable<Symbol> ETFConstituentsFilter(IEnumerable<ETFConstituentUniverse> constituents)
    {
        // Remove the ones that are not in the ETF anymore.
        var toRemove = new HashSet<Symbol>(constituents.Select(x => x.Symbol).Except(_emaBySymbol.Keys));
        foreach (var symbol in toRemove)
        {
            _emaBySymbol.Remove(symbol);
        }

        foreach (var c in constituents)
        {
            if (_emaBySymbol.TryGetValue(c.Symbol, out var ema) && c.Weight.HasValue)
            {
                // Update EMA with the latest weight.
                ema.Update(c.EndTime, c.Weight.Value);
            }
            else
            {
                // Create an EMA to filter by trend.
                _emaBySymbol[c.Symbol] = new ExponentialMovingAverage(60);
            }
        }

        // Select the ones with the increasing trend of constituent weight.
        return _emaBySymbol.Where(kvp => kvp.Value.IsReady && kvp.Value > kvp.Value.Previous)
            .Select(kvp => kvp.Key);
    }
}
```

```
class FrameworkETFConstituentsUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2023, 6, 1)
        self.set_end_date(2023, 8, 1)
        self.set_cash(10000000)

        # Add a universe of the SPY constituents.
        self.add_universe_selection(
            UptrendETFUniverseSelectionModel("QQQ")
        )
        # Add Alpha model to trade based on the selections.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(1)))
        # Equally invest in insights to dissipate the capital risk evenly.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

        # Warm up the indicators.
        self.set_warm_up(60, Resolution.DAILY)

class UptrendETFUniverseSelectionModel(ETFConstituentsUniverseSelectionModel):
    ema_by_symbol = {}

    def __init__(self, etf_ticker: str) -> None:
        super().__init__(etf_ticker, universe_filter_func=self.etf_constituent_filter)

    def etf_constituent_filter(self, constituents: list[ETFConstituentUniverse]) -> list[Symbol]:
        # Remove the ones that are not in the ETF anymore.
        to_remove = set(self.ema_by_symbol.keys()).difference(set([x.symbol for x in constituents]))
        for symbol in to_remove:
            del self.ema_by_symbol[symbol]

        for c in constituents:
            symbol = c.symbol
            if symbol in self.ema_by_symbol and c.weight:
                # Update EMA with the latest weight.
                self.ema_by_symbol[symbol].update(c.EndTime, c.weight)
            else:
                # Create an EMA to filter by trend.
                self.ema_by_symbol[symbol] = ExponentialMovingAverage(60)

        # Select the ones with the increasing trend of constituent weight.
        return [symbol for symbol, ema in self.ema_by_symbol.items() if ema.is_ready and ema.current.value > ema.previous.value]
```

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/02%20Universe%20Selection/05%20ETF%20Constituents%20Universes)