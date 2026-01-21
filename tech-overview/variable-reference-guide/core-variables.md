# Core variables

These variables define the fundamental state of any position in Twyne. Think of them as the “atoms” from which all other calculations are built.

## Borrowed Amount (External Debt) <a href="#borrowed-amount-external-debt" id="borrowed-amount-external-debt"></a>

| Context           | Notation                                                         | Description                                                                                              |
| ----------------- | ---------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **Whitepaper**    | `B`                                                              | User’s borrowed amount from external lending protocol, in some common unit of account                    |
| **Litepaper**     | `B`                                                              | Outstanding external borrow, in some common unit of account                                              |
| **Solidity**      | `maxRepay()`                                                     | Amount owed to the underlying lending protocol (e.g., Euler), denominated in borrowed asset (e.g., USDC) |
| **Plain English** | The actual debt the borrower owes to the external lending market |                                                                                                          |

**Implementation Note**: This value includes accrued interest.

## Reserved Credit (CLP Assets) <a href="#reserved-credit-clp-assets" id="reserved-credit-clp-assets"></a>

| Context           | Notation                                                                  | Description                                                                                              |
| ----------------- | ------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **Whitepaper**    | `C_LP`                                                                    | Credit reserved from Intermediate Vault, in some common unit of account                                  |
| **Litepaper**     | `C_LP`                                                                    | Outstanding CLP collateral being reserved, in some common unit of account                                |
| **Solidity**      | `maxRelease`                                                              | Amount borrowed from the Credit Vault to boost position, denominated in reserved asset (e.g., eulerWETH) |
| **Plain English** | The extra collateral borrowed from Credit LPs to increase borrowing power |                                                                                                          |

## Total Collateral <a href="#total-collateral" id="total-collateral"></a>

| Context           | Notation                                                                           | Description                                                                                   |
| ----------------- | ---------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Whitepaper**    | `C_total` or `C + C_LP`                                                            | Total collateral as seen by external protocol, in some common unit of account                 |
| **Litepaper**     | Not explicitly defined                                                             | Implicit in calculations                                                                      |
| **Solidity**      | `totalAssetsDepositedOrReserved`                                                   | Sum of user collateral and reserved credit, denominated in collateral asset (e.g., eulerWETH) |
| **Plain English** | The total collateral backing the position from the external protocol’s perspective |                                                                                               |

## Collateral (User’s Own Assets) <a href="#collateral-users-own-assets" id="collateral-users-own-assets"></a>

| Context           | Notation                                                                              | Description                                                                                                              |
| ----------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Whitepaper**    | `C`                                                                                   | User’s collateral deposited in their Collateral Vault, in some common unit of account                                    |
| **Litepaper**     | `C`                                                                                   | Simplified notation matching whitepaper, in some common unit of account                                                  |
| **Solidity**      | <p><code>totalAssetsDepositedOrReserved-maxRelease()</code></p><p>  </p>              | Actual collateral owned by the borrower excluding any reserved credit, denominated in collateral asset (e.g., eulerWETH) |
| **Plain English** | The borrower’s own assets used as collateral, not including borrowed credit from CLPs |                                                                                                                          |
