# Universe Selection

## Manual Universes

### Introduction

The `ManualUniverseSelectionModel` selects a static, fixed set of assets. It is similar to adding securities with the traditional `AddSecurity` `add_security` API methods. If your algorithm has a static universe, you can use [automatic indicators](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/automatic-indicators) instead of [manual indicators](https://www.quantconnect.com/docs/v2/writing-algorithms/indicators/manual-indicators) in your algorithm.

Manual universes can be prone to [look-ahead bias](https://www.quantconnect.com/docs/v2/writing-algorithms/key-concepts/glossary#17-look-ahead-bias). For example, if you select a set of securities that have performed well during the backtest period, you are incorporating information from the future into the backtest and the algorithm may underperform in live mode.

### Add Manual Universe Selection

To add a `ManualUniverseSelectionModel` to your algorithm, in the `Initialize` `initialize` method, call the `AddUniverseSelection` method. The `ManualUniverseSelectionModel` constructor expects a list of `Symbol` objects that represent the universe constituents.

Select Language: C#Python

```
// Use ManualUniverseSelectionModel method to select a static set of equities, similar to traditional add_security methods, for a fixed trading universe.
var tickers = new[] {"SPY", "QQQ", "IWM"};
var symbols = tickers.Select(ticker => QuantConnect.Symbol.Create(ticker, SecurityType.Equity, Market.USA));
AddUniverseSelection(new ManualUniverseSelectionModel(symbols));
```

```
# Use ManualUniverseSelectionModel method to select a static set of equities, similar to traditional add_security methods, for a fixed trading universe.
tickers = ["SPY", "QQQ", "IWM"]
symbols = [ Symbol.create(ticker, SecurityType.EQUITY, Market.USA) for ticker in tickers]
self.add_universe_selection(ManualUniverseSelectionModel(symbols))
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `symbols` | `IEnumerable<Symbol>` `List[Symbol]` | Universe constituents |  |
| `universeSettings` `universe_settings` | `UniverseSettings` | The [universe settings](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/universe-settings). If you don't provide an argument, the model uses the `algorithm.UniverseSettings` `algorithm.universe_settings` by default. | `None` |

To move the universe tickers and `Symbol` objects outside of the algorithm class, create a universe selection model that inherits the `ManualUniverseSelectionModel` class.

Select Language: C#Python

```
// Create a custom universe selection model to move universe tickers and Symbol objects outside of the algorithm class for improved modularity, reusability, and maintainability.
AddUniverseSelection(new IndexUniverseSelectionModel());

// Outside of the algorithm class
class IndexUniverseSelectionModel : ManualUniverseSelectionModel
{
    public IndexUniverseSelectionModel()
        : base(SelectSymbols()) {}

    public static IEnumerable<Symbol> SelectSymbols()
    {
        var tickers = new[] {"SPY", "QQQ", "IWM"};
        return tickers.Select(ticker => QuantConnect.Symbol.Create(ticker, SecurityType.Equity, Market.USA));
    }
}
```

```
# Create a custom universe selection model to move universe tickers and Symbol objects outside of the algorithm class for improved modularity, reusability, and maintainability.
self.add_universe_selection(IndexUniverseSelectionModel())

# Outside of the algorithm class
class IndexUniverseSelectionModel(ManualUniverseSelectionModel):
    def __init__(self):
        tickers = ["SPY", "QQQ", "IWM"]
        symbols = [Symbol.create(ticker, SecurityType.EQUITY, Market.USA) for ticker in tickers]
        super().__init__(symbols)
```

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm/Selection/ManualUniverseSelectionModel.cs)[LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm/Selection/ManualUniverseSelectionModel.py).

### Examples

The following examples demonstrate some common practices for implementing a manual universe selection model.

#### Example 1: Crypto List Selection Model

The following algorithm selects a list of preset cryptos, given we have information that they will perform better. So, we can use `ManualUniverseSelectionModel` to select this list and buy the listed cryptos.

Select Language: C#Python

```
public class FrameworkManualUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2019, 1, 1);
        SetEndDate(2024, 12, 1);
        SetCash(100000);

        // Add a universe selection model to select the most liquid cryptos manually.
        var symbols = new [] {
            QuantConnect.Symbol.Create("BTCUSD", SecurityType.Crypto, Market.Coinbase),
            QuantConnect.Symbol.Create("ETHUSD", SecurityType.Crypto, Market.Coinbase)
        };
        AddUniverseSelection(new ManualUniverseSelectionModel(symbols));

        // Sent insights on buying and holding the most liquid cryptos for one week.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(7)));
        // Evenly dissipate the capital risk among selected cryptos.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }
}
```

```
class FrameworkManualUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2019, 1, 1)
        self.set_end_date(2024, 12, 1)
        self.set_cash(100000)

        # Add a universe selection model to select the most liquid cryptos manually.
        symbols = [\
            Symbol.create("BTCUSD", SecurityType.CRYPTO, Market.COINBASE),\
            Symbol.create("ETHUSD", SecurityType.CRYPTO, Market.COINBASE)\
        ]
        self.add_universe_selection(ManualUniverseSelectionModel(symbols))

        # Sent insights on buying and holding the most liquid cryptos for one week.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(7)))
        # Evenly dissipate the capital risk among selected cryptos.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())
```

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/02%20Universe%20Selection/03%20Manual%20Universes)