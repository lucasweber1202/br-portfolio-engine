# Master implementation prompt — BR Portfolio Engine

You are the principal quant engineer, data engineer, ML engineer, risk engineer and software architect responsible for implementing this repository to production-grade research standards.

Your source of truth is `GUIDELINES.md`. Read it **completely before making any change**. Also read `CLAUDE.md`. The 103 numbered guidelines are binding acceptance requirements, not suggestions.

## Mission

Implement the BR Portfolio Engine as a Brazil-only quantitative portfolio research and decision-support system. It must combine point-in-time Brazilian market/macro/fund data, multiple independent forecasting theories, dynamic factor/regime analysis, scenario and event studies, robust risk estimation, portfolio-construction ensembles, discrete allocation, compliance gating, reproducible backtesting, auditability and a usable CLI/dashboard.

The system must remain useful after the 2026 election. The election component is a neutral macro-scenario overlay. Never hard-code candidate-specific trades, endorse political choices, rank candidates or build an internal election-winner predictor.

## Non-negotiable delivery behavior

- Do not simplify away requirements from `GUIDELINES.md`.
- Do not jump to advanced ML before the data/PIT/backtest foundation exists and passes tests.
- Do not use future data, revised values unavailable at decision time or survivorship-biased universes.
- Do not use `yfinance` as the authoritative production source.
- Do not add autonomous brokerage execution.
- Do not let an LLM directly create buy/sell orders from news text.
- Do not force a trade. Preserve `HOLD`/`NO TRADE` paths.
- Do not hide weak/failed backtests.
- Do not ask the user to perform steps that you can complete yourself. If a truly external blocker exists (credentials, licensed data, human compliance approval), complete all non-blocked engineering first and document the blocker precisely.

## Implementation strategy

Execute the 12 phases in guideline item 103. Within each phase:

1. Inspect existing code and status.
2. Create/update `docs/IMPLEMENTATION_MATRIX.md` mapping every relevant guideline number to implementation files, tests and status (`NOT_STARTED`, `PARTIAL`, `DONE`, `BLOCKED_EXTERNAL`).
3. Define typed interfaces/schemas before concrete adapters where useful.
4. Implement the smallest production-correct vertical slice.
5. Add unit, contract and integration tests.
6. Run lint/type/tests and deterministic smoke checks.
7. Update docs and matrix.
8. Commit changes coherently or leave them PR-ready with exact verification output.

## Phase requirements

### Phase 1 — foundation and data
Create Python 3.11 project structure, packaging, config models, structured logging, deterministic run IDs, DuckDB/Parquet storage, primary-source adapter interfaces and concrete supported ingestion for B3/BCB/CVM/ANBIMA first. Implement RAW/BRONZE/SILVER/GOLD layers, PIT timestamps, corporate actions, total-return logic, data contracts, quality checks and provenance. Add IBGE/TSE interfaces/adapters when required by available public endpoints. No model work is allowed to bypass missing PIT semantics.

### Phase 2 — universe/features/risk baseline
Implement historical universe snapshots without survivorship bias, feature-store contracts, price/momentum/liquidity/value/quality/growth/macro/curve/FX/commodity features and baseline rolling factor exposures. Add sample/EWMA/Ledoit-Wolf covariance estimators and baseline tail metrics.

### Phase 3 — backtesting
Implement walk-forward backtests with deterministic decision timestamps, realistic costs, discrete execution assumptions, benchmarks, expanding/rolling windows, purging/embargo where labels overlap, and full reproducibility. Add performance, risk and overfitting diagnostics. A strategy cannot advance if the backtester itself is not validated.

### Phase 4 — forecast families
Implement independent transparent baseline models: fundamental, momentum, macro/factor, rolling linear, regularized linear and selected time-series/state-space components. Use model contracts that output horizon, expected return/rank, uncertainty, confidence and metadata.

### Phase 5 — ensemble/confidence
Implement dynamic model evaluation and weighting using walk-forward IC/RankIC/error/directional metrics with regularization. Build confidence from agreement, calibration, data quality, OOD distance and regime uncertainty.

### Phase 6 — portfolio construction
Implement HRP, Black-Litterman, CVaR-aware robust optimization, risk parity, minimum variance, maximum diversification and portfolio-ensemble logic. Enforce config-driven concentration, factor, sector, turnover, cash and liquidity constraints.

### Phase 7 — scenarios/events/election overlay
Implement neutral macro scenario definitions, event-study engine, abnormal/CAR calculations, dynamic factor shock transmission, stress testing and election/macro sensitivity reporting. Never encode candidate recommendations. External scenario probabilities must preserve provenance; absent probabilities, run conditional scenarios independently.

### Phase 8 — discrete allocation
Convert continuous targets into integer executable quantities/cotas for small portfolios, including R$2,000, respecting fractional/lot mechanics where modeled, costs and residual cash. Report allocation error.

### Phase 9 — compliance
Implement fail-closed compliance states and pre-clearance workflow. `UNKNOWN` and `RESTRICTED` must block execution proposals. No direct integration to employer internal systems unless separately authorized and technically available.

### Phase 10 — interfaces
Implement Typer CLI with commands equivalent to `data sync`, `data validate`, `universe build`, `features build`, `models train`, `regimes infer`, `scenario run`, `portfolio optimize`, `backtest run`, `report generate`, `doctor`. Implement a functional Streamlit dashboard with overview, positions, signals, risk, factors, scenarios, election/macro sensitivity, backtests and system-health pages.

### Phase 11 — advanced ML/regimes
Only after prior gates pass, add HMM/change-point/Bayesian regime inference, gradient boosting if justified, optional SHAP/permutation importance and advanced models that demonstrate robust incremental value out of sample.

### Phase 12 — production hardening
Complete model registry, drift monitoring, data freshness alerts, kill switches, audit trails, weekly/monthly evaluation jobs, CI, reproducible artifacts, deterministic reports and operational runbooks.

## Required repository artifacts

Maintain at minimum:

- `GUIDELINES.md` — authoritative.
- `CLAUDE.md` — agent contract.
- `docs/IMPLEMENTATION_MATRIX.md`.
- `docs/DATA_SOURCES.md`.
- `docs/PIT_POLICY.md`.
- `docs/BACKTEST_POLICY.md`.
- `docs/MODEL_GOVERNANCE.md`.
- `docs/COMPLIANCE_BOUNDARY.md`.
- `docs/RUNBOOK.md`.
- config files specified in the guideline.
- tests for all critical invariants.

## Acceptance gates

Do not claim completion unless the relevant stage proves, through automated tests/logs, that:

- future information cannot enter training/decision data;
- historical universes include dead/delisted names where appropriate;
- corporate actions and total-return series are consistent;
- every production feature has provenance and availability semantics;
- backtests reproduce from a `run_id`/snapshot/config/git SHA;
- costs are included;
- benchmark comparisons are present;
- optimizers respect all constraints and remain numerically stable;
- discrete allocations do not exceed capital;
- compliance exclusions result in zero tradable weight;
- `NO TRADE` is produced under low confidence/data-quality/compliance failure;
- election logic contains no candidate-specific buy/sell rules;
- final recommendation explanations include drivers, counter-drivers and binding constraints.

## Definition of done

The final product must accept a Brazil-only portfolio configuration and capital amount, including an initially empty R$2,000 portfolio, and produce a fully reproducible research package containing target continuous weights, exact discrete quantities, residual cash, forecast returns with uncertainty, risk estimates, CVaR, drawdown distribution, risk/factor contributions, conditional scenario results, model agreement/confidence, per-position explanations, change versus previous run, proposed trades, estimated costs and compliance status—or a documented `HOLD`/`NO TRADE` result.

Start by reading `GUIDELINES.md` and `CLAUDE.md`, auditing the repository against all 103 items, writing `docs/IMPLEMENTATION_MATRIX.md`, and then execute Phase 1. Do not skip ahead.
