# User Types

### Credit Liquidity Provider (LP) <a href="#credit-liquidity-provider-lp" id="credit-liquidity-provider-lp"></a>

A credit LP is an entity looking to earn yield. They are likely to be lenders on existing lending markets maintaining low LTV, thus indicating they don’t borrow and are just earning yield on their assets.

They deposit their receipt tokens from the lending marker (eulerWETH) to a credit vault to earn a higher yield.

**Possible Credit LP actions**

* Deposit asset in a credit vault and receive vault shares
* Withdraw asset from an intermediate vault by burning vault shares

### Borrower <a href="#borrower" id="borrower"></a>

A borrower on Twyne is an entity either looking to borrow more aggressively than allowed on existing lending markets, or looking to protect their borrow position from liquidation.

They deploy collateral vaults to deposit collateral (eulerWETH), reserve credit (eulerWETH) from credit vault, and borrow (USDC) from the pre-determined external lending market (Euler).

### **Possible Borrower actions**

* Deploy a collateral vault via collateral vault factory. A collateral vault is configured with the collateral asset, credit vault and external lending contract.
* Deposit asset as collateral (eulerWETH) into the collateral vault, reserve credit (eulerWETH) from credit vault, and borrow (USDC) from external lending contract. A borrower pays the following interest rates:
  1. Interest to the external lending protocol on the borrowed amount. This interest accumulation is handled by the external lending contract.
  2. Interest to the credit vault. The ownership of portion of borrower’s collateral is continuously siphoned off to the credit vault.
* Withdraw collateral from the collateral vault, release credit back to intermediate vault, and repay debt to the external lending market.
* Update collateral vault’s liquidation LTV.
