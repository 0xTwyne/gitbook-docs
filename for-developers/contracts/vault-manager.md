# Vault Manager

The Twyne Vault Manager manages global Twyne parameters, including: allowed intermediate vaults, allowed liquidation LTVs, addresses of key components (such as the oracle router), safety buffer (aka external liquidation buffer) and other governance configuration parameters. This contract is owned by the Twyne multisig.

There are no user-facing functions in this contract, all state-modifying functions are admin controlled. Some view functions may be useful for protocols or bots integrating with Twyne.

## Function Calls <a href="#function-calls" id="function-calls"></a>

### Set oracle <a href="#set-oracle" id="set-oracle"></a>

Sets the oracle router used by Twyne collateral vaults.

```solidity
/// @notice Set oracleRouter address. Governance-only.
function setOracleRouter(address _oracle) external onlyOwner;
```

### Whitelist intermediate vault <a href="#whitelist-intermediate-vault" id="whitelist-intermediate-vault"></a>

Intermediate vaults are deployed by Twyne governance. They are registered with the Vault Manager via:

```solidity
/// @notice Register or unregister an intermediate vault. Governance-only.
/// @param _intermediateVault address of the intermediate vault.
/// @param _value true to register, false to unregister.
function setIntermediateVault(IEVault _intermediateVault, bool _value) external onlyOwner;
```

Only whitelisted intermediate vaults can be used when creating collateral vaults.

### Whitelist target vault <a href="#whitelist-target-vault" id="whitelist-target-vault"></a>

Every collateral vault is tied to a single intermediate vault and a single target (external) vault. Governance maintains an allowlist of `(intermediate, target)` pairs:

```solidity
/// @notice Allow a target vault to be used with a specific intermediate vault. Governance-only.
function setAllowedTargetVault(address _intermediateVault, address _targetVault) external onlyOwner;

/// @notice Remove an existing target vault from the allowlist. Governance-only.
/// @param _index The position of _targetVault inside allowedTargetVaultList[_intermediateVault].
function removeAllowedTargetVault(address _intermediateVault, address _targetVault, uint _index) external onlyOwner;
```

### Whitelist target asset (Aave-style targets) <a href="#whitelist-target-asset" id="whitelist-target-asset"></a>

For integrations where a single target vault can be used to borrow multiple assets (e.g., the Aave V3 Pool), governance additionally whitelists each borrowable asset:

```solidity
/// @notice Allow a target asset to be borrowed from a target vault. Governance-only.
function setAllowedTargetAsset(
    address _intermediateVault,
    address _targetVault,
    address _targetAsset
) external onlyOwner;
```

`CollateralVaultFactory.createCollateralVault` uses `isAllowedTargetAssets` to gate Aave V3 collateral vault creation.

### Set maximum liquidation LTV <a href="#set-maximum-liquidation-ltv" id="set-maximum-liquidation-ltv"></a>

Every borrower picks a liquidation LTV for their collateral vault. The protocol-wide upper bound is set per-intermediate-vault:

```solidity
/// @notice Set maxTwyneLiqLTV for an intermediate vault with an optional linear ramp-down. Governance-only.
/// @param _intermediateVault address of the intermediate vault.
/// @param _ltv new target maxTwyneLiqLTV (1e4 precision).
/// @param _rampDuration ramp duration in seconds. 0 for immediate update. If > 0, `_ltv` must be strictly lower
///        than the current effective value; the current value is snapshotted as the ramp start.
function setMaxLiquidationLTV(address _intermediateVault, uint16 _ltv, uint32 _rampDuration) external onlyOwner;
```

The current effective value is read via:

```solidity
function maxTwyneLTVs(address _intermediateVault) external view returns (uint16);
```

### Set external liquidation buffer <a href="#set-external-liquidation-buffer" id="set-external-liquidation-buffer"></a>

The lower bound of the user-chosen liquidation LTV is `externalLiqLTV * externalLiqBuffer`, where `externalLiqBuffer ∈ (0, 1e4]` provides the safety margin below the underlying protocol's liquidation threshold. It too supports linear ramp-down:

```solidity
/// @notice Set externalLiqBuffer for an intermediate vault with an optional linear ramp-down. Governance-only.
function setExternalLiqBuffer(address _intermediateVault, uint16 _liqBuffer, uint32 _rampDuration) external onlyOwner;

function externalLiqBuffers(address _intermediateVault) external view returns (uint16);
```

Ramp metadata (start value, target, duration) is exposed via `maxTwyneLTVFull(...)` and `externalLiqBufferFull(...)` for monitoring.

### Set LTV parameters on the intermediate vault <a href="#set-ltv-parameters-for-intermediate-vaults" id="set-ltv-parameters-for-intermediate-vaults"></a>

At creation time, `CollateralVaultFactory` configures the newly deployed collateral vault as a collateral inside its intermediate vault. This goes through the Vault Manager:

```solidity
/// @notice Set new LTV values for an intermediate vault by calling EVK.setLTV().
///         Callable by governance or the collateral vault factory.
function setLTV(
    IEVault _intermediateVault,
    address _collateralVault,
    uint16 _borrowLimit,
    uint16 _liquidationLimit,
    uint32 _rampDuration
) external onlyCollateralVaultFactoryOrOwner;
```

The factory always calls this with `(1e4, 1e4, 0)`. That is safe because the intermediate vault is not permitted to liquidate the collateral vault during normal operation — `IEVault.liquidate(...)` is only enabled as a bad-debt-settlement step after an external liquidation has been handled.

### Set resolved vault on the oracle router <a href="#set-resolved-vault" id="set-resolved-vault"></a>

Each collateral vault must be priced by the oracle router. The factory configures this via:

```solidity
function setOracleResolvedVault(address _vault, bool _allow) external onlyCollateralVaultFactoryOrOwner;

/// @notice Variant used for Aave V3 collateral vaults, where a different oracle router may be wired up.
function setOracleResolvedVaultForOracleRouter(
    address _oracleRouter,
    address _vault,
    bool _allow
) external onlyCollateralVaultFactoryOrOwner;
```

### Set collateral vault factory <a href="#set-collateral-vault-factory" id="set-collateral-vault-factory"></a>

```solidity
function setCollateralVaultFactory(address _factory) external onlyOwner;
```

### Arbitrary external call <a href="#arbitrary-external-call" id="arbitrary-external-call"></a>

`VaultManager` is the owner/admin of many contracts in the Twyne system. To avoid having to ship a new manager for every one-off admin operation it exposes:

```solidity
/// @notice Perform an arbitrary external call. Governance-only.
function doCall(address to, uint value, bytes memory data) external payable onlyOwner;
```
