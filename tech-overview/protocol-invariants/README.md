# Protocol Invariants

Core of the Twyne protocol can be encoded into two invariants.

The protocol ensures these invariants are satisfied for every user-operation.

Now let’s build up the mathematical framework that makes this system work. We’ll start simple and add complexity as we go.

#### Understanding the Two LTV Perspectives <a href="#understanding-the-two-ltv-perspectives" id="understanding-the-two-ltv-perspectives"></a>

Every position in Twyne has two different LTV calculations that matter:

1. **Twyne LTV** represents how leveraged the borrower is from their own perspective:

$$
\lambda_t = \frac{B}{C}
$$

Where B is the borrowed amount and C is the borrower’s own collateral.

2. **External LTV** represents how the position looks to the underlying protocol:

$$
\lambda_e = \frac{B}{C + C_{LP}}
$$

Where $$C_{LP}$$ is the additional collateral provided by Credit LPs.

The magic happens in the relationship between these two values. While $$\lambda_t$$ might be 85% (quite risky), $$\lambda_e$$ might only be 70% (quite safe). This gap is what Twyne exploits to provide additional borrowing power.

#### The Liquidation Conditions <a href="#the-liquidation-conditions" id="the-liquidation-conditions"></a>

A position becomes liquidatable in Twyne when EITHER of these conditions is met:

**Condition 1: Twyne Threshold Breached**

$$
B > \tilde{\lambda}_t \times C
$$

This checks if the borrower’s debt exceeds what’s allowed given their chosen Twyne liquidation LTV $$\tilde{\lambda}_t$$ and their own collateral.

**Condition 2: External Safety at Risk**

$$
B > \beta_{\text{safe}} \times \tilde{\lambda}_e \times (C + C_{\text{LP}})
$$

This ensures we maintain a safety buffer (, typically 95%) below the external protocol’s liquidation threshold.

Let’s understand why we need both conditions through an example:

Imagine Alice has:

* _1 ETH of her own collateral (C = 1)_
* _0.5 ETH reserved from CLPs (_ $$C_{LP}$$ _= 0.5)_
* _0.8 ETH worth of debt (B = 0.8)_
* _Chosen Twyne liquidation LTV of 85% (_ $$\tilde{\lambda}_t$$ _= 0.85)_
* _External protocol liquidation LTV of 75% (_ $$\tilde{\lambda}_e$$ _= 0.75)_
* _Safety buffer of 95% (_ $$\beta_{\text{safe}}$$ _= 0.95)_

Let’s check both conditions:

* _Condition 1: 0.8 > 0.85 × 1 = 0.85 ✗ (Not triggered)_
* _Condition 2: 0.8 > 0.95 × 0.75 × 1.5 = 1.069 ✗ (Not triggered)_

Alice’s position is safe. But if her debt increases to 0.86 ETH:

* _Condition 1: 0.86 > 0.85 × 1 = 0.85 ✓ (Triggered!)_
* _Condition 2: 0.86 > 1.069 ✗ (Still safe)_

Now Twyne will liquidate her position even though the external protocol still sees it as safe. This is the key to protecting CLPs.
