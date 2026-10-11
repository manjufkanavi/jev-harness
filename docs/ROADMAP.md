# jev-harness — Roadmap (DRAFT / research-driven)

> Research validation order. The design here is provisional and will change as
> experiments answer the questions below. This file exists to keep that research
> visible and ordered — it is not a schedule.

## Why this order

Risk is concentrated in two places: **calibration** and **state representation**. If a
cheap classifier cannot reliably route routine decisions, the economics collapse and
the rest of the architecture is moot. So research validates those first; everything
else (compiler, replay stability) only matters if the foundation holds.

## R1 — Can a cheap classifier route routine decisions with calibrated confidence? *(highest)*

- Build a labeled corpus of UI decision samples (operation + target) for one domain.
- Measure calibration: reliability diagram, Expected Calibration Error, Brier score.
- Decide classifier family given the calibration + latency + cost constraints.
- **Pass criterion:** ECE under a chosen bound on the sample domain.

## R2 — What is the minimal sufficient Page State representation? *(highest)*

- Compare snapshot representations (flat list vs. graph; label-only vs. includes
  visible text, position, visibility-under-overlay).
- Measure how representation quality affects wrong-call rate and escalation frequency.
- **Pass criterion:** a documented minimal representation that meets an accuracy floor
  at acceptable build cost.

## R3 — Can escalation stay rare and cheap in practice?

- With R1 + R2 fixed, measure real-world escalation rate across a sample suite.
- Confirm the hybrid cost advantage holds outside toy examples.
- **Pass criterion:** escalation rate low enough to preserve the 10–30× cost advantage.

## R4 — How do we keep replay locators stable across environments? *(flakiness)*

- Evaluate: injected `data-testid`, semantic locators, self-healing on stale locator.
- Measure flake rate across viewport / timing / DOM-drift variants at replay time.
- **Pass criterion:** flake rate under target across all tested variants.

## R5 — Compiler correctness & generality

- Round-trip fidelity: compiled script reproduces authored trajectory exactly.
- Generality over control-flow shapes (branches, loops, conditional assertions).
- **Pass criterion:** deterministic round-trip across the sample suite.

## Ongoing / operational research

- Threshold policy that minimises cost while holding a quality floor.
- Calibration drift detection and automatic threshold re-tuning over time.
- Scaling from one domain to many without per-domain retuning.
