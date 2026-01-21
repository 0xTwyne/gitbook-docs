# Collateral Vault

Based on the previous sections, we can look into the collateral vault with more depth.

A collateral vault:

* deposits and withdraws collateral from the borrower
* reserves and releases credit (same asset as collateral) from the credit vault
* borrows and repays debt to the external lending contract

How much can be borrowed and reserved depends on our invariants. The key requirement is the vault must not be liquidatable, and the details of Twyne liquidation mechanics [are explained here](../../tech-overview/liquidation-logic/).

A collateral vault can also be liquidated:

* Liquidated by Twyne
* Externally liquidated (on the external lending protocol)

Collateral vault can also be _rebalanced_. This is an action of releasing excess credit back to the intermediate vault. Further details on how rebalancing works [are in the mechanics section](../../tech-overview/rebalance-logic.md).
