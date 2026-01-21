# Borrower

Borrowers will find Twyne useful for a variety of reasons. They may want to borrow more than other protocols allow, or they may want to borrow the same amount but with more protection against losses from liquidations. A collateral vault holds the borrowers position and functions similarly to a smart account. A borrower can have multiple collateral vaults, but a collateral vault only holds 1 type of collateral and has 1 borrower (owner) at any given time. If a collateral vault is liquidated, the liquidator becomes the new owner of the collateral vault.

## Actions <a href="#function-calls" id="function-calls"></a>

A borrower must first create a collateral vault. Once the vault exists, a borrower sets 3 parameters to control their position: the amount of collateral in the collateral vault, the amount of assets borrowed from the underlying protocol, and the user-specified Twyne LTV.

### Creating the vault <a href="#creating-the-vault" id="creating-the-vault"></a>

A borrower must call the `createCollateralVault(...)` function of the CollateralVaultFactory in order to create a collateral vault. The call can happen through the EVC (inside of an [EVC batch](https://github.com/euler-xyz/ethereum-vault-connector/blob/master/docs/whitepaper.md#batch)) or with a direct call to the factory. The collateral asset, borrowed asset, and Twyne LTV should all be specified at the time of vault creation.

```solidity
/// @notice This function is called when a borrower wants to deploy a new collateral vault.
/// @param _asset address of vault asset
/// @param _targetVault address of the target vault, used for the lookup of the beacon proxy implementation contract
/// @param _liqLTV user-specified target LTV
/// @return vault address of the newly created collateral vault
function createCollateralVault(address _asset, address _targetVault, uint _liqLTV)
    external
    returns (address vault);
```

### Collateral amount <a href="#collateral-amount" id="collateral-amount"></a>

The borrower can add collateral with the following functions:

```solidity
/// @notice Deposits a certain amount of collateral asset
/// @param assets The assets to deposit.
function deposit(uint assets) external;

/// @notice Deposits a certain amount of underlying asset
/// @param underlying The underlying assets to deposit.
function depositUnderlying(uint underlying) external;

/// @notice allow users of the underlying protocol to seamlessly transfer their position to this vault
function teleport(uint toDeposit, uint toBorrow) external;
```

Only the collateral vault owner, the borrower, can call these functions.&#x20;

* `deposit(...)` allows the borrower to deposit the collateral vault asset (such as aWETH or eWETH) into the vault.
* `depositUnderlying(...)` allows the borrower to deposit WETH directly to the vault, bypassing the need to deposit into another protocol first.
* `teleport(uint,uint)` is used to allow quick transfer of a position from another protocol that includes an existing borrow, without requiring the borrower to unwind their borrow. These actions use the ERC20 `transferFrom()` function call, so an approval or Permit2 signature is necessary for them to function properly.

The borrower can remove collateral, as long as it is not being used for a borrow, with:

```solidity
/// @notice Withdraws a certain amount of assets for a receiver.
/// @param assets Amount of collateral assets to withdraw.
/// @param receiver The receiver of the withdrawal.
function withdraw(uint assets, address receiver) external;

/// @notice Withdraw a certain amount of collateral and transfers collateral asset's underlying asset to receiver.
/// @param assets Amount of collateral asset to withdraw.
/// @param receiver The receiver of the redemption.
/// @return underlying Amount of underlying asset transferred.
function redeemUnderlying(
    uint assets,
    address receiver
) external returns (uint underlying) {
```

* `withdraw(...)` does the reverse of `deposit(...)`, returning the collateral vault asset (such as aWETH or eWETH) from the vault.&#x20;
* `redeemUnderlying(...)` does the reverse of `depositUnderlying(...)` and returns the base asset (such as WETH).

### Borrowed amount <a href="#borrowed-amount" id="borrowed-amount"></a>

The borrower can increase or decrease their borrowed assets with:

```solidity
/// @notice Borrows target assets from the external lending protocol
/// @dev This function calls the internal _borrow function to handle the protocol-specific borrow logic,
/// then transfers the target asset from the vault to _receiver.
/// @param _targetAmount The amount of target asset to borrow
/// @param _receiver The receiver of the borrowed assets
function borrow(uint _targetAmount, address _receiver) external;

/// @notice Repays debt owed to the external lending protocol
/// @dev If _amount is set to type(uint).max, the entire debt will be repaid
/// @dev This function transfers the target asset from the caller to the vault, then
/// calls the internal _repay function to handle the protocol-specific repayment logic
/// @dev Reverts if attempting to repay more than the current debt
/// @param _amount The amount of target asset to repay, or type(uint).max for full repayment
function repay(uint _amount) external;
```

### Twyne Liquidation LTV <a href="#twyne-ltv" id="twyne-ltv"></a>

Borrower can set collateral vault's liquidation LTV using:

```solidity
/// @notice allow the user to set their own vault's liquidation LTV
function setTwyneLiqLTV(uint _ltv) external;
```

The LTV value must stay within the bounds required by the `checkLiqLTV(...)` function of VaultManager.
