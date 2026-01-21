# Credit LP

The credit LP deposits their assets from underlying lending protocols (aTokens from Aave, eTokens from Euler, etc.) into Twyne to receive extra yield. These tokens are then borrowed by Twyne borrowers, who pay the credit LPs yield.

### Actions <a href="#function-calls" id="function-calls"></a>

The two main actions for credit LPs are:

```solidity
function deposit(uint256 amount, address receiver) external returns (uint256);
function withdraw(uint256 amount, address receiver, address owner) external returns (uint256);
```

All the standard ERC4626 functions are provided. Since the credit vault is using the same code as [Euler’s EVK](https://github.com/euler-xyz/euler-vault-kit), you can reference the EVK codebase to understand additional functions that are available.&#x20;

Certain EVK functions have hooks added to modify how they work - for example, the liquidate function in the EVK can ONLY be called on a Twyne collateral vault under certain conditions (the collateral vault has been externally liquidated).
