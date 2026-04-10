# Borrower

Borrowers will find Twyne useful for a variety of reasons. They may want to borrow more than other protocols allow, or they may want to borrow the same amount but with more protection against losses from liquidations. A collateral vault holds the borrowers position and functions similarly to a smart account. A borrower can have multiple collateral vaults, but a collateral vault only holds 1 type of collateral and has 1 borrower (owner) at any given time. If a collateral vault is liquidated, the liquidator becomes the new owner of the collateral vault.

## Actions <a href="#function-calls" id="function-calls"></a>

A borrower must first create a collateral vault. Once the vault exists, a borrower sets 3 parameters to control their position: the amount of collateral in the collateral vault, the amount of assets borrowed from the underlying protocol, and the user-specified Twyne LTV.

### Creating the vault <a href="#creating-the-vault" id="creating-the-vault"></a>

A borrower calls `createCollateralVault(...)` on the `CollateralVaultFactory` to deploy a new collateral vault. The call can happen directly or inside an [EVC batch](https://github.com/euler-xyz/ethereum-vault-connector/blob/master/docs/whitepaper.md#batch) — the latter lets you bundle vault creation with the first deposit and borrow.

```solidity
enum VaultType { EULER_V2, AAVE_V3 }

/// @notice Deploy a new collateral vault.
/// @param _vaultType EULER_V2 or AAVE_V3
/// @param _intermediateVault address of the intermediate vault the new collateral vault will reserve credit from
/// @param _targetVault address of the external lending protocol's target (e.g., the Aave V3 Pool)
/// @param _liqLTV borrower's chosen liquidation LTV in 1e4 precision
/// @param _targetAsset asset to borrow from the target vault (e.g., USDC). Ignored for EULER_V2 but must be passed.
/// @return vault address of the newly created collateral vault
function createCollateralVault(
    VaultType _vaultType,
    address _intermediateVault,
    address _targetVault,
    uint _liqLTV,
    address _targetAsset
) external returns (address vault);
```

### Collateral amount <a href="#collateral-amount" id="collateral-amount"></a>

Only the current `borrower` of the collateral vault can add or remove collateral. For Aave V3 vaults the collateral asset is an ERC20 that wraps Aave `aTokens`; for Euler vaults it is an `eToken`. Vaults also accepts the raw underlying (e.g. WETH) through `depositUnderlying`.

```solidity
/// @notice Deposits the collateral vault's asset (the wrapped aToken / eToken).
function deposit(uint assets) external;

/// @notice Deposits the underlying of the collateral asset (e.g., WETH). The vault wraps it before storing.
function depositUnderlying(uint underlying) external;

/// @notice After an airdrop of the collateral asset, skim the idle balance into the accounted position.
///         Mainly used as the last step of a 1-click leverage batch.
function skim() external;
```

* `deposit(...)` pulls the already-wrapped collateral asset (e.g. `aWETHWrapper` or `eWETH`) from the borrower.
* `depositUnderlying(...)` pulls the bare underlying (e.g. `WETH`) and wraps it inside the vault.
* `skim()` is what leverage operators rely on: they airdrop wrapper shares to the collateral vault after supplying the flash-loaned amount to the external market, then call `skim()` inside the borrower's EVC batch so the airdropped balance is added to `totalAssetsDepositedOrReserved`.

The borrower can remove collateral, as long as it doesn't make the vault liquidatable, with:

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
