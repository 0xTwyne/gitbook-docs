# Rebalance Logic

Rebalancing is a core process in Twyne that ensures positions remain optimally sized as interest accrues over time. This document explains the purpose, mechanism, and implementation of rebalancing in Twyne.

## Purpose of Rebalancing <a href="#purpose-of-rebalancing" id="purpose-of-rebalancing"></a>

As interest accrues in Twyne, two key changes occur:

1. Borrower collateral decreases (due to the siphoning rate paid to Credit LPs)
2. Credit LP shares increase (due to receiving interest)

This creates a situation where a Collateral Vault has reserved more credit from the Intermediate Vault than it currently needs based on the borrower’s reduced collateral. This excess credit is inefficient because:

1. It’s reserved but not being used optimally
2. It could be made available to other borrowers
3. It costs the borrower interest without providing additional borrowing power

Rebalancing is the process of calculating and releasing this excess credit back to the Intermediate Vault, ensuring capital efficiency and proper position sizing.

## Excess Credit Calculation <a href="#excess-credit-calculation" id="excess-credit-calculation"></a>

### Mathematical Model <a href="#mathematical-model" id="mathematical-model"></a>

Recall that the excess credit is calculated as (refer to [Non-Negative Excess Credit](protocol-invariants/non-negative-excess-credit.md) invariant):

$$C+C_{LP} - \frac{C⋅\tilde{\lambda}_t}{\beta_{safe}⋅\tilde{\lambda}_e}$$

Where:

* &#x20;$$C+C_{LP}$$ = Total collateral in the vault
* $$C$$ = User’s collateral
* $$C_{LP}$$ = Reserved credit
* $$\tilde{\lambda}_t$$ = Twyne liquidation LTV
* $$\tilde{\lambda}_e$$ = External protocol liquidation LTV
* $$\beta_{safe}$$ = Safety buffer (typically 0.95 - 1)

The collateral vault then releases this excess credit bringing $$C_{LP}$$ down and making excess credit 0.

Practically, due to precision issues, we can never bring it down exactly to 0. In that case, we always ensure the excess credit never goes negative.

### Python Implementation <a href="#python-implementation" id="python-implementation"></a>

```
def calculate_excess_credit(total_assets, user_collateral, twyne_liq_ltv,
                          ext_lending_liq_ltv, safety_buffer=0.95):
    """
    Calculate excess credit in the position

    Parameters:
    -----------
    total_assets : float
        Total assets in the vault (C + C_LP)
    user_collateral : float
        User's collateral (C)
    twyne_liq_ltv : float
        Twyne liquidation LTV (e.g., 0.85 for 85%)
    ext_lending_liq_ltv : float
        External lending protocol liquidation LTV (e.g., 0.75 for 75%)
    safety_buffer : float
        Safety buffer to avoid external liquidation (default 0.95)

    Returns:
    --------
    float
        Excess credit amount
    """
    required_total = user_collateral * twyne_liq_ltv / (safety_buffer * ext_lending_liq_ltv)

    if total_assets <= required_total:
        return 0

    return total_assets - required_total
```

## Rebalancing Process <a href="#rebalancing-process" id="rebalancing-process"></a>

Step-by-Step Process

1. **Calculate Excess Credit**:
   * Determine the current user collateral (total assets - reserved credit)
   * Calculate how much total collateral should be required based on current user collateral
   * Compute the difference between required total and actual total
2. **Release Excess Credit**:
   * If excess credit is positive, release it back to the Intermediate Vault
   * Update internal accounting to reflect the new reserved credit amount
3. **Verify Invariants**:
   * Ensure the position remains healthy after rebalancing
   * Confirm non-negative excess credit invariant is maintained

<details>

<summary>Rebalancing Example</summary>

How rebalancing works in practice:

#### Initial State <a href="#initial-state" id="initial-state"></a>

* Borrower has 10 ETH collateral
* Twyne liquidation LTV: 85%
* External lending protocol LTV: 75%
* Safety buffer: 95%
* Required credit initially: 10 ETH _\[(0.85/(0.9&#x35;_&#x30;.75)-1] = 1.92 ETH
* Total assets in collateral vault: 11.92 ETH

#### After Interest Accrual <a href="#after-interest-accrual" id="after-interest-accrual"></a>

After a period of interest accrual:

* Borrower collateral reduced to 9.5 ETH due to interest payments
* Reserved credit increased to: 1.92 ETH + 0.5 ETH = 2.42 ETH
* Total assets in collateral vault: 11.92 ETH

Now we calculate the excess credit:

1. User collateral = 9.5 ETH
2. Required total = 9.5 \* 0.85 / (0.95 \* 0.75) = 11.333 ETH
3. Actual total = 11.92 ETH
4. Excess credit = 11.920 - 11.333 = 0.586 ETH

There’s excess credit to release for a total of 0.586 ETH. The borrower is incentivized to release it to reduce their interest payments to the Credit Vault.

</details>

## When Rebalancing Occurs <a href="#when-rebalancing-occurs" id="when-rebalancing-occurs"></a>

Rebalancing on Twyne can happen in several ways:

1. **Automatic Rebalancing**:
   * When certain collateral-modifying operations are performed
   * Examples include deposits, withdrawals, or Twyne liquidation LTV parameter changes.
2. **Manual Rebalancing**:
   * Anyone can explicitly call the `rebalance()` function
   * This might be done periodically to optimize positions

## Security Implications <a href="#security-implications" id="security-implications"></a>

Rebalancing has several security implications:

1. **Manipulation Resistance**: The rebalancing mechanism must be resistant to exploitation through rapid changes in parameters or positions
2. **Safety During Market Volatility**: Rebalancing must remain safe even during periods of high market volatility
3. **Position Health**: Rebalancing should never compromise position health or violate protocol invariants
