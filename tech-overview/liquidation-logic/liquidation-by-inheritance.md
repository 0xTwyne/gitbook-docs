# Liquidation by Inheritance

In traditional DeFi systems, liquidations function like fire sales: liquidators repay debt, seize collateral at a discount, and immediately sell it for profit. While effective, this approach introduces selling pressure during market stress and demands sophisticated infrastructure to remain profitable.

Twyne introduces a fundamentally different mechanism: liquidation by inheritance. Instead of dismantling positions, they are transferred intact to new owners, who can recapitalize them by adding collateral to satisfy the liquidation thresholds.

### How Inheritance Works <a href="#how-inheritance-works" id="how-inheritance-works"></a>

When a position becomes liquidatable, any user with sufficient collateral can “inherit” it entirely. Let’s walk through the process:

1. **Position Identification**: Alice’s position becomes liquidatable with:
   * Collateral: 100 USDC
   * Debt: 95 USDC
   * Twyne LTV: 95% (exceeding her 90% threshold)
2. **Inheritance Execution**: Bob, who has 50 USDC of his own collateral, inherits Alice’s position
3. **Post-Inheritance State**: Bob now has:
   * Collateral: 150 USDC (his 50 + Alice’s 100)
   * Debt: 95 USDC
   * Twyne LTV: 63.3% (much healthier)
4. **Profit Mechanism**: Bob effectively acquired 100 USDC worth of collateral for 95 USDC of debt, netting 5 USDC in value

### Economic Incentives for Inheritance <a href="#economic-incentives-for-inheritance" id="economic-incentives-for-inheritance"></a>

The inheritance model creates a set of unique incentives that strengthen the protocol:

#### **For Liquidators**:

* No requirement for specialized infrastructure, flash loans, or DEX liquidity, resulting in reduced technical and capital barriers to entry.
* Acquire assets passively at a discount, with the potential to profit from subsequent market recovery.
* Earn yield while holding positions prior to liquidation execution.

#### **For the Protocol**:

* Capital remains within the system (i.e., liquidated positions are retained rather than exiting through immediate forced sales)
* No adverse market impact from collateral being dumped into external markets
* Broader participation in the liquidation process, due to lower technical and capital barriers

#### **For Credit LPs**:

* Ensures that Credit LPs remain whole and incur no losses
