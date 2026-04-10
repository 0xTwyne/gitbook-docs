# Collateral Vault Factory

The primary purpose of the collateral vault factory contract is to create Twyne collateral vaults. There is also a governance-controlled pause functionality in the factory contract, which individual Twyne Collateral Vaults refer to on functions that use the `whenNotPaused()` modifier.

### Function Calls <a href="#function-calls" id="function-calls"></a>

The key function in the factory contract is `createCollateralVault()`, which borrowers call to create their Twyne collateral vault:

```solidity
enum VaultType { EULER_V2, AAVE_V3 }

/// @notice Deploy a new collateral vault.
/// @param _vaultType type of vault, only EULER_V2 or AAVE_V3 allowed
/// @param _intermediateVault address of the intermediate vault the collateral vault will reserve credit from
/// @param _targetVault address of the target vault (e.g., the Aave V3 Pool), used to look up the beacon implementation
/// @param _liqLTV user-specified liquidation LTV (1e4 precision)
/// @param _targetAsset debt token to be borrowed from the target vault (e.g., USDC).
///        For EULER_V2 this parameter is ignored by the factory but must still be supplied.
/// @return vault address of the newly created collateral vault
function createCollateralVault(
    VaultType _vaultType,
    address _intermediateVault,
    address _targetVault,
    uint _liqLTV,
    address _targetAsset
) external returns (address vault);
```

The factory validates that the combination of intermediate vault, target vault, and target asset has been whitelisted in the `VaultManager` before deploying the new `BeaconProxy`. For `AAVE_V3` vaults the factory also reads the eMode category id configured via `setCategoryId(...)` and passes it to the vault's `initialize`.

The frontend typically combines this function call in an EVC batch that also performs the user’s first deposit and borrow — the collateral vault's address can be recovered from `evc.batchSimulation(...)` before the batch is actually executed.

If you want to track creation of collateral vaults, you can monitor this event emission from this factory:

```solidity
event T_CollateralVaultCreated(address collateral_vault);
```
