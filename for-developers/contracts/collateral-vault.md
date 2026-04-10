# Collateral Vault

This is the contract that every borrower on Twyne interacts with directly. Each collateral vault has a single borrower, and most state-changing functions are gated on that borrower. The vault is neither ERC20- nor ERC4626-compliant; correctness is enforced through Twyne's invariants (see the [tech overview](../../tech-overview/protocol-invariants/)).

* `AaveV3CollateralVault` — borrows from the Aave V3 Pool using an Aave aToken wrapper (e.g. `aWETHWrapper`) as collateral asset. eMode is configured via `CollateralVaultFactory`.
* `EulerCollateralVault` — borrows from an Euler V2 eVault using the corresponding eToken as collateral asset.

The `CollateralVaultFactory` picks the right implementation based on the `VaultType` passed to `createCollateralVault`.

## Function Calls <a href="#function-calls" id="function-calls"></a>

Borrowers can deposit and withdraw collateral, borrow and repay their debt from the external lending protocol.

### Deposit collateral <a href="#deposit-collateral" id="deposit-collateral"></a>

To deposit the vault's collateral asset (e.g. `aWETHWrapper`, `eWETH`):

```solidity
/// @notice Deposits a certain amount of the vault's collateral asset.
function deposit(uint assets) external;
```

To deposit the underlying of the collateral asset (e.g. WETH) and let the vault handle wrapping:

```solidity
/// @notice Deposits a certain amount of the underlying of the collateral asset.
///         The vault wraps it (through aWETHWrapper / EVK) before crediting the position.
function depositUnderlying(uint underlying) external;
```

There is also a `skim()` hook used by the 1-click leverage flow:

```solidity
/// @notice Reconcile airdropped collateral asset sitting in the vault into the accounted position.
/// @dev The leverage operator deposits into the wrapper with the collateral vault as the recipient,
///      then calls skim() inside the borrower's EVC batch so the balance is recognized.
function skim() external;
```

### Withdraw collateral <a href="#withdraw-collateral" id="withdraw-collateral"></a>

To withdraw the collateral asset:

```solidity
/// @notice Withdraws the collateral asset to a receiver.
/// @param assets Amount of collateral assets to withdraw. Use type(uint).max to withdraw maximum.
/// @param receiver Recipient of the collateral asset.
function withdraw(uint assets, address receiver) external;
```

To withdraw the underlying of the collateral asset (e.g. unwrap `aWETHWrapper` → `WETH` in the same call):

```solidity
/// @notice Withdraws collateral and redeems it into the underlying asset before transferring.
/// @param assets Amount of collateral asset to burn (not the underlying amount).
/// @param receiver Recipient of the underlying.
/// @return underlying Amount of underlying sent to the receiver.
function redeemUnderlying(uint assets, address receiver) external returns (uint underlying);
```

### Borrow / repay target asset <a href="#borrow" id="borrow"></a>

Each collateral vault borrows a single fixed target asset.

```solidity
/// @notice Borrows the target asset from the external lending protocol.
/// @param _targetAmount Amount of target asset to borrow.
/// @param _receiver Address that receives the borrowed asset.
function borrow(uint _targetAmount, address _receiver) external;

/// @notice Repays target asset debt to the external lending protocol.
/// @dev Passing type(uint).max repays everything owed.
function repay(uint _amount) external;
```

After every operation, the vault re-runs `_handleExcessCredit(_invariantCollateralAmount())` to reserve/release credit so the ideal amount of credit is reserved.

### Adjust the liquidation LTV <a href="#twyne-liqltv" id="twyne-liqltv"></a>

```solidity
/// @notice Set this vault's liquidation LTV (1e4 precision).
/// @dev The new LTV must satisfy externalLiqLTV * buffer <= newLTV * 1e4 <= maxTwyneLTV * 1e4
function setTwyneLiqLTV(uint _ltv) external;
```

Raising `twyneLiqLTV` automatically reserves more credit; lowering it releases credit back to the intermediate vault.

### Rebalance <a href="#rebalance" id="rebalance"></a>

Over time interest accrual can leave a vault holding more reserved credit than the invariant requires. Anyone can trigger the release:

```solidity
/// @notice Returns the amount of excess credit that could be released right now. Reverts if none.
function canRebalance() external view returns (uint);

/// @notice Release excess credit back to the intermediate vault.
/// @dev Permissionless — any caller can trigger it.
function rebalance() external;
```

See the [rebalance logic](../../tech-overview/rebalance-logic.md) page for details.

### Liquidate (inheritance) <a href="#liquidate" id="liquidate"></a>

Twyne uses liquidation by inheritance. Liquidation is a multi-step flow that must happen inside a single EVC batch:

1. Call `liquidate()`. If the position is liquidatable the liquidator becomes the new `borrower` of the vault; the previous borrower receives `collateralForBorrower(B, C)` worth of the collateral asset (paid at the end of the batch as part of `checkVaultStatus`). `B`, `C` represent the value of debt and user collateral in some common unit of account.

   ```solidity
   /// @notice Begin liquidation. Must be followed by actions that restore vault health in the same batch.
   function liquidate() external;
   ```
2. Still inside the same batch, restore health by depositing more collateral and/or repaying debt.
3. A just-in-time liquidator will typically repay everything and withdraw the remaining collateral as profit — but the protocol does not force them to unwind.

The actual split between liquidator and liquidated borrower is computed by `collateralForBorrower(B, C)` in `CollateralVaultBase`, following the dynamic-incentive model described in the [liquidation logic](../../tech-overview/liquidation-logic/) pages.

### Handle external liquidation <a href="#handle-external-liquidation" id="handle-external-liquidation"></a>

From the external lending market's point of view, the collateral vault is the borrower. If Twyne liquidators fail to act in time and the external market liquidates the vault, all the operations on the collateral vault become inaccessible except `handleExternalLiquidation()` which accounts for the leftover assets:

```solidity
/// @notice Handles the aftermath of an external liquidation by the underlying lending protocol.
/// @dev Distributes any remaining collateral between the liquidator (caller), the borrower, and the
///      intermediate vault, repays any residual external debt, and shuts the vault down.
/// @dev Caller must in the same EVC batch call
///      `intermediateVault.liquidate(collateral_vault, collateral_vault, 0, 0)`
///      if any bad debt remains afterwards.
function handleExternalLiquidation() external;
```

For Aave V3, `handleExternalLiquidation`:

1. Reconciles the wrapper accounting by burning wrapper shares equal to the amount Aave already seized.
2. Uses Aave's oracles plus Twyne's `maxTwyneLTVs` to split the remaining collateral into three buckets: `C_LP` (returned to the intermediate vault), `borrowerClaim` (returned to the original borrower), and `liquidatorReward` (sent to the liquidator).
3. Pulls target asset from the liquidator to repay any residual Aave debt.
4. Transfers each bucket out and resets vault state.

If after step 4 the intermediate vault still records a debt against this collateral vault (bad debt), the caller must settle it in the same batch with:

```solidity
intermediateVault.liquidate(collateral_vault, collateral_vault, 0, 0);
```
