---
hidden: true
---

# Precision and Implementation Considerations

When working with Twyne’s codebase, keep these implementation details in mind:

#### Basis Points vs Decimals <a href="#basis-points-vs-decimals" id="basis-points-vs-decimals"></a>

* **Theory**: Uses decimals (0.85 for 85%)
* **Solidity**: Uses basis points (8500 for 85%) to avoid precision loss
* **Conversion**: Multiply decimals by 10,000 for basis points

#### Fixed-Point Arithmetic <a href="#fixed-point-arithmetic" id="fixed-point-arithmetic"></a>

* Most calculations use 18 decimal places (1e18 = 1.0)
* This prevents rounding errors in division operations
* Always scale appropriately when interfacing with external protocols

#### Rounding Direction <a href="#rounding-direction" id="rounding-direction"></a>

* **User collateral calculations**: Round down (conservative for protocol)
* **Required credit calculations**: Round up (ensures sufficient backing)
* **Interest calculations**: Follow specific rules to prevent exploitation

## Common Formula Transformations <a href="#common-formula-transformations" id="common-formula-transformations"></a>

The Solidity code often rearranges theoretical formulas for precision. Here are key examples:

#### Credit Reservation Invariant <a href="#credit-reservation-invariant" id="credit-reservation-invariant"></a>

**Theoretical** (Whitepaper Equation 7):

```
C_LP = C * (λ̃_t / (β_safe * λ̃_e) - 1)
```

**Solidity Implementation**:

```solidity
requiredCredit = userCollateral * twyneLiqLTV / (safetyBuffer * externalLiqLTV - twyneLiqLTV)
```

The rearrangement avoids intermediate division that could lose precision.

#### Excess Credit Calculation <a href="#excess-credit-calculation" id="excess-credit-calculation"></a>

**Theoretical**:

```
C_excess = C_LP - C_ideal_LP
```

**Solidity Implementation**:

```solidity
excessCredit = totalAssets > requiredTotal ? totalAssets - requiredTotal : 0
```

The ternary operator prevents underflow while maintaining clarity.

### Using This Guide <a href="#using-this-guide" id="using-this-guide"></a>

When reading Twyne documentation or code:

1. **Start with the concept** - Understand what the variable represents
2. **Check the context** - Different documents use different notation
3. **Note the units** - Basis points vs decimals, asset units vs USD
4. **Consider precision** - Implementation may rearrange formulas
5. **Follow the relationships** - Many variables are derived from others

This guide will be updated as new variables are introduced or naming conventions evolve. When in doubt, the Solidity implementation is the source of truth for how calculations are performed on-chain.
