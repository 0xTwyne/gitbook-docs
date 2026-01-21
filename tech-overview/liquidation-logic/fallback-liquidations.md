# Fallback Liquidations

Despite the best efforts of liquidators, positions may not always be inherited promptly under extreme market conditions. In such cases, the underlying protocol executes its own liquidation. This fallback process can result in potential losses for Credit LPs.

### Understanding External Protocol Liquidations <a href="#understanding-external-protocol-liquidations" id="understanding-external-protocol-liquidations"></a>

When Euler or another underlying protocol liquidates a Twyne position, it has no notion of the internal split between borrower and Credit LP collateral. From its perspective, the liquidation applies to a single combined position.

The challenge for Twyne is to determine how to fairly redistribute the remaining assets between borrowers and Credit LPs following such an external liquidation.

### Post-Liquidation Accounting <a href="#post-liquidation-accounting" id="post-liquidation-accounting"></a>

After an external liquidation, Twyne must:

1. **Assess Remaining Assets**: Determine what collateral and debt remain
2. **Prioritize CLP Protection**: Ensure CLPs recover maximum possible value
3. **Maintain Over-collaterlization**: Keep loan healthy enough that an incentive exists to inherit the position
4. **Prevent Gaming**: Avoid creating exploitable conditions

The mathematical framework for this is covered in detail in Section 6.2.1 of the whitepaper. The key insight is that we reset the borrower’s liquidation LTV to the maximum allowed value and redistribute assets accordingly.

### Calculating CLP Losses <a href="#calculating-clp-losses" id="calculating-clp-losses"></a>

Credit LPs face potential losses when external liquidations occur. The loss amount depends on several factors:

* The borrower’s risk level  $$\rho = \tilde{\lambda}_t / \tilde{\lambda}_{\text{max}_t}$$
* The external protocol’s liquidation parameters (incentive and closing factor)
* The safety buffer maintained by Twyne

The whitepaper provides detailed loss analysis showing that CLPs are protected as long as specific conditions are met. For typical parameters (5% liquidation incentive, 50% closing factor), CLPs face no losses unless borrowers choose extremely high liquidation LTVs.
