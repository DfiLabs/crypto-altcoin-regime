# Signal definition

All quantities are computed at the daily close of day `t` using data up to and including `t`. Positions are taken at the close of `t+1` (one-day execution lag). Universes `U_N` are the top `N` altcoins by trailing 30-day median dollar volume, **reconstituted weekly using only past data**, with BTC and stablecoins excluded. Delisted coins stay in the sample until delisting (no survivorship bias).

Notation: `C(i,t)` close of coin `i`, `r(i,t) = ln C(i,t) - ln C(i,t-1)`, `r_B(t)` the same for BTC.

## 1. Range position

For lookback `L` in {63, 126} days:

```
RP(i,t,L) = ( C(i,t) - min_{s in [t-L, t]} C(i,s) ) / ( max_{s in [t-L, t]} C(i,s) - min_{s in [t-L, t]} C(i,s) )

RP(t,L,N) = median over i in U_N of RP(i,t,L)
```

Values lie in [0, 1]. A high value means the typical coin sits near the top of its range.

## 2. Relative performance

For window `k` = 10 days:

```
Rel(t,N) = median_{i in U_N} [ sum_{s=t-k+1..t} r(i,s) ]  -  sum_{s=t-k+1..t} r_B(s)
```

And the depth of the rotation across universes:

```
Depth(t) = Rel(t,100) - Rel(t,30)
```

## 3. Regime score

Each component is standardised with an **expanding** window (minimum 365 days), so no future information enters the scaling:

```
z(x,t) = ( x(t) - mean_{s<=t} x(s) ) / std_{s<=t} x(s)

Score(t,N) = 0.25 * z(RP(t,63,N)) + 0.25 * z(RP(t,126,N)) + 0.25 * z(Rel(t,N)) + 0.25 * z(Depth(t))
```

Equal weights are a deliberate choice to avoid fitting. Any alternative weighting must be justified and logged as a separate experiment.

## 4. Regime state

Hysteresis to limit turnover:

```
On  when Score(t,N) exceeds its expanding 80th percentile
Off when Score(t,N) falls below its expanding 60th percentile
```

The state is binary for detection and continuous (the percentile rank `p(t)`) for sizing.

## 5. Target variable

Beta-adjusted forward return for horizon `h` in {3, 5, 10}:

```
beta(i,t) = 0.5 * beta_OLS(i, trailing 90d vs BTC) + 0.5 * 1       (shrinkage towards 1)

Y(t,h) = mean_{i in U_N} sum_{s=t+2..t+1+h} [ r(i,s) - beta(i,t) * r_B(s) ]
```

`beta` is frozen at `t`, so it carries no look-ahead. The sum starts at `t+2` because the position is entered at the close of `t+1`.

## 6. Sizing rule for a high-beta altcoin short book

```
g(t) = g0 * clip( 1 - 0.6 * p(t), 0.4, 1 )
```

where `g0` is the constant-gross benchmark and `p(t)` the percentile rank of the score. At the highest scores the short book runs at 40% of benchmark gross. The sensitivity grid in `METHODOLOGY.md` varies the 0.6 and the 0.4.

## Parameters and where they come from

All values above are round, conventional choices (63 and 126 trading-day lookbacks, a 10-day window, 80/60 percentiles). None was selected by looking at returns. Every deviation after the first real-data run counts as a new experiment.
