# Interest Rate Variables

These control how much borrowers pay for using CLP credit and how much CLPs earn.

## Utilization Rate <a href="#utilization-rate" id="utilization-rate"></a>

| Context           | Notation                                                        | Description                                                     |
| ----------------- | --------------------------------------------------------------- | --------------------------------------------------------------- |
| **Whitepaper**    | `u`                                                             | Credit utilization ratio                                        |
| **Litepaper**     | `u`                                                             | Fraction of CLP funds in use                                    |
| **Solidity**      | NA                                                              | Tracked by credit vault using total borrowed and available cash |
| **Plain English** | What percentage of available CLP funds are currently being used |                                                                 |

## Interest Rate Function <a href="#interest-rate-function" id="interest-rate-function"></a>

| Context           | Notation                                                         | Description                             |
| ----------------- | ---------------------------------------------------------------- | --------------------------------------- |
| **Whitepaper**    | `IR(u)`                                                          | Curved interest rate model              |
| **Litepaper**     | `IR(u)`                                                          | Simplified presentation                 |
| **Solidity**      | `computeInterestRate()`                                          | Implemented in `IRMTwyneCurve` contract |
| **Plain English** | The function that determines interest rates based on utilization |                                         |

## Siphoning Rate <a href="#siphoning-rate" id="siphoning-rate"></a>

| Context           | Notation                                              | Description                                                     |
| ----------------- | ----------------------------------------------------- | --------------------------------------------------------------- |
| **Whitepaper**    | `r_C`                                                 | Rate at which collateral is “siphoned” to CLPs                  |
| **Litepaper**     | `r_C`                                                 | Borrower’s interest payment rate                                |
| **Solidity**      | N/A                                                   | Siphoning rate is handled by credit vault's interest rate curve |
| **Plain English** | The interest rate borrowers pay on reserved CLP funds |                                                                 |
