# Contracts



<figure><picture><source srcset="../../.gitbook/assets/contracts5.svg" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/contracts4.svg" alt=""></picture><figcaption><p><em>Multiple such systems can operate independently</em></p></figcaption></figure>

## Credit Vault <a href="#credit-vault" id="credit-vault"></a>

* Deployed by Twyne team
* Unmodified Euler EVault based on [EVK](https://github.com/euler-xyz/euler-vault-kit), so has 1 asset that can be lent/borrowed. This asset can be used as a collateral in some lending market. For instance, we use eulerWETH as an asset which is a collateral asset to borrow USDC from Euler’s eUSDC vault
* Only collateral vaults can borrow from credit vaults

## Collateral Vault <a href="#collateral-vault" id="collateral-vault"></a>

* Deployed by the borrower via collateral vault factory
* At the time of deployment, the user specifies the collateral asset, external lending contract and liquidation LTV
* The factory then
  1. picks the intermediate vault based on the collateral asset.
  2. deploys collateral vault and configure it with the borrower-supplied parameters (collateral asset, external lending contract and liquidation LTV).
  3. configures intermediate vault (`setLTV()`) so that collateral vault can borrow from it with collateral vault’s own shares as collateral.
* Each collateral vault has one borrower, but a borrower may have multiple collateral vaults. So borrower to collateral vaults is a one-to-many relationship.

## **Terminology**

* Borrow and repay: If used standalone, these terms always refer to borrowing/repaying debt from/to the external lending market.
* Reserve and release: These terms always refer to borrowing/repaying debt (aka credit) from/to the intermediate vault.

Collateral vault enables borrowers to deposit and withdraw collateral, interact with intermediate vault (to reserve and release credit from LPs) and external lending protocol (to borrow and repay debt).
