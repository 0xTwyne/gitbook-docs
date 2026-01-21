# Safety & Liquidation Mechanisms

To safeguard the system and protect Credit-LP deposits, Twyne introduces its own liquidation framework - Liquidation by Inheritance - that acts as the first line of defense before any position becomes liquidateable on the underlying lending protocol.

By resolving distressed positions internally, Twyne protects Credit LPs and reduces their risk surface. As long as Twyne handles liquidations, Credit LPs never incur losses from liquidations.&#x20;

***

### Liquidation by Inheritance

Twyne uses a liquidation mechanism inspired by Liquity’s fallback design (which kicks in whenever their Stability Pool is out of funds).&#x20;

‘Inheriting’ means the liquidator can step into the borrower’s shoes, take over their unhealthy position and add collateral, thereby decreasing LTV. This doesn’t necessitate a swap, or even a technical setup to liquidate - only excess credit to participate.

This mechanism avoids immediate collateral sales, keeps capital within the system, and creates a profit opportunity for liquidators while adding another relief valve for unused borrowing capacity.

Currently, inheritance is limited to a single liquidator per event. A more distributed design, where liquidations are shared among multiple entities, is planned but not yet implemented.

{% hint style="info" %}
You can find more details in the [Liquidation by Inheritance](https://twyne.gitbook.io/twyne/tech-overview/liquidation-logic/liquidation-by-inheritance) section.
{% endhint %}

***

### Fallback Liquidations

If Twyne liquidators fail to act in time - usually in cases of extreme price volatility or sudden market downturns - the underlying protocol will perform its standard liquidation process.

In this case:

* From the protocol’s perspective, the borrower and Credit LPs form a single position
* The base protocol liquidates the joint position at its own liquidation thresholds (e.g., \~90% LLTV on Euler), applying its liquidation incentive

Only in cases when the liquidations are performed by the base lending market, Credit LPs may incur some losses. That said, fallback liquidation can still work in the CLP’s favor if the borrower’s collateral is sufficient to cover both debt and the liquidation incentives.

{% hint style="info" %}
Read more about the main risks of [Fallback Liquidations](https://twyne.gitbook.io/twyne/tech-overview/liquidation-logic/fallback-liquidations).
{% endhint %}
