# jev-harness

> **DRAFT — working design, not a build.** This repo holds the architecture and
> phased plan only. The design will be refined through research before any real
> implementation begins.

## One line

A **hybrid browser-test authoring harness**: a fast, cheap *System-One* classifier
handles routine UI decisions; an expensive LLM is called only on ambiguous steps.
Every decision during authoring is recorded as a **trajectory** and compiled into a
**deterministic Playwright script** that replays at zero marginal cost.

## Why this exists

Full AI-driven UI testing is slow and expensive because it calls a large model on
every step. The observation this project tests:

> In real browser automation the vast majority of decisions are routine
> ("click the button labelled X", "type into field Y"). These are bounded, typed
> questions a cheap classifier answers well. Reserve the expensive model for the
> small fraction that genuinely needs it.

That gives three worlds working together:

| Layer | Role | Cost | When it runs |
|-------|------|------|--------------|
| **System One** (Jev/OpenJev) | Routine decisions, calibrated confidence | ~sub-100ms, fractions of a cent | Every step |
| **LLM agent** (Claude/GPT) | Ambiguity, text generation, complex reasoning | Slow ($0.01–0.10+), multimodal | Only when System One is not confident |
| **Deterministic replay** | Pure Playwright code, no models | $0.00 | Test time (after authoring) |

## Architecture at a glance

```mermaid
flowchart TD
    subgraph AUTHOR["Authoring phase (models run)"]
        INT[Intent steps] --> PS[Page State extractor<br/>compact DOM snapshot]
        PS --> S1[System One classifier<br/>Jev: operation + target + confidence]
        S1 -->|confidence >= threshold?| EXEC[Execute action in browser<br/>Playwright]
        S1 -->|confidence BELOW threshold| ESC[Escalate to LLM agent<br/>screenshot + state]
        ESC --> DECOMP[Decompose into System-One-style actions]
        DECOMP --> EXEC
        EXEC --> REC[Trajectory recorder<br/>{model, op, target, loc, conf}]
    end

    subgraph REPLAY["Replay phase (no models)"]
        REC --> COMP[Compiler<br/>deterministic codegen]
        COMP --> SCRIPT[Deterministic Playwright script]
        SCRIPT --> RUN[Re-run tests at $0 cost]
    end

    REC -.calibration feedback.-> S1
```

## Repo layout (planned)

- `README.md` — this overview.
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — detailed design, diagrams, data model.
- [`docs/PHASES.md`](docs/PHASES.md) — end-to-end development / testing / deployment phases.
- [`docs/ROADMAP.md`](docs/ROADMAP.md) — research-driven open questions and validation plan.

## Draft status — where the real risk is

This design was written critically, not optimistically. The orchestrator loop is easy;
three things will make or break it and are the priority for research:

1. **Classifier calibration** — everything hinges on Jev returning *reliable*
   confidence. Miscalibrated → escalate constantly or trust wrong decisions.
2. **State representation** — the harness can only be as good as its compact Page State;
   a lossy snapshot forces wrong calls or needless escalation.
3. **Locator drift** — recorded locators must survive between authoring and replay, or
   you bake flakiness into the "deterministic" path.

See [ROADMAP.md](docs/ROADMAP.md) for the research validation order. Do **not**
treat this as final — it is a starting scaffold to be improved with research.
