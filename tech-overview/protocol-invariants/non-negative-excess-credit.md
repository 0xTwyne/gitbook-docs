# Non-Negative Excess Credit

The non-negative excess credit invariant is one of the two fundamental invariants that ensure Twyne's stability and safety. This document provides a comprehensive explanation of what this invariant means, why it's critical, how it's enforced, and the mathematical principles behind it.

### What is the Non-Negative Excess Credit Invariant?

At its core, the non-negative excess credit invariant states that:

**A collateral vault must never have negative excess credit.**

In other words, a borrower should never be allowed to reserve more credit from the Intermediate Vault than what they are entitled to based on their current collateral and parameters.

This can be expressed mathematically as:

$$\text{excessCredit} = C+C_{LP} - \frac{C \cdot \tilde{\lambda}_t}{\beta_{safety} \cdot \tilde{\lambda}_e} \geq 0$$

Where:

* &#x20;$$C+C_{LP}$$ = Total collateral in the vault
* $$C$$ = User’s collateral
* $$C_{LP}$$ = Reserved credit
* $$\tilde{\lambda}_t$$ = Twyne liquidation LTV
* $$\tilde{\lambda}_e$$ = External protocol liquidation LTV
* $$\beta_{safe}$$ = Safety buffer (typically 0.95 - 1)

### Why Is This Invariant Critical?

The non-negative excess credit invariant serves several crucial purposes:

1. **Credit LP Protection**: It ensures that borrowers reserve the credit required, which protects Credit LPs from losing more than they should in external liquidation scenarios.
2. **External Liquidation Prevention**: By ensuring sufficient credit reservation, it maintains sufficient collateralization to avoid unexpected external liquidations.
3. **Protocol Stability**: It helps maintain the mathematical relationships that power Twyne's design, ensuring the entire system operates as intended.
4. **Borrower Experience**: It helps guarantee that Borrowers receive the liquidation LTV of their choosing.

If this invariant were violated, borrowers could potentially get liquidated on the underlying lending market before they are signaled as liquidatable on Twyne, this indirectly would also put Credit LPs' funds at risk and destabilizing the protocol.

### Mathematical Derivation

Let's derive the non-negative excess credit invariant from Twyne's fundamental equations:

1. For a healthy position, we need to satisfy two constraints:
   * Twyne constraint: $$B \leq \tilde{\lambda}_t \cdot C$$
   * External constraint: $$B \leq \beta_{safe} \cdot \tilde{\lambda}_e \cdot (C+C_{LP})$$
2. For optimal credit reservation, we want these constraints to be equal at the liquidation threshold: $$\tilde{\lambda}_t \cdot C = \beta_{safe} \cdot \tilde{\lambda}_e \cdot (C+C_{LP})$$
3. Solving for $$C_{LP}$$: $$C_{LP} = \frac{\tilde{\lambda}_t \cdot C}{\beta_{safe} \cdot \tilde{\lambda}_e} - C$$
4. Therefore, the total collateral the vault should ideally have is: $$\frac{\tilde{\lambda}_t \cdot C}{\beta_{safe} \cdot \tilde{\lambda}_e}$$
5. The excess credit is then: $$C+C_{LP} - \frac{\tilde{\lambda}_t \cdot C}{\beta_{safe} \cdot \tilde{\lambda}_e}$$
6. This excess must be non-negative, which gives us our invariant.

### Python Implementation for Verification

Here's a Python function to verify the non-negative excess credit invariant:

```python
def has_non_negative_excess_credit(total_assets, user_collateral, twyne_liq_ltv,
                                 ext_lending_liq_ltv, safety_buffer=0.95):
    """
    Check if a position has non-negative excess credit

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
    dict
        Verification results including excess credit amount
    """
    # Calculate maximum allowed total assets
    calculated_max_collateral = user_collateral * twyne_liq_ltv / (safety_buffer * ext_lending_liq_ltv)

    # Calculate excess credit
    excess_credit = total_assets - calculated_max_collateral

    # Check if excess credit is non-negative
    is_valid = excess_credit => 0

    return {
        "is_valid": is_valid,
        "excess_credit": excess_credit,
        "max_allowed_total": calculated_max_collateral,
        "actual_total": total_assets
    }
```

### Connection to [Rebalancing](../rebalance-logic.md)

The non-negative excess credit invariant directly relates to the rebalancing mechanism:

1. As interest accrues, user collateral decreases and Credit LP share increases
2. This can lead to excess credit if not addressed
3. Rebalancing releases excess credit to maintain the invariant
4. The amount to release is precisely the excess credit value

### Edge Cases and Considerations

#### Parameter Boundaries

The non-negative excess credit invariant assumes certain parameter boundaries:

* `twyne_liq_LTV` must be greater than 0 and less than or equal to `MAX_LTV`
* `ext_lending_liq_ltv` must be greater than 0
* `safety_buffer` must be less than equal to 1.0 and greater than 0

These boundaries are enforced in the contract initialization and parameter setting functions.

### Impact on User Operations

The non-negative excess credit invariant affects several user operations:

1. **Deposits**: When depositing collateral, protocol reserves enough credit to have non-negative excess credit
2. **Withdrawals**: When withdrawing collateral, excess credit is released (explained in [Rebalancing](../rebalance-logic.md))
3. **Setting liquidation LTV**: When changing the Twyne liquidation LTV, credit reservation may need adjustment
4. **Interest Accrual**: As interest accrues, [rebalancing](../rebalance-logic.md) may be needed to maintain the invariant
