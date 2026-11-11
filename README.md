# Ponk's Law

**The value of a concentrated-liquidity position is its net cashflow weighted by the probability it is still alive to earn it.**

Most liquidity-provision math asks one question: *will this range earn fees?*
That question is incomplete, and acting on it is how capital bleeds. A range
only earns while price sits inside it and while the flow through it is not
picking the position off. The moment price leaves the band, or the inventory
tilts so far that every trade is adverse, the position stops being an earner and
becomes a cost. So the real question is the one Ponk's Law asks:

> Over the life the position is actually likely to survive, is the net cashflow
> it earns worth the capital locked and the cost paid to open and maintain it?

Ponk's Law is the model Ponk's autonomous agents are built on. It is two
equations: one for the value of a position, and one for the hazard that ends it.

---

## 1. The value of a position

The expected, capital-normalised value of a DLMM position is its net cashflow
integrated over time, with every future instant discounted not by a fixed rate
but by the probability the position is still **alive** to collect it:

$$
E_{\text{DLMM}} \;=\; \frac{1}{C_0}\int_{0}^{\infty}
\big(\,F_t - L_t - R_t - X_t\,\big)\,
\underbrace{e^{-\int_{0}^{t}\lambda(s)\,ds}}_{\text{survival to }t}\;dt
$$

| Symbol | Meaning |
| --- | --- |
| $C_0$ | capital deployed into the position |
| $F_t$ | fees accruing at time $t$ |
| $L_t$ | impermanent loss / divergence against simply holding the inventory |
| $R_t$ | rebalancing cost (the inventory conversion a recenter makes irreversible) |
| $X_t$ | gas and transaction cost |
| $\lambda(t)$ | the exit hazard: the instantaneous rate at which the position stops earning |

The term $e^{-\int_0^t \lambda(s)\,ds}$ is the **survival function**: the
probability the position is still in range and still worth holding at time $t$.
Fees far in the future are worth little not because of the time value of money
but because the position probably will not be there to earn them. A tight band
in a trending or toxic market has a short survival time, so its distant fees are
discounted to almost nothing no matter how high the advertised APR.

This is why fee APR alone is a trap. Two pools can advertise the same yield; the
one whose positions survive longer is worth far more, and Ponk's Law prices that
difference explicitly.

---

## 2. The hazard that ends it

The exit hazard is not constant. It rises with the forces that push price out of
the band and that turn flow against the position. Ponk's Law models it as a
proportional-hazards link, in the spirit of the Cox model, around a physical
baseline:

$$
\lambda(t) \;=\; \lambda_0 \,\exp\!\Big(
\alpha\,\frac{\sigma}{w} \;+\;
\beta\,\frac{|v|}{w} \;+\;
\gamma\,\frac{V}{L_{\text{bin}}} \;+\;
\delta\,I
\Big)
$$

| Symbol | Meaning |
| --- | --- |
| $\lambda_0$ | baseline hazard, the inverse of the physical expected time to the band edge |
| $\sigma$ | realised volatility |
| $v$ | drift (the signed, directional component of recent returns) |
| $w$ | half-width of the band |
| $V$ | flow / volume passing through the active bin |
| $L_{\text{bin}}$ | liquidity resting in the active bin |
| $I$ | inventory skew (how lopsided the position has become) |
| $\alpha,\beta,\gamma,\delta$ | sensitivities of the hazard to each force |

Each term has a plain reading:

- $\sigma / w$ : **volatility relative to how much room the band has.** A jumpy
  pool in a narrow band leaves range quickly. This is carried mostly by the
  physical baseline $\lambda_0$, so $\alpha$ is small by default.
- $|v| / w$ : **drift relative to band width.** A sustained one-way move empties
  a band far faster than volatility of the same magnitude, because it spends its
  whole budget going one direction. Recentering into a trend is paying to stand
  in front of it.
- $V / L_{\text{bin}}$ : **flow toxicity.** When the volume moving through the
  active bin is large relative to the liquidity resting there, the position is
  being selected against: it is the counterparty to informed flow, and fees do
  not compensate for it.
- $I$ : **inventory skew.** A position that has drifted to mostly one token is
  closer to its edge and more exposed to the next move, so its hazard is higher.

Raise any of these and the survival function in equation (1) decays faster, the
distant cashflows are discounted harder, and a recenter that looked profitable
on fee APR alone is revealed as a loss.

---

## What the law changes

The naive LP decision compares the fees a new range *would* earn against the gas
to open it. Ponk's Law replaces that with the only comparison that protects
capital:

> Rebalance only when the **survival-weighted** value regained over the
> position's likely remaining life exceeds the full cost of moving, by a margin
> wide enough to survive the error in the estimate itself.

Two consequences fall straight out of the math, and both are load-bearing:

1. **Doing nothing is usually right.** An agent's value is as much the actions
   it correctly declines as the ones it takes. Every recenter pays gas, a
   balancing swap that crystallises divergence, and rent. If the survival-
   weighted benefit does not clear that cost with room to spare, the position is
   worth more held than moved. An agent that fires all day is not working
   harder; it is bleeding faster.

2. **Size and pool choice dominate.** The fixed costs in $R_t$ and $X_t$ are a
   small fraction of a large position on a deep pool and a crushing fraction of
   a small one on a thin pool. The single largest determinant of whether a
   position makes money is not the cleverness of the rebalancer; it is whether
   the position should have been opened in that pool, at that size, at all.

A position is only worth opening, and only worth moving, when Ponk's Law says
the surviving cashflow pays for it.

---

## Notation summary

$$
\begin{aligned}
E_{\text{DLMM}} &: \text{expected capital-normalised value of the position} \\
F_t - L_t - R_t - X_t &: \text{net cashflow (fees, less IL, rebalance cost, gas)} \\
e^{-\int_0^t \lambda}\!\! &: \text{probability the position survives to time } t \\
\lambda(t) &: \text{exit hazard, rising with } \sigma,\,|v|,\,V/L_{\text{bin}},\,I
\end{aligned}
$$

---

*Ponk is autonomous liquidity management on Solana, Ethereum, Arbitrum, Base,
BNB Chain and Robinhood Chain. Ponk's Law governs when its agents open, hold,
move, and step aside. More at [ponk.exchange](https://ponk.exchange).*
