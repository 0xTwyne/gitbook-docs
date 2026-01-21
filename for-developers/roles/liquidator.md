# Liquidator

The purpose of a liquidator is to liquidate a Twyne collateral vault. When a vault can be liquidated, the liquidator will take ownership of the collateral vault if the liquidation succeeds. In order for the liquidation to succeed, the collateral vault must be made healthy at the end of the batch of operations. This can be done in two ways:

1. Repaying debt. The usual approach for just-in-time liquidators is to repay all debt, withdraw all collateral, and empty the position to zero.
2. Increasing the collateral. This makes sense if a liquidator wants to become a Twyne user or hold the collateral for a certain period of time.

## Actions <a href="#function-calls" id="function-calls"></a>

### Check if a collateral vault can be liquidated

To check whether a collateral vault can be liquidated, call:

```solidity
/// @notice check if a vault can be liquidated
function canLiquidate() external view returns (bool);
```

If the function returns **true**, the collateral vault can be liquidated.

### Liquidate collateral vault

A successful liquidation involves a liquidator becoming the "borrower" of the collateral vault and making the position healthy.

A position can be made healthy in multiple ways: repay some debt or deposit some collateral or both.

All these operations must happen inside of the same EVC batch in order to make the position healthy (not liquidatable).

*   First `liquidate()()` must be called to give the liquidator control over the collateral vault<br>

    ```solidity
    /// @notice Begin the liquidation process for this vault.
    /// @dev Liquidation needs to be done in a batch.
    /// If the vault is liquidatable, this fn makes liquidator the new borrower.
    /// Liquidator is then responsible to make this position healthy.
    /// Liquidator may choose to wind down the position and take collateral as profit.
    function liquidate() external;
    ```

    \
    If you want to track liquidation events, monitor the emission of the following event from Collateral Vault factory:<br>

    ```solidity
    event T_SetCollateralVaultLiquidated(address collateralVault, address liquidator);
    ```

    \
    As a result of this operation, `collateralVault.borrower` variable is now set to the liquidator's address.
* If the liquidator chooses to decrease the debt in order to make the position healthy, then `repay(uint)()` must be called to repay the debt that the collateral vault owes to the underlying protocol.\
  The `repay(uint)()` function requires that the liquidator has approved the target asset to the collateral vault, so if the collateral vault is borrowing USDC from the underlying protocol, the liquidator must approve the collateral vault for USDC before the batch containing `repay(uint)()` occurs. A `maxRepay()(uint)` helper function exists to easily return the maximum amount of debt that must be repaid, but passing `type(uint).max` as the input value to `repay(uint)()` will automatically use the `maxRepay` value. An example of this flow is found in the `test_e_liquidate_make_healthy_reduce_debt()` test [in the code repository](https://github.com/0xTwyne/twyne-contracts-v1/blob/c246f25f38225f9c27c98fad305d3465ac6b3ef0/test/twyne/EulerLiquidationTest.t.sol#L586).

### Handle external liquidation

A collateral vault is a borrower from the perspective of its external lending protocol. If the external lending protocol liquidates Twyne's collateral vault, we call it an _external liquidation_.

Twyne relies on external liquidations as a [fallback](../../tech-overview/liquidation-logic/fallback-liquidations.md) if Twyne's own liquidation mechanism fails to realize in time.

If this happens, anyone can call `handleExternalLiquidation()` , read the Natspec comments for more details:

```solidity
/// @notice Handles the aftermath of an external liquidation by the underlying lending protocol
/// @dev Called when the vault's collateral was liquidated by the external protocol (e.g., Euler)
/// @dev Steps performed:
/// 1. Calculate and distribute the remaining collateral in this vault among
///  the liquidator, borrower and intermediate vault
/// 2. Repay remaining external debt using funds from the liquidator
/// 3. Reset the vault state so that it cannot be used again
/// @dev Can only be called when the vault is actually in an externally liquidated state
/// @dev Caller needs to call intermediateVault.liquidate(collateral_vault_address, collateral_vault_address, 0, 0)
/// in the same EVC batch if there is any bad debt left at the end of this call
/// @dev Implementation varies depending on the external protocol integration
function handleExternalLiquidation() external virtual;
```

Caller first have to approve the collateral vault to spend the debt asset.&#x20;
