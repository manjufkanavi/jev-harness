# jev-harness — Roadmap (v2, research-driven)

> Research validation order, v2. Not a schedule. v1's R1 ("is ECE low enough?")
> is replaced — ECE is a population statistic; what kills tests is a
> confident-wrong decision, so the gate is now **post-escalation per-step
> error rate**. Two items (self-healing coverage, triage accuracy) move up
> because v2 made them core, not hardening.

## R1 — Can OpenJev route *your* domain within the error bound? *(highest)*

- Serve OpenJev (vLLM + shim); measure your real latency / tokens / cost per
  decision (v1's "sub-100ms" was not its number — ~210ms / ~1.5k tokens is).
- Build the labeled corpus **from the pilot app** (mined sessions + LLM
  pseudo-labels, human spot-checked). OpenJev's 84% is a *general* 10k
  benchmark; your forms may be 95% or 70%.
- For each candidate θ: confident-subset accuracy + escalation rate.
- **Pass:** post-escalation per-step error ≤ bound (draft 1%) at a θ whose
  escalation rate R3 can price. If no θ satisfies both, the project stops or
  re-scopes the domain.

## R2 — Minimal sufficient Page State, with the ladder *(v2)*

- Representation variants: flat vs tree; label-only vs +visible text /
  actionability / list-index; ladder capture on vs off.
- Measure: wrong-call rate, escalation rate, **snapshot build cost** (ms +
  prompt tokens, inside OpenJev's 52-option / ~1.5k-token budget).
- **Pass:** a documented representation that meets the accuracy floor inside
  the budget, with the boundary (canvas / WebGL / PDF / video out) stated.

## R3 — Does the hybrid stay cheap in practice? *(v2)*

- With R1 + R2 fixed, on the pilot suite: escalation rate, **text-step ratio
  and LLM-text-call count** (data-table-first hierarchy), GPU amortization per
  authored test.
- Run the **four-baseline comparison** (human Playwright / self-healing vendor
  tool / full LLM agent / jev-harness) over a 30-day replay window: authoring
  cost + time, flake rate, triage time, bug detection.
- **Pass:** the 10–30× authoring claim holds *for this domain*, and 30-day
  total ownership cost beats the best baseline. The claim is per-domain;
  form-heavy domains earn it only if the text hierarchy keeps LLM calls rare.

## R4 — Self-healing coverage *(v2 — promoted)*

- Locator ladder (testid → role+name → css → text anchor → position) across
  drift variants: class renames, element reorders, new siblings, A/B UI
  variants, viewport.
- Measure: ladder-only survival rate, exception rate, flake rate, exception
  cost; self-heal promotion effect (do exceptions trend to zero on stable
  apps?).
- **Pass:** flake rate ≤ target across all variants; the exception path stays
  the exception (≤ 5% of runs in steady state).

## R5 — Compiler round-trip

- Round-trip fidelity: compiled script re-executes the recorded trajectory
  exactly (ladder order, text, assertions).
- Generality over control-flow shapes: branches, loops, conditional
  assertions from decomposition.
- **Pass:** deterministic round-trip across the pilot suite.

## R6 — Triage accuracy *(v2 — new)*

- Build a real failure corpus: injected flakes, seeded app bugs, drifted
  replays.
- LLM verdict (bug / flake / app-change): precision + recall **per class** — a
  misrouted "flake" hides a real bug; that is the failure mode we most refuse.
- Measure verdict latency + cost; verify bug-detection parity with the golden
  suite.
- **Pass:** precision ≥ target per class; no sustained bug→flake misroutes in
  the corpus.

## Ongoing / operational research

- θ policy: minimise cost while holding the quality floor; re-tune on drift
  detection.
- Calibration drift: post-escalation error trend over time; automatic re-tune.
- Domain scaling: per-domain corpus cost; when does a domain need its own θ /
  pruning (OpenJev trains once — your labels live in the request).
- Baseline tracking: re-run the four-baseline comparison quarterly — vendor
  tools and agent models move.
