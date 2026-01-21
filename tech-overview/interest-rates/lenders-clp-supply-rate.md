# Lenders CLP Supply Rate

The **CLP Supply Rate** is the interest rate charged to borrowers for accessing the delegated borrowing power of Credit LPs. This rate is based on **utilization** — how much of the available CLP credit is currently reserved.

<div data-full-width="false"><figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption><p>Sample interest rate curves with various variables</p></figcaption></figure></div>

Twyne uses a **curved interest rate model** defined by:

```
u = CLP / C_total_LP
```

Where:

* `CLP` = amount of delegated credit in use
* `C_total_LP` = total credit capacity provided by all CLPs
* `u` = utilization rate (from 0 to 1)

The interest rate is calculated as:

```
IR(u) = (IR₀ / u₀) · u + (IR_max − (IR₀ / u₀)) · u^γ
```

With:

* `IR₀` = base interest rate at low utilization
* `IR_max` = max interest rate at full utilization
* `u₀` = kink point where the rate curve steepens
* `γ` = curvature factor (controls how sharply the rate ramps past the kink)

**Example Curve Parameters:**

* `IR₀ = 10%`, `u₀ = 0.8`, `IR_max = 120%`
* `γ` varies from 1.5 to 3+

***

#### 📈 Model Behavior

* **Below `u₀`**: Interest rate increases almost linearly
* **Above `u₀`**: Rate ramps steeply toward `IR_max`
* Designed to be gas-efficient and easy to compute on-chain
* Adapted from Keom Protocol's curved IR model

This flexible design achieves similar effects to "kinked" models (like Compound) but avoids complex on-chain logic like PID controllers.
