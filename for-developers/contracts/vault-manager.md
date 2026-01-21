# Vault Manager

The Twyne Vault Manager manages global Twyne parameters, including: allowed assets, allowed liquidation LTVs, addresses of key components (such as the oracle router), safety buffer (aka external liquidation buffer) and other governance configuration parameters. This contract will be owned by the Twyne multisig.

There are no user-facing functions in this contract, all state-modifying functions are admin controlled. Some view functions may be useful for protocols or bots integrating with Twyne.

## Function Calls <a href="#function-calls" id="function-calls"></a>

### Set oracle <a href="#set-oracle" id="set-oracle"></a>

Sets the oracle used by Twyne collateral vaults.

```solidity
/// @notice Set oracleRouter address. Governance-only.
function setOracleRouter(address _oracle) external onlyOwner;
```

### Whitelist intermediate vault <a href="#whitelist-intermediate-vault" id="whitelist-intermediate-vault"></a>

Intermediate vaults are deployed by Twyne governance. A new intermediate vault can be added to Twyne’s onchain system via:

```solidity
/// @notice Set a collateral vault's intermediate vault. Governance-only.
/// @param _intermediateVault address of the intermediate vault.
function setIntermediateVault(IEVault _intermediateVault) external onlyOwner;
```

It adds the intermediate vault to this mapping:

```solidity
mapping(address collateralAddress => address intermediateVault) internal intermediateVaults;
```

### Whitelist target vault <a href="#whitelist-target-vault" id="whitelist-target-vault"></a>

Collateral vaults are deployed with an intermediate and target vault. Collateral vaults reserve credit from intermediate vault and borrows from target vault. For each intermediate vault, `VaultManager` maintains a list of target vaults. Thus, a collateral vault can be deployed only when its intermediate and target vault is whitelisted.

```solidity
/// @notice Set an allowed target vault for a specific intermediate vault. Governance-only.
/// @param _intermediateVault address of the intermediate vault.
/// @param _targetVault The target vault that should be allowed for the intermediate vault.
function setAllowedTargetVault(address _intermediateVault, address _targetVault) external onlyOwner;
```

It adds the target vault to this mapping:

```solidity
mapping(address collateralAddress => address intermediateVault) internal intermediateVaults;
```

### Remove target vault from whitelist <a href="#remove-target-vault-from-whitelist" id="remove-target-vault-from-whitelist"></a>

Removes the target vault from the whitelist for a particular intermediate vault. This blocks creation of collateral vaults which uses this specific intermediate vault and target vault pair.

```solidity
/// @notice Remove an allowed target vault for a specific intermediate vault. Governance-only.
/// @param _intermediateVault address of the intermediate vault.
/// @param _targetVault The target vault that should be allowed for the intermediate vault.
/// @param _index The index at which this _targetVault is stored in `allowedTargetVaultList`.
function removeAllowedTargetVault(address _intermediateVault, address _targetVault, uint _index) external onlyOwner;
```

It removes the target vault from this mapping:

```solidity
mapping(address collateralAddress => address intermediateVault) internal intermediateVaults;
```

Note that existing collateral vaults using this pair will continue functioning normally.

### Set maximum liquidation LTV <a href="#set-maximum-liquidation-ltv" id="set-maximum-liquidation-ltv"></a>

Every borrower can set a liquidation LTV for its collateral vault. A range check is performed on this value to ensure Twyne works correctly. The upper bound of this range is set by:

```solidity
/// @notice Set protocol-wide maxTwyneLiqLTV. Governance-only.
/// @param _ltv new maxTwyneLiqLTV value.
function setMaxLiquidationLTV(address _collateralAddress, uint16 _ltv) external onlyOwner;
```

It updates the maximum liquidation LTV stored in this mapping:

```solidity
mapping(address collateralAddress => uint16 maxTwyneLiqLTV) public maxTwyneLTVs;
```

### Set external liquidation buffer <a href="#set-external-liquidation-buffer" id="set-external-liquidation-buffer"></a>

The lower bound on the user-specified liquidation LTV is given by: liqLTVe∗extLiqBuffer

Where liqLTVe is the external lending market’s liquidation LTV, and extLiqBuffer is value between 0 and 1. Realistically, this buffer will vary from 0.9 to 1.

This buffer can be set for each collateral asset via:

```solidity
/// @notice Set protocol-wide externalLiqBuffer. Governance-only.
/// @param _liqBuffer new externalLiqBuffer value.
function setExternalLiqBuffer(address _collateralAddress, uint16 _liqBuffer) external onlyOwner;
```

### Set collateral vault factory <a href="#set-collateral-vault-factory" id="set-collateral-vault-factory"></a>

Anyone can deploy collateral vaults via the collateral vault factory. Twyne governance can change the factory address via:

```solidity
/// @notice Set new collateralVaultFactory address. Governance-only.
/// @param _factory new collateralVaultFactory address.
function setCollateralVaultFactory(address _factory) external;
```

### Arbitrary external call <a href="#arbitrary-external-call" id="arbitrary-external-call"></a>

`VaultManager` can do an arbitrary external call via:

```solidity
/// @notice Perform an arbitrary external call. Governance-only.
/// @dev VaultManager is an owner/admin of many contracts in the Twyne system.
/// @dev This function helps Governance in case a specific a specific external function call was not implemented.
function doCall(address to, uint value, bytes memory data) external payable onlyOwner;
```

Note the actions performed by Twyne governance is still limited by the callee smart contract.

### Set LTV parameters for intermediate vaults <a href="#set-ltv-parameters-for-intermediate-vaults" id="set-ltv-parameters-for-intermediate-vaults"></a>

The collateral vault factory, at the time of collateral vault creation, sets LTV parameters under which the collateral vault can borrow from its intermediate vault:

```solidity
vaultManager.setLTV(IEVault(intermediateVault), vault, 1e4, 1e4, 0);
```

This operation is done via `VaultManager.setLTV(...)`:

```solidity
/// @notice Set new LTV values for an intermediate vault by calling EVK.setLTV(). Callable by governance or collateral vault factory.
/// @param _intermediateVault address of the intermediate vault.
/// @param _collateralVault address of the collateral vault.
/// @param _borrowLimit new borrow LTV.
/// @param _liquidationLimit new liquidation LTV.
/// @param _rampDuration ramp duration in seconds (0 for immediate effect) during which the liquidation LTV will change.
function setLTV(IEVault _intermediateVault, address _collateralVault, uint16 _borrowLimit, uint16 _liquidationLimit, uint32 _rampDuration) external onlyCollateralVaultFactoryOrOwner;
```

### Set resolved vault <a href="#set-resolved-vault" id="set-resolved-vault"></a>

Each collateral vault at the time of its deployment signals the oracle router that it needs to price it:

```solidity
vaultManager.setOracleResolvedVault(vault, true);
```

This operation is done via:

```solidity
/// @notice Set new oracleRouter resolved vault value. Callable by governance or collateral vault factory.
/// @param _vault EVK or collateral vault address. Must implement `convertToAssets()`.
/// @param _allow bool value to pass to govSetResolvedVault. True to configure the vault, false to clear the record.
/// @dev called by createCollateralVault() when a new collateral vault is created so collateral can be price properly.
/// @dev Configures the collateral vault to use internal pricing via `convertToAssets()`.
function setOracleResolvedVault(address _vault, bool _allow) external onlyCollateralVaultFactoryOrOwner;
```
