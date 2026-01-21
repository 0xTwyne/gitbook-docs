# Borrowers Interest Rate

### 1. **Borrower’s Siphoning Rate** (`r_C`)

This is the **gross rate** paid by the borrower to CLPs, calculated as:

```
r_C = (CLP · IR(u)) / C
```

Where:

* `C` = borrower’s total collateral

Borrowers also continue to earn any passive yield from the base protocol on their collateral, minus this siphoning rate (where applicable).

***

### 2. **Net Rate Paid by Borrower** (`r_net_C`)

This adjusts the gross rate to account for the borrower’s active debt:

```
r_net_C = r_C / (1 − λ_twyne)
```

Where:

* `λ_twyne` = borrower’s Loan-to-Value on Twyne
* `B` = borrower’s outstanding borrow
* `C − B` = borrower’s equity in the position

This formula ensures borrowers pay proportionally more as their position gets riskier — aligning incentives between CLPs and borrowers.
