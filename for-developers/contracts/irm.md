# IRM

The Twyne IRM implements a curved interest rate model used by the credit vaults. This is unlike many other lending markets that use a more simple two-piece linear kink model. In the Twyne IRM, the interest rate growth is smoother but retains a similar design to the two-piece linear kink model. The Twyne IRM contract uses a very similar layout as, and is interchangeable/compatible with, the Euler Finance [IRMLinearKink](https://github.com/euler-xyz/euler-vault-kit/blob/master/src/InterestRateModels/IRMLinearKink.sol) contract.

### Function Calls <a href="#function-calls" id="function-calls"></a>

```solidity
    /// @notice Perform computation of the new interest rate without mutating state
    /// @param vault Address of the vault to compute the new interest rate for
    /// @param cash Amount of assets held directly by the vault
    /// @param borrows Amount of assets lent out to borrowers by the vault
    /// @return Then new interest rate in second percent yield (SPY), scaled by 1e27
    function computeInterestRateView(address vault, uint256 cash, uint256 borrows) external view returns (uint256);
```

Credit vaults queries `computeInterestRateView(...)`  to fetch the current interest rate.

Interest rate is calculated according to the following formula (doesn't show the scaling applied in the contract):

```
(linearParamater * utilization) + (polynomialParameter * utilization^12)
```
