# Universe Selection

## Universe Settings

### Introduction

Universe settings and security initializers enable you to configure some properties of the securities in a universe.

### Resolution

The `Resolution` `resolution` setting defines the [time period](https://www.quantconnect.com/docs/v2/writing-algorithms/key-concepts/time-modeling/periods) of the asset data. The `Resolution` enumeration has the following members:

To view which resolutions are available for the asset class of your universe, follow these steps:

1. Open the [Asset Classes](https://www.quantconnect.com/docs/v2/writing-algorithms/securities/asset-classes) documentation page.
2. Click an asset class.
3. Click Requesting Data.
4. On the Requesting Data page, in the table of contents, click Resolutions.

The default value is `Resolution.Minute` `Resolution.MINUTE`. To change the resolution, in the [Initialize](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization) method,
adjust the algorithm's `UniverseSettings` before you create the Universe Selection model.

Select Language: C#Python

```
// Set universe settings to subscribe to daily data and select SPY ETF constituents as the trading universe.
UniverseSettings.Resolution = Resolution.Daily;
AddUniverseSelection(new ETFConstituentsUniverseSelectionModel("SPY"));
```

```
# Use daily data and select SPY ETF constituents as the trading universe.
self.universe_settings.resolution = Resolution.DAILY
self.add_universe_selection(ETFConstituentsUniverseSelectionModel("SPY"))
```

### Leverage

The `Leverage` `leverage` setting is a `float` `decimal` that defines the maximum amount of leverage you can use for a single asset in a non-derivative universe. The default value is `Security.NullLeverage` `Security.NULL_LEVERAGE` (0). To change the leverage, in the [Initialize](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization) method, adjust the algorithm's `UniverseSettings` `universe_settings` before you
create the Universe Selection model.

Select Language: C#Python

```
// Set unviverse leverage to 2x to increase potential returns from each asset in the non-derivative universe to increase potential returns.
UniverseSettings.Leverage = 2.0m;
// Select universe based on EMA cross signals to identify and trade assets with trending behaviors.
AddUniverseSelection(new EmaCrossUniverseSelectionModel());
```

```
# Set unviverse leverage to 2x to increase potential returns from each asset in the non-derivative universe to increase potential returns.
self.universe_settings.leverage = 2.0
# Select universe based on EMA cross signals to identify and trade assets with trending behaviors.
self.add_universe_selection(EmaCrossUniverseSelectionModel())
```

### Fill Forward

The `FillForward` `fill_forward` setting is a `bool` that defines whether or not too fill forward data. The default value is `true` `True`. To disable fill forward in non-derivative universes, in the [Initialize](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization) method, adjust the algorithm's `UniverseSettings` `universe_settings` before you
create the Universe Selection model.

Select Language: C#Python

```
// Disable fill forward to ensure each day's data is used exclusively, preventing previous day's data from influencing decisions and select options for SPY with a daily update interval to track specific option contracts.
UniverseSettings.FillForward = false;
AddUniverseSelection(
    new OptionUniverseSelectionModel(
        TimeSpan.FromDays(1),
        _ => new [] { QuantConnect.Symbol.Create("SPY", SecurityType.Option, Market.USA) }
    )
);
```

```
# Disable fill forward to ensure each day's data is used exclusively, preventing previous day's data from influencing decisions and select options for SPY with a daily update interval to track specific option contracts.
from Selection.OptionUniverseSelectionModel import OptionUniverseSelectionModel

self.universe_settings.fill_forward = False
self.add_universe_selection(
    OptionUniverseSelectionModel(
        timedelta(1), lambda _: [Symbol.create("SPY", SecurityType.OPTION, Market.USA)]
    )
)
```

### Extended Market Hours

The `ExtendedMarketHours` `extended_market_hours` setting is a `bool` that defines the trading schedule. If it's `true` `True`, your algorithm receives price data for all trading hours. If it's `false` `False`, your algorithm receives price data only for regular trading hours. The default value is `false` `False`.

You only receive extended market hours data if you create the subscription with an intraday resolution. If you create the subscription with daily resolution, the daily bars only reflect the regular trading hours.

To view the trading schedule of an asset, open [Asset Classes](https://www.quantconnect.com/docs/v2/writing-algorithms/securities/asset-classes), click an asset class, then click Market Hours.

To enable extended market hours, in the [Initializeinitialize](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization) method, adjust the algorithm's `UniverseSettings` before you create the Universe Selection model.

Select Language: C#Python

```
// Enable extended market hours for universe and manually select equity symbols for SPY, QQQ, and IWM to ensure trading during pre-market and after-hours sessions.
UniverseSettings.ExtendedMarketHours = true;
var tickers = new[] {"SPY", "QQQ", "IWM"};
var symbols = tickers.Select(ticker => QuantConnect.Symbol.Create(ticker, SecurityType.Equity, Market.USA));
AddUniverseSelection(new ManualUniverseSelectionModel(symbols));
```

```
# Enable extended market hours for universe and manually select equity symbols for SPY, QQQ, and IWM to ensure trading during pre-market and after-hours sessions.
self.universe_settings.extended_market_hours = True
tickers = ["SPY", "QQQ", "IWM"]
symbols = [ Symbol.create(ticker, SecurityType.EQUITY, Market.USA) for ticker in tickers]
self.add_universe_selection(ManualUniverseSelectionModel(symbols))
```

### Minimum Time in Universe

The `MinimumTimeInUniverse` `minimum_time_in_universe` setting is a `timedelta` `TimeSpan` object that defines the minimum amount of time an asset must be in the universe before the universe can remove it. The default value is `TimeSpan.FromDays(1)` `timedelta(1)`. To change the minimum time, in the [Initialize](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization) method, adjust the algorithm's `UniverseSettings` `universe_settings` before you
create the Universe Selection model.

Select Language: C#Python

```
// Keep each security in the universe for a minimum of 7 days.
UniverseSettings.MinimumTimeInUniverse = TimeSpan.FromDays(7);
AddUniverseSelection(new ETFConstituentsUniverseSelectionModel("QQQ"));
```

```
# Keep each security in the universe for a minimum of 7 days.
self.universe_settings.minimum_time_in_universe = timedelta(7)
self.add_universe_selection(ETFConstituentsUniverseSelectionModel("QQQ"))
```

### Data Normalization Mode

The `DataNormalizationMode` `data_normalization_mode` setting is an enumeration that defines how historical data is adjusted. This setting is only applicable for US Equities and Futures.

In the case of US Equities, the data normalization mode affects how historical data is adjusted for [corporate actions](https://www.quantconnect.com/docs/v2/writing-algorithms/securities/asset-classes/us-equity/corporate-actions). To view all the available options, see [Data Normalization](https://www.quantconnect.com/docs/v2/writing-algorithms/securities/asset-classes/us-equity/requesting-data#11-Data-Normalization). To change the data normalization mode, in the [Initializeinitialize](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization) method, adjust the algorithm's `UniverseSettings` `universe_settings` before you create the Universe Selection model.

In the case of Futures, the data normalization mode affects how historical data of two contracts is stitched together to form the [continuous contract](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/futures#12-Continous-Contracts). To view all the available options, see [Data Normalization](https://www.quantconnect.com/docs/v2/writing-algorithms/securities/asset-classes/futures/requesting-data/individual-contracts#09-Data-Normalization).

The default value is `DataNormalizationMode.Adjusted` `DataNormalizationMode.ADJUSTED`. To change the data normalization mode, in the [Initialize](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization) method, adjust the algorithm's `UniverseSettings` before you create the Universe Selection model.

Select Language: C#Python

```
// Set universe data normalization to use raw prices to reflect actual traded values without adjustments and manually select equities MSTR, MSFT, and IBM.
UniverseSettings.DataNormalizationMode = DataNormalizationMode.Raw;

var tickers = new[] {"MSTR", "MSFT", "IBM"};
var symbols = tickers.Select(ticker => QuantConnect.Symbol.Create(ticker, SecurityType.Equity, Market.USA));
AddUniverseSelection(new ManualUniverseSelectionModel(symbols));
```

```
# Set universe data normalization to use raw prices to reflect actual traded values without adjustments and manually select equities MSTR, MSFT, and IBM.
self.universe_settings.data_normalization_mode = DataNormalizationMode.RAW

tickers = ["MSTR", "MSFT", "IBM"]
symbols = [ Symbol.create(ticker, SecurityType.EQUITY, Market.USA) for ticker in tickers]
self.add_universe_selection(ManualUniverseSelectionModel(symbols))
```

### Contract Depth Offset

The `ContractDepthOffset` `contract_depth_offset` setting is an `int` that defines which contract to use for the continuous Futures contract. 0 is the front month contract, 1 is the following back month contract, and 3 is the second back month contract. The default value is 0. To change the contract depth offset, in the [Initialize](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization) method,
adjust the algorithm's `UniverseSettings` `universe_settings` before you create the Universe Selection model.

Select Language: C#Python

```
// Select the following month contract dynamically to avoid low liquidity and high volatility, for S&P 500 E-mini futures with daily updates.
UniverseSettings.ContractDepthOffset = 1;
AddUniverseSelection(
    new FutureUniverseSelectionModel(
        TimeSpan.FromDays(1),
        _ => new List<Symbol> {{ QuantConnect.Symbol.Create(Futures.Indices.SP500EMini, SecurityType.Future, Market.CME) }}
    )
);
```

```
# Select the following month contract dynamically to avoid low liquidity and high volatility, for S&P 500 E-mini futures with daily updates.
from Selection.FutureUniverseSelectionModel import FutureUniverseSelectionModel

self.universe_settings.contract_depth_offset = 1
self.add_universe_selection(
    FutureUniverseSelectionModel(
        timedelta(1),
        lambda _: [Symbol.create(Futures.Indices.SP500E_MINI, SecurityType.FUTURE, Market.CME)]
    )
)
```

### Asynchronous Selection

The `Asynchronous` `asynchronous` setting is a `bool` that defines whether or not LEAN can run universe selection asynchronously, utilizing concurrent execution to increase the speed of your algorithm. The default value for this setting is `false` `False`. If you enable this setting, abide by the following rules:

- Don't make any [history requests](https://www.quantconnect.com/docs/v2/writing-algorithms/historical-data/history-requests) in your [filter function](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/key-concepts#05-Selection-Functions). History requests can only provide data up to the [algorithm time](https://www.quantconnect.com/docs/v2/writing-algorithms/key-concepts/time-modeling/timeslices#02-Time-Frontier), but if your filter function runs asynchronously, the algorithm time may not be what you expect.
- Don't use the portfolio, security, or orders state. The `Portfolio` `portfolio`, `Securities` `securities`, and `Transactions` `transactions` objects are also functions of the algorithm time.
- If your filter function updates a class variable, don't update the class variable anywhere else in your algorithm.

To enable asynchronous universe selection, in the [Initialize](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization) method, adjust the algorithm's `UniverseSettings` `universe_settings` before you
create the Universe Selection model.

Select Language: C#Python

```
// Run universe selection asynchronously to speed up your algorithm.
UniverseSettings.Asynchronous = true;
AddUniverseSelection(new EmaCrossUniverseSelectionModel());
```

```
# Run universe selection asynchronously to speed up your algorithm.
self.universe_settings.asynchronous = True
self.add_universe_selection(EmaCrossUniverseSelectionModel())
```

### Schedule

The `Schedule` setting defines the selection schedule of the universe.
Most universes run on a daily schedule.
To change the selection schedule, call the `UniverseSettings.Schedule.On` `universe_settings.schedule.on` method with an `IDateRule` object before you create the Universe Selection model.

Select Language: C#Python

```
// Trigger selections at the begining of each month.
UniverseSettings.Schedule.On(DateRules.MonthStart());
AddUniverseSelection(new ETFConstituentsUniverseSelectionModel("QQQ"));
```

```
# Trigger selections at the begining of each month.
self.universe_settings.schedule.on(self.date_rules.month_start())
self.add_universe_selection(ETFConstituentsUniverseSelectionModel("QQQ"))
```

The following table describes the supported `DateRules`:

| Member | Description |
| --- | --- |
| `self.date_rules.set_default_time_zone(time_zone: DateTimeZone)<br>            ` `DateRules.SetDefaultTimeZone(DateTimeZone timeZone);` | Sets the time zone for the `DateRules` object used in all methods in this table. The default time zone is the [algorithm time zone](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization#12-Set-Time-Zone). |
| `self.date_rules.on(year: int, month: int, day: int)<br>            ` `DateRules.On(int year, int month, int day)` | Trigger an event on a specific date. |
| `self.date_rules.on(dates: list[datetime])` `DateRules.On(params DateTime[] dates)` | Trigger an event on specific dates. |
| `self.date_rules.every_day()` `DateRules.EveryDay()` | Trigger an event every day. |
| `self.date_rules.every_day(symbol: Symbol, extended_market_hours: bool = False)` `DateRules.EveryDay(Symbol symbol, bool extendedMarketHours = false)` | Trigger an event every day a specific symbol is trading. |
| `self.date_rules.every(days: list[DayOfWeek])` `DateRules.Every(params DayOfWeek[] days)` | Trigger an event on specific days throughout the week. To view the `DayOfWeek` enum members, see [DayOfWeek Enum](https://docs.microsoft.com/en-us/dotnet/api/system.dayofweek) in the .NET documentation. |
| `self.date_rules.month_start(days_offset: int = 0)` `DateRules.MonthStart(int daysOffset = 0)` | Trigger an event on the first day of each month plus an offset. |
| `self.date_rules.month_start(symbol: Symbol, daysOffset: int = 0)` `DateRules.MonthStart(Symbol symbol, int daysOffset = 0)` | Trigger an event on the first tradable date of each month for a specific symbol plus an offset. |
| `self.date_rules.month_end(days_offset: int = 0)` `DateRules.MonthEnd(int daysOffset = 0)` | Trigger an event on the last day of each month minus an offset. |
| `self.date_rules.month_end(symbol: Symbol, daysOffset: int = 0)` `DateRules.MonthEnd(Symbol symbol, int daysOffset = 0)` | Trigger an event on the last tradable date of each month for a specific symbol minus an offset. |
| `self.date_rules.week_start(days_offset: int = 0)` `DateRules.WeekStart(int daysOffset = 0)` | Trigger an event on the first day of each week plus an offset. |
| `self.date_rules.week_start(symbol: Symbol, days_offset: int = 0)` `DateRules.WeekStart(Symbol symbol, int daysOffset = 0)` | Trigger an event on the first tradable date of each week for a specific symbol plus an offset. |
| `self.date_rules.week_end(days_offset: int = 0)` `DateRules.WeekEnd(int daysOffset = 0)` | Trigger an event on the last day of each week minus an offset. |
| `self.date_rules.week_end(symbol: Symbol, days_offset: int = 0)` `DateRules.WeekEnd(Symbol symbol, int daysOffset = 0)` | Trigger an event on the last tradable date of each week for a specific symbol minus an offset. |
| `self.date_rules.year_start(days_offset: int = 0)` `DateRules.YearStart(int daysOffset = 0)` | Trigger an event on the first day of each year plus an offset. |
| `self.date_rules.year_start(symbol: Symbol, days_offset: int = 0)` `DateRules.YearStart(Symbol symbol, int daysOffset = 0)` | Trigger an event on the first tradable date of each year for a specific symbol plus an offset. |
| `self.date_rules.year_end(days_offset: int = 0)` `DateRules.YearEnd(int daysOffset = 0)` | Trigger an event on the last day of each year minus an offset. |
| `self.date_rules.year_end(symbol: Symbol, days_offset: int = 0)` `DateRules.YearEnd(Symbol symbol, int daysOffset = 0)` | Trigger an event on the last tradable date of each year for a specific symbol minus an offset. |
| `self.date_rules.today` `DateRules.Today` | Trigger an event once today. |
| `self.date_rules.tomorrow` `DateRules.Tomorrow` | Trigger an event once tomorrow. |

To define custom date rules, create a `FuncDateRule` object.
The `FuncDateRule` constructor expects a `name` argument of type `string` `str` and a `getDatesFunction` `get_dates_function` argument of type `Func<DateTime, DateTime, IEnumerable<DateTime>>` `Callable[[datetime, datetime], list[datetime]]`.
The `getDatesFunction` `get_dates_function` function receives the start and end dates of the algorithm and returns a list of dates for the date rule.
In live trading, the end date is 12/31/2025.
The following example demonstrates how to define a date rule that represents the 10th day of each month:

Select Language: C#Python

```
// Create a date rule that specifies the 10th day of each month.
var dateRule = new FuncDateRule(
    name: "10th_day_of_the_month",
    getDatesFunction: (start, end) => Enumerable.Range(start.Year, end.Year - start.Year + 1)
        .SelectMany(year => Enumerable.Range(1, 12).Select(month => new DateTime(year, month, 10)))
);
```

```
# Create a date rule that specifies the 10th day of each month.
date_rule = FuncDateRule(
    name="10th_day_of_the_month",
    get_dates_function=lambda start, end: [\
        datetime(year, month, 10)\
        for year in range(start.year, end.year) for month in range(1,12)\
    ]
)
```

Some date rules require a Symbol.
However, in some cases, you may not have a one.
For example, if you only add an [Equity fundamental universe](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/equity/fundamental-universes) to your [initializeInitialize](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization) method, you won't have an asset Symbol to use for the date rule.
To create a Symbol without adding a security to your algorithm, call the `Symbol.create` `QuantConnect.Symbol.Create` method.

Select Language: C#Python

```
var symbol = QuantConnect.Symbol.Create("SPY", SecurityType.Equity, Market.USA);
```

```
symbol = Symbol.create('SPY', SecurityType.EQUITY, Market.USA)
```

In live trading, scheduled universes run at roughly 8 AM Eastern Time to ensure there is enough time for the data to process.

### Configure Securities

Instead of configuring global universe settings, you can individually configure the settings of each security in the universe with a security initializer. Security initializers let you apply any [security-level reality model](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/key-concepts#02-Security-Level-Models) or special data requests on a per-security basis. To set the security initializer, in the `Initialize` `initialize` method, call the `SetSecurityInitializer` `set_security_initializer` method and then define the security initializer.

Select Language: C#Python

```
// A custom security initializer can override default models such as
// setting new fee and fill models for the security.
SetSecurityInitializer(CustomSecurityInitializer);

private void CustomSecurityInitializer(Security security)
{
    security.SetFeeModel(new ConstantFeeModel(0, "USD"));
}
```

```
# A custom security initializer can override default models such as
# setting new fee and fill models for the security.
self.set_security_initializer(self._custom_security_initializer)

def _custom_security_initializer(self, security: Security) -> None:
    security.set_fee_model(ConstantFeeModel(0, "USD"))
```

For simple requests, you can use the functional implementation of the security initializer. This style lets you configure the security object with one line of code.

Select Language: C#Python

```
// Disable the trading fees for each security by passing a functional
// implementation for the SetSecurityInitializer argument.
SetSecurityInitializer(security => security.SetFeeModel(new ConstantFeeModel(0, "USD")));
```

```
# Disable the trading fees for each security by using lambda function
# for the set_security_initializer argument.
self.set_security_initializer(lambda security: security.set_fee_model(ConstantFeeModel(0, "USD")))
```

In some cases, you may want to trade a security in the same time loop that you create the security subscription. To avoid errors, use a security initializer to set the market price of each security to the last known price. The `GetLastKnownPrices` `get_last_known_prices` method seeds the security price by gathering the security data over the last 3 days. If there is no data during this period, the security price remains at 0. When you live trade Options without the QuantConnect data provider, this method may take longer than 10 minutes to gather the historical data, causing a timeout.

Select Language: C#Python

```
// Gather the last 3 days of security prices by using GetLastKnowPrice as the seed in Initialize.
var seeder = new FuncSecuritySeeder(GetLastKnownPrices);
SetSecurityInitializer(security => seeder.SeedSecurity(security));
```

```
# Gather the last 3 days of security prices by using get_last_known_prices as the seed in initialize.
seeder = FuncSecuritySeeder(self.get_last_known_prices)
self.set_security_initializer(lambda security: seeder.seed_security(security))
```

If you call the `SetSecurityInitializer` `set_security_initializer` method, it overwrites the default security initializer. The default security initializer uses the [security-level reality models](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/key-concepts#02-Security-Level-Models) of the brokerage model to set the following reality models of each security:

- [Fill](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/trade-fills/key-concepts)
- [Slippage](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/slippage/key-concepts)
- [Fee](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/transaction-fees/key-concepts)
- [Buying Power](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/buying-power)
- [Settlement](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/settlement/key-concepts)
- [Short availability](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/short-availability/key-concepts)
- [Margin Interest Rate](https://www.quantconnect.com/docs/v2/writing-algorithms/reality-modeling/margin-interest-rate/key-concepts)

The default security initializer also sets the leverage of each security and intializes each security with a seeder function. To extend upon the default security initializer instead of overwriting it, create a custom `BrokerageModelSecurityInitializer`.

Select Language: C#Python

```
public class BrokerageModelExampleAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        // In the Initialize method, set the security initializer to set models of assets.
        // To seed the security with the last known prices, set the security seeder to FuncSecuritySeeder with the GetLastKnownPrices method.
        var securitySeeder = new FuncSecuritySeeder(GetLastKnownPrices);

        // In some case, for example, adding Options and Futures using Universes, we don't need to seed prices.
        // To avoid the overhead of and potential timeouts for calling the GetLastKnownPrices method, set the security seeder to SecuritySeeder.Null.
        securitySeeder = SecuritySeeder.Null;

        SetSecurityInitializer(new MySecurityInitializer(BrokerageModel, securitySeeder));
    }
}

public class MySecurityInitializer : BrokerageModelSecurityInitializer
{
    public MySecurityInitializer(IBrokerageModel brokerageModel, ISecuritySeeder securitySeeder)
        : base(brokerageModel, securitySeeder) {}
    public override void Initialize(Security security)
    {
        // First, call the superclass definition.
        // This method sets the reality models of each security using the default reality models of the brokerage model.
        base.Initialize(security);

        // Next, overwrite some of the reality models
        security.SetFeeModel(new ConstantFeeModel(0, "USD"));    }
}
```

```
class BrokerageModelExampleAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        # In the Initialize method, set the security initializer to set models of assets.
        # To seed the security with the last known prices, set the security seeder to FuncSecuritySeeder with the get_last_known_prices method.
        security_seeder = FuncSecuritySeeder(self.get_last_known_prices)

        # In some case, for example, adding Options and Futures using Universes, we don't need to seed prices.
        # To avoid the overhead of and potential timeouts for calling the get_last_known_prices method, set the security seeder to SecuritySeeder.NULL.
        security_seeder = SecuritySeeder.NULL

        self.set_security_initializer(MySecurityInitializer(self.brokerage_model, security_seeder))

# Outside of the algorithm class
class MySecurityInitializer(BrokerageModelSecurityInitializer):

    def __init__(self, brokerage_model: IBrokerageModel, security_seeder: ISecuritySeeder) -> None:
        super().__init__(brokerage_model, security_seeder)
    def initialize(self, security: Security) -> None:
        # First, call the superclass definition.
        # This method sets the reality models of each security using the default reality models of the brokerage model.
        super().initialize(security)

        # Next, overwrite some of the reality models
        security.set_fee_model(ConstantFeeModel(0, "USD"))
```

To set a seeder function without overwriting the reality models of the brokerage, use the standard `BrokerageModelSecurityInitializer`.

Select Language: C#Python

```
var seeder = new FuncSecuritySeeder(GetLastKnownPrices);
SetSecurityInitializer(new BrokerageModelSecurityInitializer(BrokerageModel, seeder));
```

```
seeder = FuncSecuritySeeder(self.get_last_known_prices)
self.set_security_initializer(BrokerageModelSecurityInitializer(self.brokerage_model, seeder))
```

### Examples

The following examples demonstrate some common practices for universe settings.

#### Example 1: Weekly-Updating Liquid Universe

The following algorithm demonstrates daily EMA cross, trading on the most liquid stocks. The universe is set to be updated weekly. Various universe settings have been set to simulate the brokerage environment best and for trading needs.

Select Language: C#Python

```
public class UniverseSettingsAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2021, 1, 1);
        SetEndDate(2021, 2, 1);

        // To avoid over-updating the universe, which causes insufficient time to earn the trend, update the universe weekly.
        UniverseSettings.Schedule.On(DateRules.WeekStart());
        UniverseSettings.MinimumTimeInUniverse = TimeSpan.FromDays(7);
        // Rebalancing frequency is daily, so we only need to subscribe to daily resolution.
        UniverseSettings.Resolution = Resolution.Daily;
        // To best simulate IB's margin requirement set the leverage to 50%.
        UniverseSettings.Leverage = 2;
        // Since we trade by market order, we cannot use extended market hours data to fill.
        UniverseSettings.ExtendedMarketHours = false;
        // We want to trade the EMA with raw price but not altered by splits.
        UniverseSettings.DataNormalizationMode = DataNormalizationMode.SplitAdjusted;

                // Only trade on the top 10 most traded stocks since they have the most popularity to drive trends.
        AddUniverse(Universe.DollarVolume.Top(10));
            }

    public override void OnData(Slice slice)
    {
        foreach (var (symbol, bar) in slice.Bars)
        {
            // Trade the trend by EMA crossing.
            var ema = (Securities[symbol] as dynamic).ema;
            if (bar.Close > ema)
            {
                SetHoldings(symbol, 0.05m);
            }
            else
            {
                SetHoldings(symbol, -0.05m);
            }
        }
    }

    public override void OnSecuritiesChanged(SecurityChanges changes)
    {
        foreach (var removed in changes.RemovedSecurities)
        {
            // Liquidate the ones leaving the universe since we will not trade their EMA anymore.
            Liquidate(removed.Symbol);
            // Deregister the EMA indicator to free resources
            DeregisterIndicator((removed as dynamic).ema as ExponentialMovingAverage);
        }

        foreach (var added in changes.AddedSecurities)
        {
            var security = added as dynamic;
            // Create EMA indicator for trend trading signals.
            security.ema = EMA(added.Symbol, 50, Resolution.Daily);
            // Warm up the EMA indicator to ensure its readiness for immediate use.
            WarmUpIndicator(added.Symbol, (ExponentialMovingAverage)security.ema, Resolution.Daily);
        }
    }
}
```

```
class UniverseSettingsAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2021, 1, 1)
        self.set_end_date(2021, 2, 1)

        # To avoid over-updating the universe, which causes insufficient time to earn the trend, update the universe weekly.
        self.universe_settings.schedule.on(self.date_rules.week_start())
        self.universe_settings.minimum_time_in_universe = timedelta(7)
        # Rebalancing frequency is daily, so we only need to subscribe to daily resolution.
        self.universe_settings.resolution = Resolution.DAILY
        # To best simulate IB's margin requirement, set the leverage to 50%.
        self.universe_settings.leverage = 2
        # Since we trade by market order, we cannot use extended market hours data to fill.
        self.universe_settings.extended_market_hours = False
        # We want to trade the EMA with raw price but not altered by splits.
        self.universe_settings.data_normalization_mode = DataNormalizationMode.SPLIT_ADJUSTED

                # Only trade on the top 10 most traded stocks since they have the most popularity to drive trends.
        self.add_universe(self.universe.dollar_volume.top(10))

    def on_data(self, slice: Slice) -> None:
        for symbol, bar in slice.bars.items():
            # Trade the trend by EMA crossing.
            ema = self.securities[symbol].ema.current.value
            if bar.close > ema:
                self.set_holdings(symbol, 0.05)
            else:
                self.set_holdings(symbol, -0.05)

    def on_securities_changed(self, changes: SecurityChanges) -> None:
        for removed in changes.removed_securities:
            # Liquidate the ones leaving the universe since we will not trade their EMA anymore.
            self.liquidate(removed.symbol)
            # Deregister the EMA indicator to free resources
            self.deregister_indicator(removed.ema)

        for added in changes.added_securities:
            # Create EMA indicator for trend trading signals.
            added.ema = self.ema(added.symbol, 50, Resolution.DAILY)
            # Warm up the EMA indicator to ensure its readiness for immediate use.
            self.warm_up_indicator(added.symbol, added.ema, Resolution.DAILY)
```

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/02%20Universe%20Selection/02%20Universe%20Settings)