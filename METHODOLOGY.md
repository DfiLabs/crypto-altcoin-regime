# Proposed methodology

## Stage 0: synthetic world

The point of Stage 0 is to test the machinery, not the idea.

**Generator.** Simulate 120 coins and BTC over 6 years of daily data.

- A latent two-state Markov chain `S(t)` in {calm, rotation}. Calm to rotation probability 0.02 per day, rotation to calm 0.12 per day (mean rotation length about 8 days).
- BTC return: Student-t (4 degrees of freedom) with daily volatility 3%, no drift.
- Coin `i` return: `beta_i * r_B(t) + alpha(S(t)) + e(i,t)`, with `beta_i` drawn from a lognormal around 1.2 and idiosyncratic `e` with daily volatility 4% and GARCH-style clustering.
- `alpha(rotation)` positive and cross-sectionally skewed towards smaller coins, `alpha(calm)` slightly negative, so the planted effect has the same shape as hypotheses H1 to H3.
- Market caps and volumes follow a rank-stable lognormal so that universe reconstitution can be exercised.

**Two worlds.**

1. *Planted world:* `alpha(rotation)` set so the true IC is roughly 0.05. The pipeline must recover a positive IC and a drawdown benefit.
2. *Null world:* `alpha` identical in both states. The pipeline must return an IC statistically indistinguishable from zero in at least 95% of 500 seeds. A higher false-positive rate means leakage or a test-statistic bug.

**Leakage canary.** Add a column equal to `Y(t,h)` shifted by one day and confirm the pipeline flags it or that its IC is implausibly high. The test must fail loudly if look-ahead is introduced.

## Stage 1: walk-forward evaluation

- Expanding training window, minimum 2 years.
- Test blocks of 3 months, rolled forward.
- Embargo of `h` days between train and test, to stop overlapping forward returns leaking across the boundary.
- Nothing is fitted in the baseline (equal weights, fixed thresholds), so the training window is used only for the expanding standardisation and percentile thresholds. Any optional fitted variant (for example a ridge-weighted score) is estimated on the training window only and evaluated on the following block.
- A final hold-out of the most recent 12 months is touched once, after all design decisions are frozen.

## Stage 2: predictive tests

- Daily rank IC between `Score(t)` and `Y(t,h)`, aggregated per test block.
- Newey-West standard errors with lag `h` (forward returns overlap).
- Quintile spread of mean `Y(t,h)`.
- Hit rate of the binary regime state against realised `Y(t,h) > 0`.
- Control regressions: add trailing 10-day BTC return, BTC 30-day realised volatility and the BTC dominance change. The score must keep a significant coefficient (null N1).
- Block bootstrap (block length `2h`) for confidence intervals.

## Stage 3: portfolio tests

Compare a constant-gross high-beta short book with the sized book `g(t)`.

- Costs: 10 bps one-way taker fee plus an impact term of 5 bps per 1% of daily volume traded, plus funding on shorts at observed (or, in synthetic data, simulated) rates. Run a 2x cost scenario.
- Turnover reported for both books, since sizing changes turnover.
- Execution lag of 1 day, with a 2-day lag variant (null N2).
- Borrow and liquidation constraints ignored in the baseline, flagged as a limitation.

## Metrics

Annualised return, volatility, Sharpe, Sortino, maximum drawdown, Calmar, 95% expected shortfall, worst 10-day return, turnover, and the share of the drawdown reduction that comes from the 10 worst days (to check the benefit is not a single episode).

## Robustness grid

| Dimension | Variants |
|---|---|
| Universe | Top 30, Top 40, Top 100 |
| Range lookback | 63, 126, both |
| Relative window `k` | 5, 10, 20 days |
| Thresholds | 70/50, 80/60, 90/70 percentiles |
| Sizing floor | 0.2, 0.4, 0.6 |
| Horizon `h` | 3, 5, 10 |
| Execution lag | 1 day, 2 days |
| Sub-periods | Each calendar year, bull and bear halves by BTC trend |

Results are reported for **every** cell, not only the best one. The number of cells tried is recorded so that a multiple-testing adjustment (for example a deflated Sharpe ratio) can be applied.

## Reproducibility

- Fixed random seeds, recorded in the output.
- `requirements.txt` with pinned versions.
- One command reproduces every table: `python backtest.py --world planted --seed 1`.
- Outputs written to `results/` with a hash of the config.
- Failed and null results are kept in `EXPERIMENT_LOG.md`.

## Review protocol

1. Claude commits the specification and code to `claude-research`.
2. GPT reviews independently on `gpt-review`, with a written list of criticisms.
3. Claude answers each criticism and proposes additional experiments.
4. Only items reproduced by the reviewer move to the independently verified level.
5. Merging to `main` requires human approval.

## Known limitations of this proposal

- A synthetic world encodes our own assumptions about how rotations behave, so passing Stage 0 shows the code works, not that the effect exists.
- Real data will bring exchange outages, wash volume, token unlocks and delistings that the generator does not model.
- Short availability and borrow cost for small altcoins are simplified.
- Regime labels are not observable in real data, so evaluation relies on forward returns only.
- Median aggregation across a universe hides heavy tails that matter for a short book.
