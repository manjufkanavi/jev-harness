# jev-harness — Development / Testing / Deployment Plan (DRAFT)

> Phased, gate-driven plan. Each phase has an **objective**, **development tasks**,
> a **testing approach**, and an **exit gate**. Gates are go/no-go decisions — do not
> advance a phase until its gate is met. This is a draft to be refined with research;
> the order reflects where the risk lives (calibration & state first).

## Guiding principle

De-risk the assumption that a cheap classifier can reliably route routine UI
decisions before investing in everything on top of it. Phases 0–2 exist to prove the
economics; phases 3+ build the rest.

---

## Phase 0 — Foundations & Research Validation *(de-risk first)*

- **Objective.** Prove the core hypothesis: a cheap System-One classifier can answer
  routine UI questions with well-calibrated confidence. No product code yet.
- **Development.**
  - Pick a concrete target domain (e.g., standard auth forms) — start narrow.
  - Stand up an eval harness that feeds the classifier fixed Page States and logs its
    choices + confidence.
  - Build a labeled corpus of real UI decision samples for the domain.
- **Testing.**
  - Calibration audit: reliability diagram + Expected Calibration Error (ECE) on the
    corpus. A miscalibrated classifier breaks every downstream decision.
  - Baseline accuracy of "routine vs non-routine" classification.
- **Exit gate.** Classifier confidence is calibrated (ECE under a target bound) on the
  sample domain. If it fails here, fix representation/calibration before anything else.

## Phase 1 — Page State Extraction *(the critical abstraction)*

- **Objective.** Reduce the live DOM to a compact, decision-ready snapshot.
- **Development.**
  - Snapshot builder: role / label / type / value / visible text per element.
  - Stable identifier strategy (e.g., injected `data-testid` / stable attributes).
  - State hashing for change detection between steps.
- **Testing.** Fidelity tests — does the snapshot capture everything a decision needs?
  Regression against representative pages; document what is deliberately dropped.
- **Exit gate.** Snapshot module with documented coverage and known lossiness.

## Phase 2 — System One Classifier & Confidence *(highest risk)*

- **Objective.** Ship the typed-question interface with tuned, trustworthy confidence.
- **Development.** Operation classifier (CLICK / TYPE_TEXT / SELECT / SCROLL / WAIT /
  DONE / BLOCKED) + target classifier; confidence calibration loop.
- **Testing.** Calibration on Phase 0 corpus (re-measure after tuning); threshold
  sensitivity analysis — how does escalation rate move with the threshold?
- **Exit gate.** Meets calibration + accuracy gates; a starting confidence threshold is
  chosen and its escalation-rate impact quantified.

## Phase 3 — LLM Escalation & Decomposition

- **Objective.** Add the expensive model as a rare, well-scoped fallback.
- **Development.** Narrow text-generation calls (what to type); screenshot + state prompt;
  logic that decomposes an LLM answer into ordered System-One actions.
- **Testing.** Escalation triggers at the intended rate (not too often / not too rarely);
  every decomposition is executable; per-step cost accounting.
- **Exit gate.** Escalation path with measured frequency, correctness budget, and cost.

## Phase 4 — Trajectory Recorder & Compiler (Deterministic Codegen)

- **Objective.** Turn recorded trajectories into runnable deterministic code.
- **Development.** Trajectory schema; the compiler (with control flow: branches from
  decomposition, loops over intent lines); exact-locator preservation.
- **Testing.** Round-trip test — the compiled script reproduces the authored trajectory;
  diff output against expected end state. This is where "deterministic" earns its name.
- **Exit gate.** Compiler regenerates recorded behavior deterministically on the sample domain.

## Phase 5 — Authoring Harness Integration (end-to-end)

- **Objective.** Wire the loop into a working end-to-end authoring tool.
- **Development.** Loop orchestration, state re-extraction between steps, compound/ordered
  execution, intent-line iteration.
- **Testing.** Multi-step intents on the target app; measure escalation rate, latency,
  and cost against a full-AI baseline. Target the hybrid's cost advantage here.
- **Exit gate.** Sample suite authored end-to-end within target cost/latency budget.

## Phase 6 — Deterministic Replay & Flakiness Hardening

- **Objective.** Make replay stable across environment drift.
- **Development.** Replay runner; locator-stability strategies (injected ids, semantic
  locators); self-healing on stale-locator failure.
- **Testing.** Replay across variants — viewport, timing, minor DOM drift — and measure
  flake rate. This is the make-or-break for "deterministic."
- **Exit gate.** Stable replay with flake rate under target across all variants.

## Phase 7 — Verification, Assertions & Cost Dashboard

- **Objective.** Add the assertion layer and operational observability.
- **Development.** Deterministic assertions; verification steps (e.g., "balance > $500")
  via extraction + comparison; cost + calibration dashboard. Calibration feedback loop
  that nudges the threshold over time.
- **Testing.** Assertion correctness; verify the feedback loop improves calibration or at
   least does not regress it.
- **Exit gate.** Suite with verified assertions + visible cost/calibration reporting.

---

## Deployment / Rollout Phases *(productionisation)*

| Phase | Focus | Key activity | Exit gate |
|-------|-------|--------------|-----------|
| **D0** CI integration | Author in CI, commit generated script as artifact | Pipeline step: author → compile → store deterministic test | Generated artifact reproducible in CI |
| **D1** Shadow mode | Run hybrid alongside existing tests, compare results | No enforcement; collect disagreement log | Agreement rate with golden suite above threshold |
| **D2** Gradual rollout | Opt-in per test suite; expand coverage | Enable suite-by-suite, watch cost & flakes | Per-suite quality + cost within budget |
| **D3** Production guardrails | Safety limits & fallbacks | Budget caps, automatic fallback to existing runner, alerting on quality drop | No regressions under cap; alerts functional |

---

## Cross-cutting testing strategy

- **Calibration is tested continuously**, not once — it degrades as domains change.
- **Round-trip tests** (trajectory → compiled script → same result) guard the compiler.
- **Replay-variance tests** guard flakiness across environments.
- **Cost/latency budgets** are first-class acceptance criteria, not post-hoc metrics.

## Open questions *(to be resolved by research)*

- Which classifier family best models calibrated confidence for UI decisions?
- What is the minimal-yet-sufficient Page State representation per task class?
- How do we keep replay locators stable without brittle injected attributes on every site?
- What escalation threshold policy minimises cost while holding a quality floor?
