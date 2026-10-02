# BR Portfolio Engine — Normative Guidelines

**Status:** authoritative architecture contract  
**Scope:** Brazil-only quantitative portfolio research and decision-support engine  
**Priority:** this file is normative. Any implementation choice, generated code, refactor, dependency, test, dashboard, data source, model, optimization method or workflow that conflicts with this document is invalid unless this document is explicitly amended in a dedicated change.

## Non-negotiable principles

- The system is a **research and decision-support engine**, not an autonomous trading bot.
- The election layer must **never rank, endorse, oppose, predict or hard-code a political candidate**. It models observable macro/market scenarios and their transmission to assets.
- Compliance is **fail-closed**: `UNKNOWN` is not tradable.
- Point-in-time integrity is mandatory. No look-ahead, survivorship bias, revised data leakage or hidden future information.
- `NO TRADE` is a valid and often preferred output.
- Every portfolio decision must be reproducible from immutable inputs, configuration, model artifacts and git SHA.
- Official Brazilian sources have precedence for primary market, macro, fund and electoral data whenever available.
- The implementation must remain usable after the 2026 election; election logic is an overlay, not the core architecture.

---

## 1. System objective
Build a production-grade Brazil-only quantitative portfolio engine that receives capital, current holdings, investable universe, prices, fundamentals, macro data, yield-curve data, FX, commodities, expectations, fund data, events, scenario inputs, personal constraints and compliance state; then produces research-grade forecasts, robust risk estimates, scenario analysis, portfolio targets, discrete allocations, proposed orders, explanations and audit trails.

## 2. End-to-end architecture
The canonical pipeline is: `Data -> Point-in-Time normalization -> Feature Store -> Forecast Engines -> Regime Engine -> Scenario Engine -> Risk Engine -> Portfolio Optimizers -> Portfolio Ensemble -> Discrete Allocation -> Compliance Gate -> Proposed Portfolio/Orders -> Dashboard/Reports/Audit`.

## 3. Separation of forecasting and optimization
No forecast model may directly set portfolio weights. Forecasting estimates expected returns, uncertainty or rankings. Risk models estimate covariance/tail risk. Scenario models estimate conditional shocks. Optimizers consume these outputs. This separation must be explicit in code boundaries and interfaces.

## 4. No direct candidate-to-trade mapping
Forbidden patterns include `if candidate == X: buy Y`, hard-coded candidate portfolios, political scoring, election winner prediction as an internal model, or candidate-specific buy/sell rules. Election analysis must be translated into macro/market shock vectors before reaching asset-level expected returns.

## 5. Model diversification principle
No single model may determine the final portfolio. The system must support multiple independent forecast families and multiple portfolio construction methods, then combine them through documented ensemble logic.

## 6. Uncertainty-aware sizing
Every forecast should expose uncertainty, confidence, dispersion or an equivalent calibration measure. Position size must decrease as uncertainty increases, all else equal. A point estimate without uncertainty is insufficient for production decisions.

## 7. Diversification by factors, not ticker count
The system must detect hidden concentration across tickers sharing the same economic drivers. Sector, interest-rate, FX, commodity, market, style and idiosyncratic factor exposure must be measurable and constrainable.

## 8. Point-in-time as a first-class invariant
Every time-varying observation used by models must distinguish `observation_date`, `available_at`, `ingested_at`, `source` and `source_version` where applicable. Backtests must only access values whose `available_at <= decision_time`.

## 9. Asset universes
Support separate but interoperable universes for: Brazilian listed equities; listed Brazilian vehicles such as ETFs, FIIs, FIAGROs and fixed-income ETFs; and CVM-regulated investment funds. These universes may have different liquidity, pricing and settlement semantics.

## 10. Brazil-only default scope
Production default must restrict exposure to Brazil-domiciled/listed instruments and Brazilian funds unless a future guideline amendment explicitly expands scope. Global variables may be used as explanatory factors, but the investment universe remains Brazil-only.

## 11. Data-lake layering
Use immutable raw ingestion and explicit transformations: `RAW -> BRONZE -> SILVER -> GOLD -> FEATURE STORE`. Raw data is preserved byte-for-byte or semantically equivalent to the source payload and is never silently overwritten.

## 12. RAW layer
Store source payloads, retrieval timestamps, request metadata, checksums, source URLs/endpoints and licensing/provenance metadata. No cleaning, adjustment or imputation belongs in RAW.

## 13. BRONZE layer
Parse source-native files into typed tables while preserving original fields. Parsing errors must be observable and testable; malformed rows must not disappear silently.

## 14. SILVER layer
Normalize identifiers, timestamps, units, data types, missingness conventions and corporate-action handling. Deduplication rules must be explicit and deterministic.

## 15. GOLD layer
Create analysis-ready tables for returns, fundamentals, macro, curves, fund NAVs, classifications, events, liquidity and corporate actions. GOLD tables may not violate point-in-time constraints.

## 16. FEATURE STORE
All production model features must be generated through a versioned feature pipeline. Features require definition, frequency, lookback, lag/availability rule, data source, null policy and tests.

## 17. Source hierarchy
Tier 1 primary sources should include B3, CVM, Banco Central do Brasil, IBGE, Tesouro/STN, ANBIMA and TSE when applicable. Tier 2 may include corporate investor-relations and official filings. Tier 3 contextual sources may include Reuters, Valor, InfoMoney, Bloomberg, public asset-manager material and major newspapers. Tier 3 must never replace primary source truth for prices, filings or official statistics.

## 18. Price and market data
The engine must ingest daily historical prices, volume, trading activity, instrument metadata and identifiers. It must distinguish raw prices from adjusted prices and total-return series.

## 19. Corporate actions
Handle dividends, JCP, splits, reverse splits, bonuses, subscriptions, ticker changes, mergers, acquisitions, spin-offs and delistings. Corporate-action transformations must be deterministic and regression-tested.

## 20. Total-return construction
Maintain at minimum `raw_close`, `adjusted_close` and `total_return_index`. Return-based modeling must use the correct series for the task and never assume raw close is total return.

## 21. Fund data
For CVM funds, support NAV/cota, AUM, subscriptions, redemptions, holders where available, category, administrator/manager metadata, liquidity/redemption terms and historical registration changes.

## 22. Macro data
Support Selic, inflation, activity, fiscal, external-sector, labor and other Brazilian macro series. Preserve vintages where revised series or forecast histories matter.

## 23. Focus expectations
Support historical expectation vintages for inflation, GDP, FX, Selic, fiscal and external-sector indicators. Features must use the vintage available on the decision date, never a later revised expectation history.

## 24. Yield-curve data
Represent the Brazilian interest-rate term structure across maturities and derive curve level, slope and curvature. Preserve original quotes and transformation logic.

## 25. ANBIMA/fixed-income indices
Support relevant ANBIMA index families and fixed-income benchmarks needed to characterize duration, real-rate and nominal-rate regimes and to benchmark fixed-income-sensitive assets/funds.

## 26. Commodity data
Support globally priced drivers relevant to Brazilian issuers, such as oil, iron ore, pulp, agricultural commodities, gas and others. Commodity coverage must be config-driven by exposure relevance.

## 27. FX and Brazil risk
Support USD/BRL and derived FX features. Support sovereign-risk measures or transparent proxies, with explicit source/licensing constraints. Any proxy must be labeled as a proxy, not as the underlying unavailable series.

## 28. Canonical storage technology
Initial research/quant storage should use Parquet plus DuckDB for reproducible analytical workloads. PostgreSQL may be added for application state, users, portfolio history, audit and web/API needs; it must not replace immutable research snapshots.

## 29. Canonical directory layout
Maintain logical data directories for `raw`, `bronze`, `silver`, `gold`, `features`, `models`, `cache` and `reports`. Generated data and model artifacts must not be committed to git unless intentionally small fixtures.

## 30. Canonical tables
At minimum support logical tables/entities for `assets`, `prices_daily`, `total_returns`, `corporate_actions`, `fundamentals`, `fundamentals_vintages`, `fund_nav`, `macro_series`, `macro_vintages`, `yield_curve`, `focus_expectations`, `commodities`, `sector_classification`, `universe_snapshots`, `liquidity_metrics`, `features`, `factor_exposures`, `regime_probabilities`, `model_forecasts`, `scenario_definitions`, `scenario_results`, `portfolio_holdings`, `target_weights`, `orders_proposed`, `compliance_status`, `model_registry`, `backtest_runs`, `data_quality` and `audit_log`.

## 31. Fundamental availability logic
A fiscal quarter ending on date X is not available on X unless actually disclosed then. Financial-statement features must use the filing/publication timestamp and correct market-session availability rule.

## 32. Intraday availability convention
For data released after market close, production logic must define whether it becomes usable next session. Timezone must be explicit (`America/Sao_Paulo` for Brazilian decision timestamps unless source semantics dictate otherwise).

## 33. Survivorship-bias elimination
Universe snapshots must be reconstructed historically. Delisted, merged and failed securities must remain present in historical backtests whenever they were investable at the time.

## 34. Universe engine
Build `Universe(t)` from only information known at time `t`. Filters must be config-driven and may include history length, average daily volume, trading-frequency ratio, free float, data completeness, market cap and instrument type.

## 35. Liquidity eligibility
The system must distinguish theoretical eligibility from execution practicality. Minimum liquidity, stale-price and trading-frequency rules must be testable and overridable by configuration, not hard-coded in model code.

## 36. Price features
Implement multi-horizon returns, momentum, reversal, volatility, drawdown, skew, kurtosis, gap behavior, volume surprise, relative strength, distance to highs/lows and beta-related features. Every lookback must be explicit and lag-safe.

## 37. Momentum family
Include at minimum configurable 12-1, 6-1, 3-month, short-term reversal, industry-relative momentum and relative-strength signals. Momentum must be kept as an independent family because of crash/regime behavior.

## 38. Liquidity features
Support average daily value/volume, turnover, Amihud-style illiquidity, zero-return days, spread proxies where available and volume relative to market capitalization.

## 39. Value features
Support sector- and history-aware valuation features such as P/E, P/B, EV/EBITDA, FCF yield, earnings yield and dividend yield. Raw valuation ratios should be normalized cross-sectionally and/or relative to own history where appropriate.

## 40. Quality features
Support ROE, ROIC, margin levels/stability, free cash flow, accruals, leverage, interest coverage and earnings-quality measures. Definitions must be robust to sector-specific accounting differences.

## 41. Growth features
Support revenue, EBITDA, EPS and FCF growth, plus acceleration/deceleration where meaningful. Growth features must not use future consensus or post-period filings before availability.

## 42. Macro feature family
Create features describing level and change in rates, inflation, expectations, FX, activity and risk premium. Changes should exist across multiple horizons and in standardized surprise form where justified.

## 43. Curve factors
Derive level, slope and curvature from DI/interest-rate data and retain raw maturity-level exposures. The exact decomposition method must be versioned and testable.

## 44. Dynamic factor exposures
Asset sensitivities to rates, FX, commodities, market and styles must be allowed to vary over time. Static full-sample betas are not acceptable as the sole production representation.

## 45. Rolling linear factor models
Implement transparent rolling OLS as a baseline. Support Ridge and Elastic Net where multicollinearity or regularization is beneficial. All windows and penalties must be config-driven.

## 46. State-space/Kalman models
Support dynamic-beta estimation using state-space/Kalman filtering for selected factors. Kalman output must include uncertainty and be compared with rolling baselines.

## 47. Election/macro scenario engine
The election layer must generate or ingest economic shock scenarios, not political recommendations. Canonical scenario vectors may include changes in short/long DI, BRL, risk premium, GDP expectations, inflation expectations, equity risk premium and other documented factors.

## 48. Scenario representation
Scenarios must be data/config objects with name, timestamp/version, source/rationale, shock vector, confidence/probability if externally supplied, and optional horizon. No scenario probability may be fabricated by the engine merely to force optimization.

## 49. Candidate neutrality
Scenario names should describe market/economic regimes such as `fiscal_compression`, `fiscal_stress`, `rates_down_fast`, `rates_high_for_longer`, `brl_appreciation` and `brl_depreciation`, not value judgments about political actors.

## 50. No internal election winner predictor
Do not train or deploy a model whose product purpose is estimating who will win an election. If external probabilities are supplied, preserve provenance and allow the system to run equally well with no probabilities by reporting conditional portfolios/scenarios.

## 51. Event-study engine
Support event windows around first/second rounds, major fiscal announcements, monetary decisions and other high-impact domestic events. Event definitions must be immutable inputs to a run.

## 52. Abnormal returns
Implement abnormal return and cumulative abnormal return methodology against appropriate expected-return models. Event-study code must separate expected-return estimation windows from event windows.

## 53. Regime engine
Support at least three complementary approaches over time: Hidden Markov Models, change-point detection and probabilistic/Bayesian regime estimation. Regimes must emerge from market/macro observables, not from political labels.

## 54. Regime probabilities
Production should expose probabilities, not only hard labels: `P(regime=k | X_t)`. Portfolio logic should be robust to regime uncertainty.

## 55. Fundamental forecast engine
Build an independent fundamental signal/forecast family using valuation, profitability, balance-sheet strength, growth, cash generation and revisions when legitimate PIT data exists.

## 56. Machine-learning baseline
Begin with interpretable/robust methods such as Elastic Net and gradient boosting. Neural networks are explicitly out of initial scope unless justified by out-of-sample evidence and added through a future guideline amendment.

## 57. Time-series engine
Support selected autoregressive/state-space approaches and volatility models such as GARCH where statistically justified. Do not add ARIMA/GARCH merely for sophistication; each production model needs benchmark evidence.

## 58. Cross-sectional engine
Support daily/weekly cross-sectional ranking or expected-return models for configurable horizons such as 5d, 21d, 63d and 126d. Horizon semantics must be consistent with rebalance and execution assumptions.

## 59. Black-Litterman engine
Implement Black-Litterman with market/equilibrium priors, view matrix `P`, view vector `Q` and uncertainty matrix `Omega`. Scenario or model views with lower confidence must exert proportionally less influence.

## 60. Forecast ensemble
Combine independent families such as fundamental, factor, macro, momentum, regime, event, Bayesian/Black-Litterman and ML. Each component must output a standardized forecast contract including expected return/rank, horizon, uncertainty and confidence.

## 61. Dynamic model weights
Do not use permanently fixed ensemble weights by default. Support walk-forward performance-based weighting using forecast error, information coefficient, rank IC, directional accuracy and portfolio contribution, with regularization to prevent unstable switching.

## 62. Forecast confidence engine
Create a standardized confidence score based on model agreement, calibration, historical error, data quality, distance from training distribution, regime uncertainty and forecast dispersion. Confidence must affect portfolio sizing or robust-return shrinkage.

## 63. Risk engine architecture
Risk is independent from alpha. The engine must support multiple covariance estimators and tail-risk measures and expose risk contributions at asset, sector and factor levels.

## 64. Covariance estimators
Implement sample covariance as a baseline, EWMA covariance and Ledoit-Wolf shrinkage. Production may blend estimators using configurable logic validated out of sample.

## 65. Dynamic correlation advanced module
DCC-GARCH or equivalent dynamic-correlation approaches are advanced-phase features, not MVP blockers. They must be benchmarked against simpler covariance estimators before production promotion.

## 66. Tail risk
Compute at minimum VaR, CVaR/Expected Shortfall, drawdown and downside deviation. CVaR should be available to optimization and stress reporting.

## 67. Stress engine
Given factor shocks, estimate asset and portfolio P&L through current dynamic exposures and/or scenario revaluation. Stress output must include assumptions, horizon and decomposition.

## 68. Standard scenario library
Maintain configurable macro scenarios such as Rates Down Fast, Rates Down Gradual, Fiscal Stress, BRL Appreciation, BRL Depreciation, Commodity Rally, Commodity Crash, Brazil Risk-On, Brazil Risk-Off, Global Recession, Domestic Growth and Stagflation.

## 69. Monte Carlo architecture
Support block bootstrap, stationary bootstrap and factor-based simulation. Do not assume IID Gaussian returns as the sole simulation mechanism.

## 70. Simulation outputs
Expose distributional metrics such as median return, relevant percentiles, probability of losses above thresholds, expected drawdown and tail loss. All simulated metrics must disclose horizon and methodology.

## 71. Core portfolio optimizer
The robust optimizer should be able to maximize expected return net of penalties for variance, CVaR, turnover and concentration, under explicit constraints. The exact objective terms and lambdas must be configurable.

## 72. Robust expected returns
Support uncertainty sets around expected returns. Lower-confidence forecasts should imply larger uncertainty radii, shrinking aggressive allocations and reducing optimizer sensitivity to noisy means.

## 73. Hierarchical Risk Parity
Implement HRP as an independent portfolio candidate, using clustering to identify correlated economic bets and avoid unstable mean-variance inversion behavior.

## 74. Risk parity
Implement a configurable risk-parity candidate using marginal/total risk contribution concepts. It must be benchmarked, not presumed superior.

## 75. Minimum variance
Implement a minimum-variance portfolio candidate with constraints and robust covariance handling.

## 76. Maximum diversification
Implement a maximum-diversification portfolio candidate as another independent construction method when covariance/volatility inputs are valid.

## 77. Portfolio ensemble
Final target weights may blend robust optimization, HRP, Black-Litterman/CVaR and other validated candidates. Ensemble weights must be explicit, versioned and testable; a single optimizer must not silently dominate because another failed.

## 78. Constraint system
Support long-only, max asset, max sector, max factor exposure, max company group, max state-owned exposure, max exporter exposure, max domestic-rate beta, max turnover and minimum cash. Constraints belong in config/schema, not scattered conditionals.

## 79. Liquidity constraints
Support position limits relative to average daily trading value/volume and fund redemption terms. Even if irrelevant at R$2,000, scalability must be designed from the beginning.

## 80. Discrete allocation
Convert continuous target weights to executable integer quantities/cotas while minimizing deviation from target and respecting cash, lot/fractional-market rules and minimum investment constraints.

## 81. Small-capital correctness
For portfolios such as R$2,000, the engine must report residual cash, discrete-allocation error and whether the theoretical optimum is materially distorted by indivisibility. It must not pretend continuous weights are executable.

## 82. Cost engine
Backtests and rebalance analysis must model configurable brokerage, exchange fees, spreads, slippage, applicable taxes and fund expenses. Tax/legal parameters must be configuration/versioned data because rules can change.

## 83. Turnover penalty
Portfolio optimization and rebalance logic must explicitly penalize turnover. A change in target should occur only when expected benefit exceeds costs plus an uncertainty/safety margin.

## 84. Confidence gate and NO TRADE
If forecast confidence is below threshold, data are stale, portfolio improvement is immaterial, constraints are infeasible or compliance is unresolved, the valid outcome is `NO TRADE` or `HOLD`.

## 85. Compliance architecture
Quantitative models produce a target portfolio first; a separate compliance gate then determines tradability. The optimizer may optionally re-run on the compliant universe after exclusions, but compliance logic must never be bypassed.

## 86. Compliance states
At minimum support `ALLOWED`, `RESTRICTED`, `PRE_CLEARANCE_REQUIRED` and `UNKNOWN`. `UNKNOWN` must block proposed execution. Status must include effective dates and provenance/notes without exposing confidential internal information unnecessarily.

## 87. Pre-clearance workflow
Where approval is required, the engine creates a pending proposed order, records that approval is required and waits for explicit user-provided approval metadata before marking the proposal eligible. It must not access or automate internal employer systems without an explicit supported integration and authorization.

## 88. Execution boundary
Initial versions must not execute brokerage orders automatically. Output is `PROPOSED ORDERS` only. Manual user execution occurs after compliance and independent review.

## 89. Backtesting engine
Backtesting is a separate subsystem with explicit decision timestamps, training windows, prediction timestamps, rebalance assumptions and realized-return windows. All runs must be reproducible and persisted.

## 90. Walk-forward evaluation
Use expanding and/or rolling walk-forward validation. No random train/test split for ordinary time-series investment evaluation. Temporal ordering is non-negotiable.

## 91. Purged cross-validation and embargo
For overlapping financial labels/horizons, support purged K-fold with embargo or an equivalent leakage-resistant validation scheme.

## 92. Benchmarks
Every production strategy must compare against at least relevant subsets of: Ibovespa, CDI, equal weight, buy-and-hold, minimum variance and HRP. Complexity that does not improve robust out-of-sample behavior after costs must not be promoted merely because it is complex.

## 93. Performance metrics
Track CAGR/annualized return, volatility, Sharpe, Sortino, maximum drawdown, Calmar, CVaR, turnover, hit rate, IC, Rank IC, beta, alpha, tracking error and factor exposures where applicable. Metrics must be period- and frequency-aware.

## 94. Overfitting detection
Detect train-test performance gaps, parameter instability, feature instability and signal decay. Research results must display failed/negative out-of-sample evidence rather than hiding it.

## 95. Deflated and probabilistic Sharpe
Support Deflated Sharpe Ratio and Probabilistic Sharpe Ratio or equivalent multiple-testing-aware diagnostics to reduce false discovery from extensive strategy search.

## 96. Explainability
Every portfolio target must expose `Why?` and `Why not more?` at asset level: positive drivers, negative drivers, model disagreement, risk penalties and binding constraints. Black-box “model says buy” is unacceptable.

## 97. Risk and scenario contribution
Expose asset/sector/factor risk contribution and scenario P&L contribution. Portfolio weight must never be presented as equivalent to percentage risk contribution.

## 98. Election/macro sensitivity dashboard
Provide a dedicated neutral view of interest-rate beta, FX beta, risk-premium beta, commodity beta, domestic-demand exposure and state-owned-company exposure. It may compare economic scenarios, not political choices or candidates.

## 99. News/NLP module
News ingestion is auxiliary. NLP may classify topics such as fiscal, monetary, tax, regulation, state-owned, commodity, corporate and global; extract events; and produce research features. LLM output must not directly generate expected returns or autonomous trades.

## 100. Data quality and kill switches
Every pipeline must check freshness, schema, duplicates, missingness, ranges, continuity, outliers and corporate-action consistency. Kill conditions include stale data, model drift, extreme/unbounded forecasts, covariance failure, compliance unknown, large missingness and infeasible optimization. Fail closed to `NO TRADE`.

## 101. Model registry, audit and reproducibility
Every model/run must record model ID/version, training dates, features, hyperparameters, metrics, artifact hash, git SHA, config hash, data snapshot identifiers and random seed. A historical `run_id` must reproduce the same target portfolio given preserved inputs.

## 102. Repository, configuration, CLI, dashboard, testing and operations
Use Python 3.11. Prefer `polars`, `pandas`, `numpy`, `scipy`, `pyarrow`, `duckdb`, `statsmodels`, `scikit-learn`, `cvxpy`, `arch`, `hmmlearn`, `plotly`, `pydantic`, `pandera`, `typer`, `rich`; optional validated use of `lightgbm`, `xgboost`, `shap`. Keep config in `configs/*.yaml`. Provide CLI commands for data sync/validation, universe build, feature build, model train, regime inference, scenario run, portfolio optimize, backtest, reports and `doctor`. Provide dashboard views for overview, positions, signals, risk, factors, scenarios, election/macro sensitivity, backtests and system health. Tests must include unit, integration, data contracts, regression, backtest, smoke and end-to-end. CI must run lint, type checks, tests, data-contract checks, security checks and a small deterministic backtest. Structured logs must include `run_id`, module, timestamp, status and message. Daily pipeline: fetch -> validate -> normalize -> features -> regimes -> forecasts -> covariance -> scenarios -> optimize -> compliance -> report. Weekly/monthly jobs evaluate drift/retraining/calibration; they must not retrain simply because a calendar event occurred. Core/tactical sleeves should remain separable so election overlays modify tactical risk without destroying the long-term core.

## 103. Implementation order and final acceptance contract
Implementation must proceed in this dependency order unless a documented engineering reason justifies a non-semantic reordering: **Phase 1** infrastructure, data lake, B3/BCB/CVM/ANBIMA connectors, PIT rules, corporate actions, tests; **Phase 2** universe, feature store, factors, baseline risk; **Phase 3** serious backtesting, walk-forward, costs, benchmarks; **Phase 4** fundamental/momentum/macro/factor forecasts; **Phase 5** ensemble/confidence; **Phase 6** HRP, Black-Litterman, CVaR and robust optimization; **Phase 7** scenario/event/election overlay; **Phase 8** discrete allocation for small capital; **Phase 9** compliance; **Phase 10** dashboard/CLI/reports; **Phase 11** advanced ML and regime switching; **Phase 12** monitoring, drift, registry and production hardening. The final system must be able to accept capital/current portfolio/universe/settings, produce target weights and exact discrete quantities, residual cash, expected return with uncertainty, expected volatility, CVaR, projected drawdown distribution, risk contributions, factor exposures, stress/scenario results, model agreement/confidence, explanation for each weight and binding constraint, change since prior run, proposed trades, estimated costs and compliance status; and must be able to return `HOLD`/`NO TRADE` when evidence is insufficient.

---

# Mandatory engineering guardrails

The following are explicitly forbidden unless this guideline is amended:

1. `yfinance` as the primary authoritative production data source.
2. Look-ahead bias, revised-data leakage or survivorship bias.
3. Candidate-specific portfolio weights, candidate rankings or election-winner prediction.
4. Fully automated brokerage execution in initial production scope.
5. LLM-generated buy/sell decisions without quantitative model mediation.
6. Backtests without realistic cost assumptions.
7. Unbounded optimizers or unconstrained concentration.
8. Choosing the best in-sample experiment and hiding unsuccessful experiments.
9. Retrospectively altering methodology solely to improve historical results without logging the experiment as a new version.
10. Silent fallback from failed official data to lower-quality sources.
11. Silent use of stale inputs.
12. Suppressing `NO TRADE` merely to force a recommendation.

# Repository structure contract

```text
br-portfolio-engine/
├── CLAUDE.md
├── GUIDELINES.md
├── MASTER_PROMPT.md
├── README.md
├── pyproject.toml
├── Makefile
├── .env.example
├── .gitignore
├── configs/
│   ├── data.yaml
│   ├── universe.yaml
│   ├── features.yaml
│   ├── models.yaml
│   ├── risk.yaml
│   ├── optimizer.yaml
│   ├── scenarios.yaml
│   ├── portfolio.yaml
│   └── compliance.yaml
├── data/
│   ├── raw/
│   ├── bronze/
│   ├── silver/
│   ├── gold/
│   ├── features/
│   ├── models/
│   └── cache/
├── docs/
├── notebooks/
├── reports/
├── scripts/
├── tests/
└── src/br_portfolio/
    ├── data/
    │   ├── b3/
    │   ├── bcb/
    │   ├── cvm/
    │   ├── anbima/
    │   ├── ibge/
    │   └── tse/
    ├── universe/
    ├── corporate_actions/
    ├── features/
    ├── factors/
    ├── regimes/
    ├── forecasts/
    │   ├── fundamental/
    │   ├── macro/
    │   ├── factor/
    │   ├── momentum/
    │   ├── bayesian/
    │   └── ensemble/
    ├── events/
    ├── scenarios/
    ├── risk/
    ├── portfolio/
    ├── compliance/
    ├── execution/
    ├── backtest/
    ├── validation/
    ├── monitoring/
    ├── reports/
    ├── api/
    └── cli/
```

# Acceptance invariants

At minimum, automated tests must encode invariants equivalent to:

```python
assert no_future_data_used()
assert no_survivorship_bias_in_historical_universe()
assert abs(weights.sum() - 1.0) <= tolerance
assert restricted_assets_weight == 0
assert unknown_compliance_assets_weight == 0
assert cash >= 0
assert discrete_portfolio_value <= capital + tolerance
assert no_nan_production_forecasts()
assert backtest_reproducible()
assert all_run_artifacts_have_provenance()
assert election_layer_has_no_candidate_trade_rules()
```

# Governance rule

If an agent believes any item in this file should be changed, it must **not silently change the implementation instead**. It must create a dedicated proposed amendment explaining: the conflicting guideline, reason, evidence, migration impact, tests affected and backwards-compatibility consequences. Until the amendment is accepted, the current guideline remains authoritative.
