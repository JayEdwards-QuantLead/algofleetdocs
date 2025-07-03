# Universe Selection

## Fundamental Universes

### Introduction

A `FundamentalUniverseSelectionModel` selects a universe of US Equities based on `Fundamental` data. Depending on the `Fundamental` properties you use, these universes rely on the [US Equity Coarse Universe](https://www.quantconnect.com/datasets/quantconnect-us-coarse-universe-constituents) dataset, the [US Fundamental](https://www.quantconnect.com/datasets/morning-star-us-fundamentals) dataset, or both.

These types of universes operate on daily schedule by default. In backtests, they select assets at midnight. In live trading, the selection timing depends on the [data provider](https://www.quantconnect.com/docs/v2/cloud-platform/datasets) you use. To adjust the selection schedule, see [Schedule](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/universe-settings#10-Schedule).

If you use a fundamental Universe Selection model, the only way to unsubscribe from a security is to return a list from the selection function that doesn't include the security `Symbol`. The `RemoveSecurity` `remove_security` method doesn't work with these types of Universe Selection models.

### Fundamental Selection

The `FundamentalUniverseSelectionModel` lets you select stocks based on corporate `Fundamental` data.
You can specific the selection method, which takes a list of `Fundamental` objects as argument and returns a list of `Symbol` objects.

Select Language: C#Python

```
public override void Initialize()
{
    // Run asynchronous universe selection to speed up your algorithm.
    // In this case, you can't rely on the method or algorithm state between filter calls.
    UniverseSettings.Asynchronous = true;
    AddUniverseSelection(new FundamentalUniverseSelectionModel(FundamentalFilterFunction));
}

public override List<Symbol> FundamentalFilterFunction(List<Fundamental> fundamental)
{
    return (from f in fundamental
            // Select symbols with fundamental data and a price above $10.
            where f.Price > 10 && !Double.IsNaN(f.ValuationRatios.PERatio)
            // Sort the assets in ascending order by their P/E ratio.
            orderby f.ValuationRatios.PERatio
            // Select the first 10 assets.
            select f.Symbol).Take(10);
}
```

```
def initialize(self) -> None:
    # Run asynchronous universe selection to speed up your algorithm.
    # In this case, you can't rely on the method or algorithm state between filter calls.
    self.universe_settings.asynchronous = True
    self.add_universe_selection(FundamentalUniverseSelectionModel(self._fundamental_filter_function))

def _fundamental_filter_function(self, fundamental: list[Fundamental]):
    # Select symbols with fundamental data and a price above $10.
    filtered = [f for f in fundamental if f.price > 10 and not np.isnan(f.valuation_ratios.pe_ratio)]
    # Sort the assets in ascending order by their P/E ratio.
    sorted_by_pe_ratio = sorted(filtered, key=lambda f: f.valuation_ratios.pe_ratio)
    # Select the first 10 assets.
    return [f.symbol for f in sorted_by_pe_ratio[:10]]
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `selector` | ` Func<IEnumerable<Fundamental>, IEnumerable<Symbol>>` ` Callable[[list[Fundamental]], list[Symbol]]` | Filter function to select assets based on fundamental data. |  |
| `universeSettings` `universe_settings` | `UniverseSettings` | The [universe settings](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/universe-settings). If you don't provide an argument, the model uses the `algorithm.UniverseSettings` `algorithm.universe_settings` by default. | `None` `null` |

The `Fundamental` objects have the following properties:

To move the selection function outside of the algorithm class, create a universe selection model that inherits the `FundamentalUniverseSelectionModel` class and override its `Select` `select` method.

Select Language: C#Python

```
// In the Initialize method, enable asynchronous universe selection to speed up your algorithm.
UniverseSettings.Asynchronous = true;
// Add a custom universe selection model that selects undervalued, liquid stocks.
AddUniverseSelection(new LiquidAndLowPEUniverseSelectionModel());

// Outside of the algorithm class, define the universe selection model.
public class LiquidAndLowPEUniverseSelectionModel : FundamentalUniverseSelectionModel
{
    public override IEnumerable Select(QCAlgorithm algorithm, IEnumerable fundamental)
    {
        return fundamental
            // Select symbols with fundamental data and a price above $1.
            .Where(x => x.HasFundamentalData && x.Price > 1 && !Double.IsNaN(x.ValuationRatios.PERatio))
            // Select the 100 assets with the greatest daily dollar volume.
            .OrderByDescending(x => x.DollarVolume)
            .Take(100)
            // Select the 10 assets with the lowest PE ratio.
            .OrderByDescending(x => x.ValuationRatios.PERatio)
            .Take(10)
            .Select(x => x.Symbol);
    }
}
```

```
# In the initialize method, enable asynchronous universe selection to speed up your algorithm.
self.universe_settings.asynchronous = True
# Add a custom universe selection model that selects undervalued, liquid stocks.
self.add_universe_selection(LiquidAndLowPEUniverseSelectionModel())

# Outside of the algorithm class, define the universe selection model.
class LiquidAndLowPEUniverseSelectionModel(FundamentalUniverseSelectionModel):
    def select(self, fundamental: list[Fundamental]) -> list[Symbol]:
        # Select symbols with fundamental data and a price above $1.
        filtered = [x for x in fundamental if x.price > 1 and not np.isnan(x.valuation_ratios.pe_ratio)]
        # Select the 100 assets with the greatest daily dollar volume.
        most_liquid = sorted(filtered, key=lambda x: x.dollar_volume, reverse=True)[:100]
        # Select the 10 assets with the lowest PE ratio.
        lowest_pe_ratio = sorted(most_liquid, key=lambda x: x.valuation_ratios.pe_ratio, reverse=True)[:10]
        return [x.Symbol for x in lowest_pe_ratio]
```

To return the current universe constituents from the selection function, return `Universe.UnchangedUNCHANGED`.

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/FundamentalUniverseSelectionModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/FundamentalUniverseSelectionModel.cs).

### EMA Cross Selection

The `EmaCrossUniverseSelectionModel` applies two exponential moving average (EMA) indicators to the price history of assets and then selects the assets that have their fast EMA furthest above their slow EMA on a percentage basis.

Select Language: C#Python

```
// Initialize asynchronous universe settings for faster processing and add EmaCrossUniverseSelectionModel to dynamically manage the universe based on EMA cross signals to identify trending assets.
public override void Initialize()
{
    UniverseSettings.Asynchronous = true;
    AddUniverseSelection(new EmaCrossUniverseSelectionModel());
}
```

```
# Initialize asynchronous universe settings for faster processing and add EmaCrossUniverseSelectionModel to dynamically manage the universe based on EMA cross signals to identify trending assets.
def initialize(self) -> None:
    self.universe_settings.asynchronous = True
    self.add_universe_selection(EmaCrossUniverseSelectionModel())
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `fastPeriod` `fast_period` | `int` | Fast EMA period | 100 |
| `slowPeriod` `slow_period` | `int` | Slow EMA period | 300 |
| `universeCount` `universe_count` | `int` | Maximum number of members of this universe selection | 500 |
| `universeSettings` `universe_settings` | `UniverseSettings` | The [universe settings](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/universe-settings). If you don't provide an argument, the model uses the `algorithm.UniverseSettings` `algorithm.universe_settings` by default. | `None` `null` |

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/EmaCrossUniverseSelectionModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/EmaCrossUniverseSelectionModel.py).

### Examples

The following examples demonstrate some common practices for implementing the framework fundamental universe model.

#### Example 1: 50 Stocks >$10/Share and >$10M in Daily Trading Volume

The following algorithm selects the 50 most liquid US Equities above $10/share and $10M daily volume. We pass a function to the `FundamentalUniverseSelectionModel` to filter the fundamental data for stock selection.

Select Language: C#Python

```
public class FrameworkFundamentalUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2019, 1, 1);
        SetEndDate(2019, 12, 1);
        SetCash(100000);

        // Configure the universe to update at the start of each month. Most of the top 500 don't change very frequently.
        UniverseSettings.Schedule.On(DateRules.MonthStart());
        // Add a fundamental universe with custom selection rules for filtering.
        AddUniverseSelection(new FundamentalUniverseSelectionModel(fundamental =>
            (from f in fundamental
            //Large-cap non-penny stocks have more stable trends and demand.
            where f.Price > 10 && f.DollarVolume > 10000000
            orderby f.DollarVolume descending
            select f.Symbol).Take(500)
        ));

        // Sent insights on buying and holding the selected securities.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(30)));
        // Evenly dissipate the capital risk among selected securities.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }
}
```

```
class FrameworkFundamentalUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2019, 1, 1)
        self.set_end_date(2019, 12, 1)
        self.set_cash(100000)

        # Configure the universe to update at the start of each month. Most of the top 500 don't change very frequently.
        self.universe_settings.schedule.on(self.date_rules.month_start())
        # Add a universe with custom selection rules for filtering.
        self.add_universe_selection(FundamentalUniverseSelectionModel(
            lambda fundamental: [\
                x.symbol for x in sorted(\
                    # Large-cap non-penny stocks have a more stable trend and demand.\
                    [f for f in fundamental if f.price > 10 and f.dollar_volume > 10_000_000],\
                    key=lambda f: f.dollar_volume\
                )[-500:]\
            ]
        ))

        # Sent insights on buying and holding the most liquid cryptos for a week.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(30)))
        # Evenly dissipate the capital risk among selected cryptos.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())
```

#### Example 2: 10 Stocks Above Their 200-Day EMA With >$1B of Daily Trading Volume

Another common request is to filter the universe by a technical indicator, such as only picking stocks above their 200-day [Exponential Moving Average](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/exponential-moving-average) (EMA).
The `Fundamental` object has adjusted price and volume information so that you can do any price-related analysis.
The following algorithm defines a separate class to contain the indicator of each asset. We pass a function to the `FundamentalUniverseSelectionModel` to filter the fundamental data for stock selection.

Select Language: C#Python

```
using System.Collections.Concurrent;

public class FrameworkFrameworkUniverseSelectionAlgorithm : QCAlgorithm
{
    // Create a concurrent dictionary to store the EMA data for universe selection.
    private ConcurrentDictionary<Symbol, SelectionData> _selectionDataBySymbol = new();

    public override void Initialize()
    {
        SetStartDate(2019, 1, 1);
        SetEndDate(2019, 12, 1);
        SetCash(100000);

        // Add a fundamental universe with custom selection rules for filtering.
        AddUniverseSelection(new FundamentalUniverseSelectionModel(Selection));

        // Sent insights on buying and holding the selected securities.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(1)));
        // Evenly dissipate the capital risk among selected securities.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }

    private IEnumerable<Symbol> Selection(IEnumerable<Fundamental> fundamental)
    {
        return (from f in fundamental
            // Create/Update the EMA indicators of each stock.
            let avg = _selectionDataBySymbol.GetOrAdd(f.Symbol, sym => new SelectionData(this, f.Symbol, 200))
            where avg.Update(f.EndTime, f.AdjustedPrice)
            // Select the Equities above their EMA and have a daily volume of $1B.
            // These assets are in an uptrend and are very liquid.
            where avg.Ema.IsReady && f.Price > avg.Ema.Current.Value && f.DollarVolume > 1000000000
            // Select the 10 most liquid Equities to avoid extra slippage, and their trend can be capitalized more efficiently.
            orderby f.DollarVolume descending
            select f.Symbol).Take(10);
    }
}

// Create a separate class to contain the EMA information of each asset.
public class SelectionData
{
    public readonly ExponentialMovingAverage Ema;

    // Create an EMA indicator for trend estimation and filtering.
    public SelectionData(QCAlgorithm algorithm, Symbol symbol, int period)
    {
        Ema = new ExponentialMovingAverage(period);

        // Warm up the EMA indicator.
        algorithm.WarmUpIndicator(symbol, Ema, Resolution.Daily);
    }

    // Update your variables and indicators with the latest data.
    // You may also want to use the History API here to warm up the indicator.
    public bool Update(DateTime time, decimal value)
    {
        return Ema.Update(time, value);
    }
}
```

```
class FrameworkFundamentalUniverseSelectionAlgorithm(QCAlgorithm):
	# Create a dictionary to store the EMA data for universe selection.
    _selection_data_by_symbol = {}

    def initialize(self) -> None:
        self.set_start_date(2019, 1, 1)
        self.set_end_date(2019, 12, 1)
        self.set_cash(100000)

        # Add a universe with custom selection rules for filtering.
        self.add_universe_selection(FundamentalUniverseSelectionModel(self._select_assets))

        # Sent insights on buying and holding the most liquid cryptos for a week.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(1)))
        # Evenly dissipate the capital risk among selected cryptos.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

    def _select_assets(self, fundamental: list[Fundamental]) -> list[Symbol]:
        for f in fundamental:
            # Create/Update the EMA indicators of each stock.
            if f.symbol not in self._selection_data_by_symbol:
                self._selection_data_by_symbol[f.symbol] = SelectionData(self, f.symbol, 200)
            self._selection_data_by_symbol[f.symbol].update(f.end_time, f.adjusted_price, f.dollar_volume)

        # Select the Equities above their EMA and have a daily volume of $1B.
        # These assets are in an uptrend and are very liquid.
        selected = [x for x in self._selection_data_by_symbol.values() if x.is_above_ema and x.volume > 1_000_000_000]

        # Select the 10 most liquid Equities to avoid extra slippage.
        return [ x.symbol for x in sorted(selected, key=lambda x: x.volume)[-10:] ]

# Create a separate class to contain the EMA information of each asset.
class SelectionData(object):
    def __init__(self, algorithm: QCAlgorithm, symbol: Symbol, period: int):
        # Create an EMA indicator for trend estimation and filtering.
        self.symbol = symbol
        self._ema = ExponentialMovingAverage(period)
        self.is_above_ema = False
        self.volume = 0

        # Warm up the EMA indicator.
        algorithm.warm_up_indicator(symbol, self._ema, Resolution.Daily);

    # Update your variables and indicators with the latest data.
    # You may also want to use the History API here to warm up the indicator.
    def update(self, time: datetime, price: float, volume: float) -> None:
        self.volume = volume
        if self._ema.update(time, price):
            self.is_above_ema = price > self._ema.current.value
```

In this example, the `SelectionData` class group variables for the universe selection and update the indicator of each asset.
We highly recommend you follow this pattern to keep your algorithm tidy and bug-free.
The following snippet shows an example implementation of the `SelectionData` class, but you can make whatever you need to store your custom universe filters.
Note that the preceding `SelectionData` class uses a [manual](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/manual-indicators) EMA indicator instead of the [automatic version](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/automatic-indicators).
For more information about universes that select assets based on indicators, see [Indicator Universes](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/indicator-universes).
You need to use a `SelectionData` class instead of assigning the EMA to the `Fundamental` object because you can't create custom propertiesattributes on `Fundamental` objects like you can with `Security` objects.

#### Example 3: 10 Stocks Furthest Above their 10-day SMA of Volume

The process to get the 10-day [Simple Moving Average](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/supported-indicators/simple-moving-average) (SMA) stock volume is the same process as in Example 2.
First, define a `SelectionData` class that performs the averaging.
This class tracks the ratio of today's volume relative to historical volumes.
You can use this ratio to select assets above their 10-day SMA and sort the results by the Equities that have had the most significant jump since yesterday.

We inherit the `FundamentalUniverseSelectionModel` to create a custom universe selection model. Overriding the `Select` `select`, we filter the fundamental data for stocks selection.

Select Language: C#Python

```
using System.Collections.Concurrent;

public class FrameworkFrameworkUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2019, 1, 1);
        SetEndDate(2019, 12, 1);
        SetCash(100000);

        // Add a fundamental universe with custom selection rules for filtering.
        AddUniverseSelection(new VolumeSMAUniverseSelectionModel(10));

        // Sent insights on buying and holding the selected securities.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(1)));
        // Evenly dissipate the capital risk among selected securities.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }
}

public class VolumeSMAUniverseSelectionModel : FundamentalUniverseSelectionModel
{
    // Create a concurrent dictionary to store the EMA data for universe selection.
    private ConcurrentDictionary<Symbol, SelectionData> _selectionDataBySymbol = new();
    private int _period;

    public VolumeSMAUniverseSelectionModel(int period)
        : base()
    {
        _period = period;
    }

    public override IEnumerable<Symbol> Select(QCAlgorithm algorithm, IEnumerable<Fundamental> fundamental)
    {
        return (from f in fundamental
            // Create/Update the volume SMA indicator of each stock.
            let avg = _selectionDataBySymbol.GetOrAdd(f.Symbol, sym => new SelectionData(algorithm, f.Symbol, _period))
            where avg.Update(f.EndTime, f.Volume)
            // Select the Equities with higher trading volume than their SMA, indicating higher capital flow.
            where avg.VolumeRatio > 1
            // Select the 10 Equities with the highest relative volume since they have the highest capacity
            // for scalp-trading or intra-day movement.
            orderby avg.VolumeRatio descending
            select f.Symbol).Take(10);
    }

    // Define a separate class to contain and calculate the SMA of each Equity.
    private class SelectionData
    {
        public readonly Symbol Symbol;
        public readonly SimpleMovingAverage VolumeSma;
        public decimal VolumeRatio;

        public SelectionData(QCAlgorithm algorithm, Symbol symbol, int period)
        {
            // Create an SMA of volume to track the popularity of the stock.
            Symbol = symbol;
            VolumeSma = new SimpleMovingAverage(period);

            // Warm up the volume SMA indicator.
            algorithm.WarmUpIndicator(symbol, VolumeSma, Resolution.Daily, (data) => (data as TradeBar).Volume);
        }

        public bool Update(DateTime time, decimal value)
        {
            // Update the SMA with today's data and calculate the relative volume position for filtering.
            var ready = VolumeSma.Update(time, value);
            var sma = VolumeSma.Current.Value;
            if (sma > 0)
            {
                VolumeRatio = value / VolumeSma.Current.Value;
                return ready;
            }
            return false;
        }
    }
}
```

```
from Selection.FundamentalUniverseSelectionModel import FundamentalUniverseSelectionModel

class FrameworkFundamentalUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2019, 1, 1)
        self.set_end_date(2019, 12, 1)
        self.set_cash(100000)

        # Add a universe with custom selection rules for filtering.
        self.add_universe_selection(VolumeSMAUniverseSelectionModel(10))

        # Sent insights on buying and holding the selected securities.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(1)))
        # Evenly dissipate the capital risk among selected securities.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

class VolumeSMAUniverseSelectionModel(FundamentalUniverseSelectionModel):
	# Create a dictionary to store the EMA data for universe selection.
    _selection_data_by_symbol = {}

    def __init__(self, period: int) -> None:
        self.period = period

    def select(self, algorithm: QCAlgorithm, fundamental: list[Fundamental]) -> list[Symbol]:
        # Create/Update the volume SMA indicator of each stock.
        for f in fundamental:
            if f.symbol not in self._selection_data_by_symbol:
                self._selection_data_by_symbol[f.symbol] = self.SelectionData(algorithm, f.symbol, self.period)
            self._selection_data_by_symbol[f.symbol].update(f.end_time, f.adjusted_price, f.dollar_volume)

        # Select the Equities with higher trading volume than their SMA, indicating higher capital flow.
        selected = [sd for sd in self._selection_data_by_symbol.values() if sd.volume_ratio > 1]

        # Select the 10 Equities with the highest relative volume since they have the highest capacity
        # for scalp-trading or intra-day movement.
        return [ x.symbol for x in sorted(selected, key=lambda x: x.volume_ratio)[-10:] ]

    # Create a separate class to contain the EMA information of each asset.
    class SelectionData(object):
        def __init__(self, algorithm: QCAlgorithm, symbol: Symbol, period: int) -> None:
            self.symbol = symbol
            self.volume_ratio = 0
            # Create an SMA of volume to track the popularity of the stock.
            self._sma = SimpleMovingAverage(period)

            # Warm up the volume SMA indicator.
            algorithm.warm_up_indicator(symbol, self._sma, Resolution.DAILY, lambda data: data.volume)

        def update(self, time: datetime, price: float, volume: float) -> None:
            # Update the SMA with today's data and calculate the relative volume position for filtering.
            ready = self._sma.update(time, volume)
            sma = self._sma.current.value
            if ready and sma > 0:
                self.volume_ratio = volume / sma
                return ready
            return False
```

#### Example 4: 10 "Fastest Moving" Stocks With a 50-Day EMA > 200 Day EMA

You can construct complex universe filters with the `SelectionData` helper class pattern.
To view a full example of this algorithm, see the [EmaCrossUniverseSelectionAlgorithm](https://github.com/QuantConnect/Lean/blob/master/Algorithm.CSharp/EmaCrossUniverseSelectionAlgorithm.cs) [EmaCrossUniverseSelectionAlgorithm](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Python/EmaCrossUniverseSelectionAlgorithm.py) in the LEAN GitHub repository or take the [related Boot Camp lesson](https://www.quantconnect.com/learning/task/153/).

Select Language: C#Python

```
using System.Collections.Concurrent;

public class FrameworkFrameworkUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2019, 1, 1);
        SetEndDate(2019, 12, 1);
        SetCash(100000);

        // Add a fundamental universe with custom selection rules for filtering.
        AddUniverseSelection(new EmaUniverseSelectionModel(10));

        // Sent insights on buying and holding the selected securities.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(1)));
        // Evenly dissipate the capital risk among selected securities.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }
}

public class EmaUniverseSelectionModel : FundamentalUniverseSelectionModel
{
    // tolerance to prevent bouncing
    const decimal Tolerance = 0.01m;
    // Create a concurrent dictionary to store the indicator data for universe selection.
    private ConcurrentDictionary<Symbol, SelectionData> _averages = new();
    private int _selectionNumber;

    public EmaUniverseSelectionModel(int selectionNumber = 10)
        : base()
    {
        _selectionNumber = selectionNumber;
    }

    public override IEnumerable<Symbol> Select(QCAlgorithm algorithm, IEnumerable<Fundamental> fundamental)
    {
        return (from f in fundamental
            // Grab the SelectionData instance for this symbol
            let avg = _averages.GetOrAdd(f.Symbol, sym => new SelectionData(algorithm, f.Symbol))
            // Update returns true when the indicators are ready, so don't accept until they are
            where avg.Update(f.EndTime, f.AdjustedPrice)
            //Only pick symbols that have their 50-day ema over their 100-day ema
            where avg.Fast > avg.Slow * (1 + Tolerance)
            // prefer symbols with a larger delta by percentage between the two averages
            orderby avg.ScaledDelta descending
            // We only need to return the symbol and return selectionNumber symbols
            select f.Symbol).Take(_selectionNumber);
    }

    // class used to improve the readability of the coarse selection function
    private class SelectionData
    {
        public readonly ExponentialMovingAverage Fast;
        public readonly ExponentialMovingAverage Slow;

        public SelectionData(QCAlgorithm algorithm, Symbol symbol)
        {
            Fast = new ExponentialMovingAverage(100);
            Slow = new ExponentialMovingAverage(300);

            // Warm up the indicators.
            algorithm.WarmUpIndicator(symbol, Fast, Resolution.Daily);
            algorithm.WarmUpIndicator(symbol, Slow, Resolution.Daily);
        }

        // computes an object score of how large the fast is than the slow
        public decimal ScaledDelta
        {
            get { return (Fast - Slow)/((Fast + Slow)/2m); }
        }

        // updates the EMA50 and EMA100 indicators, returning true when they're both ready
        public bool Update(DateTime time, decimal value)
        {
            return Fast.Update(time, value) && Slow.Update(time, value);
        }
    }
}
```

```
from Selection.FundamentalUniverseSelectionModel import FundamentalUniverseSelectionModel

class FrameworkFundamentalUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2019, 1, 1)
        self.set_end_date(2019, 12, 1)
        self.set_cash(100000)

        # Add a universe with custom selection rules for filtering.
        self.add_universe_selection(EmaUniverseSelectionModel(10))

        # Sent insights on buying and holding the selected securities.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(1)))
        # Evenly dissipate the capital risk among selected securities.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

class EmaUniverseSelectionModel(FundamentalUniverseSelectionModel):
	# Create a dictionary to store the EMA data for universe selection.
    averages = {}

    def __init__(self, selection_number: int) -> None:
        super().__init__()
        self.selection_number = selection_number

    def select(self, algorithm: QCAlgorithm, fundamental: list[Fundamental]) -> list[Symbol]:
        # We are going to use a dictionary to refer to the object that will keep the moving averages
        for f in fundamental:
            if f.symbol not in self.averages:
                self.averages[f.symbol] = self.SelectionData(algorithm, f.symbol)

            # Updates the SymbolData object with current EOD price
            avg = self.averages[f.symbol]
            avg.update(f.end_time, f.adjusted_price)

        # Filter the values of the dict: we only want up-trending securities
        values = list(filter(lambda x: x.is_uptrend, self.averages.values()))

        # Sorts the values of the dict: we want those with greater differences between the moving average
        values.sort(key=lambda x: x.scale, reverse=True)

        # we need to return only the symbol objects
        return [ x.symbol for x in values[:self.selection_number] ]

    # class used to improve the readability of the fundamental selection function
    class SelectionData(object):
        def __init__(self, algorithm: QCAlgorithm, symbol: Symbol) -> None:
            self.symbol = symbol
            self.tolerance = 1.01
            self.fast = ExponentialMovingAverage(100)
            self.slow = ExponentialMovingAverage(300)
            self.is_uptrend = False
            self.scale = 0

            # Warm up the indicators.
            algorithm.warm_up_indicator(symbol, self.fast, Resolution.DAILY)
            algorithm.warm_up_indicator(symbol, self.slow, Resolution.DAILY)

        def update(self, time: datetime, price: float) -> None:
            if self.fast.update(time, price) and self.slow.update(time, price):
                fast = self.fast.current.value
                slow = self.slow.current.value
                self.is_uptrend = fast > slow * self.tolerance

            if self.is_uptrend:
                # computes an object score of how much larger the fast is than the slow
                self.scale = (fast - slow) / ((fast + slow) / 2.0)
```

#### Other Examples

For more examples, see the following algorithms:

Demonstration Algorithms

[FundamentalUniverseSelectionAlgorithm.py Python](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Python/FundamentalUniverseSelectionAlgorithm.py) [EmaCrossUniverseSelectionFrameworkAlgorithm.py Python](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Python/EmaCrossUniverseSelectionFrameworkAlgorithm.py) [FundamentalUniverseSelectionAlgorithm.cs C#](https://github.com/QuantConnect/Lean/blob/master/Algorithm.CSharp/FundamentalUniverseSelectionRegressionAlgorithm.cs) [EmaCrossUniverseSelectionFrameworkAlgorithm.cs C#](https://github.com/QuantConnect/Lean/blob/master/Algorithm.CSharp/EmaCrossUniverseSelectionFrameworkAlgorithm.cs)

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/02%20Universe%20Selection/04%20Fundamental%20Universes)