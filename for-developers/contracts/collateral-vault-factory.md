# Collateral Vault Factory

The primary purpose of the collateral vault factory contract is to create Twyne collateral vaults. There is also a governance-controlled pause functionality in the factory contract, which individual Twyne Collateral Vaults refer to on functions that use the `whenNotPaused()` modifier.

### Function Calls <a href="#function-calls" id="function-calls"></a>

The key function in the factory contract is `createCollateralVault()`, which users call through the EVC to create their Twyne collateral vault:

```solidity
/// @notice This function is called when a borrower wants to deploy a new collateral vault.
/// @param _asset address of vault asset
/// @param _targetVault address of the target vault, used for the lookup of the beacon proxy implementation contract
/// @param _liqLTV user-specified target LTV
/// @return vault address of the newly created collateral vault
function createCollateralVault(address _asset, address _targetVault, uint _liqLTV)
```

The frontend combines this function call in a batch that includes the user’s first deposit and borrow.

If you want to track creation of collateral vaults, you can monitor this event emission from this factory:

```solidity
event T_CollateralVaultCreated(address collateral_vault);
```
