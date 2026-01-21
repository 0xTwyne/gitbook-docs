# Liquidation Logic

Liquidations are the cornerstone of any lending protocol’s risk management system. In Twyne, they take on even greater importance because we’re allowing borrowers to access higher leverage while protecting the Credit LPs who make this possible. This guide will take you from the conceptual foundations through the mathematical theory to the practical implementation details.

### Conceptual Overview: The Two Safety Nets <a href="#conceptual-overview-the-two-safety-nets" id="conceptual-overview-the-two-safety-nets"></a>

To understand Twyne’s liquidation system, imagine a tightrope walker (the borrower) crossing between two buildings. Traditional lending protocols give them one safety net below. Twyne gives them two safety nets at different heights.

The first safety net (Twyne’s liquidation threshold) catches them early, allowing for a controlled landing where someone else can take over the walk. The second safety net (the external protocol’s liquidation threshold) is the last resort - if you hit this one, it’s messy and potentially painful for everyone involved.

This dual-safety-net system is what allows Twyne to offer higher leverage while maintaining security. The borrower can walk higher on the tightrope (borrow more), but we ensure they’re caught before they’d hit the ground (external liquidation).

### Why Two Liquidation Thresholds? <a href="#why-two-liquidation-thresholds" id="why-two-liquidation-thresholds"></a>

In traditional lending protocols like Aave or Euler, there’s a single liquidation threshold. When your position becomes undercollateralized beyond this point, liquidators swoop in, repay part of your debt, and take your collateral plus a bonus. It’s efficient but limiting - the protocol must be conservative to protect all lenders equally.

Twyne introduces a paradigm shift. By creating a two-tier system, we can offer borrowers higher effective LTVs while ensuring the underlying protocol never sees an unsafe position. Think of it like subletting an apartment - the landlord (external protocol) only cares that the total rent is paid, not how the tenants (borrower + CLPs) split it among themselves.

### When Is A Borrower Liquidated?

Whenever [Position Health Invariant](../protocol-invariants/position-health-invariant.md) is in violation.
