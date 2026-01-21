# Credit Vault

The main user-facing purpose of the credit vault is to handle all the interactions with credit LPs. The credit vault code is unmodified Euler EVault based on [EVK](https://github.com/euler-xyz/euler-vault-kit).

Credit LPs deposit their assets for additional yield. Additionally, this contract:

* Lends these assets to the collateral vaults. In Twyne terminology, this action is called _extending credit_.
* Maintains an interest rate curve according to which it charges interest from the collateral vaults.

## Function Calls <a href="#function-calls" id="function-calls"></a>

This is a standard ERC-4626 vault, hence credit LPs deposit and withdraw their assets in a familiar fashion:

### Deposit <a href="#deposit" id="deposit"></a>

```solidity
/// @notice Transfer requested amount of underlying tokens from sender to the vault pool in return for shares
/// @param amount Amount of assets to deposit (use max uint256 for full underlying token balance)
/// @param receiver An account to receive the shares
/// @return Amount of shares minted
/// @dev Deposit will round down the amount of assets that are converted to shares. To prevent losses consider using
/// mint instead.
function deposit(uint256 amount, address receiver) external returns (uint256);

/// @notice Transfer underlying tokens from sender to the vault pool in return for requested amount of shares
/// @param amount Amount of shares to be minted
/// @param receiver An account to receive the shares
/// @return Amount of assets deposited
function mint(uint256 amount, address receiver) external returns (uint256);
```

### Withdraw <a href="#withdraw" id="withdraw"></a>

```solidity
/// @notice Transfer requested amount of underlying tokens from the vault and decrease account's shares balance
/// @param amount Amount of assets to withdraw
/// @param receiver Account to receive the withdrawn assets
/// @param owner Account holding the shares to burn
/// @return Amount of shares burned
function withdraw(uint256 amount, address receiver, address owner) external returns (uint256);

/// @notice Burn requested shares and transfer corresponding underlying tokens from the vault to the receiver
/// @param amount Amount of shares to burn (use max uint256 to burn full owner balance)
/// @param receiver Account to receive the withdrawn assets
/// @param owner Account holding the shares to burn.
/// @return Amount of assets transferred
function redeem(uint256 amount, address receiver, address owner) external returns (uint256);
```

### Liquidate <a href="#liquidate" id="liquidate"></a>

A credit vault cannot be liquidated except in _one scenario_. This is possible to achieve since it lends only to collateral contracts in the Twyne system. The responsibility of maintaining the health of this debt position falls on the collateral vaults.

#### **The exception scenario**

As with any lending market, there is a possibility of bad debt creation. If a collateral vault isn’t liquidated in time, it be liquidated by the external lending protocol.

After handling this scenario on the collateral vault, there may be some leftover bad debt. Only in this case, `liquidate(...)` function can be called to settle that bad debt.

```solidity
/// @notice Attempts to perform a liquidation
/// @param violator Address that may be in collateral violation
/// @param collateral Collateral which is to be seized
/// @param repayAssets The amount of underlying debt to be transferred from violator to sender, in asset units (use
/// max uint256 to repay the maximum possible amount). Meant as slippage check together with `minYieldBalance`
/// @param minYieldBalance The minimum acceptable amount of collateral to be transferred from violator to sender, in
/// collateral balance units (shares for vaults).  Meant as slippage check together with `repayAssets`
/// @dev If `repayAssets` is set to max uint256 it is assumed the caller will perform their own slippage checks to
/// make sure they are not taking on too much debt. This option is mainly meant for smart contract liquidators
function liquidate(address violator, address collateral, uint256 repayAssets, uint256 minYieldBalance) external;
```

To settle the bad debt, it needs to called as follows:

```solidity
liquidate(collateral_vault_address, collateral_vault_address, 0, 0);
```

### Interest accrual <a href="#interest-accrual" id="interest-accrual"></a>

Each credit vault follows an interest rate model to charge interest for all the credit extended to collateral vaults. This is explained in [interest rate model](irm.md) section.

### Security <a href="#security" id="security"></a>

The Twyne Credit Vault uses the exact same code as the EVK vaults on Euler Finance. This means that the Twyne Credit Vaults inherit the security that Euler invested into the design, which was [roughly $4 million](https://www.euler.finance/blog/securing-euler). Euler’s docs have [much more information](https://docs.euler.finance/developers/core/euler-vault-kit/whitepaper) about their EVK code.
