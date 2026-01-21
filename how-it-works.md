# How It Works

Whenever a user supplies to a lending market, they’re issued an `IOU token` (aETH, eUSDC etc) that represents their position. These tokens generate yield and unlock borrowing capacity.&#x20;

Enter Twyne: it builds on top of established lending markets like Aave. Twyne simply moves IOU tokens around to use them more efficiently. User capital never leaves the underlying lending market, allowing users to earn the underlying interest while also benefiting from Twyne.

Lenders (Credit-LPs) deposit their IOU tokens into a shared Credit Vault to delegate unused borrowing power and earn extra yield. Borrowers deposit their own IOU tokens into a Collateral Vault, giving them temporary access to that delegated borrowing power.

<figure><picture><source srcset=".gitbook/assets/twyne_overview_dark (3).svg" media="(prefers-color-scheme: dark)"><img src=".gitbook/assets/twyne_overview_light (1).svg" alt=""></picture><figcaption><p>How users interact with Aave through Twyne</p></figcaption></figure>

Users can interact with Twyne in two ways: as lenders or borrowers. Below is the typical user flow for each:<br>

1. **Lender (Credit LP)**&#x20;

* Deposit IOU tokens from the underlying lending market to Twyne’s Credit Vault
* Delegate borrowing capacity
* Earn delegation yield on top of the base lending APY<br>

2. &#x20;**Borrower**&#x20;

* Deposit IOU tokens from the underlying lending market to Twyne’s Borrow Vault
* Pull credit from the Credit Vault to increase borrowing capacity
* Pay an interest rate for the delegated credit&#x20;

***

#### Pooled Credit, Isolated Borrowing

Twyne operates pooled Credit Vaults, allowing Credit-LPs to deposit and withdraw as long as liquidity is available. Any additional risk introduced by Twyne is fully contained within this layer.

If losses ever occur, Credit-LPs absorb them, while borrowers remain isolated from one another and retain the solvency guarantees of the underlying lending market. The underlying base layer remains fully isolated, while risk-on users interact with a separate sandbox to earn or borrow more.
