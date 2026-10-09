# Synthetic example: altcoin regime detection

Status: **proposal only**. No experiment has been run, nothing here is independently verified.

This folder is a worked template for how a regime-detection study should be specified before any real data is touched. It uses no proprietary data, no DFI Labs positions and no confidential information. All numbers are illustrative parameters, not findings.

| File | Content |
|---|---|
| `HYPOTHESIS.md` | Economic rationale, falsifiable statements, pre-registered success criteria |
| `SIGNAL.md` | Exact formulas for the regime score and the sizing rule |
| `METHODOLOGY.md` | Synthetic data generator, walk-forward design, costs, metrics, robustness checks |

## Purpose

1. Fix the definitions and the evaluation protocol before looking at real returns, so results cannot be tuned after the fact.
2. Provide a synthetic world with a known regime process, so the backtest code can be tested for leakage and bias. If the pipeline cannot recover a regime we planted, it cannot be trusted on real data.

## Classification of claims

| Level | Meaning | Items in this folder |
|---|---|---|
| Hypothesis | Stated, not tested | All of `HYPOTHESIS.md` |
| Completed experiment | Run, code and output committed | None |
| Independently verified | Reproduced by the GPT reviewer | None |

## Next steps

1. Implement the generator and the signal in `backtest.py` (not included here).
2. Run the pipeline on synthetic data with a planted regime and with no regime (null world).
3. Submit to GPT review on the `gpt-review` branch before any real-data run.
