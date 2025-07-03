# Algorithm Framework

## Overview

### Introduction

Since QuantConnect was formed in 2013, the community has created more than 2 million algorithms. Although they all seek to achieve different results, they can all be abstracted into the same core features. The Algorithm Framework is our attempt to provide this well-defined structure to the community, encouraging good design and making your work more reusable. LEAN provides many pre-built Algorithm Framework models you can use in your algorithms.

### Strategy Styles

The algorithm framework works best for portfolio rebalancing, ranking, and multi-factor style strategies. The default portfolio construction models are designed to weigh and invest in the assets as the signals values shift, aiming to be fully invested as much as possible.


Some strategies might find it difficult to fit this model. Technical entry signals are often tightly coupled with their exit criteria, they may not have defined hold periods, and they might follow the market to determine when to exit. This style of strategy can be addressed in a few ways:

1. Short Term Signals - Emitting insights for short periods until they are no longer valid.
2. Cancel Signals - Updating an insight's expiry time so that it's expired upon the exit signal.
3. Alpha Exit Signal - Treating exit as its own signal, a separate Alpha model could indicate exit.
4. Risk Model - Tracking exit using a risk model that attempts to lock in gains.

These techniques are still not ideal for many strategies. For example, with only a single signal, the built in `EqualWeightingPortfolioConstructionModel` will allocate 100% of buying power to a single asset, over concentrating risk. If a second signal comes a few days later, it would sell 50% of the portfolio to free cash and invest in the second asset. This can create high trading costs through portfolio churn.

With a [custom portfolio construction model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/key-concepts#01-Introduction), signals can mean what you need for your specific strategy style. For example, you might create a " `MultiBetPortfolioConstructionModel`" that uses each insight to open a specific "bet" that is separately allocated capital and tracked.

### Important Terminology

The following table defines important Algorithm Framework terminology:

| Term | Description |
| --- | --- |
| [Universe Selection model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/universe-selection/key-concepts) | A framework module that selects assets for your algorithm. |
| [Alpha model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/alpha/key-concepts) | A framework module that generates trading signals on the assets in your universe. |
| [Insight](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/alpha/key-concepts#06-Insights) | An object that represents a trading signal. The Alpha model generates these objects for the Portfolio Construction model. |
| [Portfolio Construction model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/key-concepts) | A framework module that determines position size targets based on the `Insight` objects it receives from the Alpha model. |
| [PortfolioTarget](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/key-concepts#05-Portfolio-Targets) | An object that represents the target position size for an asset in the universe. The Portfolio Construction model generates these objects for the Risk Management model. |
| [Risk Management](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/risk-management/key-concepts) | A framework module that manages market risks and ensures the algorithm remains within target parameters. This model adjusts the `PortfolioTarget` objects it receives from the Portfolio Construction model before they reach the Execution model. |
| [Execution](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/execution/key-concepts) | A framework module that places trades to reach the target risk-adjust portfolio based on the `PortfolioTarget` objects it receives from the Risk Management model. |

### Framework Advantages

We recommend you design strategies with the Algorithm Framework in most cases. The Algorithm Framework has many advantages over the classic algorithm design.

#### Pluggable Algorithm Modules

With the Algorithm Framework, your code can instantly utilize all modules built within the framework. The modules clip together in well-defined ways, which enable them to be shared and swapped interchangeably.

#### Focus On Your Strengths

If you write code in modules, you can focus on your strengths. If you are great at building universes, you can build Universe Selection modules. If you have risk management experience, you could write reusable risk monitoring technology.

#### Reduce Development By Using Community Modules

Easily share modules you've made between algorithms or pull in ones produced by the community. The strict separation of duties makes the Algorithm Framework perfect for reusing code between strategies.

### System Architecture

The Algorithm Framework is built into the `QCAlgorithm` class, so your strategy can access all the normal methods you use for algorithm development. You should be able to copy and paste your algorithms across without any changes. You don't need a separate class.

![The Algorithm Framework modules communicate w](https://cdn.quantconnect.com/web/i/docs/algorithm-framework/class-structure_rev2.png)

The framework data output of each module flows into the following module. The assets that the Universe Selection model selects are fed into the Alpha model to generate trade signals ( `Insight` objects). The `Insight` objects from the Alpha model are fed into the Portfolio Construction model to create `PortfolioTarget` objects, which contain the target number of units to hold for each asset. The `PortfolioTarget` objects from the Portfolio Construction model are fed into the Risk Management model to ensure the targets are within safe risk parameters and to adjust the `PortfolioTarget` objects if necessary. The `PortfolioTarget` objects from the Risk Management model are fed into the Execution model, which efficiently places trades to acquire the target portfolio.

![The flow of data between the Algorithm Framework modules](https://cdn.quantconnect.com/web/i/docs/algorithm-framework/algorithm-framework.png)

Select Language: C#Python

```
public class MyFrameworkAlgorithm : QCAlgorithm
{
    public override void Initialize()
    {
        SetUniverseSelection(new EmaCrossUniverseSelectionModel());
        AddAlpha(new RsiAlphaModel());
        SetPortfolioConstruction(new EqualWeightingPortfolioConstructionModel());
        SetExecution(new ImmediateExecutionModel());
        AddRiskManagement(new NullRiskManagementModel());
    }
}
```

```
class MyFrameworkAlgorithm(QCAlgorithm):
    def initialize(self) -> None:
        self.set_universe_selection(EmaCrossUniverseSelectionModel())
        self.add_alpha(RsiAlphaModel())
        self.set_portfolio_construction(EqualWeightingPortfolioConstructionModel())
        self.set_execution(ImmediateExecutionModel())
        self.add_risk_management(NullRiskManagementModel())
```

For simple strategies, it may seem like overkill to abstract out your algorithm concepts. However, even simple strategies can benefit from reusing the ecosystem of modules available in QuantConnect. Imagine pairing your EMA-cross Alpha model with a better execution system or plugging in an open-source [trailing stop Risk Management model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/risk-management/supported-models#07-Trailing-Stop-Model).

### Separation of Concerns

The separation of concerns principle is an algorithm design principle where individual components are isolated and their responsibilities don't overlap. The Algorithm Framework is designed to uphold this design principle. The individual framework components shouldn't rely on the state of the other framework components in order to operate.

In the Algorithm Framework, the models should not communicate. The universe constituents and the `Insight` objects you emit should be the same, regardless of what you select for the Portfolio Construction, Risk Management, and Execution models. If your algorithm logic naturally violates the separation of concerns design principle, it's more appropriate to use the classic or [hybrid design](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/hybrid-algorithms).

### Framework vs Classic Design

Select the classic or Algorithm Framework design based on the needs of your algorithm. If your algorithm uses special order types or passes information between the different algorithm components, use the classic design. If you can reuse existing modules and the Algorithm Framework modules can work in isolation, use the Algorithm Framework design.

You can also see our
[Videos](https://www.youtube.com/user/QuantConnect/videos).
You can also get in touch with us via [Discord](https://www.quantconnect.com/discord).


Did you find this page helpful?

Yes No

Contribute to the documentation: [![](https://cdn.quantconnect.com/i/tu/docs_github_icon_rev0.png)](https://github.com/QuantConnect/Documentation/tree/master/03%20Writing%20Algorithms/34%20Algorithm%20Framework/01%20Overview)