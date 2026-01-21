# Collateral Vault

This contract is the collateral vault contract that all borrowers on Twyne interact with. In Twyne v1, every collateral vault has a single owner. Many functions in the collateral vault can be called only by the vault owner. This limits the ability for anyone else to interact with the vault, enhancing the security of the user’s collateral assets.

It isn’t ERC20 or ERC4626 compliant, partially the current design requires a single owner of vault value, meaning the vault shares must be non-transferrable. Correctness is maintained through our invariants, explained in more detail in the mechanics section.

## Function Calls <a href="#function-calls" id="function-calls"></a>

Borrowers can deposit and withdraw collateral, borrow and repay their debt from the external lending protocol.

### Deposit collateral <a href="#deposit-collateral" id="deposit-collateral"></a>

Euler collateral vaults have assets like eWETH (yield bearing, Euler wrapped WETH) as collateral. To deposit these assets, use:

```solidity
/// @notice Deposits a certain amount of assets for a receiver.
/// @param assets The assets to deposit.
function deposit(uint assets) external;
```

To deposit the underlying asset (like WETH), use:

```solidity
/// @notice Deposits a certain amount of underlying asset for a receiver.
/// @param underlying The underlying assets to deposit.
function depositUnderlying(uint underlying) external;
```

This function first trasfers the underlying asset, converts it to the corresponding Euler wrapped asset and then registers it as collateral.

### Withdraw collateral <a href="#withdraw-collateral" id="withdraw-collateral"></a>

To withdraw your collateral, use:

```solidity
/// @notice Withdraws a certain amount of assets for a receiver.
/// @param assets Amount of collateral assets to withdraw.
/// @param receiver The receiver of the withdrawal.
function withdraw(uint assets, address receiver) external
```

In case you want to receive the underlying asset, use:

```solidity
/// @notice Withdraw a certain amount of collateral and transfers collateral asset's underlying asset to receiver.
/// @param assets Amount of collateral asset to withdraw.
/// @param receiver The receiver of the redemption.
/// @return underlying Amount of underlying asset transferred.
function redeemUnderlying(uint assets, address receiver) external returns (uint underlying);
```

Note that, you still specify the amount of collateral (which is eWETH in our running example). The contract will remove that amount as collateral, convert it to the underlying asset, and transfer all the redeemed underlying asset to `receiver`.

### Borrow <a href="#borrow" id="borrow"></a>

Each collateral vault can only borrow a single asset called _target asset_. To borrow that target asset, use:

```solidity
/// @notice Borrows target assets from the external lending protocol
/// @dev This function calls the internal _borrow function to handle the protocol-specific borrow logic,
/// then transfers the target asset from the vault to _receiver.
/// @param _targetAmount The amount of target asset to borrow
/// @param _receiver The receiver of the borrowed assets
function borrow(uint _targetAmount, address _receiver) external;
```

### Repay <a href="#repay" id="repay"></a>

```solidity
/// @notice Repays debt owed to the external lending protocol
/// @dev If _amount is set to type(uint).max, the entire debt will be repaid
/// @dev This function transfers the target asset from the caller to the vault, then
/// calls the internal _repay function to handle the protocol-specific repayment logic
/// @dev Reverts if attempting to repay more than the current debt
/// @param _amount The amount of targe asset to repay, or type(uint).max for full repayment
function repay(uint _amount) external;
```

### Liquidate <a href="#liquidate" id="liquidate"></a>

Liquidating a collateral vault is a multi-step process.

1.  Call `liquidate()` which sets the collateral vault’s `borrower` variable to the caller:

    ```solidity
    /// @notice Begin the liquidation process for this vault.
    /// @dev Liquidation needs to be done in a batch.
    /// If the vault is liquidatable, this fn makes liquidator the new borrower.
    /// Liquidator is then responsible to make this position healthy.
    /// Liquidator may choose to wind down the position and take collateral as profit.
    function liquidate() external;
    ```

    This essentially transfers the entire position from the borrower to the liquidator, hence liquidator becomes the new borrower.
2. In the same batch, the liquidator has to make the position healthy, either by depositing more collateral, repaying some debt, or both.
3. The liquidator will likely unwind the position completely for profit, but it isn’t mandated by the protocol.

### Handle external liquidation <a href="#handle-external-liquidation" id="handle-external-liquidation"></a>

Collateral vault is a borrower w.r.t. external lending protocol. In case, the liquidation doesn’t happen on Twyne and the external debt position keeps deteriorating, the external lending protocol will liquidate the collateral vault.

Once this happens, the following function needs to be called to process this external liquidation on that collateral vault:

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

In a nutshell, this process unwinds the entire position, distributes the collateral asset balance of the collateral vault among relevant parties, and then shuts down.

In some cases, this process may be create some bad debt for the intermediate vault as described as the exception scenario in [credit vault doc](https://twyne-tech-docs.onrender.com/developers/Twyne_Collateral_Vault/Credit_Vault.md#the-exception-scenario).

To settle the bad debt, call `intermediateVault.liquidate(...)` as follows:

```solidity
liquidate(collateral_vault_address, collateral_vault_address, 0, 0);
```
