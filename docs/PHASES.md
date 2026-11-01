# jev-harness — Development / Testing / Deployment Plan (v2, DRAFT)

> Gated, phased plan, v2. Each phase: objective, build, test, exit gate. Gates
> are go/no-go — do not advance until the gate is met. v2 reorders v1:
> self-healing moves from a late-phase patch (v1 Phase 6) to a core-design
> phase (Phase 7), and failure triage gets its own phase (Phase 8), because
> both are where the product's value lives.

## Guiding principle

De-risk the two assumptions that, if false, kill the product: (1) a
typed-decision model can route routine UI decisions with a tolerable
*post-escalation* error rate on **your own domain**, and (2) multi-locator
replay + bounded LLM exceptions can hold flake rate under target. Everything
else (compiler, dashboards) only matters if those hold.

---

## Phase 0 — Foundations: OpenJev serving + R1 gate *(de-risk first)*

- **Objective.** Prove or kill the core hypothesis with your own numbers. No
  product code beyond the eval harness.
- **Build.**
  - Serve OpenJev: vLLM (fp8) + shim; measure your real latency / tokens /
    cost per decision on your hardware.
  - One concrete pilot domain (e.g., the auth flows of one app you already
    test).
  - Labeled corpus of UI decision samples **from the pilot app**
    (snapshot, step, correct operation, correct target) — mined from your own
    recorded sessions + LLM pseudo-labels with human spot-check.
  - Eval harness: fixed snapshots in, OpenJev answers + probabilities out.
- **Test.**
  - OpenJev per-decision accuracy on *your* samples (vs its 84% general
    benchmark — expect a delta in either direction).
  - Confident-subset analysis: for each candidate θ, measure {p ≥ θ} accuracy
    + escalation rate.
  - **Gate metric: post-escalation per-step error rate.** ECE / Brier recorded
    as supporting evidence, not the gate.
- **Exit gate.** Post-escalation per-step error ≤ bound (draft: 1%) at an
  escalation rate compatible with the cost model (R3). If it fails, the
  project stops or re-scopes the domain — that is the correct place for it to
  stop.

## Phase 1 — Page State: snapshot with boundary + locator ladder

- **Objective.** A compact, decision-ready snapshot the classifier can consume
  inside OpenJev's option budget, carrying the multi-locator ladder replay will
  need.
- **Build.** role / label / type / value / visible text; **actionable flag +
  covered_by + disabled**; **list_index** for repeated / virtualized elements;
  same-origin iframes + open shadow flattened; **locator-ladder capture per
  element** (testid → role+name → css → text anchor → position); stable
  element identity (content-hash fingerprint that survives re-render); state
  hashing for change detection.
- **Boundary (documented in the module).** In: DOM tree incl. same-origin
  iframes, open shadow, text, roles, actions. **Out: canvas / WebGL / PDF /
  video UIs — declared out of pilot scope.**
- **Test.** Fidelity suite over representative pilot pages: does the snapshot
  express everything a decision needs? Measure ladder capture cost (ms +
  tokens). Regression on a frozen page corpus; document what is deliberately
  dropped.
- **Exit gate.** Snapshot module with documented coverage, explicit loss list,
  and measured ladder cost (R2 pass).

## Phase 2 — System One integration + text binding

- **Objective.** OpenJev in the loop with a tuned θ, and text coming from data
  before models.
- **Build.** Candidate pruning (page → ≤ 52 options); θ tuning per task class;
  **text supply hierarchy**: spec data row → Faker pool → LLM (last resort;
  generated values pinned back into the data table).
- **Test.** On the pilot suite: post-escalation error (re-measure in the loop),
  escalation rate, **text-step ratio and LLM-text-call count**, per-step
  latency.
- **Exit gate.** Error / escalation / text-ratio all within the model R3
  prices.

## Phase 3 — Structured test case layer *(v2 — fix 5)*

- **Objective.** Deterministic, diffable authoring per spec.
- **Build.** Spec schema (steps + assertions + data table; spec_id + version);
  **one-shot intent interpretation, pinned** in the spec (pin hash, model +
  date recorded); spec → trajectory keying.
- **Test.** **Determinism check:** two authoring runs of the same spec against
  the same app produce the same trajectory (diff = zero). Spec version bump →
  clean, reviewable diff.
- **Exit gate.** Spec layer ships; re-authoring a pinned spec carries no
  interpretation variance.

## Phase 4 — LLM escalation + decomposition

- **Objective.** The expensive model as a rare, well-scoped fallback.
- **Build.** Escalation prompt (snapshot + optional screenshot) → ordered
  System-One-style actions; decompose-then-verify (each decomposed action
  re-gated by OpenJev where applicable); per-call cost accounting.
- **Test.** Escalation fires at the modeled rate; every decomposition
  executable; per-step cost measured.
- **Exit gate.** Escalation path within cost + correctness budget.

## Phase 5 — Trajectory recorder + compiler

- **Objective.** Trajectory → runnable deterministic code, with the ladder.
- **Build.** Trajectory schema (ARCHITECTURE §10); compiler: ladder
  preservation, control flow (branches from decomposition, loops over steps),
  assertions as extract + compare with ladder treatment; spec pin + version in
  the script header.
- **Test.** **Round-trip:** compiled script reproduces the recorded trajectory
  exactly (ladder order, text, assertions); diff against recorded end state.
- **Exit gate.** Deterministic round-trip across the pilot suite (R5).

## Phase 6 — Authoring harness E2E

- **Objective.** Wire the loop end-to-end; measure the cost claim.
- **Build.** Loop orchestration, snapshot re-extraction between steps,
  compound / ordered execution, spec iteration.
- **Test.** Multi-step specs on the pilot app; **four-baseline comparison
  (ARCHITECTURE §9)** — hybrid vs full-LLM-agent vs human-authored vs
  self-healing vendor tool — on authoring cost + time, including GPU
  amortization.
- **Exit gate.** Pilot suite authored end-to-end within the cost / latency
  budget the model claims.

## Phase 7 — Deterministic replay + self-healing *(v2 — promoted to core, fix 3)*

- **Objective.** Make replay survive real drift; this is the product.
- **Build.** Replay runner walking the locator ladder (deterministic order);
  **bounded exception path**: on total ladder failure → one LLM call with
  screenshot + snapshot; K retries; self-healed locators **promoted** into the
  ladder.
- **Test.** Replay across drift variants — viewport, timing, class renames,
  element reorders, new siblings, A/B UI variants — measuring flake rate,
  exception rate, exception cost.
- **Exit gate.** Flake rate ≤ target across all variants (R4); exception path
  bounded and logged; $0 path verified $0 on the no-drift variant.

## Phase 8 — Failure triage loop *(v2 — new, fix 4)*

- **Objective.** Close the loop: replay failures get a verdict, not a shrug.
- **Build.** Triage classifier (bug / flake / app-change) on the exception
  path; routing: flake → retry; app-change → **surgical re-author** of just the
  stale step (spec version bump, old trajectory archived); bug → normal
  failure + diagnosis attached; **triage budget cap** (max LLM calls per run /
  per suite / per day).
- **Test.** Build a real failure corpus (drifted replays, injected flakes,
  seeded app bugs); measure **triage precision / recall per class** and
  verdict latency + cost; verify bug-detection parity with the golden suite —
  the harness must never classify a real bug as a flake.
- **Exit gate.** Triage precision ≥ target per class (R6); re-author pipeline
  round-trips; budget cap enforced.

## Phase 9 — Verification + observability

- **Objective.** Assertions layer + the dashboards operators need.
- **Build.** Assertion extraction + comparison (spec assertions, re-checked
  every replay); dashboards: cost per authoring / per run, escalation rate,
  **post-escalation error rate over time (calibration drift)**, flake rate,
  triage accuracy, budget-cap usage; drift-triggered θ re-tune job.
- **Test.** Assertion correctness on the golden suite; feedback-loop test:
  drift detection re-tunes θ without regressing error rate; budget caps fire.
- **Exit gate.** Suite with verified assertions + live cost / quality reporting.

---

## Deployment / rollout

| Phase | Focus | Key activity | Exit gate |
|-------|-------|--------------|-----------|
| **D0** CI integration | Author → compile → commit script as artifact | Pipeline step | Generated artifact reproducible in CI |
| **D1** Shadow mode | Hybrid runs alongside the existing suite; compare results **and triage verdicts** | Disagreement log | Agreement + triage precision above threshold |
| **D2** Gradual rollout | Opt-in per suite; expand | Per-suite cost / flake / triage watch | Per-suite quality + cost within budget |
| **D3** Production guardrails | Budget caps (author + replay exceptions + triage), automatic fallback to existing runner, quality-drop alerts | No regressions under caps; alerts functional |

## Cross-cutting testing strategy *(v2)*

- **Calibration is monitored as post-escalation error rate** (the
  decision-relevant metric), continuously — it drifts as domains change.
- **Round-trip tests** guard the compiler; **replay-variance tests** guard
  self-healing; **triage-corpus tests** guard the exception path.
- **Cost / latency / flake / triage budgets are first-class acceptance
  criteria**, measured against the four baselines (ARCHITECTURE §9) — including
  the 30-day total-ownership comparison, not one-day snapshots.

## Open questions *(resolved by research, see ROADMAP)*

- Which classifier family best models calibrated confidence for UI decisions?
  (v2's working answer: OpenJev — validate, don't assume.)
- What is the minimal-yet-sufficient Page State per task class, inside OpenJev's
  option budget?
- How do we keep locators stable across environments without requiring injected
  attributes on every site? (v2: ladder + promotion; validate coverage in R4.)
- What θ policy minimises cost while holding the quality floor?
- Can the exception path (self-heal + triage) be held to ≤ 5% of runs?
