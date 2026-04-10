# User Types

### Credit Liquidity Provider (LP) <a href="#credit-liquidity-provider-lp" id="credit-liquidity-provider-lp"></a>

A Credit LP is an entity looking to earn extra yield. They are typically already passive lenders on an underlying lending market (Aave, Euler, …) with low LTV — meaning they aren't using their own borrowing power, and their receipt tokens are just sitting there earning the base lending APY.

They deposit those receipt tokens into a Twyne intermediate vault to earn a delegation fee on top of the base APY. For the Aave V3 integration the deposited asset is a Twyne-maintained ERC20 wrapper around the `aToken` (for example `aWETHWrapper`); for the Euler V2 integration it is the `eToken` itself (e.g. `eWETH`).

**Possible Credit LP actions**

* Deposit asset into an intermediate vault and receive vault shares.
* Withdraw asset from an intermediate vault by burning vault shares.

### Borrower <a href="#borrower" id="borrower"></a>

A borrower on Twyne is an entity either looking to borrow more aggressively than the underlying market allows, or wanting extra safety margin against liquidation on their existing borrow position.

They deploy collateral vaults to deposit collateral (e.g. `aWETHWrapper` or `eWETH`), reserve credit in that same asset from an intermediate vault, and borrow a target asset (e.g. USDC) from the pre-determined external lending market (Aave V3 Pool, Euler V2 eUSDC, …).

### **Possible Borrower actions**

* Deploy a collateral vault via `CollateralVaultFactory.createCollateralVault(...)`. The vault is configured with the vault type (`AAVE_V3` or `EULER_V2`), the intermediate vault (which determines the collateral asset), the target vault, the target asset, and the user-chosen liquidation LTV.
* Deposit the collateral asset (or its underlying) into the collateral vault, reserve credit from the intermediate vault, and borrow the target asset from the external lending market. A borrower pays two components of interest:
  1. Interest on the target asset debt to the external lending protocol. This is accrued on the external market directly.
  2. Interest to the intermediate vault on the reserved credit. In practice, ownership of a portion of the borrower's collateral is continuously siphoned off into the intermediate vault at the credit vault's IRM rate.
* Repay the target asset debt, release credit back to the intermediate vault, withdraw collateral.
* Adjust the collateral vault's liquidation LTV at any time (subject to bounds set by governance).
* Use leverage / deleverage / teleport operators to perform 1-click looping and migration of existing external positions (see the [borrower integration guide](../../borrower-integration-guide.md)).
