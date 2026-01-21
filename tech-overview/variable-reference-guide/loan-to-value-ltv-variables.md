# Loan-to-Value (LTV) Variables

These parameters control when positions can be liquidated and how much can be borrowed. Understanding their relationships is crucial for grasping Twyne’s dual-LTV system.

## Twyne LTV (User Perspective) <a href="#twyne-ltv-user-perspective" id="twyne-ltv-user-perspective"></a>

| Context           | Notation                                                           | Description                                                                                            |
| ----------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| **Whitepaper**    | $$\lambda_t$$$$\l$$                                                | Twyne loan-to-value ratio                                                                              |
| **Litepaper**     | $$\lambda_{twyne}$$                                                | Simplified notation                                                                                    |
| **Solidity**      | `twyneLTV`                                                         | <p>Calculated with basis point precision<br><code>1e4 * USD(maxRepay) / USD(userCollateral)</code></p> |
| **Plain English** | The borrower’s effective LTV considering only their own collateral |                                                                                                        |

**Formula**: `λ_t = B / C`

## External LTV (Protocol Perspective) <a href="#external-ltv-protocol-perspective" id="external-ltv-protocol-perspective"></a>

| Context           | Notation                                                  | Description                  |
| ----------------- | --------------------------------------------------------- | ---------------------------- |
| **Whitepaper**    | `λ_e`                                                     | External loan-to-value ratio |
| **Litepaper**     | `λ_ext`                                                   | External protocol’s view     |
| **Solidity**      | Function call to external protocol                        | External protocol’s view     |
| **Plain English** | How the external lending protocol sees the position’s LTV |                              |

**Formula**: `λ_e = B / (C + C_LP)`

## Twyne Liquidation LTV <a href="#twyne-liquidation-ltv" id="twyne-liquidation-ltv"></a>

| Context           | Notation                                                            | Description                                                   |
| ----------------- | ------------------------------------------------------------------- | ------------------------------------------------------------- |
| **Whitepaper**    | $$\tilde{\lambda}_t$$                                               | Twyne liquidation LTV threshold (tilde indicates liquidation) |
| **Litepaper**     | $$\tilde{\lambda}^{liq}_{twyne}$$                                   | User-selected liquidation parameter                           |
| **Solidity**      | `twyneLiqLTV`                                                       | Stored as basis points (e.g., 9300 = 93%)                     |
| **Plain English** | The Twyne LTV at which the position becomes liquidatable internally |                                                               |

**Implementation Note**: Users select this value within protocol-defined bounds.

## Maximum Twyne Liquidation LTV <a href="#maximum-twyne-liquidation-ltv" id="maximum-twyne-liquidation-ltv"></a>

| Context           | Notation                                                         | Description                                |
| ----------------- | ---------------------------------------------------------------- | ------------------------------------------ |
| **Whitepaper**    | $$\tilde{\lambda}^{max}_t$$                                      | Maximum allowed Twyne liquidation LTV      |
| **Litepaper**     | $$\tilde{\lambda}^{max}_t$$                                      | Protocol-enforced upper bound              |
| **Solidity**      | `maxTwyneLiqLTV`                                                 | Stored in VaultManager per collateral type |
| **Plain English** | The highest liquidation LTV a user can select for their position |                                            |

## External Protocol Liquidation LTV <a href="#external-protocol-liquidation-ltv" id="external-protocol-liquidation-ltv"></a>

| Context           | Notation                                                             | Description                               |
| ----------------- | -------------------------------------------------------------------- | ----------------------------------------- |
| **Whitepaper**    | $$\tilde{\lambda}_e$$                                                | External protocol’s liquidation threshold |
| **Litepaper**     | $$\lambda^{liq}_{ext}$$                                              | Underlying protocol’s liquidation LTV     |
| **Solidity**      | `liqLTV_external`                                                    | Retrieved from external protocol          |
| **Plain English** | The LTV at which the external protocol (e.g., Euler) would liquidate |                                           |
