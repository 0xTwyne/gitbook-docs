# Borrowers

By tapping into other lenders’ idle credit power, borrowers on Twyne can boost the Liquidation Loan-to-Value ratio (LLTV) dictated by the underlying lending market. Borrowers can use this extra line of credit in two main ways:&#x20;

1. Increase their leveraged exposure by looping more
2. Create a safety buffer that protects against liquidations&#x20;

Example: Twyne’s stETH↔ETH vault allows borrowers to boost their Liquidation LTV from 95% up to 98% and squeeze up to 32 loops, compared to a maximum of 14 loops allowed by the underlying lending market (Euler).

***

### Interest Rate

Borrowers on Twyne pay two interest components:

* The base borrow rate from the lending protocol&#x20;
* The CLP Supply rate which is the fee paid to Credit-LPs for providing credit.

The CLP Supply Rate is dynamic and grows with protocol utilization. When demand is lower than Twyne’s utilization target, it translates to low CLP supply rates. When demand is higher than Twyne’s utilization target, it leads to a sharp increase in the CLP supply rate.&#x20;

The CLP supply rate is paid only on the extra credit the borrower reserved, not their total debt. This makes the CLP supply rate a tiny fraction of the typical liquidation costs.

{% hint style="info" %}
Read more details about borrower [Interest Rates](https://twyne.gitbook.io/twyne/interest-rates).
{% endhint %}
