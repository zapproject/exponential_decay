# Zap v3: Bonding Curve with Tax-Reinforced Stability

**White Paper**

**Author:** Nick Spanos

**Date:** November 03, 2025

## Abstract

Zap v3 introduces a bonding curve mechanism enhanced by an exponential decay exit tax. When tokens are sold, 99% of the exit tax simulates buying additional tokens into the reserve without delivering them to any holder, effectively raising the price floor as if those tokens were purchased but left locked in the contract. This boosts the reserve pool, creates a self-reinforcing stability loop, and aligns incentives for long-term holding. The remaining 1% of the tax is burned for deflationary pressure. The tax decays from 99% to near 0% over 100 days, transitioning to organic demand-driven growth.

## 1. Introduction

Traditional bonding curves in token launches suffer from high volatility and dump risks due to early speculation. Zap v3 addresses this by reinvesting exit taxes in a way that simulates additional buys, increasing reserves without expanding circulating supply. This transforms sell pressure into price support, fostering sustainable growth.

## 2. Mechanism Design

### 2.1 Bonding Curve Basics

The base bonding curve uses an exponential price function:

\[
P(s) = 0.0001 \times e^{0.0001s}
\]

Where \(P(s)\) is the price at supply \(s\), ensuring increasing prices with supply growth.

### 2.2 Exit Tax and Simulated Reinvestment

Upon selling \(x\) tokens held for \(t\) days, an exit tax \(T(t)\) is applied. The total tax collected is \(x \times P(n) \times T(t)\), where \(P(n)\) is the current price at total supply \(n\).

- **Tax Decay Formula**:
  \[
  T(t) = 0.99 \times e^{-0.04652t} \quad \text{(Decays from 99% at t=0 to ~0.94% at t=100 days)}
  \]

  **Derivation**: Starts at 99% (t=0: \(e^0=1\), so \(0.99 \times 1 = 0.99\)). The constant 0.04652 solves \(e^{-k \times 100} \approx 0.0095\) for \(k\), where the threshold is ~1% of initial, ensuring asymptotic decay to near 0% over 100 days.

  **Half-Life**: \(t_{half} = \ln(2)/0.04652 \approx 14.9\) days. Tax halves every ~15 days (e.g., 99% → ~49.5% at day 15, ~24.75% at day 30).

- **Reinvestment Breakdown**: 99% of the tax simulates buying additional tokens:
  \[
  \Delta R = (x \times P(n) \times T(t)) \times 0.99
  \]
  This \(\Delta R\) is added to the reserve as if buying tokens at current price, raising \(P(n)\) without minting or delivering new tokens—they remain locked in the contract, preventing supply inflation.

  **Burned Amount**: 
  \[
  \text{Burned} = (x \times P(n) \times T(t)) \times 0.01
  \]
  This provides deflationary pressure by permanently removing tokens.

### 2.3 Updated Bonding Curve with Simulated Reinvestment

The price is:
\[
P(n) = \frac{R(n)}{n}
\]
Where:
\[
R(n) = \int_0^n P(s) \, ds + \sum \Delta R
\]

- \(R(n)\): Total reserve (initial liquidity + cumulative simulated buys from taxes).
- \(\sum \Delta R\): Sum of reinvested taxes from all sales, simulating buys that boost reserves without supply increase.

This creates a feedback loop: higher reserves → higher \(P(n)\) → larger future taxes → further reserve growth.

## 3. Key Features

### 3.1 Price Stability

Early sales trigger high taxes, with 99% simulating buys that counteract sell pressure by inflating reserves and raising the price floor.

### 3.2 Positive Feedback Loop

High initial taxes (99%) boost reserves exponentially, turning speculation into compounded growth.

### 3.3 Long-Term Growth

As tax decays to 0%, the system relies on organic demand, with early reinvestments providing a permanent price uplift.

### 3.4 Example Scenario

- **Day 1**: Alice sells 100,000 tokens at \(P(n) = \$0.0025\):
  - Tax: \(T(1) \approx 94.5\%\).
  - Total tax: \(100,000 \times 0.0025 \times 0.945 = \$236.25\).
  - Simulated reinvestment: \(\$236.25 \times 0.99 = \$233.89\) added to reserve, raising price as if ~93,556 tokens were bought and locked.
  - Burned: \(\$236.25 \times 0.01 = \$2.36\) (equivalent tokens burned).
  - Effect: Reserve increases, bumping \(P(n)\) for next buyers.

- **Day 30**: Cumulative simulated buys from repeated sales push \(P(n)\) above original curve levels.

## 4. Mathematical Proof of Stability

### 4.1 Reserve Growth Equation

After \(m\) sales:
\[
R(n) = \underbrace{\int_0^n 0.0001 \times e^{0.0001s} \, ds}_{\text{Original Reserve}} + \underbrace{\sum_{i=1}^m (x_i \times P(n_i) \times T(t_i) \times 0.99)}_{\text{Simulated Buys from Taxes}}
\]

- Result: \(R(n)\) grows super-linearly, steepening the curve as taxes compound.

### 4.2 Price Impact

Simulated reinvestments increase reserves faster than supply:
\[
P_{\text{new}}(n) = \frac{R(n)}{n} > P_{\text{original}}(n)
\]

- **Expansion**: \(\frac{dP}{dn} = \frac{\frac{dR}{dn} - P(n)}{n} > 0\) since \(\frac{dR}{dn} > P(n)\) from taxes. In limit, \(P(n) \sim k n^\alpha\) with \(\alpha > 0\), steeper than linear.

- **Example**: At \(n = 50,000\), \(P_{\text{original}} = \$0.82\); with reinvestments, \(P_{\text{new}}\) could reach \$1.20.

### 4.3 Impact on Reserve Dynamics

Assuming exponential base \(P(s) = c \times e^{d s}\), simulated buys create: higher \(R(n)\) → higher \(P(n)\) → larger taxes → exponential growth beyond baseline.

### 4.4 Simulation Insight

Early high-volume sells (small \(t\), large \(T(t)\)): \(\Delta R\) dominates. E.g., 10% supply sold at t=1: \(\Delta R \approx 0.99 \times (0.1 n P(n) \times 0.945) \approx 0.093 n P(n)\), adding ~9.3% to \(R(n)\), increasing \(P(n)\) by ~9.3%. Later, growth shifts to buys.

### 4.5 Edge Cases

- \(t \to \infty\): \(T(t) \to 0\), reverts to pure bonding curve.
- No sales: \(R(n) =\) original integral.
- Rapid dumps: Taxes recapture value, simulating buys that minimize supply impact while inflating reserves, making dumps net positive.

## 5. Tokenomics

| Parameter          | Value              | Effect                          |
|--------------------|--------------------|---------------------------------|
| Initial Tax       | 99%                | Maximizes early reserve boosts |
| Tax Decay Rate    | 0.04652/day        | Smooth decay to 0% over 100 days |
| Burn Rate         | 1% of tax          | Deflationary pressure          |
| Reserve Growth    | 99% of tax (simulated buys) | Price stability + upside      |

## 6. Benefits

1. **Dump Resistance**: Early sellers fund simulated buys, raising prices for holders.
2. **Holder Confidence**: Growing reserves provide auditable token backing.
3. **Sustainable Demand**: Merges viral bonding with compounding growth from locked simulated purchases.

## 7. Technical Implementation

- **Smart Contract Modifications**:
  1. Track \(R(n)\) in real-time: reserve = initial liquidity + simulated buy amounts - burned.
  2. On sale: Calculate tax, simulate buy by adding \(\Delta R\) to reserve without minting tokens, update \(P(n) = R(n)/n\).
  3. Burn 1% equivalent tokens.

- **User Dashboard**: Show reserve growth, tax decay, and projected prices from simulated reinvestments.

## 8. Comparison to Pump.fun

| Feature            | Pump.fun          | Zap v3                     |
|--------------------|-------------------|---------------------------------|
| Price Stability   | Low               | High (Tax-Simulated Buys)      |
| Reserve Growth    | None              | Exponential via Locked Buys    |
| Dump Impact       | Severe            | Net Positive (Boosts Reserve)  |

## 9. Conclusion

Zap v3 reinvents bonding curves by using exit taxes to simulate buys that lock value in the contract, transforming speculation into stability. This flywheel—where volatility fuels growth—aligns all holders for long-term success.

## Appendices

- **Appendix A**: Reserve Growth Simulations (Tax vs. No-Tax).
- **Appendix B**: Smart Contract Code Snippets for Simulated Reinvestment.
