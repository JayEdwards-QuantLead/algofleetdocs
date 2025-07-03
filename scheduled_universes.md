# Universe Selection

## Scheduled Universes

### Introduction

A Scheduled Universe Selection model selects assets at fixed, regular intervals. In live trading, you can use this type of model to fetch tickers from Dropbox or to perform periodic historical analysis and select some assets.

### Scheduled Universe Selection

The `ScheduledUniverseSelectionModel` selects assets on the schedule you provide. To use this model, provide a `DateRule`, `TimeRule`, and a selector function. The `DateRule` and `TimeRule` define the selection schedule. The selector function receives a `datetime` `DateTime` object and returns a list of `Symbol` objects. The `Symbol` objects you return from the selector function are the constituents of the universe.

Select Language: C#Python

```
// Enable asynchronous universe selection to speed up your algorithm.
UniverseSettings.Asynchronous = true;
// Add a universe that selects assets at the beginning of each month.
AddUniverseSelection(
    new ScheduledUniverseSelectionModel(
        DateRules.MonthStart(),
        TimeRules.Midnight,
        // Select SPY for October. Otherwise, select QQQ.
        dt => new List<Symbol> {
            QuantConnect.Symbol.Create(dt.Month == 10 ? "SPY" : "QQQ", SecurityType.Equity, Market.USA)
        }
    )
);
```

```
# Enable asynchronous universe selection to speed up your algorithm.
self.universe_settings.asynchronous = True
# Add a universe that selects assets at the beginning of each month.
self.add_universe_selection(
    ScheduledUniverseSelectionModel(
        self.date_rules.month_start(),
        self.time_rules.midnight,
        # Select SPY for October. Otherwise, select QQQ.
        lambda dt: [Symbol.create("SPY" if dt.month == 10 else "QQQ", SecurityType.EQUITY, Market.USA)]
    )
)
```

The following table describes the arguments the model accepts:

| Argument | Data Type | Description | Default Value |
| --- | --- | --- | --- |
| `dateRule` `date_rule` | `IDateRule` | Date rule defines what days the universe selection function will be invoked |  |
| `timeRule` `time_rule` | `ITimeRule` | Time rule defines what times on each day selected by date rule the universe selection function will be invoked |  |
| `selector` | `Func<DateTime, IEnumerable<Symbol>>` `Callable[[datetime], list[Symbol]]` | Selector function accepting the date time firing time and returning the universe selected symbols |  |
| `settings` | `UniverseSettings` | The [universe settings](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/universe-settings). If you don't provide an argument, the model uses the `algorithm.UniverseSettings` `algorithm.universe_settings` by default. | `None` `null` |

The model assumes the `timeRule` `time_rule` argument is in Coordinated Universal Time (UTC). To use a different timezone, pass a `timeZone` argument of type `DateTimeZone` before the `dateRule` `date_rule` argument.

To return the current universe constituents from the selector function, return `Universe.UnchangedUNCHANGED`.

Select Language: C#Python

```
// Add a universe that selects assets on Monday, Tuesday, and Thursday at 00:00, 06:00, 12:00, and 18:00 UTC.
AddUniverseSelection(new ScheduledUniverseSelectionModel(
    DateRules.Every(DayOfWeek.Monday, DayOfWeek.Tuesday, DayOfWeek.Thursday),
    TimeRules.Every(TimeSpan.FromHours(6)),
    SelectSymbols
));

// Define the selection function.
private IEnumerable<Symbol> SelectSymbols(DateTime dateTime)
{
    var tickers = new[] {"SPY", "AAPL", "IBM"};
    return tickers.Select(ticker =>
        QuantConnect.Symbol.Create(ticker, SecurityType.Equity, Market.USA));
}
```

```
# Add a universe that selects assets on Monday, Tuesday, and Thursday at 00:00, 06:00, 12:00, and 18:00 UTC.
self.add_universe_selection(ScheduledUniverseSelectionModel(
    self.date_rules.every(DayOfWeek.MONDAY, DayOfWeek.TUESDAY, DayOfWeek.THURSDAY),
    self.time_rules.every(timedelta(hours = 6)),
    self.select_symbols # selection function in algorithm.
))

# Define the selection function.
def select_symbols(self, date_time: datetime) -> list[Symbol]:
    tickers = ['SPY', 'AAPL', 'IBM']
    return [Symbol.create(ticker, SecurityType.EQUITY, Market.USA)\
        for ticker in tickers]
```

To view the implementation of this model, see the [LEAN GitHub repository](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Framework/Selection/ScheduledUniverseSelectionModel.cs).

### Date Rules

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

### Time Rules

The following table describes the supported `TimeRules`:

| Member | Description |
| --- | --- |
| `self.time_rules.set_default_time_zone(time_zone: DateTimeZone)<br>            ` `TimeRules.SetDefaultTimeZone(DateTimeZone timeZone)` | Sets the time zone for the `TimeRules` object used in all methods in this table, except when a different time zone is given. The default time zone is the [algorithm time zone](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization#12-Set-Time-Zone). |
| `self.time_rules.before_market_open(symbol: Symbol, minutes_before_open: float = 0, extended_market_open: bool = False)` `TimeRules.BeforeMarketOpen(Symbol symbol, double minutesBeforeOpen = 0, bool extendedMarketOpen = false)` | Trigger an event a few minutes before market open for a specific symbol (default is 0). This rule doesn't work for Crypto securities or custom data. |
| `self.time_rules.after_market_open(symbol: Symbol, minutes_after_open: float = 0, extended_market_open: bool = False)` `TimeRules.AfterMarketOpen(Symbol symbol, double minutesAfterOpen = 0, bool extendedMarketOpen = false)` | Trigger an event a few minutes after market open for a specific symbol (default is 0). This rule doesn't work for Crypto securities or custom data. |
| `self.time_rules.before_market_close(symbol: Symbol, minutes_before_close: float = 0, extended_market_open: bool = False)` `TimeRules.BeforeMarketClose(Symbol symbol, double minutesBeforeClose = 0, bool extendedMarketOpen = false)` | Trigger an event a few minutes before market close for a specific symbol (default is 0). This rule doesn't work for Crypto securities or custom data. |
| `self.time_rules.after_market_close(symbol: Symbol, minutes_after_close: float = 0, extended_market_open: bool = False)` `TimeRules.AfterMarketClose(Symbol symbol, double minutesAfterClose = 0, bool extendedMarketOpen = false)` | Trigger an event a few minutes after market close for a specific symbol (default is 0). This rule doesn't work for Crypto securities or custom data. |
| `self.time_rules.every(interval: timedelta)` `TimeRules.Every(TimeSpan interval)` | Trigger an event every period interval starting at midnight. This time rule triggers every period, regardless of whether or not the market is open. |
| `self.time_rules.now` `TimeRules.Now` | Trigger an event at the current time of day. |
| `self.time_rules.midnight` `TimeRules.Midnight` | Trigger an event at midnight. |
| `self.time_rules.noon` `TimeRules.Noon` | Trigger an event at noon. |
| `self.time_rules.at(hour: int, minute: int, second: int = 0)` `TimeRules.At(int hour, int minute, int second = 0)` | Trigger an event at a specific time of day (e.g. 13:10). |
| `self.time_rules.at(hour: int, minute: int, second: int, time_zone: DateTimeZone)` `TimeRules.At(int hour, int minute, int second, DateTimeZone timeZone)` | Trigger an event at a specific time of day in the given time zone (e.g. 13:10 UTC). |

To define custom time rules, create a `FuncTimeRule` object.
The `FuncTimeRule` constructor expects a `name` argument of type `string` `str` and a `createUtcEventTimesFunction` `create_utc_event_times_function` argument of type `Func<IEnumerable<DateTime>, IEnumerable<DateTime>>` `Callable[[list[datetime]], list[datetime]]`.
The function receives the list of dates from the date rule and then returns a list of `DateTime` `datetime` that define the time rule.

Select Language: C#Python

```
var timeRule = new FuncTimeRule(
    name: "CustomTimeRule",
    createUtcEventTimesFunction: dates => dates.Select(d => d.AddHours(10)));
```

```
time_rule = FuncTimeRule(
    name="CustomTimeRule",
    create_utc_event_times_function=lambda dates: [d + timedelta(hours=10) for d in dates]
)
```

Some time rules require a Symbol.
However, in some cases, you may not have a one.
For example, if you only add an [Equity fundamental universe](https://www.quantconnect.com/docs/v2/writing-algorithms/universes/equity/fundamental-universes) to your [initializeInitialize](https://www.quantconnect.com/docs/v2/writing-algorithms/initialization) method, you won't have an asset Symbol to use for the time rule.
To create a Symbol without adding a security to your algorithm, call the `Symbol.create` `QuantConnect.Symbol.Create` method.

Select Language: C#Python

```
var symbol = QuantConnect.Symbol.Create("SPY", SecurityType.Equity, Market.USA);
```

```
symbol = Symbol.create('SPY', SecurityType.EQUITY, Market.USA)
```

### Examples

The following examples demonstrate some common practices for implementing a dateless scheduled universe selection model.

#### Example 1: From External Source

The following example selects its universe every week starting before the market opens from a source URL of Dropbox. It reloads on every selection so that you can hook up to the continuously updating source file.

Select Language: C#Python

```
public class FrameworkScheduledUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2023, 6, 1);
        SetEndDate(2023, 8, 1);
        SetCash(10000000);

        // Add a universe of that read from a Dropbox source to select the stocks 30 minutes before the week's first trading day.
        var spy = QuantConnect.Symbol.Create("SPY", SecurityType.Equity, Market.USA);
        AddUniverseSelection(new ScheduledUniverseSelectionModel(
            DateRules.WeekStart(spy),
            TimeRules.BeforeMarketOpen(spy, 10),
            SelectSymbols)
        );
        // Add Alpha model to trade based on the selections; the signals last for a week until the next selection.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(7)));
        // Equally invest in insights to dissipate the capital risk evenly.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }

    private string DownloadUniverseFile(DateTime dt)
    {
        // Download the universe CSV file. Dropbox links require the "dl=1" URL parameter.
        var file = Download(
            "https://www.dropbox.com/scl/fi/fbrxitk4ec3w91nse1raa/df.csv?rlkey=7r042rukzkthp7y1srloyhkov&st=5r4sdfwd&dl=1"
        );
        // Convert the CSV file data into a dictionary where the key is the date and
        // the value is a comma-separated string of stock tickers.
        foreach (var line in file.Split('\n').Skip(1))
        {
            // Skip empty lines.
            if (line.IsNullOrEmpty())
            {
                continue;
            }
            var items = line.Split("\"");
            var date = Parse.DateTimeExact(items[0].Split(",")[0], "yyyy-MM-dd").Date;
            // Return the universe if the date matches.
            if (date == dt.Date)
            {
                return items[1];
            }
        }
        return string.Empty;
    }

    private IEnumerable<Symbol> SelectSymbols(DateTime dt)
    {
        // Re-download the CSV file each day to get today's row.
        var tickers = DownloadUniverseFile(dt);
        // If there isn't an entry for the current date, return an empty universe.
        if (tickers.IsNullOrEmpty())
        {
            return Enumerable.Empty<Symbol>();
        }
        // Convert the stock tickers in the CSV file to Symbol objects.
        return tickers
            .Split(',')
            .Select(x => QuantConnect.Symbol.Create(x, SecurityType.Equity, Market.USA));
    }
}
```

```
from io import StringIO

class FrameworkScheduledUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2023, 6, 1)
        self.set_end_date(2023, 8, 1)
        self.set_cash(10000000)

        # Add a universe of that read from a Dropbox source to select the stocks 30 minutes before the week's first trading day.
        spy = Symbol.create("SPY", SecurityType.EQUITY, Market.USA)
        self.add_universe_selection(ScheduledUniverseSelectionModel(
            self.date_rules.week_start(spy),
            self.time_rules.before_market_open(spy, 10),
            self.select_symbols
        ))
        # Add Alpha model to trade based on the selections; the signals last for a week until the next selection.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(7)))
        # Equally invest in insights to dissipate the capital risk evenly.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

    def _download_universe_file(self, dt: datetime) -> str:
        # Download the universe CSV file. Dropbox links require the "dl=1" URL parameter.
        file = self.download(
            "https://www.dropbox.com/scl/fi/fbrxitk4ec3w91nse1raa/df.csv?rlkey=7r042rukzkthp7y1srloyhkov&st=5r4sdfwd&dl=1"
        )
        # Convert the CSV file data into a dictionary where the key is the date and
        # the value is a comma-separated string of stock tickers.
        df = pd.read_csv(StringIO(file), index_col=0).iloc[:, 0]
        df.index = pd.to_datetime(df.index).date
        date = dt.date()
        # Return the universe if the date matches.
        if date in df:
            return df.loc[dt.date()]
        return ''

    def select_symbols(self, dt: datetime) -> list[Symbol]:
        # Re-download the CSV file each day to get today's row.
        tickers = self._download_universe_file(dt)
        # If there isn't an entry for the current date, return an empty universe.
        if not tickers:
            return []
        # Convert the stock tickers in the CSV file to Symbol objects.
        return [Symbol.create(x, SecurityType.EQUITY, Market.USA) for x in tickers.split(",")]
```

#### Example 2: Seasonality

The following algorithm selects between SPY and short-term bond ETF according to the current month. It holds SPY on May, June, July, November, and December since they are historically better-performing months while holding SHV to preserve funds and earn interest in the remaining months.

Select Language: C#Python

```
public class FrameworkScheduledUniverseSelectionAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetStartDate(2019, 1, 1);
        SetEndDate(2024, 1, 1);

        // Add a universe of that select based on the time to trade seasonality.
        AddUniverseSelection(new ScheduledUniverseSelectionModel(
            DateRules.WeekStart(),
            TimeRules.At(0, 0),
            SelectSymbols)
        );
        // Add an Alpha model to trade based on the selections; the signals will last until the next selection.
        AddAlpha(new ConstantAlphaModel(InsightType.Price, InsightDirection.Up, TimeSpan.FromDays(7)));
        // Equally invest in insights to dissipate the capital risk evenly.
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
    }

    private IEnumerable<Symbol> SelectSymbols(DateTime dateTime)
    {
        // May/Jun/Jul/Nov/Dec are statistically better-performing months of SPY.
        if (new[] { 5, 6, 7, 11, 12 }.Contains(dateTime.Month))
        {
            return new[] { QuantConnect.Symbol.Create("SPY", SecurityType.Equity, Market.USA) };
        }
        // In other months, hold short-term bonds to preserve funds while earning interest.
        return new[] { QuantConnect.Symbol.Create("SHV", SecurityType.Equity, Market.USA) };
    }
}
```

```
class FrameworkScheduledUniverseSelectionAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_start_date(2019, 1, 1)
        self.set_end_date(2024, 1, 1)

        # Add a universe of that select based on the time to trade seasonality.
        self.add_universe_selection(ScheduledUniverseSelectionModel(
            self.date_rules.week_start(),
            self.time_rules.at(0, 0),
            self.select_symbols
        ))
        # Add an Alpha model to trade based on the selections; the signals will last until the next selection.
        self.add_alpha(ConstantAlphaModel(InsightType.PRICE, InsightDirection.UP, timedelta(7)))
        # Equally invest in insights to dissipate the capital risk evenly.
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())

    def select_symbols(self, date_time: datetime) -> list[Symbol]:
        # May/Jun/Jul/Nov/Dec are statistically better-performing months of SPY.
        if date_time.month in [5, 6, 7, 11, 12]:
            return [Symbol.create("SPY", SecurityType.EQUITY, Market.USA)]
        # In other months, hold short-term bonds to preserve funds while earning interest.
        return [Symbol.create("SHV", SecurityType.EQUITY, Market.USA)]
```

#### Other Examples

For more examples, see the following algorithms:

Demonstration Algorithms

[ScheduledUniverseSelectionModelRegressionAlgorithm.py\\
Python](https://github.com/QuantConnect/Lean/blob/master/Algorithm.Python/ScheduledUniverseSelectionModelRegressionAlgorithm.py) [ScheduledUniverseSelectionModelRegressionAlgorithm.cs\\
C#](https://github.com/QuantConnect/Lean/blob/master/Algorithm.CSharp/ScheduledUniverseSelectionModelRegressionAlgorithm.cs)

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/02%20Universe%20Selection/06%20Scheduled%20Universes)