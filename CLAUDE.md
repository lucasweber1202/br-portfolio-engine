# Claude Code / Work operating contract

Before planning, editing, generating or reviewing code in this repository, read **`GUIDELINES.md` in full**.

`GUIDELINES.md` is the normative architecture contract. It takes precedence over convenience, speed, framework defaults and any inferred preference. Do not weaken, skip or silently reinterpret its requirements.

## Required behavior

1. Start every implementation task by identifying which guideline items are in scope.
2. Preserve point-in-time correctness, reproducibility, auditability and fail-closed behavior.
3. Do not implement candidate-specific political recommendations or election-winner prediction.
4. Do not introduce automatic brokerage execution.
5. Do not use weak data sources as silent substitutes for official sources.
6. Implement in dependency order. Do not build advanced ML before the PIT data/backtest foundation is trustworthy.
7. Add tests together with code. A feature without its critical tests is incomplete.
8. Prefer explicit contracts, typed schemas, config-driven behavior and deterministic pipelines.
9. Never make the system trade merely because a model ran. `NO TRADE` must remain reachable.
10. If credentials, licensed datasets or a human compliance approval are missing, finish every non-blocked part, provide a deterministic adapter/interface and document the exact external blocker. Do not fabricate secrets/data.
11. Record architectural deviations only through an explicit proposed amendment to `GUIDELINES.md`; never silently diverge.
12. Before declaring a phase complete, run the relevant tests, deterministic smoke/backtest and `doctor` checks that exist at that stage.

## Work style

- Work autonomously through non-blocked tasks.
- Make small coherent commits/PR-ready changes grouped by phase or subsystem.
- Keep a living implementation matrix mapping guideline item -> code path -> tests -> status.
- Do not optimize for demo appearance before quantitative correctness.
- Treat failed validation as useful evidence, not something to hide.
