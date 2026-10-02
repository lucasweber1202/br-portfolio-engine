# BR Portfolio Engine

A Brazil-only quantitative portfolio research and decision-support engine designed around strict point-in-time data integrity, multiple independent forecasting theories, robust risk/optimization, scenario analysis, discrete allocation, compliance gating and reproducibility.

## Architecture authority

**Read `GUIDELINES.md` first.** Its 103 numbered items are the repository's normative architecture contract. `CLAUDE.md` tells coding agents how to operate. `MASTER_PROMPT.md` is the implementation mission for Claude Code / Work.

## Core rule

The election module is **not** a candidate recommender or election predictor. It translates neutral macro/market scenarios into factor shocks, conditional asset behavior and portfolio stress/tilts.

## Implementation phases

1. Foundation + PIT data + corporate actions
2. Universe + feature store + factor/risk baseline
3. Backtesting + costs + benchmarks
4. Forecast families
5. Ensemble + confidence
6. Portfolio construction
7. Scenario/event/election overlay
8. Discrete allocation
9. Compliance
10. CLI/dashboard/reports
11. Advanced ML/regimes
12. Production hardening

## Intended first use

The engine should be able to analyze a Brazil-only portfolio starting with R$2,000 while remaining architecturally suitable for much larger portfolios.

## Safety / operational boundary

This repository is for research and decision support. Initial scope does not include autonomous brokerage execution. Compliance checks are fail-closed and must be respected before any manual execution.
