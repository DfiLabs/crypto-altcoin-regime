# Hypothesis

## Economic rationale

Altcoin returns can be decomposed into market beta (mostly BTC) and a residual. Short positions in high-beta altcoins are usually a bet that the residual is negative or flat. That bet fails in short, violent windows when capital rotates down the market-cap curve and the residual turns strongly positive.

Three mechanisms could make such windows partly predictable:

1. **Range position.** Altcoins trading near the top of their 3-month or 6-month range have attracted momentum buyers and leverage. Breakouts from a long range tend to cluster in time across coins, because they share liquidity conditions.
2. **Breadth of outperformance.** If many coins beat BTC over the recent window, the move is a market-wide rotation rather than an idiosyncratic story, and rotations persist for days as flows arrive with a lag.
3. **Depth of the rotation.** If the Top 100 universe outperforms the Top 30 (small beats large), speculative risk appetite is high and tends to precede further residual strength in the short run.

## Falsifiable statements

Let `Y(t,h)` be the forward beta-adjusted altcoin return over `h` days (defined in `SIGNAL.md`).

- **H1.** The regime score at `t` has a positive rank correlation with `Y(t,h)` for `h` in {3, 5, 10}.
- **H2.** Top-quintile regime scores are followed by a higher mean `Y(t,h)` than bottom-quintile scores, net of the cost of acting on the signal.
- **H3.** Reducing the gross of a high-beta altcoin short book when the score is high lowers maximum drawdown versus a constant-gross book, without a material loss of Sharpe ratio.
- **H4.** The effect has the same sign in the Top 30, Top 40 and Top 100 universes.

## Null hypotheses

- **N1.** The regime score carries no information beyond a trailing 10-day BTC return or realised volatility.
- **N2.** Any measured edge disappears after realistic costs or after a 1-day execution lag.

## Pre-registered success criteria

Fixed before any real-data run. The study fails on a criterion if it is not met out of sample.

| Criterion | Threshold |
|---|---|
| Out-of-sample rank IC, h = 5 | Mean above 0.03, t-stat above 2 (Newey-West, lag h) |
| Same-sign IC across three universes | All three positive |
| Max drawdown of sized book vs constant gross | At least 15% relative reduction |
| Sharpe of sized book vs constant gross | Not worse by more than 0.1 |
| Sensitivity | Sign preserved across all parameter variants in `METHODOLOGY.md` |

## What would change our mind

A positive in-sample result that vanishes in walk-forward testing, or that is explained by BTC volatility alone, is treated as a negative result and logged in `EXPERIMENT_LOG.md`.
