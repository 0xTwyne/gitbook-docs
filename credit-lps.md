# Credit-LPs

Twyne lets CLPs extend their lending activity into additional lending agreements, similar to how restaking extends staking.

Beyond pure lending, CLPs delegate their idle borrowing capacity to others by depositing their IOU tokens (e.g., aETH) into a Credit Vault. This makes otherwise unused credit productive again and allows CLPs to earn additional yield on top of the base lending APY

{% hint style="info" %}
You can dive deeper into the math behind the CLP supply rate[ here](https://twyne.gitbook.io/twyne/safety-and-liquidation-mechanisms).
{% endhint %}

***

### Risk & protection

Delegating credit effectively backs another borrower’s loan. As a result, Credit-LPs deliberately take on higher risk than passive lenders on Aave.

To mitigate CLP’s exposure, Twyne acts as the first-order liquidator, resolving borrower positions before they become eligible for liquidation on the underlying protocol. As long as the borrower is liquidated on Twyne, Credit LPs face no losses.

In cases of extreme price volatility and swift market downturns, there might be instances where nobody has liquidated the position on Twyne in time. In such scenarios, the underlying protocol would kick in as the fallback liquidator, which could (though not always) incur losses for the Credit LP.

{% hint style="info" %}
We have conducted thorough research and rigorous testing of the added risks that Credit LPs may face when using Twyne. You can read more about them in the [Safety Mechanism section](https://twyne.gitbook.io/twyne/safety-and-liquidation-mechanisms).
{% endhint %}
