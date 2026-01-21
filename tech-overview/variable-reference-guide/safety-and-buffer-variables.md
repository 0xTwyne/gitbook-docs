# Safety and Buffer Variables

These parameters ensure Twyne triggers liquidations before the external protocol, protecting Credit LPs from losses.

## Safety Buffer <a href="#safety-buffer" id="safety-buffer"></a>

| Context           | Notation                                                                   | Description                              |
| ----------------- | -------------------------------------------------------------------------- | ---------------------------------------- |
| **Whitepaper**    | $$\beta_{safe}$$                                                           | Safety buffer below external liquidation |
| **Litepaper**     | $$\beta_{safe}$$                                                           | Safety buffer below external liquidation |
| **Solidity**      | `externalLiqBuffer`                                                        | Typically 0.95-0.99 (95%-99%)            |
| **Plain English** | How much “breathing room” to maintain below external liquidation threshold |                                          |

## Dynamic Safety Buffer (coming soon) <a href="#dynamic-safety-buffer" id="dynamic-safety-buffer"></a>

| Context           | Notation                                                       | Description                                |
| ----------------- | -------------------------------------------------------------- | ------------------------------------------ |
| **Whitepaper**    | $$\beta^{dyn}_{safe}$$                                         | User-specific dynamic safety buffer        |
| **Litepaper**     | Not covered                                                    | Advanced topic                             |
| **Solidity**      | Not implemented                                                | Adjusts based on user’s risk level         |
| **Plain English** | An adaptive safety margin that increases for riskier positions | Currently NOT implemented in Solidity code |

**Formula**: $$\beta^{dyn}_{safe} = max(\rho, \beta_{safe})$$ where $$\rho = \tilde{\lambda}_t / \tilde{\lambda}^{max}_t$$.
